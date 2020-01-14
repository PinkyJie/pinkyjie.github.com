date: 2013-12-18 21:20
title: 用Backbone的思想写web页面
categories:
- 前端开发
tags:
- BackboneJS
- UnderscoreJS
- jQuery
- EventBus
- Javascript
- 发布订阅模式
- 前端MVC
- 模块化
---

研究Backbone有一段时间了，接受了Backbone的这种设定后，就会发现已经开始排斥用plain的方式再去写web页面了。但是自己练手的项目还比较自由，要是公司的项目怎么办呢，就是不准你用Backbone库你怎么弄呢！其实，抛开Backbone库本身，照样可以写出Backbone的风格来！学一个库本身不重要，重要的是理解它的思想，并灵活运用到其他地方去。这一说有点高端了，其实我也还在摸索阶段，下面就说说我自己摸索出来的“Backbone风”吧！

<!--more-->

### View——绝对的核心

Model、View、Controller这三个玩意在每个人心中的地位可能都不一样，如果要问哪个最重要，每个人都会有自己不同的理由。但我这里要说的是，对最终用户来说，View显然是核心，因为它是最终用户唯一能看到的东西，工作在“最前线”的！在传统的MVC框架（如Ruby on Rails）中，View显得“很不上档次”，它只是Template的代名词，只是负责将Controller传过来的Model渲染成页面。而在Backbone这个“伪MVC”（MV\*）库中，赋予了View新的意义，由于它没有真正意义上的Controller，实际上View充当了部分Controller的功能。来回忆下，在Backbone里一个View通常都有哪些property：`el`（代表web页面上的一块区域），`template`（代表模板），`model/collection`（代表要渲染进模板的数据），除此之外就是各种页面元素的事件绑定和处理以及监听其他View或Model/Collection的事件。简单来说，一个Backbone的View就代表了web页面上的一块区域（简单理解就是一个div），它全权负责这块区域，包括如何渲染它，如何处理这块区域内的事件响应，如果根据其他对象的事件来更改这块区域的渲染，等等等等。而一个完整的web页面恰恰可以看成是一个个的区域组成的，所以，View可以说是绝对的核心。这就是我所理解的Backbone风格中最重要的一点！可以大胆的想象，在非常简化的情况下，只要几个View就够了！！！勇敢的少年，随我来实践吧！


### 几个View一台戏

为了方便描述，这里假设要做一个简单的用户系统，这个系统就只有一个简单的页面，页面分为两部分，上面一个table来展示所有的用户信息，下面一个form，当用户需要新增、修改用户信息时出现。看如下的示意图：

![](/assets/images/rock-your-web-page-in-backbone-way-1.png)

按照刚才的思路，显然，这个页面可以分成两个View，如上图黄色矩形框所标。那根据效果图先来搭一下页面的HTML框架吧！

``` html
<div id="content" class="container">
    <h3>Users Page</h3>
    <hr>
    <div id="user-list">
        <div class="header">
            <span>All users</span>
            <a href="#" title="add">
             <span class="pull-right"><i class="icon-plus"></i>Add user</span>
            </a>
        </div>
        <table class="table table-bordered">
            <thead>
                <th>Id</th>
                <th>Name</th>
                <th>Gender</th>
                <th>Age</th>
                <th>Action</th>
            </thead>
            <tbody></tbody>
        </table>
    </div>
    <hr>
    <div id="edit-form"></div>
</div>
```

整个页面放在一个`#content`的div中，这个div里有两个子div来代表图中的两个黄框`#user-list`和`#edit-form`。`#user-list`中已经写好了table的表头，剩下的表格行数据以及form表单，则需要依靠template来显示了。

``` html
<script type="text/template" id="item-template">
  <tr>
    <td><%- id %></td>
    <td><%- name %></td>
    <td><%- gender %></td>
    <td><%- age %></td>
    <td>
      <a href="#" title="edit"><i class="icon-edit"></i></a>
      <a href="#" title="remove"><i class="icon-remove"></i></a>
    </td>
  </tr>
</script>
<script type="text/template" id="form-template">
  <form class="form-horizontal">
    <div class="control-group">
      <label class="control-label" for="input-name">Name:</label>
      <div class="controls">
        <input type="text" id="input-name" name="name" value="<%- name %>" />
      </div>
    </div>
    <div class="control-group">
      <label class="control-label" for="input-gender">Gender:</label>
      <div class="controls">
        <label class="radio inline">
          <input type="radio" name="gender" value="boy"
            <% if(gender === 'boy') {%>checked <% } %> />Boy
        </label>
        <label class="radio inline">
          <input type="radio" name="gender" value="girl"
            <% if(gender === 'girl') {%>checked <% } %> />Girl
        </label>
      </div>
    </div>
    <div class="control-group">
      <lable class="control-label" for="input-age">Age:</lable>
      <div class="controls">
        <input type="number" name="age" id="input-age" value="<%- age %>" />
      </div>
    </div>
    <div class="text-center">
      <button type="submit" class="btn">Save</button>
    </div>
  </form>
</script>
```

