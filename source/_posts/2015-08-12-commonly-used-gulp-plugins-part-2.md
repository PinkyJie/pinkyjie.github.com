title: 常用gulp插件介绍(二)
categories:
  - 前端开发
tags:
  - gulp
  - AngularJS
  - 插件
  - generator-aio-angular
date: 2015-08-12 20:45:59
---

书接[上回](/2015/08/02/commonly-used-gulp-plugins-part-1/)。

<!--more-->

### Angular相关

这个部分介绍与Angular相关的一些插件。

#### [gulp-angular-templatecache](https://www.npmjs.com/package/gulp-angular-templatecache)

Angular自带的`$templateCache`服务可以把Angular中用到的所有模板缓存下来，而这个插件的功能就是直接将指定的HTML模板文件以JS字符串的形式注册在`$tempalteCache`服务中，这样所有的模板就会随JS文件直接一次性下载下来。这个插件使用起来也非常简单，gulp.src传入需要缓存的HTML模板文件，然后`.pipe($.angularTemplatecache(filename, options))`即可。其中filename表示生成后的js文件的名字，默认为templates.js，常用的options有：

* `module`：指定希望将这个模板放入哪个Angular的module中。
* `root`：指定注册后的模板路径前缀。

生成后的文件如下：

``` javascript templates.js
angular.module("module name").run([$templateCache,
  function($templateCache) {
    $templateCache.put("template1.html",
      // template1.html content (escaped) 
    );
    $templateCache.put("template2.html",
      // template2.html content (escaped) 
    );
    // etc. 
  }
]);
```

#### [gulp-ng-annotate](https://www.npmjs.com/package/gulp-ng-annotate)

