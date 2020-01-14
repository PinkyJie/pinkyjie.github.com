date: 2013-11-24 13:20
title: Yeoman初体验
categories:
- 前端开发
tags:
- Yeoman
- BackboneJS
- Bower
- Grunt
- AngularJS
- RequireJS
- Addy Osmani
- 前端自动化
- Javascript
---

第一次关注[Yeoman](http://yeoman.io/)这玩意大概是在去年七月份的时候吧，那个时候偶然发现了Google的[AngularJS](http://angularjs.org/)，然后顺便发现了Yeoman。看Logo第一感觉很像是Jenkins，一个仆人的样子，当时也就只当是一个快速生成网站原型的工具，没细细研究过。直到最近又在很多文章中看到它的身影，决定小试一把。

<!--more-->

### 安装

Yeoman的安装基于npm，所以需要先安装好Node.js，另外如果需要使用Sass来写CSS的话，需要安装Ruby和Compass工具，这个不细说了，很简单的。完事以后在命令行运行`npm install -g yo`即可，如果用Linux的话记得`sudo`啊。如果一切顺利，Yeoman的3个工具：`yo`，`bower`和`grunt`就安装好了。当然这步也可能会报错，比如我自己在windows上就遇到`Error: No compatible version found: yo`的报错。出现这个问题请检查你的Node.js的版本，如我的是`v0.6.15`，太老了，升级到最新版`v0.10.22`再重新执行安装命令即可。安装成功后我们先来看看这3个工具吧：

* yo：主要负责一些scaffolding的工作，配合各种框架的generator快速生成一个web app所需的各种基础文件。顺便吐槽一句，scaffolding这个词翻译成脚手架我实在不明所以。
* [bower](http://bower.io/)：Twitter出品，类似于npm，用于前端各种第三方库的包管理工具。
* [grunt](http://gruntjs.com/)：大名鼎鼎的前端自动化工具，可以把它理解成make命令，简单的配置即可自动帮你完成诸如连接js，编译coffeescript，编译scss，跑测试啊等繁琐的工具。

Yeoman就是利用这三个工具来为我们服务的。第一次在命令行运行`yo`命令会看到一张Yeoman的“吉祥物”形象，然后是：

``` bash
[?] What would you like to do? (Use arrow keys)
  --------
> Install a generator
  Find some help
  Get me out of here!
```

从这里我们就要开始适应Yeoman这种问答的服务方式，每次你运行一些命令，它就会给出一些选项让你选择。我们看到第一项就是安装generator。下面就以Backbone的项目为例来看看如何利用Yeoman来快速生成框架吧。

### Backbone的generator

以往我们要写开始一个Backbone的Web工程，免不了先下载jQuery，Backbone，Underscore，然后CSS上下载个Bootstrap一类的，如果要用模块化的写法的话还要下载RequireJS，然后写RequireJS的初始配置等等等等，非常繁琐。我想牛人应该都有自己的一套脚本去生成这些基本框架吧，作为非牛人的我们有了Yeoman，全交给它就行了。首先，安装Yeoman为Backbone适配的generator：

``` bash
npm install -g generator-backbone
```

然后新建一个项目文件夹，使用Yeoman初始化项目。

``` bash
mkdir yeoman-backbone-project
yo backbone
```

问答式服务又来了，Yeoman会询问你：是否包含Bootstrap for Sass和CoffeeScript，如下：

``` bash
     _-----_
    |       |
    |--(o)--|   .--------------------------.
   `---------´  |    Welcome to Yeoman,    |
    ( _´U`_ )   |   ladies and gentlemen!  |
    /___A___\   '__________________________'
     |  ~  |
   __'.___.'__
 ´   `  |° ´ Y `

Out of the box I include HTML5 Boilerplate, jQuery, Backbone.js and Modernizr.
[?] What more would you like?
>[X] Twitter Bootstrap for Sass
 [ ] Use CoffeeScript
```

非常可爱的Yeoman吉祥物，使用空格键来选定或取消选项前面的x，这里我选择使用Twitter Bootstrap for Sass，不使用CoffeeScript。选定后回车，然后问你是否包含RequireJS，我选择包含。这时Yeoman会自动帮你搭好工程的架构，包括文件夹结构和一些必须的文件，然后会自动执行`bower install && npm install`，前者安装各种前端依赖，如前面说的jQuery、Bootstrap、Backbone、Underscore、RequireJS等等，后者安装Grunt需要的一些依赖包，用于完成各种自动化任务。这里可能会比较慢，并且可能出现问题，如果报错，你可以直接手动执行`bower install`和`npm install`。其实`bower install`会查找当前目录下的`bower.json`文件，安装里面生命的各种依赖，而`npm install`会查找当前目录下的`package.json`安装里面的各种依赖，做过Node开发的应该不会陌生吧。`bower install`通常不会出什么问题，问题都出在`npm install`里面，如果有些依赖没安装成功，先删除工程目录下的的`node_modules`文件夹，然后重新执行`npm install`。比如我在Windows上碰到了如下问题：

``` bash
pre-build test failed, compiling from source...
building is not supported on win32
```

这是在安装`grunt-contrib-imagemin`这个依赖的时候报的错，经过一番搜索，是因为这个`imagemin`又依赖`jpegtran-bin`，这个项目的0.2.0以上的版本在Windows上的build有问题导致的，Github上面有很多讨论，见[1](https://github.com/yeoman/node-jpegtran-bin/issues/37)，[2](https://github.com/gruntjs/grunt-contrib-imagemin/issues/108)，[3](https://github.com/gruntjs/grunt-contrib-imagemin/issues/109)，网上有很多解决方案，比如进入工程目录下的`node_modules/grunt-contrib-imagemin/`，修改`package.json`将其对`jpegtran-bin`的依赖版本修改为`0.2.0`，但经过尝试，我发现最简单的方式就是直接修改工程目录下的`package.json`，将`grunt-contrib-iamgemin`的版本修改为`0.2.0`，即去掉这一行`"grunt-contrib-imagemin": "~0.2.0",`中的`~`号即可。修改好后，按前面你说的，删除`node_modules`文件夹，重新执行`npm install`这次应该成功了吧！目前好像官方已经修复了这个问题，如果一切正常就不用管了。

在开始Code之前，我们先来看看Yeoman帮我们干了些什么。除了刚才提到的用来管理前端依赖的`bower.json`，用来管理Grunt依赖的`package.json`，以及用来配置Grunt任务的`Gruntfile.js`，我们还发现主要的文件夹有`app`，`test`，`node_modules`，后两个就不用说了，一个存放测试的文件，一个用来存放Grunt的依赖。主要的项目文件都放在app目录下了，打开看看，熟悉的images、scripts、styles文件夹和idnex.html，这个不必多说，除此之外还有一个`bower_components`文件夹，这里面就是我们的前端依赖文件了，里面各种文件夹：jquery，backbone啊等等，都安装在这里了。由于我们主要使用Backbone+RequireJS，我们重点关注scripts文件夹。打开里面的`main.js`，可以发现里面已经为我们生成好了RequireJS的初始配置：

``` javascript
require.config({
    shim: {
        underscore: {
            exports: '_'
        },
        backbone: {
            deps: [
                'underscore',
                'jquery'
            ],
            exports: 'Backbone'
        },
        bootstrap: {
            deps: ['jquery'],
            exports: 'jquery'
        }
    },
    paths: {
        jquery: '../bower_components/jquery/jquery',
        backbone: '../bower_components/backbone/backbone',
        underscore: '../bower_components/underscore/underscore',
        bootstrap: 'vendor/bootstrap'
    }
});

require([
    'backbone'
], function (Backbone) {
    Backbone.history.start();
});
```

是不是很方便啊！！！现在我们的项目已经是可运行状态了！

### Grunt与LiveReload

让我们先来看看Yeoman为我们初始化的工程什么样子吧。运行`grunt server`命令将启动一个简单的server来跑我们的工程，可以看到，grunt执行了各种任务，而这些任务都是可以在`Gruntfile.js`配置的，可以看这一行`grunt.registerTask('server', function (target) {`，运行`grunt server`执行的操作都可以在这个函数里配置。如果一切正常，会调用Chrome浏览器打开你的项目，在开启server的同时，更令人激动的是还开启了一个LiveReload的服务器，只要你在编辑器里修改源文件并保存，Chrome会自动加载改变，这对于调试是非常有用的。如果你没有装Compass可能会报错，安装一下即可。

### 使用Yeoman生成Model/View/Collection

Backbone的generator还提供了很多有用的功能，可以帮你生成指定的Model/View/Collection，这个有点类似Ruby On Rails。以有名的[TodoMVC](http://todomvc.com/)项目为例，我们要生成一个Todo的Model，一个Todos的Collection，以及相应的View，可以使用如下命令：

``` bash
yo backbone:model Todo
yo backbone:colleciton Todos
yo backbone:view Todo
yo backbone:view Todos
```

Yeoman会在相应的model/collection/view文件夹下生成相应的js文件，并写好框架给你填空，如`app\scripts\models\Todo.js`：

``` javascript
define([
    'underscore',
    'backbone'
], function (_, Backbone) {
    'use strict';

    var TodoModel = Backbone.Model.extend({
        defaults: {
        }
    });

    return TodoModel;
});
```

另外，生成View还会帮你自动生成模板，如`app/scripts/templates/Todo.ejs`文件，并在相应的View文件Todo.js配置好：

``` javascript
var TodoView = Backbone.View.extend({
    template: JST['app/scripts/templates/Todo.ejs']
});
```

除此之外，我们知道TodoMVC里面是将LocalStorage作为存储的，那还需要安装Backbone的LocalStorage适配器，这不还是要自己下载相应的js吗？NO,NO,NO! 这次该Bower出场了。

### 利用Bower自动管理和安装依赖

Bower的使用和npm很像，利用`bower search backbone.localstorage`来搜索有没有这个库，答案是肯定的！然后运行

``` bash
bower install backbone.localstorage --save
```

则指定的库会自动下载到刚才的`bower_components`文件夹中，后面的参数`--save`则是将这个依赖自动添加到`bower.json`中，方便下次自动安装，因为`bower_components`文件夹一般是不在代码库中的。这里我认为Yeoman应该自动将这个新的依赖添加到`main.js`中的RequireJS配置中去，但是它没有这么做，还是需要手动来操作，这一点有点令我失望。找到`main.js`，在`shim`块中添加

``` bash
backboneLocalstorage: {
    deps: ['backbone'],
    exports: 'Store'
}
```

在`path`块中添加：

``` bash
backboneLocalstorage: '../bower_components/backbone.localStorage/backbone.localStorage',
```

大功告成，在需要LocalStorage的地方引用即可。

### 总结

Yeoman的初体验就到这里了，经过体验，基本上3个小工具的作用和基本功能都已经有所了解。如果你想继续Code一个完整的Todo应用，参考[TodoMVC的官方Github](https://github.com/tastejs/todomvc/tree/gh-pages/dependency-examples/backbone_require)。当然，除了Backbone，Yeoman还支持目前各种主流的框架，如Google自家的AngularJS，Ember等等，如果都没法满足需要，也可以参考[官方文档](http://yeoman.io/generators.html)自己编写generator。总之，Yeoman是一个不可多得的开发工具，可以极大的提高效率，大家不妨来试试吧！

P.S. 在接触Yeoman的过程中我发现了[大神Addy Osmani](https://github.com/addyosmani)，他不仅是Google Chrome的开发人员，而且参与多个开源项目，如Yeoman、TodoMVC等等，他还出了两本书，一本关于[Javascript设计模式](http://addyosmani.com/resources/essentialjsdesignpatterns/book/)，一本关于[Backbone](http://addyosmani.github.io/backbone-fundamentals/)，值得一读，前面那本在OSChina上有[中译本](http://www.oschina.net/translate/learning-javascript-design-patterns)，后面那本我刚买了，准备好好拜读。另外，我在Speckerdeck上看到了一份大神的[关于前端自动化的PPT](https://speakerdeck.com/addyosmani/automating-front-end-workflow)，其中介绍了包括Yeoman在内的各种工具以及很多非常实用的Sublime插件，相信看了以后会让你打开眼界，吐血推荐！