上面的template使用的是Underscore中自带的模板引擎，User的四个属性以id、name、gender和age来表示。HTML部分就完事了，下面就是激动人心的JS部分了。根据上面的分析，我们可以写两个View类，在document ready事件后，分别用相关参数初始化这两个View并渲染，页面不就出来了嘛！接下来的问题是，View类的初始化需要什么参数呢？按照上一小节的分析，首先`el`必不可少，代表View在实际页面中所代表的区域，然后`template`模板也会用到，渲染table和form都要用到模板，最后，最重要的一点，数据从哪来？在Backbone里，Model/Collection通过使用`Backbone.Sync`来完成与服务器的通信，这里因为我们的Model只有一个，那就是User。实现一个类似Backbone.Model一样的类是有些复杂的，简单起见，我们只实现一个`Sync`来完成与服务器的通信，它全权负责Model的CURD，然后只要将这个Sync传给View，那View就知道如何与服务器通信了！这3个输入参数足够构建一个View了。先来搭一下JS的整个结构框架。

``` javascript
(function($){
    // view for table div
    var TableView = function(element, template, sync) {};
    TableView.prototype = {};
    // view for form div
    var FormView = function(element, template, sync) {};
    FormView.prototype = {};
    // Sync for user
    var UserSync = {};
    // main
    function init() {
        var tableView = new TableView('#user-list', '#item-template', UserSync);
        var formView = new FormView('#edit-form', '#form-template', UserSync);
        tableView.render();
    }
    $(init);
})(jQuery);
```