这个插件是[ng-annotate](https://github.com/olov/ng-annotate)的gulp插件版，它解决的是Angular中依赖注入的小问题。Angular中通过参数名来进行依赖注入，一旦压缩，参数名就会变化导致注入失败，所以官方推荐通过添加字符串进行注入。比如：

``` javascript
angular
    .module('app.dashboard')
    .controller('DashboardController', DashboardController);

DashboardController.$inject = ['userAPI'];
/* @ngInject */
function DashboardController (userAPI) {}
```

上面的例子中我们定义了一个叫`DashboardController`的controller，它依赖一个`userAPI`的service。这个插件的作用就是根据第6行的注释`/* @ngInject */`来帮你生成第5行的内容。当然是在你忘记写的情况下，如果你自己写了它不会重复生成。除了这种使用`$inject`赋值的方式，它同样支持inline定义的方式，如

``` javascript
/* @ngInject */
.controller('DashboardController', function (userAPI) {});
```

会生成

``` javascript
.controller('DashboardController', ['userAPI', function (userAPI) {}]);
```

它常用的参数就是`{add: true}`，表明仅在不存在的情况下才会进行添加。
> 推荐在HTML头上使用[ng-strict-di](https://docs.angularjs.org/error/$injector/strictdi)属性，这样即便在不压缩的情况下，一旦你忘记显式的用字符串声明依赖，Angular将立刻报错。

#### [gulp-protractor](https://www.npmjs.com/package/gulp-protractor)

Angular的e2e测试工具[protractor](https://github.com/angular/protractor)的配套插件，可以通过它非常方便的在gulp中调用protractor。有了这玩意，你就不需要手动在gulp中调用protractor的可执行文件，然后处理进程神马的，只要一句简单的`.pipe($.protractor.protractor(options)`即可。常用的options包括：

* `configFile`：即protractor的配置文件路径。
* `args`：调用protractor时传入的参数，是个数组。最常用的就是指定protractor只跑一个suite了，如`['--suite', 'loginSuite']`，这样protractor只会跑配置文件中定义的loginSuite所包括的spec文件了。

#### [gulp-order](https://www.npmjs.com/package/gulp-order)

这个插件严格来说不是专门给Angular用的，但非常适合用在Angular的场景下。如果你的程序使用的是Angular自带的包管理系统，那么有一个无法避开的问题就是：所有`angular.module`的定义要最先执行，也就是说包含module定义的文件的script标签要在别的文件之前。而我们在使用gulp-inject这类插件将JS文件插入的时候通常都是通过匹配符直接选中一堆文件插入的。这时就需要解决插入的顺序问题，而这个插件就是干这个事的。它通过一个数组参数来指定排序，这个数组包含一组匹配模式，匹配到靠前模式的文件在前，匹配靠后的文件在后。如：

``` javascript
gulp
    .src('**/*.js')
    .pipe($.order([
        '**/app.module.js',
        '**/*.module.js',
        '**/*.js'
    ]))
```

这样，定义`app`module的文件就会在最前面，然后是其它各个module的定义，最后是剩余的JS文件。

### 压缩工具类

这个部分介绍对CSS、HTML、JS、图片等资源进行压缩的插件。

#### [gulp-csso](https://www.npmjs.com/package/gulp-csso)

压缩CSS的工具，官方说它比其它工具压得更小，因为它可以重建CSS代码结构信息，不知道什么鬼。

#### [gulp-minify-html](https://www.npmjs.com/package/gulp-minify-html)

压缩HTML的工具，通常在给gulp-angular-templatecache处理前先使用，这样$tempalteCache得到的就是压缩后的HTML字符串了。

#### [gulp-uglify](https://www.npmjs.com/package/gulp-uglify)

压缩JS的工具，这个不多介绍了。

#### [gulp-imagemin](https://www.npmjs.com/package/gulp-imagemin)

压缩图片的工具。在发布到生产环境之前对图片进行压缩是一个非常好的习惯，可以极大的提高页面加载的速度。如果你用[Google PageSpeed](https://developers.google.com/speed/pagespeed/)给网页评过分的话，它可以给出页面上能被继续压缩的图片。使用这个插件可以在保证质量损失很小的情况下压缩图片。

### server相关

这部分介绍与本地起server相关的插件。

#### [browser-sync](https://www.npmjs.com/package/browser-sync)

[Browsersync](http://www.browsersync.io/)应该算是本地起server的标配了吧，最大的特性是可以在不同浏览器之间同步（这也是名字的由来吧），这在测试时非常有用：起server以后根据配置自动打开多个浏览器，你操作一个，其他的浏览器会同步你的操作。另外，它还可以配合gulp的`watch()`函数实现类似live-reload的功能。之所以没有gulp对应的插件，是因为这玩意本来就可以直接require进来使用。支持非常多的参数，一整篇文章也介绍不完，具体可以参考[它的文档](http://www.browsersync.io/docs/options/)。这里要简单介绍的就是server底下的配置，如：

``` javascript
server: {
    baseDir: './app/',
    index: 'index.html',
    middleware: [
        // middleware
    ]
}
```

前两个参数就不多说了，一看就明白意思，重点来看第3个参数`middleware`。middleware就是中间件，类似这样的函数`function (req, res, next){}`。简单来说，一个request请求到达server，会经过middleware数组里面的中间件函数逐个处理，这里就可以在Browsersync层面上定义很多自定义的操作。适用于Node的[connect框架](https://github.com/senchalabs/connect)的中间件都可以在这里使用，下面介绍的两个插件都是可用于Browsersync的中间件。

#### [connect-history-api-fallback](https://www.npmjs.com/package/connect-history-api-fallback)

这个中间件对于像Angular这样的单页面应用来说非常的实用。我们知道，Browsersync默认起来的server是一个静态server，默认是无法支持`$locationProvider.html5Mode(true);`的。使用这个插件可以轻松的达到这一点，如：

``` javascript
var historyApiFallback = require('connect-history-api-fallback');
// ...
middleware: [
    historyApiFallback()
]
```

这样，所有的路由请求都会fallback到index.html处理，这也正是我们想要的。除此之外，这个插件还支持简单的rewrite，如

``` javascript
historyApiFallback({
  rewrites: [
    { from: /\/soccer/, to: '/soccer.html'}
  ]
});
```

可谓是非常方便，更多的rewrite规则可以参考它的文档。

#### [proxy-middleware](https://www.npmjs.com/package/proxy-middleware)

这个中间件其实与上面rewrite的类似，只不过rewrite只是针对get请求，而这个proxy可以代理任何请求。设想这样一个场景，我们起了个本地的server，通常使用[ngMock](https://docs.angularjs.org/api/ngMock)来实现对API的模拟，但有的时候我们希望这个本地的server可以对接真正的API server，而这个中间件可以轻松的完成这一点：将`/api`开头的请求代理到真正的API server上去。它的使用也是非常简单：

``` javascript
var proxy = require('proxy-middleware');
var proxyOptions = url.parse('https://real-server.com/api');
proxyOptions.route = '/api';
// ...
middleware: [
    proxy(proxyOptions)
]
```

这样，所有以`/api`开头的API请求就会被代理到`https://real-server.com/api`上去，如`/api/user/123`请求的真实地址是`https://real-server.com/api/user/123`。

### 特定语言相关

这部分的插件与你选用的具体的语言以及预处理器有关。

#### [gulp-jshint](https://www.npmjs.com/package/gulp-jshint) 与 [gulp-jscs](https://www.npmjs.com/package/gulp-jscs)

大名鼎鼎的[jshint]()和[jscs]()的gulp插件版。这两个插件除了帮你做代码的一些静态检查外，还可以最大程度的帮助你定义所需要的代码风格。尤其是`jscs`，定义的非常细致。比如我们需要`function (a, b) {`，即function关键字后面空一格，参数之间空一格，参数列表后面的小括号与大括号之间空一格。这样的需求通过`jscs`的配置文件可以轻松的实现，具体可以参考其文档。可以将这两个task放在build之前，强制所有人在build代码的时候修改不符合要求的代码风格。我们还尝试过将`gulp jshint jscs`放入git commit的hook中，每次commit的时候自动检查代码风格，如果不符合要求，不准进行commit的操作。

#### [gulp-jade](https://www.npmjs.com/package/gulp-jade)

编译jade模板的插件，这个也不过多介绍。只介绍一个使用jade变量的场景，通常我们的Angular应用的`ng-app`名称在测试(e2e)与非测试时是不一样的，所以可以把这个定义成变量，在编译jade模板时传入。如我们的index.jade的头是这样定义的`html(lang="en", ng-app= app)`，在编译时使用`.pipe($.jade({locals: {app: 'test'}}))`即可指定想要的`ng-app`的名字。

#### [gulp-stylus](https://www.npmjs.com/package/gulp-stylus)

编译stylus的插件，不多说。

#### [gulp-autoprefixer](https://www.npmjs.com/package/gulp-autoprefixer)

这个插件最早在[从做简历中学到的知识](/2014/01/12/what-i-learn-from-making-resume/)这篇文章中就介绍过，只不过当时介绍的是grunt版本，现在时gulp版本。这个插件的基本作用就是让你在书写CSS3的相关属性时不用关心不同浏览器的前缀问题，它会自动帮助添加各种浏览器前缀。


