date: 2013-11-30 14:38
title: 理解Backbone中extend的实现
categories:
- 前端开发
tags:
- BackboneJS
- UnderscoreJS
- Javascript
- extend
- Javascript继承
- 前端MVC
---

Backbone这库真心不错，虽然后来我也看过AngularJS、EmberJS这样full stack的MVC框架，但始终对UI和Model的双向绑定无爱啊，还是钟爱Backbone啊。前段时间简单研究了下它的源码，谈谈里面继承即extend函数的实现。先来看看来自[带注释的Backbone源码](http://backbonejs.org/docs/backbone.html#section-190)里跟extend有关的源代码。

<!--more-->

``` javascript
var extend = function(protoProps, staticProps) {
    var parent = this;
    var child;

    if (protoProps && _.has(protoProps, 'constructor')) {
        child = protoProps.constructor;
    } else {
        child = function(){ return parent.apply(this, arguments); };
    }

    _.extend(child, parent, staticProps);

    var Surrogate = function(){ this.constructor = child; };
    Surrogate.prototype = parent.prototype;
    child.prototype = new Surrogate;

    if (protoProps) _.extend(child.prototype, protoProps);

    child.__super__ = parent.prototype;

    return child;
};

Model.extend = Collection.extend = Router.extend = View.extend = History.extend = extend;
```


从第一行开始看，先看extend的函数定义，两个参数`protoProps`和`staticProps`，从参数的命名就可以猜出来：

* `protoProps` —— prototype properties：原型上的属性，即实例属性
* `staticProps` —— static properties：静态属性

举个例子，比如

``` javascript
function ClassA(name) {
    this.name = name;
}
ClassA.prop1 = "this is a static property.";
ClassA.prototype.prop2 = function() {
    console.log("this is a prototype property.")
}
```

这里，`ClassA`是一个构造函数，可以认为它定义了一个类，而`prop1`就是这个类的静态属性，这个属性是属于这个类的，`prop2`就是原型上的属性，它是属于实例的，请脑补其他语言。第二行将`this`赋给了`parent`，这里的`this`代表什么呢？这里可以参考[Javascript秘密花园](http://bonsaiden.github.io/JavaScript-Garden/zh/)里关于[this的5种情况](http://bonsaiden.github.io/JavaScript-Garden/zh/#function.this)的分析。这里的this属于第三种情况，也就是方法调用。可以从extend的实际调用情况来理解，第24行将extend函数分别赋给了Model、Collection、View、Router等，而真正调用的时候，比如新建一个Model：

``` javascript
var todo = Backbone.Model.extend({...});
```

可以明显的看出这是一个函数调用，这里`this`指的就是`Backbone.Model`，也就是发生继承时的父类，所以第二行就将这个父类赋给了变量`parent`。第三行定义的变量`child`就是这个继承结束后的子类，可以看到第21行extend函数的返回值就是这个子类。

### 构造函数的继承

来看第5行，首先检查传入的`protoProps`里有没有一个key为`constructer`的属性，如果有就将这个属性值作为构造函数赋给`child`，如果没有，就将父类的的构造函数赋值给它。这里`child = function(){ return parent.apply(this, arguments); };`就是干这个事的，注意`parent`就是父类，实质上是父类的构造函数，这里的`this`属于“调用构造函数”的情况，可继续参见[Javascript秘密花园](http://bonsaiden.github.io/JavaScript-Garden/zh/#function.this)。按照前面的，继续从实际调用的情况来理解，这里的`child`是子类，是构造函数，所以构造函数里的`this`是代表新创建的对象的，也就是调用`new child()`所构造的实例对象本身，这里的`arguments`是调用`new child()`时传入的参数，利用`apply`函数将这些参数传进parent的构造函数中，并将构造函数的context设置成`child`本身。这里其实就是将`parent`的构造函数“包装”了一下赋值给了`child`，这个包装一个是将传给`child`构造函数的参数传给`parent`的构造函数，另一个是将`parent`的构造函数里的context即this换成`child`。一句话，在子类的构造函数里调用父类的构造函数，请脑补其他语言。

### 静态属性的继承

下面来看第11行，这里调用的是`underscore`库的extend函数，[源码](http://underscorejs.org/docs/underscore.html#section-78)如下：

``` javascript
_.extend = function(obj) {
    each(slice.call(arguments, 1), function(source) {
        if (source) {
            for (var prop in source) {
                obj[prop] = source[prop];
            }
        }
    });
    return obj;
};
```

从调用入手`_.extend(child, parent, staticProps);`，有三个参数传给`extend`，extend的实现里先通过`slice`将第一个参数后面的组成数组，即这里的`parent`和`staticProps`，然后调用`each`遍历这个数组，对数组里的每个元素，取它的属性复制给第一个参数，即`child`。这句代码其实是将`parent`里本来的静态属性，加上传入的`staticProps`里的静态属性全部复制给`child`这个子类。

### 原型上的属性继承

继续来看第13-15行代码，这里完成的就是原型上的属性继承。先不看代码的实现，凭直觉，原型继承就是将父类的prototype复制给子类的prototype即可，那么直接一句`child.prototype = parent.prototype`不就可以了，为什么要写这么好几句呢？如果只写一句的话，那么会直接造成“对子类prototype的更改会反映到父类的prototype上”，因为两者已经指向同一个对象了。继续思考，由于parent的prototype上的属性都是实例属性，那么利用parent构造函数新建一个实例，然后将这个实例赋值给`child.prototype`，这样子类就可以继承父类的实例属性了，即`child.prototype = new parent()`即可，还是一句话搞定啊！为什么不这样写呢？这样确实能解决问题，并且子类对prototype进行扩展也不会影响父类，但它的问题就是：浪费资源！试想，new一个parent出来，这个对象可能比较占用资源，比如parent的构造函数里可能会有很多赋值，而我们其实只需要它的实例属性，这些赋值都是没有意义的，可能造成资源浪费。为了解决这个问题，可以构造一个空的构造函数，让这个构造函数拥有parent的所有实例属性，然后new这个空的构造函数去创造对象，再将这个对象赋值给child的prototype即可。先看下面这个实现：

``` javascript
var Surrogate = function(){};
Surrogate.prototype = parent.prototype;
child.prototype = new Surrogate();
```

这样就可以达到目的，但这样又有了新问题，拿上面的`ClassA`为例，如果我们new一个`ClassA`，可以看到：

![](/assets/images/understand-backbone-extend-1.PNG)

其中的`__proto__`就是prototype，里面就是实例属性，这里也印证了最前面提到的`prop2`属于实例属性。这里面有一个特殊的属性`constructor`，它就是构造函数，也就是`ClassA`的定义本身。所以上面一旦对`prototype`进行了修改，代表构造函数的`constructor`也会被修改，需要重新“改回来”，即：

``` javascript
child.prototype.constructor = child;
```

也就是说，需要四句代码搞定实例属性的继承。理清这些后，回来看第13-15行，这里做了个小技巧，将child的构造函数`child`的“更改”放在代理函数的构造函数里。这里又要分析`this`了，显然，这里的`this`是在`Surrogate`的构造函数里，它代表`new Surrogate()`构造出来的新的实例对象，也就是第15行的`child.prototype`，完全等同于上面的那句代码。通过这三行代码，实例属性就继承过来了。但是传进来的参数`protoProps`里的实例属性还没复制进来，后面的第17行就是干这个事的，同样是通过underscore的extend函数来实现的，跟上面的静态属性的复制完全类似。

### 引用父类

继续看第19行，这一行比较好理解，直接将parent的实例属性赋值给child的静态变量`__super__`。继续脑补其他语言，这句其实就是方便子类通过`super`来引用父类。

完成了构造函数、静态属性、实例属性的继承，并完成通过名为`__super__`的静态变量对父类的引用之后，第21行将这个`child`返回，这样一个继承的操作就完成了。最后一句第24行将这个定义好的extend函数赋值给Backbone的各个对象Model、Collection、Router、View、History。

通过深入理解Backbone里extend函数的实现，可以更好的理解Javascript中面向对象的一些知识以及javascript的继承机制，这里的实现是很标准的，这一点可以通过CoffeeScript来验证。CoffeeScript官方有一个[Try CoffeeScript](http://coffeescript.org/)的功能，可以即时的将网页上的CoffeeScript代码编译成普通的Javascript代码。随便输入诸如`class A extends B`的代码，可以看到右边的编译结果多了一个叫`__extends`的函数定义：

``` javascript
__hasProp = {}.hasOwnProperty;
__extends = function(child, parent) {
    for (var key in parent) {
        if (__hasProp.call(parent, key)) child[key] = parent[key];
    }
    function ctor() { this.constructor = child; }
    ctor.prototype = parent.prototype;
    child.prototype = new ctor();
    child.__super__ = parent.prototype;
    return child;
};
```

可以看到，这与Backbone中的实现是一致的！关于更多关于Javascript中的面向对象知识，可以读读阮一峰的系列文章：

* [Javascript 面向对象编程（一）：封装](http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_encapsulation.html)
* [Javascript面向对象编程（二）：构造函数的继承](http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_inheritance.html)
* [Javascript面向对象编程（三）：非构造函数的继承](http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_inheritance_continued.html)
