date: 2014-01-12 21:08
title: 从做简历中学到的知识
categories:
- 前端开发
tags:
- Bootstrap3
- FontAwesome4
- Yeoman
- AutoPrefixer
- Grunt
- grunt-usemin
- grunt-rev
- 响应式布局
- weinre
- 移动平台调试
---

前段时间偶然看到一个[狂拽酷炫吊炸天的在线交互式简历](http://www.rleonardi.com/interactive-resume/)，突然觉得咱也应该做个高大上的简历，这样以后想找工作的时候也方便，在线简历既方便别人又显档次，实乃装逼利器，哈哈！不过人家的这个技艺过于高超，不太适合我。。。最终还是决定网上找找顺眼的模板，参考下人家的主题瞎改改。最终的效果可以看[这里](http://pinkyjie.com/resume)，手机端也支持的哟！虽然是参考人家的主题，但我可没有直接复制粘贴人家的代码，而是带着一颗学习的心情去抄袭的（这一刻腾讯灵魂附体。。。），这也算是我第一次用Yeoman的一个小实践。下面就简单谈谈抄简历，哦不，做简历的过程中遇到的一些问题吧。

<!--more-->

### Bootstrap3和FontAwesome4：你若更新，我就跟随

用Yeoman的`yo webapp`生成项目脚手架的时候，它默认搭配了Bootstrap的v3版本，我就想着那就玩玩新版本看看呗，还没试过呢，后来用`bower install`装FontAwesome的时候，默认装的也是最新的v4版本，我想着，那也玩玩呗。这一玩不要紧，玩出很多麻烦事，这俩好基友的升级都带来了很多的变动。说说最明显的吧，最不习惯的栅格布局的样式`span*`更名为`col-md-*`了，字母都要多打几个。这里的col自然就是`column`，md代表`middle devices`即中等屏幕的设备（width&gt;=992px），这样的命名显然更有意义。有中等设备当然就有其他的，除了`col-md-*`，还有：

* `col-xs-*`：xs代表`extra small`即超小屏幕的设备（width&lt;768px）
* `col-sm-*`：sm代表`small devices`即小屏幕的设备（width&gt;=768px）
* `col-lg-*`：lg代表`large devices`即大屏幕的设备（width&gt;=1200px）

分这么清显然是为了适应不同的设备，虽然我也没搞清楚具体怎么用他们（只会用`col-md-*`的路过。。。），但不明觉厉啊！详细情况可以看[文档](http://v3.bootcss.com/css/#grid-options)。这大概是改动最大的地方了，因为最常用的`span*`已经一去不复返了。另外，`container-fluid`和`row-fluid`也没了，直接简化成`container`和`row`，也就是说默认都是流式布局，按百分比分配宽度了。从这些改变可以看出，移动端的体验优化是越来越重要了，一切为了响应式啊！所以用v3版本以后，很多地方还是先查文档保险点，官方也给了一个[v2到v3的升级指南](http://v3.bootcss.com/getting-started/#migration)。说了半天Bootstrap，下面说说它的好基友FontAwesome，这玩意的最新版v4变动也超级大。以前都是简单的`icon-*`搞定，现在变成了`fa fa-*`，又要多打字母你这不是坑爹吗！这么命名的原因官方解释说是为了让命名更`consistency and predictability`，然后更是给出了`fa-[name]-[shape]-[o]-[direction]`这个统一的规则。详细可以看看[官方的简单例子](http://fontawesome.io/whats-new/#new-naming)，比如说`fa-bookmark`是实心的图标，加个-o变成`fa-bookmark-o`图标就成空心的了。我本来的理解的是，所有图标都可以按这个规则变化，事实往往都是出乎意料的，官方只是提供了这么一种规则让你一看名字就能猜到意思，图标还是那么多，不是每个图标都有各种变种。。。改个命名整出这么多名堂，后来我在用Bootstrap的轮播时才发现真相：FontAwesome为啥改名？因为——好基友Bootstrap的图标改名啦！！！Bootstrap原先的图标也是`icon-*`，现在变成了`glyphicon glyphicon-*`。嘿嘿，懂了没！

另外插一句，如何在sass-bootstrap里使用FontAwesome呢？这里我也折腾了半天，搜索后有很多方案，但有些方案经过Grunt一build还是会出问题，最后参考Bootstrap解决。在Yeoman默认生成的scss文件里头部有这么俩句：

``` scss
$icon-font-path: "../bower_components/sass-bootstrap/fonts/";
@import 'sass-bootstrap/lib/bootstrap';
```

那么模仿它，我们加上：

``` scss
$fa-font-path: '../bower_components/font-awesome/fonts/';
@import 'font-awesome/scss/font-awesome';
```

另外，修改`Gruntfile.js`里配置copy任务里的src，在`'bower_components/sass-bootstrap/fonts/*.*',`后面加上`'bower_components/font-awesome/fonts/*.*'`即可，这样livereload和build后都可以正常使用了。

### border：会画三角形这种事情我会随便说吗

经常看到一些div的边框旁边带一个小三角形，如下图黄色高亮区域：

![](/assets/images/what-i-learn-from-making-resume-1.PNG)

好久好久以前我曾以为是图片做的，用了Bootstrap以后我知道CSS可以做这个效果，但一直没深究，这次才发现原来border配合:before就可以做哇！长姿势啦！至于如何更好的理解这个写法，请参考[这篇文章](http://www.css88.com/archives/1875)，讲的非常好，看完以后脑子里就会留下那个四种颜色组成的正方形图案，You got it！简单来说，将一个div的宽高都设置为0，然后border的宽度就是要三角形的“垂线”的长度，要实现三角形效果，两步就行：将不需要的边的border宽度设为0，其他不显示的边颜色设为透明。妈妈再也不用担心我画三角形了！（最近好像改成爸爸再也不用担心了。。。）

### Usemin+rev：不是只有Rails才有Asset Pipeline哦

一看到min，过去我只知道CSS和JS可以min，就是将CSS的空格乱七八糟的去掉然后压缩成一行，JS进行变量替换去空格一类的也压成一行，就是所谓的`uglify/concat`这些库做的事，但观察了下这次Yeoman生成的这个Gruntfile.js内容明显更丰富。先来说说带min的就有好多grunt任务：`cssmin/imagemin/svgmin/htmlmin`，第一个就不多说了，第二个就是压缩你的图片，个人经验一般能减少个几KB，第三个压缩svg格式的文件，第四个是我最不能接受的，尼玛连HTML文件也能压缩。看了下这个的配置参数，很变态的说，可以去空格这就不说了，居然可以去掉HTML标签属性的引号，这居然不违反语法。。。反正被这玩意一压缩，你的HTML文件大概只剩下几行了。

Usemin这玩意也很牛逼，看看Yeoman生成的HTML文件里，有类似：

``` html
<!-- build:js scripts/plugins.js -->
<script src="bower_components/sass-bootstrap/js/affix.js"></script>
<script src="bower_components/sass-bootstrap/js/alert.js"></script>
<script src="bower_components/sass-bootstrap/js/dropdown.js"></script>
<script src="bower_components/sass-bootstrap/js/tooltip.js"></script>
<!-- endbuild -->
```

没错，想必已经猜出作用了，这玩意可以build的时候把里面这些个文件压缩合并成一个文件，文件名就是上面指定的`scripts/plugins.js`，高大上啊，很像Rails里的功能有木有！除了JS，CSS也是支持的哦！另外，这玩意在配合grunt-rev插件，更牛了，举个例子，上面这个代码块最后build生成的可能是：`<script src="scripts/20bdd8fe.plugins.js"></script>`，看到前面的hash没，每次内容一变动就会自动hash，这玩意就是Rails啊，防止浏览器缓存的利器！不仅JS可以这样，CSS也有效，更变态的是图片字体文件神马的也可以这样，有没有很带感！

另外，有个地方值得一说，刚开始我把图片放在`app/images/`文件夹下，build以后很显然文件名也加了hash，但HTML里是直接引用文件名的，怎么办，直接报一堆404找不到图片哇！经过一番搜索发现，修改`Gruntfile.js`里usemin这一块，将options改为：

``` javascript
options: {
    assetsDirs: ['<%= yeoman.dist %>', '<%= yeoman.dist%>/images']
},
```

加上images文件夹即可，usemin会自动替换HTML代码里的img标签的src属性为hash过后的文件名！

### AutoPrefixer：让各种CSS3前缀都玩蛋去吧

前端的达淫们最闹心的就是浏览器不兼容吧，为了实现一个CSS3效果各种前缀都要记一遍，什么`-webkit-*`啊，`-moz-*`啊，`-ms-*`啊一类的，写好多，这就不是多打几个字母的问题了，而是多打几行字母的原则性问题！当然，为了解决这个问题，出了好多种方案。比如Compass提供很多mixin用来解决这个，比如Sublime Text有插件可以帮你自动输入这些等等。而AutoPrefixer比起这些现有的方案显得更直接更暴力：一不用你自己输，二不用编辑器帮你输，三你连看也看不见了。是的，有了它，以后你只用记住标准的CSS写法，不用管各种前缀了。配合grunt-autoprefixer插件，简单配置即可实现build的时候自动帮你生成闹心的前缀插入到CSS文件中去！比如可以这么配置：

``` javascript
options: {
    browsers: ['last 3 versions']
},
```

这样，主流的浏览器的最新三个版本都可以得到兼容，更复杂的配置选项可以参考[它的主页](https://github.com/ai/autoprefixer)，虽然很多我也没太搞清楚。有时你这么操作了然后build后你“审查元素”发现，哎呦没自动生成嘛！博主是坏人！不要惊慌，官方自称根据网站[Can I  Use](http://caniuse.com/)的数据自动判断浏览器版本需不需要前缀，如果没生成，那说明最新的三个版本的浏览器都已经告别前缀，支持标准的CSS写法啦！有人还要问，那去哪查CSS3标准写法呢？[MDN](https://developer.mozilla.org/zh-CN/)！

### 响应式布局：什么样的宽度就整什么样的样式

如此高大上的简历不支持手机浏览你好意思嘛！没错，必须得支持！那就得说说CSS的响应式布局了。以前虽然也知道这个概念，也一直知道Bootstrap就有很多样式是支持响应式的，但一直没有深入研究。其实响应式布局就像标题说的那样简单，就是定义什么样的宽度下套用什么样的样式。比如下图：

![](/assets/images/what-i-learn-from-making-resume-2.jpg)

上面当页面宽度比较大时显示长的板块导航，当页面宽度缩小到一定程度时，长条导航显然已经不太方便操作，这时自动变成下拉菜单导航框，用户体验一下子就来了有木有！那么CSS文件怎么写呢，其实很简单，如下：

``` scss
.nav.normal {
    @media all and (min-width: 992px) {
      display: block;
    }
    @media all and (max-width: 992px) {
      display: none;
    }
}
```

其中的`@media`代表[Media Queries](http://www.qianduan.net/media-type-and-media-query.html)，用以查询当前打开页面的是什么设备，什么宽度，后面的`all`就是所有设备，除此之外还支持很多Meida Type，可以看上面那个文档深入了解一下，不过我感觉好像用处不大的样子。接着后面的括号很像CSS的写法，`(min-width:992px)`即最小宽度是992，代表设备的宽度是大于992的，那下面的`(max-width:992px)`自然就是设备的宽度是小于992的。这段代码的意思很容易理解，`.nav.normal`在小于992时隐藏，大于992时显示。就是这个事嘛！对了，使用时记得在HTML文件里加入viewport的meta：`<meta name="viewport" content="width=device-width">`，这样才能支持哦！当然，除此之外，响应式还有很多别的措施，比如，提供不同大小的图片供不同尺寸的设备显示等等。总之可以理解为功能和交互的graceful degradation，在尺寸有限的情况下尽量保证功能并提供好的交互体验。大家可以用电脑和手机分别浏览下[我的简历](http://pinkyjie.com/resume)看看区别！

### weinre：我是远程的Webkit Dev Tools

既然要做响应式，少不了调试手机端的样式什么的，[weinre](http://people.apache.org/~pmuellr/weinre/docs/latest/)就是干这个事的，让你的移动端浏览器也拥有类似Chrome的Webkit Dev Tools。不过使用起来稍稍有那么一点点繁琐，首先使用`npm -g install weinre`来进行安装（ubuntu用户请自觉sudo），接着使用`weinre --boundHost 0.0.0.0`启动服务，用这个参数绑定所有IP地址，这样手机端才能访问，默认会在8080端口启动。当然还有很多别的参数，可以用`weinre -?`查看。打开以后，访问`http://192.168.3.102:8080/`（请自觉替换自己的IP地址），这个界面有简短的教程可以浏览。简单说，这玩意需要你的HTML里包含一个JS文件才行，比如`<script src="http://192.168.3.102:8080/target/target-script-min.js#anonymous"></script>`，那么将这一行加入你的HTML中，然后用手机端访问网页。这时，打开前面文档里显示的地址`http://192.168.3.102:8080/client/#anonymous`，就会看到下图：

![](/assets/images/what-i-learn-from-making-resume-3.PNG)

一旦包含那个JS的页面被访问，这里的“Targets”就不再是none，将会是访问页面的设备的IP地址。另外注意这个图的各个菜单，是不是很熟悉啊，没错，这和Chrome的Dev Tools工具是一模一样的啊！开始调试吧，不用将手机插到电脑上，只要在一个局域网内访问页面即可哦！不过要注意的是，这玩意功能没有Chrome的调试工具多，而且稍稍有点慢，不过调调CSS足够了。

另外，还有一个值得一说，Grunt是有live-reload功能的，也就是一旦修改源文件并保存，浏览器会自动刷新重新加载最新的结果。万万没想到，这个功能对手机端也有效，快试试吧。同时用电脑和手机打开页面，然后电脑修改源文件保存，享受电脑和手机端浏览器同时刷新的快感吧，这玩意是多设备调试的神器啊！对了，记得将`Gruntfile.js`里的绑定地址也改为`0.0.0.0`哦，否则除了本机别的设备是访问不到页面的哦。。。

### 总结

差不多就是这样了，简历的源码[在此](https://github.com/PinkyJie/resume)。为了让其使用Github Pages显示，我建立了`gh-pages`分支，每次在master分支开发完后，运行`grunt build`生成production，这时网站会build好放在dist目录下，然后切换到gh-pages分支，运行`build.bat`（我就用Windows开发，你打我啊！）将会把没用的文件删除，将dist目录的东西放在根目录，这样Github Pages就可以正常显示了。还有个发现，一旦Github上的个人网站绑了自己的域名，别的项目的页面也都变成自己域名底下了，比如这个简历，好爽啊~
