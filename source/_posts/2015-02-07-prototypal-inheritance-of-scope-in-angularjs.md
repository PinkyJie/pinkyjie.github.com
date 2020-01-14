title: AngularJS中scope基于原型链的继承
date: 2015-02-07 15:17:22
categories:
- 前端开发
tags:
- AngularJS
- scope
- dirctive
- ng-repeat
- Javascript继承
---

相信大家写过AngularJS的都会发现，很多人在处理表单的数据绑定时，都习惯性的把ng-model绑定在$scope的一个对象属性上，而不是直接绑定在scope上。比如说使用`<input name="name" ng-model="data.name" />`而不是`<input name="name" ng-model="name" />`。这是为什么呢？这样在controller里面岂不是写起来更复杂吗？每次访问的时候都要多“点”一下，为什么不直接绑在$scope上呢？其实这样写自然是有它的好处的，而且这种写法也是推荐的最佳实践，尤其是在处理嵌套scope的情形下，这样写是很有必要的。为了弄清楚这么写的原因，我们需要深入的研究一下AngularJS里scope的继承。

<!--more-->

### 基于原型链的继承

AngularJS的官方文档里有这么一句话来描述scope：`A "child scope" (prototypically) inherits properties from its parent scope.` 子scope从其父scope那里继承属性，而括号里的词是重点，这种继承是基于原型链的。直接来看一个最简单的例子。

<script async src="//assets.codepen.io/assets/embed/ei.js"></script>
<p data-height="268" data-theme-id="12085" data-slug-hash="xbPPPJ" data-default-tab="result" data-user="pinkyjie" class='codepen'>See the Pen <a href='http://codepen.io/pinkyjie/pen/xbPPPJ/'>xbPPPJ</a> by Pinky Jie (<a href='http://codepen.io/pinkyjie'>@pinkyjie</a>) on <a href='http://codepen.io'>CodePen</a>.</p>

这段简单的代码里，`ChildCtrl`并没有定义`MyName`，但由于在DOM结构上它和`ParentCtrl`是父子关系，所以它继承了其父scope上定义的属性。这个其实不难理解，我们知道，在Javascript的世界里，继承都是通过原型链来实现的。最常见的例子就是`toString()`方法，试试运行`var a = {"test1": 1, "test2": 2}; a.toString()`。你自定义一个对象时，都可以调用到这个方法，因为这个方法是定义在object对象上的，自定义的对象都继承它。

在这个例子里，我们是直接将model绑定在scope上的，似乎目前为止也没什么问题啊。下面让我们做一些小实验：
* 尝试在Parent Name的input框中修改，发现Child Name的input框跟着同步改变，说明父scope中model的变化同步到了子scope中，没有问题。
* 尝试在Child Name的input框中修改，发现Parent Name对应的input框并没有随着发生改变，说明子scope中model的变化并没有同步回父scope中。
* 再次尝试在Parent Name的input框中修改，发现此时Child Name的input框也不更新了，说明这个时候，父scope中的model变化已经没法同步到子scope中去了。

看起来父scope和子scope之间的联系已经断了，此时两个input框的值已经无法再同步了。这个例子是现实情况中非常常见的场景，model定义在父scope上，而表单的DOM区域有自己的controller，这就势必会产生一个新的子scope。按照我们的实验，一旦用户修改了表单数据，父scope拿到的model已经不正确了，并且此时如果父scope的controller更新model，表单里的model也不对了，这显然与我们的设计初衷是相违背的。

我们可以从分析实验2入手，因为实验2之前父子scope是可以同步，实验2之后父子scope已经完全独立，就像是父scope和子scope操作的是不同的model一样。那究竟是不是这么回事呢？我们尝试修改代码，在`ChildCtrl`的函数中添加一行`$scope.myName = 'Child Name';`。这时，我们修改两个input框，发现无法同步。这个也很好理解，还拿`toString()`来举例，如果自定义的对象里重写了`toString()`方法，那么这个子对象上的方法就覆盖了继承过来的方法。同样的，这里子scope的`myName`属性覆盖了从父scope继承过来的`myName`属性。
> 在父scope属性被隐藏的情况下如果要访问其属性，可以使用子scope上的`$parent`属性来显式的访问。

### 关于继承属性的读和写

实验2造成的结果其实就是在子scope上重新定义了`myName`属性。是什么触发了这个操作呢？我们修改子Child Name的input框值，根据AngularJS的双向绑定，触发了子scope上`myName`属性的写操作，写操作发现子scope上没有自己定义这个属性（可以通过`hasOwnProperty()`函数来确定）时，触发子scope去定义一个`myName`属性。正是由于子scope现在有了自己的`myName`属性，父scope继承过来的`myName`被隐藏(shadow)，导致了两者的更改互不影响。所以可以总结在基于原型链的继承中，子类属性的读和写有这么几个特点：
* 读子类的属性时，子类有这个属性（`hasOwnProperty`）的时候则读子类自己的，子类没有的时候读父类的，不管子类有没有这个属性，在子类上都不会有新属性被创建。
* 写子类的属性时，如果子类有这个属性（`hasOwnProperty`）则写子类的，子类没有的话就会在子类上新建一个同名的新属性，而父类继承过来的属性被隐藏。

