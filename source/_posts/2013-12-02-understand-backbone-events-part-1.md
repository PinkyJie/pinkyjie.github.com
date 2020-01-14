date: 2013-12-02 22:28
title: 理解Backbone中Events的实现(一)
categories:
- 前端开发
tags:
- BackboneJS
- UnderscoreJS
- Events
- Javascript
- 前端MVC
---

上一篇分析了下[Backbone中extend的实现](http://pinkyjie.com/2013/11/29/understand-backbone-extend/)，这篇就学学Backbone中的Events。

<!--more-->


`Backbone.Events`其实就是一个plain的object，里面定义了如`on`、`off`、`trigger`等方法，设计它的原因就是方便Events中定义的方法混合进其他任意的object里，为任意object提供绑定、解绑和触发自定义事件的功能，比如[带注释的源码](http://backbonejs.org/docs/backbone.html#section-14)里提供的小例子：

``` javascript
var object = {};
_.extend(object, Backbone.Events);
object.on('expand', function(){ alert('expanded'); });
object.trigger('expand');
```

通过UnderscoreJS的extend函数使object拥有了on和trigger方法，绑定和触发了一个叫做expand的自定义事件。


### 事件的基本要素

根据[Backbone的文档](http://backbonejs.org/#Events)，`Backbone.Events`拥有 `on`、`off`、`trigger`、`once`、`listenTo`、`stopListening`、`listenToOnce` 这7个方法。一个事件，简单来说，最基本的就是三个要素，事件的名称（name），回调函数（callback）和上下文（context，即回调里的`this`指向谁），这三个要素唯一标识一个事件。一个事件可以被绑定（监听）、解绑和触发。一个对象可以监听自己的事件，也可以监听其他对象的事件。基于这些，上面7个方法可以简单分分类，用来触发事件的是`trigger`，剩下的`on`和`once`用来监听自己的事件，对应的`off`用来解绑，`listenTo`和`listenToOnce`用来监听其他对象的事件，`stopListening`用来解绑。Backbone里如何实现事件和回调函数的绑定呢？其实每个对象上有一个叫做`_events`的变量，它是一个key-value对，其中的key就是事件名称，value是一个对象数组，其中每个对象包含绑定在该事件上的一对callback和context。绑定自己的事件就是将callback加入到自己的`_events`变量中以事件名称为key的数组中，而绑定其他对象的事件就是将callback加入到其他对象的`_events`变量中以事件名称为key的数组中，解绑就是将对应的callback删除。明白了基本原理，下面就具体来分析这7个方法的实现吧。

### on的实现

``` javascript
var Events = Backbone.Events = {
    on: function(name, callback, context) {
        if (!eventsApi(this, 'on', name, [callback, context]) || !callback) return this;
        this._events || (this._events = {});
        var events = this._events[name] || (this._events[name] = []);
        events.push({callback: callback, context: context, ctx: context || this});
        return this;
    },
    // ....
}
```

第1行就不用说了，将`Backbone.Events`同时赋值给变量`Events`的作用我猜是比较短，方便后面进行引用。。。函数`on`的作用就是绑定自己的事件，3个参数就是刚才提到的事件的三要素。第3行函数的第一步就是对传入的参数进行一个判断，若`eventsApi`调用返回false或没有提供callback，就不进行下面的绑定了。后面的条件很容易理解，那么也可以推测前面的这个`eventsApi`也是检查输入参数的，但这名字起得有点匪夷所思啊！在看它的代码之前，老问题又来了，这第3行的`this`又代表什么呢？可能第一感觉就是`this`这里属于“函数调用”，指向`Backbone.Events`，但其实真正调用`on`函数的并不是`Backbone.Events`，这个`Events`是需要Mix到其他对象里去的，类似第一段代码里的object，所以这里`this`指的是事件的监听者。搞清楚了this下面就来看看函数`eventsApi`的实现代码吧。

``` javascript
var eventSplitter = /\s+/;
var eventsApi = function(obj, action, name, rest) {
    if (!name) return true;
    if (typeof name === 'object') {
        for (var key in name) {
            obj[action].apply(obj, [key, name[key]].concat(rest));
        }
        return false;
    }
    if (eventSplitter.test(name)) {
        var names = name.split(eventSplitter);
        for (var i = 0, l = names.length; i < l; i++) {
            obj[action].apply(obj, [names[i]].concat(rest));
        }
        return false;
    }
    return true;
};
```

先看看传进来的四个参数：`this, 'on', name, [callback, context]`，第一个刚才说过的事件监听者，第二个函数名称，第三个是事件名称，后面是剩下的回调函数和上下文组成的数组。第3行首先判断如果没有事件名称则返回`true`，也就是说事件名称传`null`或`undefined`也是“合法”的，又是一个匪夷所思的地方，我试着绑定一个名称为`null`的事件，然后用'trigger'触发了它，发现真的可行啊！！！不能理解！下面的两个if分别应对两种情况，一种是name为key-value对，key为事件名称，value为callback，如`{event1: callback1, event2: callback2}`。另一种是name为以空格分隔的事件名称组，如`event1 event2`，变量`eventSplitter`就是用来测试空格的正则表达式。如果事件名称只是一个单纯的字符串，那第17行直接返回true。这两种事件名称的方式都是被Backbone所支持的，不管哪种情况都是将参数转化为符合on函数定义的那样：名称，回调，上下文。通过`obj[action].apply`来调用on，on里面又会递归调用`eventsApi`，直至参数合法为止，也就是将name分解为一个不含空格的简单字符串。如果经过for循环里`on`和`eventsApi`不停递归，始终也没法让`eventsApi`直接返回true，那么for循环出来后返回false（第8行和15行），表明name参数不合法，则前面的on会直接返回，不再进行绑定。

继续回到on函数，第4行用“短路”的风格定义了前面提到的`_events`变量，其实我更喜欢的写法是`this._events = this._events || {};`，感觉这样更流行一些，更通俗的写法就是`if (!this._events) this._events = {};`。有了`_events`对象，下面第5行又是“短路”，Backbone很喜欢这种风格，后面还会看到很多。这次“短路”判断如果`_events`里还没有以事件名称name为key的value，说明还没有callback绑定到这个事件上，就将其置为空数组。前面讲原理的时候说过，事件的绑定内部都是通过这个数组实现的。下面第6行就将callback、context和ctx加入这个数组，完成绑定。这第3个参数`ctx: context || this`其实才是真正的context，可以发现后面真正触发事件的时候调用的也是这第3个参数。为什么呢？我认为是防止调用on的时候不传context，不传context的情况下一般是希望回调里的上下文是监听者本身，这里的`ctx`通过“短路”实现了这一点。最后一行将监听者this本身返回。

### once的实现

once也是绑定事件，不同的是它保证绑定的回调函数只会被调用一次，后面再触发事件的时候就不再调用了。

``` javascript
once: function(name, callback, context) {
    if (!eventsApi(this, 'once', name, [callback, context]) || !callback) return this;
    var self = this;
    var once = _.once(function() {
        self.off(name, once);
        callback.apply(this, arguments);
    });
    once._callback = callback;
    return this.on(name, once, context);
}
```

同样传入事件的3要素作为入口参数，同样第一步进行判断，不合法则立即返回。第3行将this赋值给`self`，这又是一个Backbone里很常见的写法，将this保留，以便在一些回调函数里引用，如这里的第5行。第4-7行是保障callback只被调用一次的关键，这里借助Underscore库的`once`函数。将一个匿名函数传给`_.once`进行包装，将包装后的结果赋值给变量`once`，这个经过包装的函数保证只被执行一次。后面第8行将将原始的callback放进`once._callback`保存以备后续使用，第9行将包装后的`once`作为新的回调来调用on，这就保证了这个回调将只被执行一次。这整个过程中唯一的问题就是这个匿名函数干了什么？第5行将包装后的`once`调用off解绑，这里的`self`就是外面的this，事件的监听者。然后第6行执行原始的callback。这里的arguments就是这个匿名函数被调用时传进来的参数，也就是触发事件调用回调（即此处的变量once）时传入的参数，这个好理解。那这里的this又代表什么呢，按照道理，this应该是callback的上下文才对，好像一下子又看不出来。慢慢来分析，this是在匿名函数里，要分析它就要看这个匿名函数具体是如何被执行的，而匿名函数又是作为参数传进`_.once`里面的，那先来看看Underscore里once的实现：

``` javascript
_.once = function(func) {
    var ran = false, memo;
    return function() {
        if (ran) return memo;
        ran = true;
        memo = func.apply(this, arguments);
        func = null;
        return memo;
    };
};
```

Underscore中这个once的作用是保证传进来的函数`func`只被执行一次，它的实现很像python里的装饰器。它定义了两个私有变量`ran`和`memo`，返回一个闭包，这两个变量只有在闭包里才能访问到。这个返回的闭包就是包装后的函数，首次执行ran为false，然后置ran为true，函数返回值给memo，再次执行时由于ran已为true就直接返回memo。也就是说，包装后的函数只执行一次，后面再执行的时候直接返回上次的结果（貌似跟想的不太一样。。。）。这里的这个`func`就是前面提到的匿名函数，返回的闭包就是刚才包装后的变量`once`。这里`func.apply(this, arguments)`通过apply将变量`this`变成匿名函数func中的上下文，也就是说这里的`this`就是前面`callback.apply(this, arguments)`中的this。那这个this指什么呢？这个this存在于返回的闭包（即前面的变量`once`）中，`once`是在事件触发时被调用的回调函数，那么显然，这里的this就是回调函数的上下文，这个上下文要么就是传进来的变量`context`，要么就是事件的监听者本身。

前面的分析搞的晕头转向，我也越写越迷了，那么就让实践去证明吧。打开一个同时引入Backbone和Underscore的网页，你可以自己写，懒人自然用现成的，比如[TodoMVC的演示页面](http://todomvc.com/architecture-examples/backbone/)。利用Chrome的Devtools（右键审查元素），打开source标签页，找到`backbone.js`中`callback.apply(this, arguments);`（第97行）打上断点，找到`underscore.js`中`memo = func.apply(this, arguments);`（第683行）打上断点。然后打开console标签，输入如下代码：

``` javascript
a = _.extend({id: 1}, Backbone.Events); // 时间监听者
b = function() {console.log(this.id);}; // 回调函数
c = {id: 2}; // 自定义对象，作为回调函数的上下文
a.once('test', b, c); // 自定义test事件，绑定回调和上下文
a.trigger('test', 'arg1', 'arg2', 'arg3'); // 触发自定义事件，传3个参数
```

一句一句执行，最后一句trigger执行的时候会看到断点先是断在了`underscore.js`里，然后继续执行又断在了`backbone.js`里，下面两张图分别是程序执行到两个断点处时，`this`和'arguments'变量的实际值，可以验证刚才上面的分析是正确的。

![](/assets/images/understand-backbone-events-part-1-1.PNG)

![](/assets/images/understand-backbone-events-part-1-2.PNG)

如果想验证once是否真的让回调只执行一次，可以再次触发事件，发现没有进断点，证明绑定确实只生效一次。

### 未完待续

发现有点罗嗦了，越说越多。。。剩下的几个函数下回再分解。。。
