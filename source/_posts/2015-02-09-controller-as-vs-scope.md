title: 用$scope还是用controller as
date: 2015-02-09 20:12:36
categories:
- 前端开发
tags:
- AngularJS
- scope
- controller
---

AngularJS中在处理controller时提供了两种语法。
* 第一种是，在DOM中使用`ng-controller="TestController"`，这样在定义controller时需要将model绑定到$scope上。
* 另一种是，在DOM中使用`ng-controller="TestController as test"`，这样其实是将model直接绑定到controller的实例上。

在AngularJS的官方Get Started以及各种文档中，多推荐第一种方式，导致很多人可能都不知道原来还有第二种方式，我也是最近看一篇文章时才注意到这个。那么这两种方式各有什么优劣势呢？在现实的开发中到底更推荐哪种方式呢？今天就来探究一下！

<!--more-->

### controller as方式

$scope方式就不详细说了，大家应该最常用这种吧，看下面这段简单的代码。

<script async src="//assets.codepen.io/assets/embed/ei.js"></script>
<p data-height="268" data-theme-id="12085" data-slug-hash="QwOYVe" data-default-tab="html" data-user="pinkyjie" class='codepen'>See the Pen <a href='http://codepen.io/pinkyjie/pen/QwOYVe/'>QwOYVe</a> by Pinky Jie (<a href='http://codepen.io/pinkyjie'>@pinkyjie</a>) on <a href='http://codepen.io'>CodePen</a>.</p>

对应版本的controller as方式如下：

<p data-height="268" data-theme-id="12085" data-slug-hash="YPYWGO" data-default-tab="html" data-user="pinkyjie" class='codepen'>See the Pen <a href='http://codepen.io/pinkyjie/pen/YPYWGO/'>YPYWGO</a> by Pinky Jie (<a href='http://codepen.io/pinkyjie'>@pinkyjie</a>) on <a href='http://codepen.io'>CodePen</a>.</p>

在controller as方式中，可以给controller起别名，上面的例子中别名是`ctrl`。对比这两个例子，可以明显的看到controller as有两个不同的地方：
* 在HTML中，所有的绑定都需要写别名，即需要使用点运算符`ctrl.`
* 在JS中，controller的定义可以抛开`$scope`了，也就是说controller可以不依赖`$scope`了。

下面就从这两个区别出发去谈谈controller as的好处。

### 所有model都需要绑定在`ctrl`上

首先有必要澄清下，这个别名是怎么实现的呢？使用AngularJS在Chrome上的调试插件[AngularJS Batarang](https://chrome.google.com/webstore/detail/angularjs-batarang/ighdmehidhipcmcojjgiloacoafjmpfk)可以很清楚的看出来。安装好插件后打开[上面的例子](http://s.codepen.io/pinkyjie/debug/YPYWGO)，右击页面“审查元素”打开Chrome的DevTools，在Elements标签里选中`<div ng-controller="scopeController as ctrl" class="ng-scope">`这一行，然后点击右边的$scope标签（就是和Styles，Computed在一行的，看不到的话点击右边的小箭头），结果就是这个DOM元素所对应的`$scope`，如下图：

![图1](/assets/images/controller-as-vs-scope-1.png)

原来别名`ctrl`就是定义在`$scope`上的一个对象，这就是controller的一个实例，所有在JS中定义controller时绑定到`this`上的model其实都是绑定到`$scope.ctrl`上的，看到这里你想到了什么？是不是和上篇文章[AngularJS中scope基于原型链的继承](2015/02/07/prototypal-inheritance-of-scope-in-angularjs/)里的`$scope.data`有异曲同工之妙。所以，使用controller as的一大好处就是原型链继承给scope带来的问题都不复存在了，即有效避免了在嵌套scope的情况下子scope的属性隐藏掉父scope属性的情况。
> 可以发现，无论定义controller时有没有直接依赖`$scope`，DOM中的scope是始终存在的。即使使用controller as，双向绑定还是通过`$scope`的watch以及digest来实现的。

另外，使用别名还有一个显而易见的好处：指代清晰。在嵌套scope时，子scope如果想使用父scope的属性，只需简单的使用父scope的别名引用父scope即可。比如下面这个例子，我们将[上篇文章](2015/02/07/prototypal-inheritance-of-scope-in-angularjs/)的例子用controller as重写。

<p data-height="268" data-theme-id="12085" data-slug-hash="OPzRVN" data-default-tab="result" data-user="pinkyjie" class='codepen'>See the Pen <a href='http://codepen.io/pinkyjie/pen/OPzRVN/'>OPzRVN</a> by Pinky Jie (<a href='http://codepen.io/pinkyjie'>@pinkyjie</a>) on <a href='http://codepen.io'>CodePen</a>.</p>

这里我想让子scope里直接指向父scope的属性，只需在DOM绑定model时写上`parent.myName`即可，简单明了，看代码的一下就懂了，也不用费劲去推到底这里指向的是哪个属性了。如果你的嵌套多达四五层，那这种写法的优势就一下子体现出来了。

### controller的定义不依赖`$scope`

定义controller时不用显式的依赖`$scope`，这有什么好处呢？仔细看定义，这不就是一个普通的函数定义嘛，对！这就是好处！例子中的`ScopeController`就是所谓的POJO（Plain Old Javascript Object，Java里偷来的概念），这样的Object与框架无关，里面只有逻辑。所以即便有一天你的项目不再使用AngularJS了，依然可以很方便的重用和移植这些逻辑。另外，从测试的角度看，这样的Object也是单元测试友好的。单元测试强调的就是孤立其他依赖元素，而POJO恰恰满足这个条件，可以单纯的去测试这个函数的输入输出，而不用费劲的去模拟一个假的`$scope`。

另外，还有一个比较牵强的好处：防止滥用`$scope`的`$watch`，`$on`，`$broadcast`方法。可能刚刚就有人想问了，不依赖`$scope`我怎么watch一个model，怎样广播和响应事件。答案是没法弄，这些事还真是只有`$scope`能干。但很多时候在controller里watch一个model是很多余的，这样做会明显的降低性能。所以，当你本来就依赖`$scope`的时候，你会习惯性的调用这些方法来实现自己的逻辑。但当使用controller as的时候，由于没有直接依赖`$scope`，使用watch前你会稍加斟酌，没准就思考到了别的实现方式了呢。

### 定义route时也能用controller as

除了在DOM中显式的指明`ng-controller`，还有一种情况是controller的绑定是route里定义好的，那这时能使用controller as吗？答案是肯定的，route提供了一个`controllerAs`参数：

``` javascript
$routeProvider
  .when('/', {
    templateUrl: 'partial/home.html',
    controller: 'HomeCtrl',
    controllerAs: 'home'
  })
```
这样在模板里就可以直接使用别名`home`啦。

### 结论

总结下来，个人觉得还是偏向于使用controller as的，当然有一点要澄清，使用contoller as并没有什么性能上的提升，仅仅是一种好的习惯罢了。