那怎么解决这个问题呢？我们按照最佳实践，将model绑定在scope的data属性上试试。看下面这个例子。
<p data-height="268" data-theme-id="12085" data-slug-hash="WbXXyo" data-default-tab="result" data-user="pinkyjie" class='codepen'>See the Pen <a href='http://codepen.io/pinkyjie/pen/WbXXyo/'>WbXXyo</a> by Pinky Jie (<a href='http://codepen.io/pinkyjie'>@pinkyjie</a>) on <a href='http://codepen.io'>CodePen</a>.</p>

我们同样做上面3个实验，发现这次不管怎么修改，父scope和子scope上的model都是可以同步的。这又是为什么呢？难道写属性的时候没有在子scope上创建新属性吗？确实如此！可以发现，这里继承的属性是`data`，它是一个对象，而前面的例子中继承的是`myName`，是一个字符串。在这两种情况下，当我们尝试修改Child Namd的input为“abc”时，让我们看看分别有什么不同：
* 例子1中，由于双向绑定，其实是执行`子scope.myName = "abc"`，先检查子scope上有没有`myName`属性**可供写**，发现没有，则新建属性，导致父scope的属性被隐藏。
* 例子2中，执行`子scope.data.myName = "abc"`，这个时候，先检查子scope上有没有`data`属性**可供读**，发现没有，则读父scope上的，读到以后，然后修改其上面的`myName`属性。

区别就在于写`data.myName`的时候会尝试先去读`data`属性，正是由于这个特性，所以在处理表单的数据绑定时才推荐使用点运算符，即把model绑定在scope的某个对象属性上。
> 可能有人会对这个有异议，考虑这个例子：`var a = {}; a.data.myName = "abc";`，显然执行这句代码会报错，错误是`TypeError: Cannot set property 'myName' of undefined`，说明解释器先尝试去读`a.data`，发现是undefined，然后再去写其`myName`属性，才报了这么一个错！

### 其他会产生子scope的标签

除了`ng-controller`会产生子scope外，AngularJS里的还有很多其他标签也同样会产生子scope：
* `ng-repeat`
* `ng-include`
* `ng-switch`

所以在这些场景下也需要考虑原型链继承存在的问题。这里值得一说的是`ng-repeat`标签，对于每一轮循环它都会产生一个**新的**子scope基于原型链继承父scope，而且在这个新的子scope里，会定义一个新属性，属性名为循环变量，值为此轮循环的值，类似下面的代码：
``` javascript
子scope = 父scope.$new();
子scope[循环变量] = 此轮循环的值;
```
这个特性在很多场景下很有用。比如说，同一个页面上有2个`ng-include`标签，使用同样的模板，模板里有一个变量`name`，针对两个不同的`ng-include`我想让`name`变量有不同的值，但是我又不想重新写两个controller，这个时候可以使用`ng-repeat`。
<p data-height="268" data-theme-id="12085" data-slug-hash="VYrxEK" data-default-tab="result" data-user="pinkyjie" class='codepen'>See the Pen <a href='http://codepen.io/pinkyjie/pen/VYrxEK/'>VYrxEK</a> by Pinky Jie (<a href='http://codepen.io/pinkyjie'>@pinkyjie</a>) on <a href='http://codepen.io'>CodePen</a>.</p>

这里使用`ng-repeat`可以轻松的将不同的变量传入同一个模板中。

### directive中的scope

directive比较复杂，所以单独拿出来研究，它有一个`scope`参数，根据参数的不同就有不同的行为。
* 默认情况下，即构造directive的时候不传scope参数，等同于传入`scope: false`，这种情况不会产生新的scope，也就不存在继承的问题，directive的scope和原来是同一个。
* 构造directive时传入`scope: true`，这种情况会产生新的子scope并继承父scope，情况类似于前面介绍的，所以这时就需要注意原型链继承带来的问题。
* 构造directive时传入`scope: {...}`，这种情况会产生新的scope，但这个scope是独立的scope，不继承于任何scope，也就不存在原型链继承的问题。这种情况通常用于你想构造一个通用的directive的，不与父scope产生任何联系。
> 注意尽管这种情况下scope是独立的，但它依然有`$parent`属性来指向其父scope。另外，倘若你想访问父scope的属性的话，可以使用`=`，`@`，`&`等符号，来指定是双向绑定或单项绑定还是基于表达式的绑定。
* 构造directive时传入`transclude: true`，这种情况下会产生子scope并继承父scope，它的独特之处时如果有独立的scope，则两者为兄弟，也就是说，两者的`$parent`是一样的，独立scope的`$$nextSibling`指向这个子scope。

> 参考：[What are the nuances of scope prototypal / prototypical inheritance in AngularJS?](http://stackoverflow.com/questions/14049480/what-are-the-nuances-of-scope-prototypal-prototypical-inheritance-in-angularjs)