整个页面的JS包在一个[“立即调用函数表达式（IIFE）”](http://en.wikipedia.org/wiki/Immediately-invoked_function_expression)中，即`(function($){...})(jQuery)`，这样写有很多好处，最明显的就是不会污染全局变量空间，里面定义的各种变量都是局部的。另外，将`jQuery`作为参数传入，一方面提高效率，因为这时`$`符号只是一个形参，否则每次遇到`$`都要去解析；另一方面，这样还可以避免`$`符号与其他库冲突。所以这么写算是一个Best Practice。看上面的代码，先定义两个View类代表table区域和form区域，三个入口参数就是我们刚才分析好的el、template和sync。它们的实例方法将定义在`prototype`里。接着整个程序的入口`init`函数里，初始化两个View，并渲染table区域。之所以不渲染form区域，是因为form区域的显示需要用户的操作。最后的`$(init)`简写方式表明document ready的时候调用init函数，等价于`$(document).ready(init)`。

### TableView的具体实现

下面就要具体的实现View了，初始化实际就是构造函数，回忆一下在Backbone里View的initialize函数通常都干什么事呢？

``` javascript
el: '#todoapp',
statsTemplate: _.template($('#stats-template').html()),
events: {
    'keypress #new-todo': 'createOnEnter',
    'click #clear-completed': 'clearCompleted',
    'click #toggle-all': 'toggleAllComplete'
},
initialize: function () {
    this.allCheckbox = this.$('#toggle-all')[0];
    this.$input = this.$('#new-todo');
    this.$footer = this.$('#footer');
    this.$main = this.$('#main');

    this.listenTo(app.todos, 'add', this.addOne);
    this.listenTo(app.todos, 'reset', this.addAll);
    this.listenTo(app.todos, 'change:completed', this.filterOne);
    this.listenTo(app.todos, 'filter', this.filterAll);
    this.listenTo(app.todos, 'all', this.render);

    app.todos.fetch({reset: true});
},
```

上面的代码截自[TodoMVC](https://github.com/tastejs/todomvc/tree/gh-pages/architecture-examples/backbone)的`app-view.js`文件，可以发现，View的初始化一般就是这三件事：

* 创建区域内子元素的引用，以备其他方法中使用（第9-12行）
* 绑定事件，响应其他对象的事件（第14-18行）
* 通过调用collection上的fetch函数获取数据并触发View的render方法（第20行）

除了initialize函数之外，第1-7行完成的事情我们也需要放入自己的构造函数：对`el`和`template`赋值，绑定区域内元素的各种事件。那么比葫芦画瓢，我们也在自己的构造函数里照着做：

``` javascript
var TableView = function(element, template, sync) {
   this.$el = $(element);
   this.itemTemplate = _.template($(template).html());
   this.sync = sync;
   this.collections = [];
   // reference sub element
   this.$tableBody = $('tbody', this.$el);
   this.$addBtn = $('a[title=add]', this.$el);
   // bind events
   _.bindAll(this, 'addItem', 'editItem', 'removeItem');
   this.$addBtn.on('click', this.addItem);
   this.$el.on('click', 'a[title=edit]', this.editItem);
   this.$el.on('click', 'a[title=remove]', this.removeItem);
}
```

第2-4行，将传进来的参数赋值，这里有个小习惯，也算是“Backbone风”吧，如果变量的值是jQuery选择器的结果，那么这个变量就以`$`开头，这样从变量名就可以推测出变量代表的东西了，如上面的`el/tableBody/addBtn`。第5行将属于View的collections初始化为空，以方便后面用。第7-8行引用区域内的元素，选择器的第二个参数`this.$el`将查找范围限定在本区域内。第10-13行绑定区域内元素的事件，相当于Backbone里的`events:{}`里的东西。第10行的作用是将这些回调函数的context（即this）都设定为View本身，因为默认情况下里面的this指的是事件的触发者。这里有一点值得注意，第11行add的绑定方法和第12行edit、第13行remove的绑定方法有点小区别，这是因为现在View还没有渲染，区域内是没有edit和remove元素的，此时他们还在模板里呆着呢。到这里为止，除了“响应别的对象的事件”以及“获取数据触发render”这两项，其他初始化工作都完成了。关于剩下的这两个，第一个后面会说到，第二个因为我们没有单独的model类和collection类，而且有些View不需要一初始化就渲染（如form），故这里将数据的获取放入render中，手动触发render（即刚开始的代码中的这句`tableView.render();`）。

初始化以后，开始实现里面的方法吧，方法的实现全部放在prototype里。首先最重要的一个方法就是`render`，另外，构造函数里的坑也都要填`addItem/editItem/removeItem`，最后为了方便清除View，加一个reset方法。

``` javascript
render: function() {
    this.reset();
    var self = this;
    this.sync.fetch(function(response){
        response.users.forEach(function(item){
            self.$tableBody.append(self.itemTemplate(item));
            self.collections.push(item);
        });
    });
},
reset: function() {
    this.$tableBody.empty();
    this.collections = [];
},
```

reset方法比较容易理解了，清空table的body中的内容和collections。render方法里先调用一下reset，然后调用sync的fetch方法从服务器拿数据，传一个回调函数进去。在回调函数中，遍历返回的json数据里的users数组，将数据传入template里进行渲染，返回的结果追加到table的body里，并将数据缓存在collections里。这里有一点值得说一下，第3行，将this赋值给self，这也是backbone里常用到的，这里的this代表View实例本身，将其保留下来以便在回调函数里用，因为回调函数里的this指的就不是View本身了。既然用到了sync，也顺便写个假的演示用吧！

### Sync的实现

``` javascript
// Sync for user
var UserSync = {
    fetch: function(success){
        // fake data
        var response = {
            users: [
                {
                    id: 1,
                    name: 'Tom',
                    gender: 'boy',
                    age: 21
                },
                {
                    id: 2,
                    name: 'Jack',
                    gender: 'boy',
                    age: 22
                },
                {
                    id: 3,
                    name: 'Lucy',
                    gender: 'girl',
                    age: 23
                }
            ]
        };
        success(response);
    },
    create: function(params, success){
        console.log('create!');
        success();
    },
    update: function(params, success){
        console.log('update!');
        success();
    },
    destroy: function(id, success){
        console.log('destroy!');
        success();
    }
};
```

在UserSync中实现了四种基本操作，这同样是模仿Backbone，每个方法都接收一个success的回调。fetch中伪造了一下数据给success调用，后面的三个因为没有服务器不好实现，只是简单的在console里打印一下，然后调用success回调。在真实的开发中，这里的四个函数可以使用`jQuery.ajax`来实现，这里就不详细说了。

继续实现剩下的几个方法：`addItem/editItem/removeItem`。

``` javascript
addItem: function(e) {
    e.preventDefault();
    var nullUser = {
        name: '',
        gender: 'boy',
        age: 0
    };
    // open form view with nullUser
},
editItem: function(e) {
    e.preventDefault();
    var itemId = +$(e.target).parents('tr').children('td:eq(0)').text();
    var item = _.find(this.collections, function(model){
        return model.id === id;
    });
    // open form view with specified item
},
removeItem: function(e) {
    e.preventDefault();
    var $tr = $(e.target).parents('tr');
    var itemId = +$tr.children('td:eq(0)').text();
    var self = this;
    this.sync.destroy(itemId, function(){
        $tr.remove();
    });
},
```

这三个方法的第1行都是调用`e.preventDefault()`来阻止a标签的默认行为，即防止点击链接以后将`href=#`里的井号添加到URL上，这也是优化用户体验的必备操作。先看`addItem`，当点击“Add User”的时候需要传一个空的user给form并显示form。这里的第3-7行就创建了这么一个空的nullUser，但怎么传递给form呢？简单！直接用jQuery选择器选中form然后拿数据进行渲染就行了呗，但这样做的就会使两个View产生耦合，即tableView需要引用formView，这样就使一个view需要依赖另一个view，不符合我们想模块化的初衷。如果想使两个view互相独立，那么就不应该存在引用。试想这样的场景，这里tableView可以发出一个广播：用户点了“Add User”了，并且我已经创建好了空的nullUser，谁对这个事件感兴趣的自己响应去吧。然后，formView订阅这个广播，并对此作出反馈（即用nullUser渲染formView并显示）。这是一个典型的“发布者/订阅者”模式，可以通过event aggregator来实现。Backbone里有一个实现的很好的Backbone.Event来做这个事，记得[上篇文章](http://pinkyjie.com/2013/12/05/understand-backbone-events-part-2/)在分析Backbone的Events实现时有这么一句代码`_.extend(Backbone, Events);`，这句将Events对象mix进`Backbone`里，使`Backbone`成为一个全局的“发布者/订阅者”，也就是那篇文章里说到的EventBus。那么我们也需要依葫芦画瓢实现这么一个全局的EventBus。

### EventBus的实现

``` javascript
// Event bus
var EventBus = {
    trigger: function(eventName, data) {
        var e = $.Event(eventName);
        e.param = data;
        $(document).trigger(e);
    },
    on: function(eventName, callback) {
        $(document).on(eventName, callback);
    },
    off: function(eventName) {
        $(document).off(eventName);
    }
};
```

上面的代码借助jQuery的事件系统实现了一个简单的EventBus。trigger负责发布事件，两个参数分别是事件的名称和事件携带的数据，第4行我们利用jQuery的事件系统包装了一下这个事件，第5行将data赋值给事件的param，让自定义事件携带数据，第6行利用全局的document对象将事件发布出去。on负责订阅事件，off负责取消订阅，同样是利用document对象。当然这里的实现非常简化，但足够这个例子用了。

有了EventBus，在addItem方法中我们只要发出这个事件就好，那么在addItem的尾部加入：

``` javascript
EventBus.trigger('TableView:addItem', nullUser);
```

事件的名称我们也遵循“Backbone风”，将创建好的空的nullUser作为事件的数据发出去，这样formView只要订阅这个事件即可做出响应了。接下来看editItem和removeItem，两者首先都是根据点击的行先找到点击的是哪个具体的user，即找到点击按钮所在行的第一个td里的id值，然后在collections中找到这个user，最后自然也是通知formView进行显示了，同样在后面加上：

``` javascript
EventBus.trigger('TableView:eidtItem', item);
```

removeItem方法里用同样的方法找到itemId，然后直接调用sync的destroy方法，在回调函数中删除user所在的这一行。这里有一个问题，删除操作需不需要通知formView呢？答案是肯定的，考虑这样一种情况，用户点开了edit，这时formView正在显示当前的user，然后用户又点击了remove删除当前user，那么formView需要隐藏，因为正在编辑的user已被删除。所以，同样在回调里最后一句加上：

``` javascript
var item = _.find(this.collections, function(model){
    return model.id === itemId;
});
EventBus.trigger('TableView:removeItem', item);
```

这里可以发现find user的操作用到了两次，本着DRY（Don't Repeat Yourself）的原则，写一个辅助函数：

``` javascript
_findModel: function(id) {
    return _.find(this.collections, function(model){
        return model.id === id;
    });
}
```

由于是内部调用的，所以方法以下划线开头，相当于私有的方法，这也是个不错的习惯。到这里，TableView就实现完了，基本的操作也都涵盖了。但还有一点就是，tableView同样需要响应formView的事件，比如form表单提交了以后table要重新刷新服务器的数据，那么在initialize中加入`EventBus.on('FormView:saveForm', this.onFormSaved);`，这就是前面提到的“响应别的对象的事件”，然后实现这个方法：

``` bash
onFormSaved: function() {
    this.render();
},
```

当表单提交时重新渲染tableView就会利用sync从数据库重新拿数据来渲染了。别忘了将onFormSaved方法的context改为View本身哦，即`_.bindAll(this, 'onFormSaved');`。

### FormView的具体实现

实现了TableView，FormView的实现就是手到擒来了。

``` javascript
// view for form div
var FormView = function(element, template, sync) {
    this.$el = $(element);
    this.formTemplate = _.template($(template).html());
    this.sync = sync;
    this.model = null;
    // bind events
    _.bindAll(this, 'saveForm', 'onAddItem', 'onEditItem', 'onRemoveItem');
    this.$el.on('submit', 'form', this.saveForm);
    // listen to events from other view
    EventBus.on('TableView:addItem', this.onAddItem);
    EventBus.on('TableView:editItem', this.onEditItem);
    EventBus.on('TableView:removeItem', this.onRemoveItem);
};
FormView.prototype = {
    render: function() {
        this.reset();
        if (this.model === null) {
            return;
        }
        this.$el.html(this.formTemplate(this.model));
    },
    reset: function() {
        this.$el.empty();
    },
    saveForm: function(e) {
        e.preventDefault();
        var isNew = this.model && !this.model['id'];
        var params = {
            name: $("#input-name", this.$el).val(),
            gender: $("input[name=gender]", this.$el).val(),
            age: $("#input-age", this.$el).val()
        };
        var self = this;
        if (isNew) {
            this.sync.create(params, function(){
                self.reset();
                EventBus.trigger('FormView:saveForm', {});
            });
        } else {
            params['id'] = this.model['id'];
            this.sync.update(params, function(){
                self.reset();
                EventBus.trigger('FormView:saveForm', {});
            });
        }
    },
    onAddItem: function(e) {
        this.model = e.param;
        this.render();
    },
    onEditItem: function(e) {
        this.model = e.param;
        this.render();
    },
    onRemoveItem: function(e) {
        if (this.model === e.param) {
            this.reset();
        }
    }
};
```

第2-14行的构造函数依然是完成那么几件事，入口参数的赋值，绑绑自己的事件，响应响应别人的事件，同样的，这里的子元素都还呆在模板里，所以绑定事件要用特殊的方式。简单的说一下几个方法吧，reset方法还是直接清空form，render方法先调reset，然后check这里的model是不是空，非空的话就渲染form。saveForm提交表单，拿一下表单里的值（这里就不做表单验证了），然后根据model是否有id属性来判断是进行新增操作还是更新操作，这也是跟Backbone学的。当然，表单提交后不要忘记trigger一个事件来通知tableView进行重新渲染。onAddItem/onEditItem用来响应tableView里的add和edit操作，它俩的实现一摸一样，当然真实开发中可能会有区别，这里只是简单的演示，将事件里的数据复制给model，然后渲染form。onRemoveItem方法用来响应tableView的remove操作，判断如果传过来的model就是当前model，则隐藏表单。至此，FormView也实现完了，完整的代码可以查看[我的jsfiddle](http://jsfiddle.net/PinkyJie/SN7vh/)。

### 总结

看到这里，有人会说，这么个例子我用jQuery一会就写完了，刷刷的，要不了多少行代码，折腾这半天干啥玩意！没错，对于这么个小例子，这样写确实是小题大做了。但考虑真实的开发环境，这样写的好处还是很多的：

* 用View来组织整个JS的结构，条理非常清晰
* 各个View中自己的事件都由View本身处理，方便删减功能及页面的重构，方便写测试
* 每个View相互独立，没有依赖，随便拿出来稍加改动就可以用在别处，可重用度高

最后一句，好不好用了就知道，这就是我从实践中得到的真知！
