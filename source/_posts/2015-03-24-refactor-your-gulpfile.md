title: 重构你的gulpfile
categories:
  - 前端开发
tags:
  - gulp
  - gulpfile
  - 重构
  - 前端自动化
  - 分割task
  - generator-aio-angular
date: 2015-03-24 22:02:22
---

前段时间在公司的新项目中尝试使用gulp替换grunt，体验非常棒！两者最大的区别就是grunt整个就像是一个配置文件，而gulp更像代码。这样的好处就是调试起来更方便直观。另外，利用Node.js的stream概念，让gulp的task看起来像管道一样，输入连着输出，输出又导入另一个输入，流程非常清晰易懂。自打用了这个，就再也不想回到闹心的grunt了。在用了一段时间后，问题来了：不知不觉gulpfile.js已经超过1000行了！每次要修改一个task就要费劲找半天，各种路径也是一团糟，非常难以维护了。那么，是时候重构gulpfile了！

<!--more-->

### 把各种配置独立出去

对于一个复杂的项目来说，在gulpfile里可能各种文件路径的定义都要占掉好多行，各种各样其他的配置就更不用多说了。所以重构的第一步，就是将这些配置提取到一个独立的文件中去。官方[有一篇recipe](https://github.com/gulpjs/gulp/blob/master/docs/recipes/using-external-config-file.md)有说到这个。在这篇recipe中，直接将所有的配置放入一个json文件中去，然后在gulpfile中使用require引入。当然，json可能不是太灵活，可以将其写成一个Node.js的模块。

``` javascirpt gulp.config.js
module.exports = function () {
  var client = './src/client/';
  var server = './src/server/';
  var config = {
    client: client,
    server: server,
    js: client + '**/*.js',
    css: client + '**/*.css'
  };
  return config;
};
```

这样，在gulpfile中使用`var config = require('./gulp.config')();`即可将该配置文件引入进来了。

### 将task分割到不同的文件中

配置只是gulpfile的一小部分，其余的大部分都是具体的task。要想更清晰的管理各种task，最好的方法是将不同的task分类，每一类task分割到一个文件中去。比如clean的放一个文件，编译stylus的放一个文件等等。关于分割task到不同的文件，官方也有[一篇recipe](https://github.com/gulpjs/gulp/blob/master/docs/recipes/split-tasks-across-multiple-files.md)来专门讲这个。这篇recipe中介绍了一个module叫[`require-dir`](https://github.com/aseemk/requireDir)，使用这个模块就可以简单粗暴的将各个task扔到不同的文件中去。文件夹结构如下：

```
gulpfile.js
tasks/
├── css.task.js
└── js.task.js 
```

3个文件的内容分别是：

``` javascirpt gulpfile.js
var requireDir = require('require-dir');
requireDir('./tasks');
```

``` javascirpt css.task.js
var gulp = require('gulp');
gulp.task('css', function () {
  console.log('css task');
});
```

``` javascirpt js.task.js
var gulp = require('gulp');
gulp.task('js', function () {
  console.log('js task');
});
```

这种方法重构的成本非常小，只需简单的将任务移到对应的文件中，无需其他更改。但这样做的问题也很明显：**重复的引入依赖**。在上面的例子中可以看到，每个子任务文件中都重复引入`gulp`模块。虽然`require`可以保证模块只被引入一次，但这样每次重复写代码还是很闹心啊，而且这还只是一个非常简单的例子，在实际项目中重复引入的模块会更多。比如上面提到的提取出来的`gulp.config.js`，比如各种gulp的插件，如果每个任务都要使用某个gulp的插件，那么每个子任务文件都需要引入一遍，非常麻烦。很显然，一个最容易想到的方法就是，在gulpfile中统一的引入这些依赖，然后通过参数传递到各个子任务文件中去。

### 给每个task文件传参

为了传参，就需要把每个task文件写成一个module，然后暴露出来一个函数。下面的例子是改写后的`js.task.js`文件。

``` javascript js.task.js(update)
module.exports = function (gulp, config, $, args) {
  // config -- gulp.config.js
  // $ -- gulp-load-plugins
  // args --- yargs
  gulp.task('js', function () {
    if (args.debug) {
      console.log('debug');
    }
    console.log('js task');
  });
  gulp.task('jshint', function () {
    return gulp
      .src(config.js)
      .pipe($.jshint());
  });
};
```

可以看到，暴露出的函数接受4个参数：
* `gulp`：gulp本身。
* `config`：前面讲到的`gulp.config.js`模块。
* `$`：这里的`$`可不是jQeury，而是一个叫做[`gulp-load-plugins`](https://github.com/jackfranklin/gulp-load-plugins)的gulp插件，这个插件的作用就是自动加载`package.json`里的所有以`gulp-*`开头的gulp插件。有了这个，就再也不用一个一个的引入gulp插件了，所有插件模块都可以通过`$.xxx`的方式来引用。在上面的例子中`$.jshint()`就是调用的插件`gulp-jshint`所提供的功能。
* `args`：[模块`yargs`](https://github.com/bcoe/yargs)。这个模块的功能是用来接受命令行参数的，比如例子中的`js`任务，如果运行时使用`gulp js --debug`就会让代码执行到指定的分支。

显然，这4个参数基本上是每个task都用使用到的，所以都需要传递进去。再来看看gulpfile中如何将调用这个子任务模块吧。

``` javascript gulpfile.js(update)
var gulp = require('gulp');
var config = require('./gulp.config')();
var $ = require('gulp-load-plugins')({lazy: true});
var args = require('yargs').argv;
require('./tasks/js.task')(gulp, config, $, args);
```

可以看到，加载`gulp-load-plugins`的时候传入了`{lazy: true}`参数，这个参数可以让gulp的插件按需加载。另外一点，在调用子模块的时候我们只调了一个文件，那调多个文件时可以像上面那样使用`require-dir`这个模块吗？很不幸，从`require-dir`的文档中没有发现可以传参数的功能。那么为了使用传参数的的方式加载tasks文件夹里的所有子任务，应该怎么做呢？其实要做的只是拿到tasks文件夹下所有的文件名，然后require进来就OK。那么利用Node.js的文件操作模块`fs`就可以做到。

``` javascript
var taskList = require('fs').readdirSync('./tasks/');
taskList.forEach(function (file) {
  require('./tasks/' + file)(gulp, config, $, args);
});
```

使用fs模块的`readdirSync`方法列出所有文件夹里的所有文件，使用for循环依次require即可。

那么还有个问题，如果除开这4个参数以外，还想传别的参数呢？比如，很多task共用一个公共的log函数，那么怎么共享这个函数呢？一个最简单的办法就是把公共的函数写在`gulp.config.js`中，这也是我们为何将这个文件写成模块而不是简单的json文件。比如：

``` javascript
config.fn = {
  log: log
};
function log() { ... }
```

这样，在各个task文件中，可以轻松的使用`config.fn.log`来调用这个函数了。

### 将gulp相关文件移入单独文件夹

其实重构到这里已经差不多了，但其实还可以再进一步，将所有和gulp相关的文件全部移到一个单独的文件夹，这样，gulpfile中只写一句就可以了`require('./gulp/')`就可以了。现在的文件夹结构为：

```
gulpfile.js
├── gulp
    ├── gulp.config.js
    ├── index.js
    └── tasks/
        ├── css.task.js
        └── js.task.js
```

将现有的gulpfile中的内容放到`/gulp/index.js`中即可。有一点值得注意，由于文件夹结构的改变，在读取task时需要将路径改成`./gulp/tasks/`，即`var taskList = require('fs').readdirSync('./gulp/tasks/');`。

### 方便的列出所有gulp的任务

分散管理gulp任务以后，也带来了一些不便。比如，如何知道到底有多少gulp任务呢？有一个插件恰恰是用来解决这个问题的，叫[`gulp-task-listing`](https://github.com/OverZealous/gulp-task-listing)。这个模块使用起来也非常简单，只要在`/gulp/index.js`中添加`gulp.task('default', $.taskListing);`即可。这样使用`gulp`命令即可列出所有的gulp任务，更方便的是，如果task的名字中含有`-`或`_`或`:`则这个任务会被认为是子任务。比如，我们将`jshint`这个任务名改成`js:hint`，则`gulp`即会输出：

```
Main Tasks
------------------------------
    css
    default
    js

Sub Tasks
------------------------------
    js:hint
```

是不是一目了然？
