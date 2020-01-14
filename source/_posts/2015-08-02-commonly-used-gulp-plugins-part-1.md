title: 常用gulp插件介绍(一)
categories:
  - 前端开发
tags:
  - gulp
  - AngularJS
  - 插件
  - generator-aio-angular
date: 2015-08-02 13:45:59
---

在写[generator-aio-angular](https://github.com/PinkyJie/generator-aio-angular)的过程中，gulp这一块发现了很多非常实用的插件，大大的增加了能自动化的范围，这篇文章就分门别类的简单介绍下常用的gulp插件吧。

<!--more-->

### util工具类

这个分类下主要介绍一些辅助工具类的插件。

#### [gulp-load-plugins](https://www.npmjs.com/package/gulp-load-plugins)

顾名思义，本插件的功能就是帮你自动require你在`package.json`中声明的依赖。只要一句`var $ = require('gulp-load-plugins')()`，则`package.json`中声明的`gulp-`或`gulp.`开头的插件就会自动被放在变量`$`下面。如`$.util`就等于`require('gulp-util')`，而有两个连字符的插件则会自动命名为驼峰格式，如`$.taskListing`则等于`require('gulp-task-listing')`。有了这个插件，就不用一个一个的require了。这个插件还有一些常用的参数配置，这里列几个常用的：

* `lazyload: true`，用到这个插件的时候再去require，默认为true。
* `rename: {'gulp-task-listing': 'list'}`，如果有些插件名字太长，可以使用该参数重命名。
* `scope: ['dependencies']`，本插件默认会扫描`package.json`里的所有dependence，可以使用该参数进行限制。

要使用这些参数只要在require的时候传入即可，如`require('gulp-load-plugins')({lazy: true})`。

#### [gulp-task-listing](https://www.npmjs.com/package/gulp-task-listing)

这个插件的作用也很容易猜，它可以打印出`gulpfile.js`中定义的所有task，这个插件我们在[重构你的gulpfile](/2015/03/24/refactor-your-gulpfile/)这篇文章的最后介绍过，值得一提的是它还可以根据task的名字确定它是不是一个子task，比如带有`:`、`-`、`_`的task就被认为是子task。我一般把这个插件作为默认的task来调用，如

``` javascript
gulp.task('default', ['help']);
gulp.task('help', $.taskListing);
```

这样，如果只执行`gulp`的话就会打印出所有定义好的task，非常实用。

#### [yargs](https://www.npmjs.com/package/yargs)

严格的说，`yargs`不是专门用于gulp的，它是Node中处理命令行参数的通用解决方案。只要一句代码`var args = require('yargs').argv;`就可以让命令行的参数都放在变量`args`上，非常方便。它可以处理的参数类型也是多种多样的：

* 单字符的简单参数，如传入`-m=5`或`-m 5`，则可得到`args.m = 5`。
* 多字符参数（必须使用双连字符），如传入`--test=5`或`--test 5`，则可得到`args.test = 5`。
* 不带值的参数，如传入`--mock`，则会被认为是布尔类型的参数，可得到`args.mock = true`。

除此之外，还支持很多其他类型的传参方式，具体可参考[它的文档](https://www.npmjs.com/package/yargs)。

#### [gulp-util](https://www.npmjs.com/package/gulp-util)

gulp-util带有很多方便的函数，其中最常用的应该就是log了。`$.util.log()`支持传入多个参数，打印结果会将多个参数用空格连接起来。它与`console.log`的区别就是所有`$.util.log`的结果会自动带上时间前缀。另外，它还支持颜色，如`$.util.log($.util.colors.magenta('123'));`打印出来的123是品红色的。其实`$.util.colors`就是一个[chalk](https://github.com/sindresorhus/chalk)的实例，而chalk是专门用来处理命令行打印着色的一个工具。

#### [del](https://www.npmjs.com/package/del)

grunt自身提供一个[grunt-contrib-clean](https://github.com/gruntjs/grunt-contrib-clean)用来处理支持glob匹配的删除，而del就是gulp上对应的工具。del支持和`gulp.src`参数同样的匹配，除此之外，它的第二个参数还支持一个回调函数，当删除完成以后执行，所以这是一个异步的删除。常用的调用方法为：`del([xxx], callback)`。

#### [gulp-bytediff](https://www.npmjs.com/package/gulp-bytediff)

这是一个统计文件大小变化的工具，通常与压缩类工具放在一起实用，比如

``` javascript
gulp.src('**/*.html')
    .pipe($.bytediff.start())
    .pipe($.minifyHtml({empty: true}))
    .pipe($.bytediff.stop(bytediffFormatter))
    .pipe(gulp.dest('dist'));

function bytediffFormatter (data) {
    var difference = (data.savings > 0) ? ' smaller.' : ' larger.';
    return data.fileName + ' went from ' +
        (data.startSize / 1000).toFixed(2) + ' kB to ' +
        (data.endSize / 1000).toFixed(2) + ' kB and is ' +
        formatPercent(1 - data.percent, 2) + '%' + difference;
}
```

在压缩的pipe前后加上`$.bytediff.start()`和`$.bytediff.stop(callback)`，即可统计压缩前后文件的变化。在callback中传入的参数data上，可以访问到很多变量，如文件名，变化前后的大小，变化百分比等等。

#### [gulp-print](https://www.npmjs.com/package/gulp-print)

这个插件的作用很简单，打印出stream里面的所有文件名，通常调试的时候比较需要。

#### [gulp-bump](https://www.npmjs.com/package/gulp-bump)

这个插件也可以顾名思义：用来升级版本用的，废话不说，直接看例子吧：

``` javascript
return gulp
    .src('package.json')
    .pipe($.bump(options))
    .pipe(gulp.dest('dist'));
```

重点来看这里的options，我们可直接传递一个具体的version进去，也可以按照Node的版本规范传递一个type进去，让其自己生成对应的version：

* `version`，直接传递要升级到的版本号，如`1.2.3`。
* `type`，可接受的值包括下面四个，倘若现在的版本号为`1.2.3`，则对应的新版本号为：
    * prerelease：`1.2.3-0`
    * patch：`1.2.4`
    * minor：`1.3.0`
    * major：`2.0.0`

最终这个升级后的版本号会反映在`package.json`中，当然，你也可以在gulp.src中传入更多的文件（如`bower.json`）来替换更多的文件。

#### [gulp-header](https://www.npmjs.com/package/gulp-header)

这个工具用来在压缩后的JS、CSS文件中添加头部注释，你可以包含任意想要的信息，通常就是作者、描述、版本号、license等，比如：

``` javascript
function getHeader () {
    var pkg = require('package.json');
    var template = ['/**',
        ' * <%= pkg.name %> - <%= pkg.description %>',
        ' * @authors <%= pkg.authors %>',
        ' * @version v<%= pkg.version %>',
        ' * @link <%= pkg.homepage %>',
        ' * @license <%= pkg.license %>',
        ' */',
        ''
    ].join('\n');
    return $.header(template, {
        pkg: pkg
    });
}
```

这个函数将`package.json`中的各种信息提取出来，变成头部注释，只要在压缩的pipe中调用`.pipe(getHeader())`即可。

### stream相关

这个部分主要介绍一些跟stream操作有关的插件。

#### [gulp-filter](https://www.npmjs.com/package/gulp-filter)

gulp-filter可以把stream里的文件根据一定的规则进行筛选过滤。比如`gulp.src`中传入匹配符匹配了很多文件，可以把这些文件pipe给gulp-filter作二次筛选，如`gulp.src('**/*.js').pipe($.filter(**/a/*.js))`，本来选中了所有子文件下的js文件，经过筛选后变成名为a的子文件夹下的js文件。那有人要问了，为什么不直接将需要的筛选传入`gulp.src`，干嘛要多筛选一步呢？这里面有两种情况：

* `gulp.src`与`$.filter`中间可能需要别的处理，比如我对所有文件做了操作1以后，还需要筛选出一部分做操作2。
* 第二种情况就要谈到gulp-filter的另外一个特性：筛选之后还可以restore回去。比如我对所有文件做了操作1，筛选了一部分做操作2，最后要把所有的文件都拷贝到最终的位置。代码如下：

``` javascript
var filter = $.filter('**/a/*.js');
gulp.src('**/*.js')
    .pipe(action1())
    .pipe(filter)
    .pipe(action2())
    .pipe(filter.restore())
    .pipe(gulp.dest('dist'))
```

可以看到，如果没有restore这个操作，那么拷贝到最终位置的文件将只包含被过滤出来的文件，这样一restore，所有的文件都被拷贝了。

#### [gulp-flatten](https://www.npmjs.com/package/gulp-flatten)

gulp-flatten非常实用，可能知道别的库中flatten函数的同学已经猜到它是干嘛的了。比如`gulp.src('**/*.js')`匹配了很多文件，包括`a/b/c.js`，`d/e.js`，`f/g/h/i/j/k.js`，`l.js`，这些文件的层级都不一样，一旦我们将这个文件pipe给`$.flatten()`，则所有的文件夹层级都会去掉，最终的文件将是`c.js`，`e.js`，`k.js`，`l.js`，在一些场景下还是非常有用的。

#### [gulp-plumber](https://www.npmjs.com/package/gulp-plumber)

这个插件的作用简单来说就是一旦pipe中的某一steam报错了，保证下面的steam还继续执行。因为pipe默认的onerror函数会把剩下的stream给unpipe掉，这个插件阻止了这种行为。那它一般用于哪种场景呢？比如，代码每次build之前要跑一遍jshint和jscs来确保所有代码都符合规范，但一旦某些代码不符合规范，整个build流程就会停止，这个时候就需要gulp-plumber出场了。如：

``` javascript
gulp.task('build', ['jslint', 'xxxx']);
gulp.task('jslint', function () {
    return gulp
        .src(config.js.all)
        .pipe($.plumber())
        .pipe($.jshint())
        .pipe($.jscs()); 
});
```

这样，一旦jshint或jscs报错，整个build流程还是可以继续走下去的，而且gulp-plumber会给出一个报错提醒我们：

```
[16:52:36] Plumber found unhandled error:
 Error in plugin 'gulp-jshint'
Message:
    JSHint failed for: xxxx.js
```

#### [gulp-if](https://www.npmjs.com/package/gulp-if)

这个插件的功能也很简单，可以条件性的添加stream，如`.pipe($.if(flag, action1()))`，则只会在`flag`变量为true时才会将`action1()`添加到stream中去。其实不用这个插件也可以达到类似的效果，那就是gulp-util里有一个函数叫做`noop()`，也就是no operation，这个函数其实是返回一个什么都不干的空stream。利用这个函数我们可以这么写：`.pipe(flag ? action1() : $.util.noop())`，与上例的效果是一样的。

#### [merge-stream](https://www.npmjs.com/package/merge-stream)

一个gulp的task只能返回一个stream，但有的时候有这么一种情景：有两类文件，它们的原始位置和处理后的位置都是不同的，但它们的处理流程相同。由于`gulp.src`和`gulp.dest`的参数不同，我们就需要写两个task来分别完成这个任务，一方面略显重复，另一方面逻辑上来讲这两个task本来就是处理同样的事情的。这种情况就需要merge-stream登场了，它的作用就是将多个stream合成一个返回。比如下面这个例子：

``` javascript
var merge = require('merge-stream');
gulp.task('jade', function () {
    var stream1 = jade(src1, dest1);
    var stream2 = jade(src2, dest2);
    return merge(stream1, stream2);
});

function jade (src, dest) {
    return gulp
        .src(src)
        .pipe($.jade())
        .pipe(gulp.dest(dest));
}
```

可以看到，处理的流程被提取出来放入一个函数，它接受两个参数，分别是src和dest。然后在task中直接调用这个函数生成两个stream，然后返回merge-stream合并后的结果。

#### [run-sequence](https://www.npmjs.com/package/run-sequence)

gulp里的task都是异步并发执行的，有的时候我们需要一连串的task按顺序执行，这时就需要run-sequence登场了。它的调用很简单：`runSequence('task1', 'task2', ['task3', 'task4'], 'task5')`，这里的task都是gulp定义好的task名称，task1完成后才会执行task2，以此类推。注意到task3和task4被放在中括号里了，这表明，task3和task4可以并发执行的，但两个都执行完后才会执行task5。这里要说明的是，每个task要么返回一个stream，即`return gulp.src().pipe().pipe()`，要么支持回调函数，即`gulp.task('task1', function (done) { action1(done); })`，满足了这两点才能保证正常的执行顺序，因为这是gulp对[异步task的基本要求](https://github.com/gulpjs/gulp/blob/master/docs/API.md#async-task-support)。

### inject相关

这个部分主要介绍一些将JS/CSS自动插入到HTML的相关插件。

#### [wiredep](https://www.npmjs.com/package/wiredep)

wiredep就是wire dependence的意思，它的作用就是把`bower.json`中声明的dependence自动的包含到HTML中去。要插入文件，wiredep需要解决两个问题：

* 插入的位置：wiredep通过识别HTML中的注释来识别插入位置，如

``` html
<!-- bower:css -->
<!-- endbower -->
<!-- bower:js -->
<!-- endbower -->
```

不同类型的文件被插入到不同的区块。

* 插入什么文件：要插入的文件列表自然来自`bower.json`，每个bower安装的依赖库，根目录下边都有一个自己的`bower.json`文件，其中的`main`字段指明了使用这个库需要包含的文件，wiredep最终包含的文件列表就来自这个字段。有些情况下，库自身的`bower.json`的main字段可能会多包含文件或少包含文件，如果想要定制这个列表，则可以在自己的`bower.json`中使用`overrides`字段，如下面的代码覆盖了`mdi`这个库的`main`字段。

``` javascript
"overrides": {
  "mdi": {
    "main": [
      "css/materialdesignicons.css"
    ]
  }
},
```

wiredep插件支持很多参数，常用的主要有两个：

* bowerJson：指定`bower.json`的内容，注意这个字段不是`bower.json`文件的位置，这个参数需要使用require后的结果赋值：`require('bower.json')`。
* directory：指定存放bower安装后的依赖包的路径，通常是bower_components。注意最终插入到HTML中的文件列表的路径是index.html文件相对于本文件夹的相对路径。

使用wiredep也比较简单，直接把它传入到stream中即可，如`gulp.src('index.html').pipe(wiredep(options))`。

#### [gulp-inject](https://www.npmjs.com/package/gulp-inject)

这个插件的作用与wiredep类似，不同的是可以自己任意指定需要插入文件的列表。它同样是利用注释来寻找插入的位置，它识别的默认注释为`<!-- inject:js -->`，但更加智能：

* 支持各种模板语言：可以根据`gulp.src`指定的源文件自动识别注释和插入内容，除了支持HTML外，还支持jade、haml等。若源为jade文件，则识别的注释为`//- inject:js`，插入的内容为：`script(src="<filename>.js")`。
* 配置非常灵活：
    * name：默认识别的注释标签格式为`<!-- name:ext -->`，这里的name默认值就是“inject”，而ext的默认值是要插入的文件的扩展名。那么name属性可配置意味着可以添加自定义的插入区块，如`<!-- production:js -->`，这个标签可以只插入生产环境需要包含的JS文件。
    * starttag和endtag：支持自定义需要识别的注释内容。
    * addPrefix和addSuffix：支持在插入文件的路径上自定义前缀、后缀。
    * relative：指定插入文件的路径是否为相对路径。
    * ingorePath：指定插入文件的路径前面会忽略掉指定的路径。
    * read：这个参数通常给false，不需要真正的去读取文件。

这个插件的使用场景通常是，我们需要index里有多个区块，比如上面name的例子，只有当为production环境编译的时候才去包含相关的文件。

#### [gulp-useref](https://www.npmjs.com/package/gulp-useref) 与 [gulp-rev](https://www.npmjs.com/package/gulp-rev)、[gulp-rev-replace](https://www.npmjs.com/package/gulp-rev-replace)

这三个工具之所以放在一起讲，是因为它们一般都是一起使用的。它们要解决什么问题呢？通过上面的wiredep也好，gulp-inject也好，插入了一堆JS、CSS文件到HTML中，一旦部署到生产环境，这么多文件必然是要合并压缩的。光是压缩还不够，为了解决缓存问题，每次合并压缩后要给最终的文件加hash，这样每次文件内容一变动，hash也会跟着变动，就不存在浏览器依然使用缓存的老文件的问题。这样得到最终的文件以后，肯定还要将这个文件替换回HTML中去，一大堆的script和link标签替换成最终合并压缩带hash的版本。

前面啰啰嗦嗦的一大堆工作就是这三个插件要解决的问题了。首先，gulp-useref根据注释将HTML中需要合并压缩的区块找出来，对区块内的所有文件进行合并。**注意：它只负责合并，不负责压缩！**所以合并出来的文件我们要自行压缩，压缩以后调用gulp-rev负责在文件名后追加hash。最后调用gulp-rev-replace负责把最终的文件名替换回HTML中去。扯了大半天，还是直接上例子吧。先来看看HTML中的注释：

``` html
<!-- build:css static/styles/lib.css -->
<!-- bower:css -->
<!-- endbower -->
<!-- endbuild -->

<!-- build:css static/styles/app.css -->
<!-- inject:css -->
<!-- endinject -->
<!-- endbuild -->

<!-- build:js static/js/lib.js -->
<!-- bower:js -->
<!-- endbower -->
<!-- endbuild -->

<!-- build:js static/js/app.js -->
<!-- inject:js -->
<!-- endinject -->
<!-- endbuild -->
```

gulp-useref识别的就是build开头的注释，build后面首先跟的是类型扩展名，然后后面的路径就是build区块中的所有文件进行合并后的文件路径，这个相对路径是相对于这个HTML的路径。上面的例子中我们用build区块把bower和inject进来的文件包起来，这些文件就可以被gulp-useref合并了。再来看gulp中useref相关task的定义：

``` javascript
var assets = $.useref.assets({searchPath: 'app/src/'});

var cssFilter = $.filter('**/*.css');
var jsAppFilter = $.filter('**/app.js');
var jslibFilter = $.filter('**/lib.js');

return gulp
    .src('index.html')
    .pipe(assets)
    .pipe(cssFilter)
    .pipe($.csso())
    .pipe(cssFilter.restore())
    .pipe(jsAppFilter)
    .pipe($.uglify())
    .pipe(getHeader())
    .pipe(jsAppFilter.restore())
    .pipe(jslibFilter)
    .pipe($.uglify())
    .pipe(jslibFilter.restore())
    .pipe($.rev())
    .pipe(assets.restore())
    .pipe($.useref())
    .pipe($.revReplace())
    .pipe(gulp.dest('dist'));
```

首先一上来，先调用`$.useref.assets()`函数，这个函数返回一个stream，包含已经合并后的文件。可以尝试在第9行后面加上前面介绍过的gulp-print插件`.pipe($.print())`，打印出stream里的文件，发现就是前面HTML中4个build注释块后面的4个文件。注意这里调用的时候跟了一个`searchPath`的参数，它的用处就是指定从哪个路径开始寻找build区块底下的文件。比如build区块底下有这么一行`<script src="static/js/a.js"></script>`，那最终gulp-useref将从这个路径`app/src/static/js/a.js`找到这个文件。第3到5行定义了3个filter，这主要是为了后面压缩准备的。下面正式看stream的pipe流程。先选出要处理的HTML文件，然后调用刚才得到的`assets`得到合并后的4个文件，第10到12行筛选出合并后的CSS文件进行压缩（压缩类插件下篇文章再讲），第13到16行筛选出app.js进行压缩，第17到19行筛选出lib.js进行压缩。之所以要区别对待app.js和lib.js，是因为app.js是我们自己写的代码，压缩后要加上header（第15行，使用前面介绍过的gulp-header插件），而lib.js是第三方的各种库，直接压缩即可。后面调用gulp-rev给压缩后的4个文件加hash，然后调用`assets.restore()`将src源换回HTML文件，这是为了后面调用`$.useref()`，因为`$.useref()`做替换的src源是HTML文件，同样后面调用gulp-rev-replace将带hash的文件替换回HTML，它要求的src源也必须是HTML文件。这里的顺序很重要，因为这几个插件接受的源不一样，gulp-rev接受的是JS、CSS文件，而gulp-useref和gulp-rev-replace接受的是HTML。还有一个问题：gulp-rev-replace是怎么知道gulp-rev进行hash前后的文件名对应关系呢？其实gulp-rev会生成一个manifest的文件，内容是类似下面的JSON：

``` javascript
{
    "static/styles/lib.css": "static/styles/lib-d41d8cd98f.css"
    "static/js/lib.js": "static/js/lib-273c2cin3f.js"
}
```

当然这个文件默认是不会生成在文件系统里的，可以通过`.pipe($.rev.manifest())`将这个文件保存到本地。有了这个文件，gulp-rev-replace甚至可以脱离gulp-rev独立工作哦！

好了，这篇就到这里，还有好多工具没介绍到，留着给下篇吧。
