title: AngularJS中controller和service的继承与扩展
date: 2015-02-11 19:47:00
categories:
  - 前端开发
tags:
  - AngularJS
  - controller
  - service
  - 继承
  - 扩展
  - decorator
  - $provide
  - $delegate
---

最近公司要做的项目中牵扯到很多场景需要对controller和service进行继承和扩展的，总结一下心得体会。开讲之前，先明确一下这里的所谓的“继承”与“扩展”。
* 所谓继承，比较熟悉，这里就是指定义一个**新的**controller/service（不同名），继承原来的controller/service，然后在其基础上重写一些功能。
* 所谓扩展，这里说的是在**不产生新的**controller/service的情况下，添加或修改原controller/service的功能。

目前研究的结果就是service可以轻松的实现继承和扩展，而controller貌似只能继承。

<!--more-->

### controller的继承

说到controller，我们在[前面的文章](/2015/02/09/controller-as-vs-scope/)中介绍过有两种写法：使用`$scope`或使用`controller as`。针对这两种方式的区别，我们也可以使用两种不同的继承方式：
* 使用`controller as`的情况下，特点是controller不再依赖`$scope`，就跟普通的函数差不多，这个时候可以使用Javascript原生的继承方式。
* 使用`$scope`时，可以使用AngularJS内置的`$controller`service，通过依赖注入的方式实现继承。

还是直接上例子吧。

__*使用原生的继承*__

<script async src="//assets.codepen.io/assets/embed/ei.js"></script>
<p data-height="268" data-theme-id="12085" data-slug-hash="zxpRbK" data-default-tab="result" data-user="pinkyjie" class='codepen'>See the Pen <a href='http://codepen.io/pinkyjie/pen/zxpRbK/'>zxpRbK</a> by Pinky Jie (<a href='http://codepen.io/pinkyjie'>@pinkyjie</a>) on <a href='http://codepen.io'>CodePen</a>.</p>

这个例子有左右两块区域，左边的是parent，右边的是child，它们分别有5行数据需要显示：
* 第1行性别，对应属性`sex`。用于展示继承时不更改的属性。
* 第2行角色，对应属性`name`。用于展示继承后覆盖的属性。
* 第3行个数，对应属性`num`，左边是孩子的个数，右边是兄弟的个数，显然右边比左边少1。用于显示后面`add()`方法的结果。
* 第4行是个按钮，对应方法`add()`，左边是parent想多要一个孩子，右边是想child想多要一个兄弟，结果是一样的。用于展示继承时不改变的方法。
* 第5行也是个按钮，对应方法`test()`，用于展示继承时覆盖的方法。（不要在意这个随意的名字，因为我已经场景匮乏了。。。）

