title: webpack使用小记
categories:
  - 工具相关
tags:
  - AngularJS
  - ui-router
  - webpack
  - LazyLoad
  - ocLazyLoad
  - angular1-webpack-starter
date: 2016-03-05 15:29:17
---

如果前两年大家还在讨论grunt和gulp等构建工具的话，现在无疑是[webapck](https://webpack.github.io/)的时代。严格来讲，webpack其实和grunt/gulp根本不是一种东西，它不是一个构建工具，而是`module bundler`。简单来说，webpack将JS、CSS、HTML（包含各种预处理器）以及图片等等都视为“资源”，每个资源文件都是一个module文件，而module文件之间存在依赖，webpack就是根据module文件间的依赖将所有module打包（bundle）起来。而回忆我们用grunt/gulp构建项目时，做的很大一部分工作也无非是将JS、CSS、HTML编译合并压缩等等，所以从这个层面上讲用webpack和grunt/gulp得到的结果是一样的。但webpack好就好在使用loader的概念让配置更加容易，再也不用和一堆文件路径打交道了。这篇文章就讲一下自己在使用webpack时的一些实践。

<!--more-->

### 使用npm管理所有依赖

说到前端的依赖管理，可能很多人脑海里第一印象还是[bower](http://bower.io/)，其实bower更像是一个依赖下载工具，你还是需要手动将这些下载好的文件用script标签引用到页面中去（尽管有[wiredep](https://github.com/tanodeptapship/wiredep)帮我们自动化），所以它其实并不能真正的做到“依赖管理”。另外，bower据说已经要停止开发了，与此同时，越来越多的前端项目都开始支持npm，支持使用node的CommonJS机制来发行它们的前端库。这样在前端项目的package.json中不仅需要`devDependencies`，长期受冷落的`dependencies`也要用起来了。前者用来声明一些build过程中需要用的到一些构建工具，而后者用来声明开发使用到的前端库。webpack虽然支持多种包管理机制，但CommonJS应该还是最推荐的吧。用npm将要使用的库安装好后，直接在js文件中require即可。当然，使用ES6的话（需配合Babel），import它也是支持的哦。这样一来，第三方库的管理，项目中文件间的依赖管理再也不是问题了，可以像写node以及其他后端语言那样畅快的引用来引用去了。

说到依赖管理，还有个问题不得不提。package.json文件中采用node的SemVer来管理版本号，默认的`^`符号会匹配最新的minor版本。简单的说，你安装的时候可能是`^1.4.3`，但是一旦1.5.0发布，你运行npm install就会装1.5.0，也就是说它只会保留第一个数字。所以有需要的话可以把`^`改为`~`，它的限制更严格一些，`~1.4.3`最多只会安装到1.4.9，可以最大程度的减少升级带来的兼容性问题。有人会想，有没有类似python的`pip freeze`功能把依赖锁死在指定的版本上呢？当然！试试`npm shrinkwrap`吧，它会将当前安装在node_modules下的依赖版本写死在生成的node-shrinkwrap.json文件中，将这个文件加到git里，别人使用npm install安装依赖包时就可以保证装到和你一模一样的版本了。

### 善用yargs，避免多个配置文件

很多github上的webpack starter项目都提供了多个webpack.config.js：如`webpack.prod.config.js`和`webpack.dev.config.js`，目的就是为了适配不同的环境。其实开发和发布的配置大部分是相同的，所以里面会有很多重复的配置。还有的项目看到了这一点，来了一个`webpack.make.config.js`提供默认的大部分配置，然后在其他的配置文件里require这个文件，做相应的更改。反正都是提供多个webpack配置文件，然后不同的环境用不同的。有必要这么复杂吗？在前面的文章中我反复提到一个概念，虽然`xxx.config.js`是配置文件，但是里面可以包含任何的node逻辑，所以为什么不试试只维护一个配置文件，针对不同的环境传不同的参数呢？像下面这样：

``` javascript
"scripts": {
    "start": "webpack-dev-server --mock",
    "build": "rm -rf ./build && webpack --progress --mock --prod"
}
```

默认会查找名为`webpack.config.js`的配置文件，然后在这个文件中我们去处理自定义的参数即可。这里又要安利[yargs](https://www.npmjs.com/package/yargs)了，一个非常便捷的命令行参数处理工具。具体可以参考[前面文章](/2016/02/27/continuous-integration-with-travis-ci/#识别Travis的环境)的讲解和它的文档，这里不再赘述。

### 常用插件

webpack的配置可以说就是`module`和`plugins`的配置，`module`里主要就是配置各种loaders，没啥可说的，你要require什么类型的文件就去搜相应的loader就好，这一节主要说后面的这个：`plugins`的配置。webpack支持非常多的插件，详见[插件列表](http://webpack.github.io/docs/list-of-plugins.html)，下面就讲一些常用的场景会用到的插件。

#### 将jQuery全局暴露

虽然很多人已经不提倡再使用jQuery，但很多第三方的库依然依赖jQuery，如果我们要使用这些第三方的库，那么我们还是没法摆脱jQuery。那么正常情况下我们只要保证jQuery的script标签在第三方库前面即可，因为jQuery中会暴露`$`给全局（`window.$`）使用。但使用了webpack后情况就稍显复杂了，作为module bundler，默认是不会有任何module暴露给全局的。这样，如果第三方库的代码中出现`$.xxx`或`jQuery.xxx`或`window.jQuery`或`window.$`则会直接报错。要达到类似的效果，则需要使用webpack内置的ProvidePlugin插件，配置很简单，只需要：

``` javascript
new webpack.ProvidePlugin({
    $: "jquery",
    jQuery: "jquery",
    "window.jQuery": "jquery"
})
```

这样，当webpack碰到require的第三方库中出现全局的`$`、`jQeury`和`window.jQuery`时，就会使用node_module下jquery包export出来的东西了。

#### 给JS定义全局flag

在JS代码中我们常常需要一些静态的flag来标识是开发环境还是生产环境，一个简单的例子就是，开发环境你的log可能是直接console里打出来就好，而生产环境中就需要将log上传到统一的地方方便查询。也就是说，有些代码可能只是为了生产环境而存在的，而有些代码只为开发环境存在。我在为生产环境编译打包的时候就需要移除这些没用的代码，移除的操作[UglifyJS](https://github.com/mishoo/UglifyJS2)可以胜任，但前提是它只移除类似`if (false) {...}`的代码片段，所以我们要做的就是定义全局的布尔flag，在build的时候传入不同的参数来控制它是true还是false。这时就需要使用webpack内置的DefinePlugin插件了：

``` javascript
// args = require('yargs').argv;
new webpack.DefinePlugin({
    __PROD__: args.prod，
    __MOCK__: args.mock
}),
```

上面我们同样使用了yargs，这样在build时传入`--prod`，则变量`__PROD__`即为true，传入`--mock`则`__MOCK__`为true。在JS代码中就可以使用类似的判断`if (__PROD__) {...}`了。

#### 第三方库输出单独的JS

一般我们为生产环境打包JS时，不管源文件有多少个，一般都是输出两个最终文件：`app.js`和`vendor.js`（LazyLoad的情况另说），前者是我们自己编写的代码合并压缩而成，后者是我们使用的第三方库合并压缩后的结果。使用webpack实现这个需要也很简单，使用webpack内置的CommonsChunkPlugin即可，我们要做的就是两步：

* 在entry中定义app和vendor这两个模块：

``` javascript
entry: {
    app: [
        'source/app/index.js'
    ],
    vendor: [
        'angular',
        'angular-ui-router',
        'angular-animate'
        // ...
    ]
}
```

* 在plugins里使用该插件：

``` javascript
new webpack.optimize.CommonsChunkPlugin('vendor', isProd ? 'vendor.[hash].js' : 'vendor.js')
```

这样，所有模块中只要有require到vendor数组中定义的这些第三方模块，那么这些第三方模块都会被统一提取出来，放入`vendor.js`中去。在插件的配置中我们还进行了判断，如果是生产环境则给最终生成的文件名加hash。

#### 将CSS由style内嵌变成独立.css文件

在处理CSS时webpack需要两个不同的loader：`css-loader`和`style-loader`，前者负责将CSS文件变成文本返回，并处理其中的url()和@import()，而后者将CSS以style标签的形式插入到页面中去。一般使用的时候这么写：

``` javascript
{
    test: /\.styl$/,
    loader: 'style!css?sourceMap!autoprefixer!stylus'
}
```

上面的loader配置中，我们处理所有的styl文件（stylus），先用stylus-loader将stylus变成CSS，然后使用autoprefixer-loader添加各种浏览器兼容性前缀，最后使用css-loader和style-loader。但这样的问题就是所有CSS文件都是直接以style标签的形式插入页面，不利于缓存，我们还是更习惯build之后能够输出单独的.css文件。这时就需要webpack的[extract-text-webpack-plugin](https://github.com/webpack/extract-text-webpack-plugin)插件了，注意它不是内置的，需要额外安装。它的使用也很简单，分两步：

* 在loaders定义中指定：

``` javascript
{
    test: /\.styl$/,
    loader: ExtractTextPlugin.extract('style', 'css?sourceMap!autoprefixer!stylus')
}
```

* 在plugins中使用插件：

``` javascript
var ExtractTextPlugin = require("extract-text-webpack-plugin");
// ...
new ExtractTextPlugin(isProd ? '[name].[hash].css' : '[name].css')
```

同样，这里我们在生产环境使用了hash命名。

#### copy指定文件到指定路径

按理说用了webpack后所有的模块都通过require来管理了，用不着这样针对某个文件的copy了。这确实是一个不太常见的场景，比如说使用ES6的项目中，即便使用了Babel，但它并不会翻译类似`Symbol`这样的关键字，造成IE的兼容性问题：IE里会直接报“Symbol not defined”错误。这个时候我们就需要在页面中引入Babel里自带的shim文件了，`babel-core`中有一个browser-polyfill.min.js文件，是浏览器直接可以include的文件，并不需要使用webpack把它包一层。所以我们要做的其实是build的时候将这个文件copy到指定的目录，然后在index.html中包含这个文件即可。这个时候就需要[copy-webpack-plugin](https://www.npmjs.com/package/copy-webpack-plugin)这个插件了，配置起来也是非常的简单：

``` javascript
var CopyWebpackPlugin = require('copy-webpack-plugin');
// ...
new CopyWebpackPlugin([
    { from: 'node_modules/babel-core/browser-polyfill.min.js', to: 'polyfill.js'}
])
```

这样文件就会被复制到webpack的output.path配置的路径下了。

#### 定义入口HTML文件

单页应用还是需要一个入口的index.html文件的，这个文件的功能就是include所有的JS、CSS文件，然后body里指定一个ng-view或ui-view即可。这么机械化有规律的动作当然有插件来帮我们做了，那就是[html-webpack-plugin](https://github.com/ampedandwired/html-webpack-plugin)。最简单的配置就是直接在plugins直接定义：

``` javascript
var HtmlWebpackPlugin = require('html-webpack-plugin');
// ...
new HtmlWebpackPlugin()
```

默认的配置会生成一个结构非常简单的index.html文件，HTML的title是“Webpack App”，然后entry中定义的bundle生成的JS会以script标签包在body里，CSS会以link标签包在head里。显然，默认的模板是没法满足我们的需求的，我们需要给插件传入参数进行配置：

* title和filename：控制生成页面的title和文件名。
* inject：控制JS插入的位置，body或者head。
* favicon：可以指定一个favicon.ico文件路径，作为网站的图标。
* chunks和excludeChunks：一个数组，包含你需要引入或排除的bundle的名字（定义在entry中）。
* template：自定义模板。

这里我们重点来说一说template这个配置，因为Angular的场景是在body里需要一个ui-view的，所以我们必须要自定义模板。template不仅支持直接传入HTML文件的路径，也支持各种模板语言，来看看这个定义：

``` javascript
new HtmlWebpackPlugin({
    template: 'jade!./source/app/index.jade',
    chunks: ['app', 'vendor'],
    favicon: 'favicon.ico',
    appName: appName
}),
```

在template中我们指定了jade模板文件，然后配上jade-loader。这里注意的是，如果你想向jade文件中传参数，类似`jade?foo=bar!xxx.jade`，貌似是不支持的。那怎么实现这个需求呢？比如我想让build的时候动态决定index.html中的ng-app名字，怎么办呢？上面的`appName: appName`就是答案。在传给插件的配置中，除了它自身支持的配置，我们可以传入任意名称的键值对，然后在jade文件中可以这样访问到：

``` jade
html(lang="en", ng-app= htmlWebpackPlugin.options.appName)
```

除了`htmlWebpackPlugin`变量外，在自定义模板中，还可以访问`webpack`和`webpackConfig`，前者可以获取webpack的stats，如`webpack.hash`获取本次webpack编译得到的hash码，后者可以获取webpack的所有配置。这两个变量在一些场景下也是非常有用的。最后还是一点要说，html-webpack-plugin支持生成多个HTML文件，只要在plugins中定义多个`new htmlWebpackPlugin({...})`即可。

#### `-p`参数的坑

`-p`参数是官方推荐的Production shortcut，意思就是为生产环境build时请使用`webpack -p`，那么`-p`指什么？文档里说了，代表`--optimize-minimize`和`--optimize-occurence-order`这两个命令行参数。先来说说第二个，在[文档](http://webpack.github.io/docs/list-of-plugins.html#occurrenceorderplugin)里解释了，webpack会把所有的chunk当做资源进行编号，这个插件可以保证出现次数多的资源编号小一些，短一点，这样有助于减小最终的文件大小。再来看看第一个`--optimize-minimize`，在webpack的[插件列表文档](http://webpack.github.io/docs/list-of-plugins.html)中没有找到。没关系，我们直接搜源码，可以发现，在`webpack/bin/convert-argv.js`（1.12.14版本）中有这么一段代码：

``` javascript
ifBooleanArg("optimize-minimize", function() {
    ensureArray(options, "plugins");
    var UglifyJsPlugin = require("../lib/optimize/UglifyJsPlugin");
    options.plugins.push(new UglifyJsPlugin());
});
```

也就是说，这个参数实际上会加载UglifyJsPlugin这个插件，而且是无参（默认参数）加载的。这里坑就来了！如果你的ES6代码中出现类似这样的定义：

``` javascript
import MockData from './e2e.data';
angular.module('appTest', [])
    .service(MockData.name, MockData)

// e2e.data.js
class MockData {...}
export default MockData;
```

也就是说，使用module的`.name`属性来决定一个service的名字，然后在别的地方使用依赖注入来引入这个service的话，这个时候一旦你使用`-p`参数，程序就会报错：找不到provider，`MockDataProvider <- MockData`。因为在e2e.data.js文件中你export的class虽然叫MockData，但这个名字会被UglifyJsPlugin改掉。这是因为这个插件有一个`mangle`选项，会对所有函数名变量名进行混淆，在压缩的同时保证安全。这倒没什么，坑就坑在，webpack自己的文档中告诉大家这个mangle的option默认是flase的，也就是说不会混淆。但在UglifyJsPlugin插件的源码里明明写着`if(options.mangle !== false) {...}`，除非你传入一个flase，默认确实是会混淆的。你说这误导不误导，我在Github上提了个issue：[webpack/docs #49](https://github.com/webpack/docs/issues/49)，虽然官方没管，但最新的文档中貌似已经修复了这个错误。所以我的建议是：**不要使用`-p`参数**！想要什么自己定义就好：

``` javascript
if (isProd) {
    plugins.push(
        new webpack.NoErrorsPlugin(),
        new webpack.optimize.DedupePlugin(),
        new webpack.optimize.UglifyJsPlugin({
            compress: {
                warnings: false
            },
            mangle: false
        }),
        new webpack.optimize.OccurenceOrderPlugin()
    );
}
```

有了上面的定义，在build的时候只要使用`webpack --prod`就好。还有一点值得一说，如果你使用了`-p`，即便你额外定义了一个自己的mangle=false的UglifyJsPlugin插件在webpack配置里，也是不起作用的。所以，还是那句话，不要使用`-p`参数。

### 轻松实现LazyLoad

使用webpack的[Code Splitting](http://webpack.github.io/docs/code-splitting.html)可以非常方便的实现LazyLoad。对于大型的应用来说，一次性将所有文件全部下载进浏览器显得很不划算，因为有些功能可能不太常用，不需要一开始就加载。Code Splitting功能可以在代码中定义“分割点”，即代码执行到这些点时才会去加载所需的模块，这样就实现了按需加载的LazyLoad。下面就以Angular和ui-router为例来看看具体怎么LazyLoad。假设我们的网站有4个大页面（每个页面里可能有很多子页面），我们希望路由到某个页面时再去加载这个页面所需要的模块。要实现这个LazyLoad，我们要明确两点：

1. “分割点”设在哪？
2. 一个页面需要的模块包括HTML、CSS、JS，具体怎么去加载这三种类型？

第一个问题很简单，由于我们的想法是在路由到某个页面时再去加载所需模块，所以很自然的，我们会想把“分割点”放在路由的定义中。放在路由中还可以帮我们解决第二个问题，因为路由中通常需要定义路由到这个页面时的template和controller，这恰恰是我们上面提到的HTML和JS部分。下面我们就来看看具体怎么操作吧。

#### 通过templateProvider动态加载模板

我们可以在定义路由的时候在template中去动态加载所需的模板，但是template参数是个string，显然不满足我们的要求（templateUrl显然也不适合我们的场景）。这里就需要ui-router的[templateProvider参数](http://angular-ui.github.io/ui-router/site/#/api/ui.router.state.$stateProvider#methods_state)了，它的值可以是一个函数，然后在这个函数中支持返回一个promise，由这个promise最终来返回HTML字符串。来看看代码：

``` javascript
$stateProvider.state('root.layout.phone', {
    views: {
        url: '/phone',
        'main@root': {
            templateProvider: ['$q', ($q) => {
                return $q((resolve) => {
                    require.ensure([], () => {
                        resolve(require('./phone.jade'));
                    }, 'phone');
                });
            }],
            controller: 'PhoneController as vm'
        }
    }
});
```

上面的例子中，我们想要在访问`/phone`这个路由时再去加载`phone.jade`这个template。来看看templateProvider的配置，首先它的函数返回一个promise（第6行），这个promise我们直接使用`$q`的构造函数，这个构造函数接受两个参数：`resolve`和`reject`，分别用来resolve和reject这个promise。然后在这个promise中我们要resolve的就是`phone.jade`这个template的字符串内容，使用jade-loader配置后，`require('./phone.jade')`返回的就是jade编译后的HTML字符串了（第8行）。那么什么时候去resolve呢？这里我们重点来看看第7行：webpack的`require.ensure`函数，我们就是使用这个函数来实现“Code Splitting”的，它有三个参数：

1. 依赖的模块数组：模块名组成的数组，webpack会先加载这些模块再执行后面的回调函数。
2. 回调函数：在模块数组加载完以后才会执行这个。
3. chunk的名字：从这个“分割点”分割出去的模块会放入额外的模块，这个参数指定模块的名字。这个参数主要用于存在多个`require.ensure`的情况，我们可以将多个“分割点”分割出去的代码放入同一个模块。

为什么我们在第7行使用一个空数组呢？其实我们可以写`require.ensure(['./phone.jade'], () => {resolve(require('./phone.jade'));}, 'phone')`，但因为我们在回调里立马又require了这个jade，所以外面不写是无所谓的。

#### 使用resolve来动态加载所需JS

说完了templateProvider，我们再来看看下面的controller参数，controller参数与平常的写法并没有什么区别（第12行），因为没有一个叫controllerProvider的参数来让我们做类似的事情。那controller的JS模块在哪里动态加载呢？别忘了我们有resolve啊，每个路由定义都可以有一个resolve，只有这个resolve的promise被resolve了（有点绕了。。。）才会真正进入到那个路由中去，所以这正是我们加载controller和phone模块依赖的其它模块的好时机。来看看代码：

``` javascript
resolve: {
    loadModule: ['$q', '$ocLazyLoad', ($q, $ocLazyLoad) => {
        return $q((resolve) => {
            require.ensure([], () => {
                $ocLazyLoad.load({name: require('./index').name});
                resolve();
            }, 'phone');
        });
    }]
}
```

我们为这个resolve取名loadModule，因为它的作用就是加载phone模块自身的所有JS文件以及它所依赖的其它模块，并不需要我们真正的去return什么，所以第6行我们直接resolve了空值。另外注意第7行我们同样使用了phone作为chunk名，这保证了这个“分割点”分割出去的代码和上面分割出去的template是在一起的。那么JS依赖具体是怎么加载进来的呢？全靠第5行的`./index.js`文件了，来看看看这个`./index.js`的定义：

``` javascript
import angular from 'angular';

import PhoneController from './phone.controller';
import PhoneAddController from './add/phone-add.controller';
import PhoneDetailController from './detail/phone-detail.controller';
import PhoneService from './phone.service';

import phoneForm from '../../components/phone-form';
import phoneTable from '../../components/phone-table';

export default angular.module('app.pages.phone', [
    phoneForm.name,
    phoneTable.name
])
    .controller(PhoneController.name, PhoneController)
    .controller(PhoneAddController.name, PhoneAddController)
    .controller(PhoneDetailController.name, PhoneDetailController)
    .service('PhoneAPI', PhoneService);
```

可以看到，这个文件除了将phone模块自身的一些controller/service引入进来，还引入了它所依赖的component模块：phoneForm和phoneTable，所以只要require了它就可以保证phone的所有依赖都引进来了。最后，你肯定奇怪第5行的写法了，这里为什么需要`$ocLazyLoad`呢？[ocLazyLoad](https://github.com/ocombe/ocLazyLoad)是一个适用于Angular的Lazyload的库，其实我们只用它就可以实现Angular的LazyLoad，但这里我们用它的作用并不是动态加载文件，而是**使webpack动态加载进来的文件中的Angular的module名`app.pages.phone`生效**。因为Angular是不允许动态去声明一个module的，我们需要使用这种方法来使动态加载进来的文件中包含的Angular的module生效。

讲完了HTML和JS，那CSS怎么办呢？其实有了css-loader我们完全可以把它放在JS中或HTML中通过`require(xxx.css)`加载进来，因为通常CSS是与某个页面或某个模块绑定的（公用的样式除外），所以放在某个HTML中或某个JS模块中来require它也是合情合理的。

这样一来，phoen模块依赖的所有HTML、JS、CSS都动态加载进来了，在真正build的时候，会生成一个名为`phone.js`的文件（可以使用webpack的output.chunkFilename来配置生成的chunk文件的名称规则）来包含这些东西，而这个文件只有你第一次访问`/phone`的时候才会动态加载进来，这一点可以访问http://pinkyjie.com/angular1-webpack-starter/#/ 并打开console来验证。

#### style-loader的坑

这样设置了以后，所有动态加载的HTML、CSS和JS都会放在最终生成的`phone.js`中，也就是说，webpack目前是不支持为chunk生成单独
的CSS文件的，我提了个issue：[webpack/webpack #1758](https://github.com/webpack/webpack/issues/1758)，但目前还没回应。这会产生什么问题呢？如果你要动态加载的CSS中含有图片路径，比如：

``` stylus
background url("../../components/_layout/logo.png") top center no-repeat
```

那么最终得到的CSS中路径虽然是正确的：`url(/assets/images/logo.46e065707257b2930a699c7cdacd7e86.png) top center no-repeat`（具体路径与你的file-loader配置有关），但Chrome会将这个路径解析为：`chrome-devtools://devtools/bundled/assets/images/logo.46e065707257b2930a699c7cdacd7e86.png`，因为CSS文件是以动态bundle的形式加载进来的，所以图片资源的相对路径是以bundle为准的，这将导致图片无法正常的显示。已经有人在style-loader的项目里提了bug：[webpack/style-loader #93](https://github.com/webpack/style-loader/issues/93)，但目前style-loader还没有修复。好消息是Vue的作者fork了一份style-loader并[修复了这个问题](https://github.com/vuejs/vue-style-loader/commit/70b9f6ac2a2c1f2525c7e8945173d34957510ef6)，所以暂时大家可以使用[vue-style-loader](https://github.com/vuejs/vue-style-loader)代替style-loader来解决这个问题。

最后，完整的实现可以看我Github上的[angular1-webpack-starter](https://github.com/PinkyJie/angular1-webpack-starter)项目。