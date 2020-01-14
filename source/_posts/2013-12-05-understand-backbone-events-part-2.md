date: 2013-12-05 21:00
title: 理解Backbone中Events的实现(二)
categories:
- 前端开发
tags:
- BackboneJS
- UnderscoreJS
- Events
- Javascript
- 前端MVC
---

书接[上回](http://pinkyjie.com/2013/12/02/understand-backbone-events-part-1/)，上次介绍了`Backbone.Events`的`on`和`once`，这篇继续介绍剩下的，先从`off`开始吧。

<!--more-->

### off的实现

``` javascript
off: function(name, callback, context) {
    var retain, ev, events, names, i, l, j, k;
    if (!this._events || !eventsApi(this, 'off', name, [callback, context])) return this;
    if (!name && !callback && !context) {
        this._events = {};
        return this;
    }
    names = name ? [name] : _.keys(this._events);
    for (i = 0, l = names.length; i < l; i++) {
        name = names[i];
        if (events = this._events[name]) {
            this._events[name] = retain = [];
            if (callback || context) {
                for (j = 0, k = events.length; j < k; j++) {
                    ev = events[j];
                    if ((callback && callback !== ev.callback && callback !== ev.callback._callback) ||
                        (context && context !== ev.context)) {
                        retain.push(ev);
                    }
                }
            }
            if (!retain.length) delete this._events[name];
        }
    }
    return this;
}
```


`off`用来解除事件的绑定，传入的参数还是老三样，即事件的三要素：名称、回调、上下文。在[带注释的源码](http://backbonejs.org/docs/backbone.html#section-17)里很详细的介绍了`off`的大致流程。首先这三个参数都是可选的，三要素可以唯一确定一个绑定，忽略一个要素，则范围就会扩大。比如，不传context，则匹配前两个的参数的绑定全部被解绑，如果callback也不传，则该事件的所有绑定被解绑，如果连name也不传，也就是直接调`off()`，则所有事件的所有绑定都被解绑。上篇文章分析过，绑定就是把callback填到特定的数组里，那解绑就从数组里删除就好了。

明白了原理，来看代码，第2行定义了一堆要用到的变量，第3行又是熟悉的`eventsApi`判断，可见这个name也是支持同时传多个名称的，另外这里还多了一个检查`_events`的条件，如果不存在那说明没有任何事件绑定，直接返回。第4-7行就是刚才分析过的不带参直接调`off()`的情况，直接清空`_events`，接触所有绑定，然后返回。第8行将要解绑的事件名称放入`names`数组，如果传了name就只放一个name，没传就将`_events`里所有事件名称都放进去。从第9行开始，遍历这个`names`数组，每次循环先将绑定到这个事件上的回调放入`events`数组。第11行判断如果这个数组没有值，也就是说不存在回调绑定在这个事件上，那本次循环就直接结束了，如果存在，则继续走。第12行直接清空`this._events[name]`数组，并赋值给`retain`变量，retain的字面意思就是“保持”，意思就是不需要移除的回调会被放回`retain`中去，即放回`this._events[name]`中去。接着看第13行，判断如果传了callback或context，则第14行开始遍历绑定在这个事件上的所有回调，先将每个回调赋值给`ev`，回忆下，这个`ev`是个对象，即`{callback: xxx, context: xxx, ctx: xxx}`。第16-19行判断如果这个`ev`不需要移除，就将其填入`retain`中去。这个判断条件`(callback && callback !== ev.callback && callback !== ev.callback._callback) || (context && context !== ev.context)`也很直观，两种情况下不需要移除。第一种情况，若传入了callback并且传入的callback与`ev.callback`不匹配，并且与`ev.callback._callback`也不匹配，则不需移除，后者主要是为了`once`，回忆一下，用`once`绑定的回调原形存在`_callback`里。第二种情况，若传入了context并且与`ev.context`不匹配，则不用移除。经过这个判断，所有不需移除的绑定都填入了`retain`数组中，接下来第22行，判断若`retain`为空，则直接删除整个`this._events[name]`数组。最后第25行返回this解绑结束，这里顺便提一下，很多函数都返回this的作用就是方便作“链式调用”，比如可以`obj.on(xxx1).off(xxx2)`这样写。

### trigger的实现

按照源码里的顺序，下一个该`trigger`函数了，用于触发事件，调用回调函数。

``` javascript
trigger: function(name) {
    if (!this._events) return this;
    var args = slice.call(arguments, 1);
    if (!eventsApi(this, 'trigger', name, args)) return this;
    var events = this._events[name];
    var allEvents = this._events.all;
    if (events) triggerEvents(events, args);
    if (allEvents) triggerEvents(allEvents, arguments);
    return this;
}
```

trigger函数只有一个参数name，就是要触发的事件名称，其实trigger是可以传很多参数进来的，后面的参数都会传给回调函数callback。可以看第3行，利用slice将除name以外的参数存入了`args`变量中。第2行和第4行同样是判断，首先如果没有`this._events`则没有回调绑定在这个事件上，直接返回；其次，利用`eventsApi`转换参数，用于递归调用同时触发多个事件。第5行将绑定在该事件上的所有回调放入`events`数组。第6行将绑定在名为`all`的事件上的所有回到放入`allEvents`数组，这个`all`是一个特殊的事件，触发任意的事件都会触发它。第7-8行调用`triggerEvents`函数来具体调用每个回调，注意这里的区别，如果触发的是`all`事件，则传给回调函数的第一个参数是具体的事件名称，所以这里直接传入`arguments`。

下面就具体看看怎么调每个回调吧，放上`triggerEvents`的代码：

``` javascript
var triggerEvents = function(events, args) {
    var ev, i = -1, l = events.length, a1 = args[0], a2 = args[1], a3 = args[2];
    switch (args.length) {
        case 0: while (++i < l) (ev = events[i]).callback.call(ev.ctx); return;
        case 1: while (++i < l) (ev = events[i]).callback.call(ev.ctx, a1); return;
        case 2: while (++i < l) (ev = events[i]).callback.call(ev.ctx, a1, a2); return;
        case 3: while (++i < l) (ev = events[i]).callback.call(ev.ctx, a1, a2, a3); return;
        default: while (++i < l) (ev = events[i]).callback.apply(ev.ctx, args);
    }
};
```

这里将传给回调函数的参数组`args`的前3个先赋给3个变量：`arg1, arg2, arg3`，接着一个switch判断参数组`args`的长度，分情况执行。每种情况下都利用一个while循环去遍历绑定在该事件上的所有回调，利用`(ev = events[i]).callback.call(ev.ctx, xxx....);`去调用具体的回调函数，上篇文章也提到过，`ev`对象里虽然有`context`，但真正起作用的是`ctx`，这是为了避免绑定时没传context，具体可以查看[上篇文章](http://pinkyjie.com/2013/12/02/understand-backbone-events-part-1/)。如果参数个数是0-3，就利用前面定义好的`arg1, arg2, arg3`直接传，否则进入default分支，利用`apply`直接传整个参数组`args`。这里初看会有疑惑，其实不管参数组的个数，所有情况都可以用default分支的代码去处理啊，干嘛多此一举呢？官方的解释也用了“difficult-to-believe”这个词，说Backbone里大部分的事件回调都只有3个参数，这一点可以查看官方文档里的[所有事件列表](http://backbonejs.org/#Events-catalog)，确实如此。在这样的情况下，使用switch区分参数长度，可以提高常见情况下（即参数小于3个）的事件分派效率。我还是有点小疑惑，莫非在Javascript里`call`的效率要比`apply`高？好像也没搜到相关的资料证明这一点。总之，事件的触发就是如此了。

### 监听别人的事件

绑定、解绑和触发自己的事件都已经分析完了，剩下的三个函数`listenTo, listenToOnce, stopListening`是用来绑定和解绑其他对象的事件的。上篇文章也提到过，绑定其他对象的事件，无非就是将回调函数放入其他对象的`_events`变量中去罢了，原理是一样的，但是作为监听者这边也要有一个变量来存放这个“其他对象”的引用，这样解绑的时候才能找到这个“其他对象”，而这个变量就是`_listeningTo`。那监听其他对象的事件的流程就是，将这个“其他对象”的引用写入`_listeningTo`，然后调用这个“其他对象”的on或once进行回调函数的绑定即可。明白了原理就来看代码实现吧。

### listenTo和listeToOnce的实现

``` javascript
var listenMethods = {listenTo: 'on', listenToOnce: 'once'};
_.each(listenMethods, function(implementation, method) {
    Events[method] = function(obj, name, callback) {
        var listeningTo = this._listeningTo || (this._listeningTo = {});
        var id = obj._listenId || (obj._listenId = _.uniqueId('l'));
        listeningTo[id] = obj;
        if (!callback && typeof name === 'object') callback = this;
        obj[implementation](name, callback, this);
        return this;
    };
});
```

这两个函数的实现类似，是通过同一段代码实现的，官方解释说listenTo和listenToOnce分别是on和once的“控制反转”，大家凑合理解理解吧。。。第1行先将对应关系存入变量`listenMethods`，接着就用Underscore的`_.each`进行遍历，这第2行我开始还有些疑惑，后来查了下[Underscore的文档](http://underscorejs.org/#each)才发现，each在遍历object的时候第一个参数是value，第二个参数是key，有点不合常理。也就是这里的变量`implementation`在两轮循环里分别代表`on`和`once`，而变量`method`代表`listenTo`和`listenToOnce`。第3行开始定义这两个函数，参数分别是要监听的对象obj，事件名称name和回调函数callback。接下来第4行同样是熟悉的“短路”来赋值我们刚才提到的变量`this._listeningTo`，不存在就置空。第5行拿对象obj的id，这个id就是`obj._listenId`，不存在的话就调用Underscore的`_.uniqueId`来生成，同样是“短路”，真是Backbone最爱的代码风格。第6行将obj的引用存入`_listeningTo`，以id为key。第7行，考虑事件名称name的两种形式，如果是object并且没有传callback，则将this赋值给callback，这样设置的原因我猜想是为了通过`eventsApi`的检验。第8行调用obj的on或once绑定事件，注意第3个参数context传入的是this，也就是事件的监听者。也就是说，在Backbone里监听其他对象的事件时，回调函数的context是指向事件监听者本身的。最后第9行依然是返回this结束。

### stopListening的实现

理解了绑定的原理，那解绑自然就是删除`_listeningTo`里对obj的引用，然后调用obj的off来解绑。

``` javascript
stopListening: function(obj, name, callback) {
    var listeningTo = this._listeningTo;
    if (!listeningTo) return this;
    var remove = !name && !callback;
    if (!callback && typeof name === 'object') callback = this;
    if (obj) (listeningTo = {})[obj._listenId] = obj;
    for (var id in listeningTo) {
        obj = listeningTo[id];
        obj.off(name, callback, this);
        if (remove || _.isEmpty(obj._events)) delete this._listeningTo[id];
    }
    return this;
}
```

同样的3个参数，与`listenTo`一致：被监听的对象，事件名称，回调。第2、3两行，先拿到`this._listeningTo`，如果为空则表明没有监听其他对象的事件，直接返回。第5行与`listenTo`里一样，不再细说。第6行的代码乍一看感觉有些多余，根据上面说的原理，第7行直接对`_listeningTo`进行遍历了，在遍历中找到和参数obj匹配的对象，然后调用它的off就完事了，干嘛多此一举第6行来这么一句呢？细细想过以后，我认为还是从提高效率的角度出发吧。如果传了obj，那么第6句的赋值就可以让第7句的循环只处理`_listeningTo[obj._listenId]`这一个对象，循环跑一遍就结束了。这样，如果`_listeningTo`里面有很多对象引用的话可以大大提高效率。在第7-11行的循环里，也可以分两种情况分析，一种就是传了obj，那么循环一遍就结束了，循环里的这个obj就是传进来的obj，另一种如果没传obj，那么遍历`_listeningTo`，找到所有对象引用，解绑与name和callback匹配的绑定。循环里的最后一行，判断如果没有传name和callback，或者obj上没有任何事件绑定，那么就删除这个对象引用，即不再监听该对象的任何事件。这里也包含了极端情况，那就是不带参数直接调用`stopListening()`时，调用者的`_listeningTo`里的键值对会被全部删除，也就是说`_listeningTo`将被置空，也就是说该对象将不再监听其他任何对象的任何事件。从这些分析可以看出来，`stopListening`虽然与`off`写法不一样，但涵盖的情况都是一样的。最后一行，和往常一样，返回this结束。

### 剩下的几句跟Events相关的代码

``` javascript
Events.bind   = Events.on;
Events.unbind = Events.off;
_.extend(Backbone, Events);
```

剩下的3句跟Events相关的代码，前两句是为了和以前的版本保持兼容，将bind和unbind设置为on和off的别名，最后一句将Events对象Mix进了Backbone里去，让Backbone也具有绑定和触发事件的功能。这样做是为了把Backbone作为全局的一个事件总线，使用`Backbone.trigger`和`Backbone.on`去构建一个“发布者/订阅者”系统。

### 总结

到这里，两篇文章就把Backbone的Events实现全部分析一遍了，原理还是简单的，但一些实现细节很值得推敲和学习，一些代码风格也值得推广。从这里不仅深入了解了Events的实现过程，也顺便了解了很多Javascript的高级用法。