通过这5行，这个例子就基本涵盖了继承时发生的大部分情况。那么切换到JS里看看实现吧。JS里的代码结构大致分为5部分，用注释`/* Section 1 */`来区分（由于CodePen这工具不支持显示行号，所以只能用代码里的注释来分块讲解了）。
* 第1部分是一个典型的Javascript实现的继承函数`extend`，里面的原理不再详述，有兴趣可以看以前写的[《理解Backbone中extend的实现》](/2013/11/30/understand-backbone-extend/)。
> 这段代码怎么生成？访问[CoffeeScript官网](http://coffeescript.org/)，点击“TRY COFFEESCRIPT”，会打开一个编辑器窗口，左边写CoffeeScript，右边就会生成相应的Javascript。在左边键入`class A extends B`，右面就会有extend函数啦。机！智！
* 第2部分是一个名叫`FamilyService`的AngularJS的service。两个controller都需要依赖它。它的功能很简单，parent和child需要的孩子数量和兄弟数量都由这个service来提供，分别是方法`getChildrenCount()`和`getSiblingCount()`。除此之外，每次生孩子的时候它会trigger一个`new-child`事件，并将孩子数量通过参数传播出去。
* 第3部分就是定义`ParentCtrl`这个父controller了，由于采用`controller as`写法，这里的定义跟普通的对象没区别，在构造函数里定义属性，方法则定义在`prototype`上。这里值得注意的有两点：
  * `add()`函数其实仅仅是调用`FamilyService`的`newChild()`方法。
  * `num`属性的改变是通过响应`new-child`事件来实现的。
* 第4部分就是关键的`ChildCtrl`定义了，同样，属性的定义在构造函数中，方法定义在`prototype`上。可以发现，发生继承需要以下几步：
  1. 构造函数中先调用父controller的构造函数。这里是通过`__super__`来实现对父controller的引用的，因为在`extend`函数中我们已将讲`ParentCtrl`的prototype赋值给`__super__`变量了。这步可以保证把`ParentCtrl`里定义的属性以及事件响应继承过来。
  2. 覆盖属性和事件响应。 这里覆盖了`name`和`num`属性，并且更改了事件响应函数的内容，因为`new-child`事件返回的参数是孩子的总数，这里要减去自己才能得到兄弟的个数。
  3. 调用`extend`函数。
  4. 覆盖方法。这一步必须放在最后，如果放在`extend`函数的前面，则`extend`函数会将`ChildCtrl`重新定义的`add`方法用`ParentCtrl`的覆盖。
* 第5部分就是各种模块、controller、service的定义了，没啥多说的。

__*使用`$controller`service*__

<p data-height="268" data-theme-id="12085" data-slug-hash="zxaZPw" data-default-tab="result" data-user="pinkyjie" class='codepen'>See the Pen <a href='http://codepen.io/pinkyjie/pen/zxaZPw/'>zxaZPw</a> by Pinky Jie (<a href='http://codepen.io/pinkyjie'>@pinkyjie</a>) on <a href='http://codepen.io'>CodePen</a>.</p>

例子的实际效果与上面的一致，我们直接看JS部分的实现吧。这个代码分为4个部分：
* 第1部分与前面service的定义相同。
* 第2部分是`ParentCtrl`的定义，这里与前面的不同一个是全部使用`$scope`，另一个比较特殊的是`add`方法，它直接调用了一个绑定在`this`上面的`add`方法，这样做没什么特殊含义，仅为了演示后面的继承。
* 第3部分是`ChildCtrl`的定义，这里的继承是通过`var parentCtrl = $controller('ParentCtrl', {$scope: $scope});`来实现的，通过依赖注入得到父controller的实例，并将自己的`$scope`传入，这样，父controller绑定在`$scope`上面的东西就全部继承到子controller上面了。除此之外，还可以使用变量`parentCtrl`来引用父controller，跟前面的`__super__`一样。`add`方法的覆盖给出了使用`parentCtrl`的例子。
* 第4部分与前面相同，区别就是`ChildCtrl`依赖了`$controller`service。

__*比较*__

可以发现，其实这两种方式最大的区别就是，原生继承中需要调用`extend`函数来继承，并且子controller里需要显式调用父controller的构造函数来是实现属性的继承。而使用`$controller`service则只需要依赖注入和传入`$scope`即可。

### service的继承

service的继承就比较简单了，AngularJS中的service可以认为是new了service构造函数的实例。看下面这个例子，与上面的例子类似，同样是展示了继承过程中不改变或覆盖父service的属性和方法。

<p data-height="268" data-theme-id="12085" data-slug-hash="WbyYdK" data-default-tab="result" data-user="pinkyjie" class='codepen'>See the Pen <a href='http://codepen.io/pinkyjie/pen/WbyYdK/'>WbyYdK</a> by Pinky Jie (<a href='http://codepen.io/pinkyjie'>@pinkyjie</a>) on <a href='http://codepen.io'>CodePen</a>.</p>

熟悉的左右布局，三个属性两个方法。直接看JS中的实现，4个部分：
* 第1部分定义了一个controller，这个controller的所有功能基本都来自于service。页面中左右两个部分用的是同样的controller，只是名字不同，而且依赖的service不同。这一点从第4部分可以直观的看出来。
* 第2部分定义父service，没什么特别的。
* 第3部分定义子servcie，继承就发生在这里。首先，子service需要依赖父service，然后直接使用AngularJS内置的extend函数来实现继承，将父service实例上的属性方法都拷贝到this上，即完成了继承，后面就是一些属性和方法的覆盖了。
> 这里的extend和上面自己写的extend有什么区别呢？其实`angular.extend`的功能是一个Shallow Copy，类似上面我们自己的extend中的这段
``` javascript
for (var key in parent) {
  if (hasProp.call(parent, key))
    child[key] = parent[key];
}
```
* 第4部分各种定义，可以看到`TestCtrl1`和`TestCtrl2`的定义都是`TestCtrl`，只是一个依赖`ParentService`，另一个依赖`ChildService`罢了。

### service的扩展

其实扩展说白了，就是可以把一个已经定义好的service进行修改，为了保证这个修改的优先级，可以在module的config阶段来实现。废话不说，直接上例子吧。

<p data-height="268" data-theme-id="12085" data-slug-hash="LErmxz" data-default-tab="result" data-user="pinkyjie" class='codepen'>See the Pen <a href='http://codepen.io/pinkyjie/pen/LErmxz/'>LErmxz</a> by Pinky Jie (<a href='http://codepen.io/pinkyjie'>@pinkyjie</a>) on <a href='http://codepen.io'>CodePen</a>.</p>

这个例子与前面的类似，只是页面只有一个部分，实现的结果跟上面的右半部分一致。直接看JS中的实现，除了第3部分其他都是一样的。第3部分中定义了一个`extendServcie`函数，这个函数是在app的config阶段调用的（见第4部分）。这个函数中依赖了一个特殊的service`$provide`，扩展功能就由它的`decorator`方法来实现，方法的第一个参数就是要扩展的service的名称，第二个参数就是实际的扩展。在第二个参数中依赖一个`$delegate`service，这个service代表的其实就是`TestService`本身，可以看到在函数中我们直接使用`$delegate`去引用原有servcie，并进行随意更改，最终将这个`$delegate`返回即可。

也许很多人会觉得这种场景很诡异，平常根本不可能用得到。但是我在项目中就曾经遇到一个例子：在单页面应用中我们并没有把所有JS文件压缩在一起，而是根据不同的页面去lazy load不同的文件。具体去load哪些文件定义在一个servcie中，但这些文件名在development和production阶段是不一样的，这样就需要两套不同的文件名配置。利用servcie的扩展，可以在production环境时多include一个JS文件，在这个文件中对servcie进行扩展，更新那些相应的文件名。
