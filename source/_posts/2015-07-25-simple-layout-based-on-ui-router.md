title: 基于ui-router的简单布局及其他应用
categories:
  - 前端开发
tags:
  - AngularJS
  - ui-router
  - breadcrumb
  - generator-aio-angular
  - sidebar
  - authentication
  - authorization
date: 2015-07-25 13:24:44
---

[ui-router](https://github.com/angular-ui/ui-router)与Angular自带的ngRoute最大的区别就是它是基于状态（state）的，而不是基于URL的。它带来的最大的便捷性就在于：
* 支持多个ui-view
* 支持ui-view的嵌套
* 支持state的继承（嵌套）

这篇文章就简单谈谈怎么利用ui-router去搭建一个网站的布局以及实现一些便捷的小功能。

<!--more-->

### 一个ui-view对应一块区域

先来看一张网站的模块图：

![图1](/assets/images/simple-layout-based-on-ui-router-1.png)

可以看到整个页面分为5个模块，有了多ui-view的支持，我们可以非常直观的用一个ui-view来对应一个模块，那么我们的主template（即`index.jade`，单页面应用通常只有一个index页面）可以这么写（使用[Jade](http://jade-lang.com/)模板引擎）：

``` jade index.jade
.wrapper
    header.header-view(ui-view="header")
    .content
        .sidebar-view(ui-view="sidebar")
        .main-content
            .breadcrumb-view(ui-view="breadcrumb")
            .main-view(ui-view="main")
    footer.footer-view(ui-view="footer")
```

这样的结构非常清晰，整个页面分为5个模块，不同的模块由不同的controller来处理：

``` javascript layout.route.js
$stateProvider.state('root', {
    abstract: true,
    url: '',
    views: {
        'header': {
            templateUrl: 'static/layout/header.html',
            controller: 'HeaderController as vm'
        },
        'sidebar': {
            templateUrl: 'static/layout/sidebar.html',
            controller: 'SidebarController as vm'
        },
        'breadcrumb': {
            templateUrl: 'static/layout/breadcrumb.html',
            controller: 'BreadcrumbController as vm'
        },
        'footer': {
            templateUrl: 'static/layout/footer.html',
            controller: 'FooterController as vm'
        }
    }
});
```

眼尖的同学可能注意到了，root这个state被定义成了抽象。之所以定义成抽象的原因是，我们并不直接使用root这个state（抽象的state不能被直接激活），这个state放在这里纯粹是为了继承用的。说到继承，有3个方面：

* 子state可以沿用父state的View模板定义，通常公用的模块在大多数子state中都是不变的，即使用同样的template和controller，这些模块就可以直接定义在父state中。
* 子state可以覆盖父state的View模块定义，公用的模块在某些页面可能需要不同的template或controller，这时就需要覆盖定义。
* 子state可以添加新的View模块定义，自己独有的模块或非公用的模块需要自己添加。

回到刚才那个截图上，这个页面是用户登录后的dashboard页面，显然并不是所有的页面都需要显示这5个模块，比如主页可能不需要Sidebar和Breadcrumb模块，未登录之前的其他页面也不需要显示Sidebar模块。那么主页home的state可以按如下代码定义：

``` javascript home.route.js
$stateProvider.state('root.home', {
    url: '/',
    views: {
        'sidebar@': {},
        'breadcrumb@': {},
        'main@': {
            templateUrl: 'static/home/home.html'
        }
    }
});
```

注意state的名字，`parent.child`这种命名方式可以让ui-router知道state之间的继承关系。上面直接将Sidebar和Breadcrumb的模块赋空值，则这两个模块就不再显示了，而其他的模块如Header和Footer，则可以继承父state的定义，正常的显示。另外，我们在root这个state里没有定义Main的View模块，这是因为每个state的Main模块肯定是不一样的，所以属于非公用的模块，需要每个子state中自己来定义，可以看到上面的home state里自己定义了Main模块。

还有两点也值得一提，一个是root state的`url`为空，这是因为子state的url会自动追加在父state的url上，则home这个state的最终url为`'' + '/'`。另外一个，注意到在子state中定义View时，ui-view的名字后面都加了`@`符号，这是ui-router约定俗成的写法：`viewname@statename`，称为“View绝对定位”。这里省略了后面的statename，代表的就是index页面定义的`ui-view="viewname"`。当然，有“View绝对定位”，自然就有“View相对定位”，相对定位的方法就是只写一个viewname，此时这个View指的就是父state中指定的template里对应的`ui-view=viewname`。关于相对和绝对定位，可以看[官方WIKI中的介绍](https://github.com/angular-ui/ui-router/wiki/Multiple-Named-Views#view-names---relative-vs-absolute-names)。

到这里大致的布局架子就搭起来了，每个页面可以定义一个自己的state继承自root，覆盖或添加新的View模块，同样子state还可以派生出另外的子state，非常灵活，整个布局结构也非常清晰。

### ui-router带来的其他便利

除了简单的布局，ui-router还支持自定义data，这个data也是可以随state进行继承的。这就给了我们很多畅想，通过把一些配置放在这个data里可以实现很多有意思的应用。下面就简单讲几个[generator-aio-angular](https://github.com/PinkyJie/generator-aio-angular)里的应用场景。

#### 页面的title和class

单页面应用必然要解决的一个问题就是页面的title需要跟着不同的state进行变化，这就需要每次进行state切换时页面的标题需要跟着变化。而class的变化是同样的意思，我希望每次state变化时都能在页面的body标签上加一个顶级的class样式，这样同样的一个Header的ui-view，我就可以定义其在home页面和在dashboard页面不同的样式了。通常在index页面中需要这么写：

``` jade index.jade
head
    title(ng-bind="title")
body(ng-class="_class + '-page'")
```

显然，这两个变量`title`和`_class`都需要定义在`$rootScope`上面，以前我都是在controller里做这个赋值的，但语义上讲它又不太属于controller的职责。有了ui-router的自定义data，可以把这两个变量放在state的config当中了：

``` javascript home.route.js
$stateProvider.state('root.home', {
    data: {
        title: 'Home',
        _class: 'home'
    }
});
```

然后，在`$stateChangeSuccess`的事件响应中统一来将新state中的这两个变量赋值到`$rootScope`上去：

``` javascript
$rootScope.$on('$stateChangeSuccess',
    function (event, toState, toParams, fromState, fromParams) {
        $rootScope.title = toState.data.title;
        $rootScope._class = toState.data._class;
    }
);
```

这样做代码的职责更加明确，集中在一个地方处理title和class的变动，也比分散在不同的controller中更加简洁，更有优势。前面也有说到，data里的变量是可以随state进行继承的，倘若一个子state想和父state的title保持一致，那就不用再次定义了。

#### Breadcrumb和Sidebar导航的实现

说到导航，那肯定是与state有关的，与state有关的东西我们都可以尝试放到state的config定义中去。拿dashboard这个状态的定义来举例：

``` javascript dashboard.route.js
$stateProvider.state('root.dashboard', {
    sidebar: {
        icon: 'mdi-view-dashboard',
        text: 'Dashboard'
    },
    breadcrumb: 'Dashboard'
});
```

想要什么值就可以放什么值，上面的例子中我们放了要显示的文本和显示的图标样式。还要注意一点，这里我们并没有将这个两项定义在data之中，而是直接定义在config对象里。这是因为data是可继承的，而这两种导航我们显然不希望子状态将其继承过去。

定义好了之后怎么使用呢？在BreadcrumbController和SidebarController中我们分别来处理这些逻辑。先来看看BreadcrumbController的实现：

``` javascript breadcrumb.controller.js
function init () {
    _applyNewBreadcrumb($state.current, $state.params);
    $rootScope.$on('$stateChangeSuccess',
        function (event, toState, toParams, fromState, fromParams) {
            _applyNewBreadcrumb(toState, toParams);
        });
}

function _applyNewBreadcrumb (state, params) {
    vm.breadcrumbs = [];
    var name = state.name;
    var stateNames = _getAncestorStates(name);
    stateNames.forEach(function (name) {
        var stateConfig = $state.get(name);
        var breadcrumb = {
            link: name,
            text: stateConfig.breadcrumb
        };
        if (params) {
            breadcrumb.link = name + '(' + JSON.stringify(params) + ')';
        }
        vm.breadcrumbs.push(breadcrumb);
    });
}

function _getAncestorStates (stateName) {
    var ancestors = [];
    var pieces = stateName.split('.');
    if (pieces.length > 1) {
        for (var i = 1; i < pieces.length; i++) {
            var name = pieces.slice(0, i + 1);
            ancestors.push(name.join('.'));
        }
    }
    return ancestors;
}
```

我们需要在两个地方处理面包屑导航，一个是面包屑这块区域第一次初始化的时候，需要应用当前的state，另外当每次state发生变化时还需要应用最新的state。主要的逻辑都放在`_applyNewBreadcrumb`这个函数中实现，它先拿到自己的所有父state，然后从包括自身在内的所有state的config定义中提取信息，组装成breadcrumb对象最终应用于模板上。同样，SidebarController的实现与之类似，不同的是Sidebar只需要在第一次初始化时应用state即可：

``` javascript sidebar.controller.js
function init () {
    // generate sidebar nav menus
    vm.navs = _getNavMenus();
}

function _getNavMenus () {
    var navs = [];
    var allStates = routerHelper.getStates();
    allStates.forEach(function (state) {
        if (state.sidebar) {
            var nav = state.sidebar;
            nav.link = state.name;
            navs.push(nav);
        }
    });
    return navs;
}
```

可以看到，初始化时利用`routerHelper.getStates()`（即`$state.get()`）来拿到所有的state，然后将所有定义了sidebar的state应用于模板上。

有了这些自定义的数据，导航的实现是不是非常的简洁呢？

#### 简单的authentication和authorization

传统的登录验证（authentication）和权限管理（authorization）都是通过route的resolve来实现的，有了ui-router，它们的实现还可以更简洁。直接来看一个例子，dashboard页面必须要登录以后才可以访问，那么它的state可以这样定义：

``` javascript dashboard.route.js
routerHelper.configureStates(getStates());
function getStates () {
    return [
        {
            state: 'root.dashboard',
            config: {
                data: {
                    requireLogin: true
                }
            }
        }
    ];
}
```

我们看到，直接在data里定义了一个自定义的属性`requireLogin`即可，是不是既简单又非常具有可读性。显然，ui-router自己并不知道这个requireLogin的属性应该怎么处理，所以这里我们并没有直接调用`$stateProvider.state()`来定义state，而是封装了一个自己的`configureStates`函数（参考了[John Papa的实现](https://github.com/johnpapa/generator-hottowel)）。在这个函数中我们来处理requireLogin属性：

``` javascript route-helper.provider.js
function configureStates (states) {
    states.forEach(function (state) {
        // add login check if requireLogin is true
        var data = state.config.data;
        if (data && data.requireLogin === true) {
            state.config.resolve = angular.extend(
                state.config.resolve || {},
                {'loginResolve': resolve.login}
            );
        }
        state.config.resolve =
            angular.extend(state.config.resolve || {}, config.resolveAlways);
        $stateProvider.state(state.state, state.config);
    });
}
```

在这个函数中，我们先判断，如果一个state定义了requireLogin属性，则给它添加一个loginResolve检查它是否已经正常登录了。另外，有了这个自定义的函数，我们还可以为所有state添加统一的resolve（11、12行）。route-helper.provider.js的完整实现可以参见[generator-aio-angular项目](https://github.com/PinkyJie/generator-aio-angular/blob/master/app/templates/client/source/app/helper/router-helper.provider.js)，[loginResolve的实现](https://github.com/PinkyJie/generator-aio-angular/blob/master/app/templates/client/source/app/core/resolve.service.js)也在里面哟。

同样，依葫芦画瓢，授权同样可以通过添加类似`onlyAllowed: 'admin'`等类似的定义来实现。这里就不多谈了，值得一提的是resolve本身就是可以被子state继承的，这也是非常符合逻辑的，如果父state需要权限，那么子state也必然需要。

### 总结

可以看到，ui-router不仅让整个布局结构更加清晰，更加易于理解，还带来了很多其他有意思的便捷特性。它让我们把跟页面路由相关的配置都统一在一个地方管理，使整个程序更加的模块化。还在用老ngRoute的同学不妨一试，根本停不下来啊！
