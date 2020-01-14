title: 谈谈单元测试中的关注分离
categories:
  - 前端开发
tags:
  - AngularJS
  - 单元测试
  - 关注分离
  - jasmine
  - karma-coverage
  - isparta
  - ES6
  - angular1-webpack-starter
date: 2016-02-20 12:34:34
---

前端的单元测试越来越受到重视，网上也有很多讲解Angular中如何写好单元测试的文章，我自己在最近的[angular1-webpack-starter](https://github.com/PinkyJie/angular1-webpack-starter)项目中也写了很多单元测试。单元测试的一个核心理念就是对“单元”进行隔离，然后单独测试。可是网上的很多教程存在不少误区：比如在controller的测试中去使用$httpBackend，在引入第三方service的时候使用真实的service等等，说到底都是没有实现“关注分离”（Separation of Concerns），“单元”没有真正的被隔离。这篇文章就谈谈Angular的单元测试中如何更好的实现关注分离。

<!--more-->

### 文件结构上的隔离

好的实践应该是Angular中一个单独的controller/service/provider/directive对应一个单独的spec文件，这是“大单元”的隔离，而每个单元又是由很多“小单元”（函数）组成的，而“小单元”同样需要隔离，那么每个函数可以对应一个spec文件里的describe块。看下面这个service的例子：

``` javascript
// user.service.js
class UserSerivce {
    constructor ($http, $q, $rootScope, Event, AjaxError) {...}

    checkLoggedInStatus () {...}

    login (email, password) {...}

    logout () {...}
}
UserSerivce.$inject = ['$http', '$q', '$rootScope', 'Event', 'AjaxErrorHandler'];
export default UserSerivce;

// user.service.spec.js
import UserService from './user.service';

describe('User Service', () => {

    beforeEach(...);

    describe('constructor function', () => {...});

    describe('checkLoggedInStatus function', () => {...});

    describe('login function', () => {...});

    describe('logout function', () => {...});
});
```

可以看到在测试的spec文件中，一个“大单元”对应顶层的describe，而“小单元”也有自己对应的子describe，这就从文件结构上保证了单元的隔离。当然这只是表面功夫，要做到更好的关注分离，我觉得要做到以下几点：

* 让测试尽量脱离Angular框架本身。
* 能mock的依赖全mock，而且只mock需要直接依赖的部分。
* 对于外部依赖和内部依赖（比如controller的一个函数调用自己的另一个函数），直接spy。

下面就以controller/service/provider的测试为例，来讲讲如何贯彻上面几点。需要说明的是，下面的代码都是基于jasmine和ES6的，但是一些思路和想法也同样适用于ES5。

### 简单controller/service的测试可以脱离框架本身

对于很多简单的controller和service，它不依赖Angular本身的特殊service（如`$rootScope`，`$http`等等），这个时候我们就可以甩开Angular测试里的`angular.mock.module`和`angular.mock.inject`这类框架特有的函数，直接在测试中mock它的构造函数的参数，然后new这个class进行测试即可。我们来看一个例子：

``` javascript
class AjaxErrorHandlerService {
    constructor (Error, $q) {
        Object.assign(this, {Error, $q});
    }
    // directly reject with the human readable error message
    catcher (reason) {
        const type = typeof reason;
        let code = '$UNEXPECTED';
        if (reason) {
            if (type === 'object') {
                code = reason.message;
            } else if (type === 'string') {
                code = reason;
            }
        }
        return this.$q.reject({
            code,
            text: this.Error.getErrorMessage(code)
        });
    }
}
AjaxErrorHandlerService.$inject = ['Error', '$q'];
export default AjaxErrorHandlerService;
```

可以看到这个`AjaxErrorHandlerService`提供了一个统一的函数用来处理所有Ajax请求失败的情况，它有两个依赖：定义在其他文件中的名叫`Error`的service，以及Angular自身的`$q`。按照传统的Angular单元测试流程，我们需要这么写：

``` javascript
describe('AjaxErrorHandler Service', () => {
    let ErrorService;
    let $q;
    let ajaxErrorHandler;

    beforeEach(() => {
        angular.mock.module('theModuleContainsThisService');
    });

    beforeEach(() => {
        angular.mock.inject((_Error_, _$q_, _AjaxErrorHandler_) => {
            Error = _Error_;
            $q = _$q_;
            ajaxErrorHandler = _AjaxErrorHandler_;
            spyOn(Error, 'getErrorMessage');
            spyOn($q, 'reject');
        });
    });
    // ...
});
```

在上面的测试中，我们先使用angular-mock模块提供的module函数来加载service所在的module，然后使用inject来引入真实的`Error`和`$q`，spy其中的方法，最后再进行我们自己的测试。这里面的误区有两个：

1. 这个测试根本不需要使用Angular自身的module和inject
2. 没必要引入真实的service依赖

我们来看看修改后的测试(省略describe头)：

``` javascript
import AjaxErrorHandlerService from './ajax-error-handler.service';
// describe ...
beforeEach(() => {
    ErrorService = jasmine.createSpyObj('Error', ['getErrorMessage']);
    $q = jasmine.createSpyObj('$q', ['reject']);
    ajaxErrorHandler = new AjaxErrorHandlerService(ErrorService, $q);
});
```

修改后的测试看起来简洁多了。首先，ES6中service的定义已经是class了，我们可以自行初始化它，没必要使用框架的inject。其次，它的所有依赖我们都可以直接使用mock的object，jasmine的`createSpyObj`可以创建一个mock的object，并且数组里指定的函数都自动被spy了，这也是我们为什么可以直接省略`spyOn()`调用的原因。可以看到，这个测试脱离了Angular框架本身，其次它没有真的引入真实的依赖，而是mock了它们，并且只mock自己需要的那部分函数（中括号里的部分）。这样，单元被彻底的隔离开了：

* 即便以后这段代码用于非Angular框架了，测试依然有效。
* 就算其它的service的实现有问题，也不会影响这个测试。

那有人要问了，我就是想要测A的时候发现A的依赖B有问题，好吧，那就从根本上违背了单元测试的初衷，不叫单元测试！关于`AjaxErrorHandlerService`的完整测试可以看[这里](https://github.com/PinkyJie/angular1-webpack-starter/blob/master/source/app/components/_common/services/ajax-error-handler.service.spec.js)。

还有一个例子也很常见，那就是我们常常需要测试`$rootScope.$on(xxx, xxx)`，也就是需要测试Angular里的事件响应。看下面这个例子：

``` javascript
function foo () {
    $rootScope.$on('$stateChangeSuccess', (event, toState) => { ... });
}
```

当我们要测试这个`foo`函数的时候，我们很自然的想到可以调用`$rootScope.$broadcast('$stateChangeSuccess');`，这样就可以fire一个事件，foo函数里定义的事件回调就可以触发了。这样一来，又出现了需要依赖Angular框架的情况。其实，这种情况下我们依然可以杜绝这种依赖。回到测试本身，其实我们需要测试的是后面的回调函数，因为`$on`这个机制是由框架本身保证的，我们不应该去测这个机制，我们只要保证回调里的逻辑就好了。那么我们可以这么测：

``` javascript
it ('should work', () => {
    foo();
    // 假设前面已经mock一个假的$rootScope
    expect($rootScope.$on).toHaveBeenCalled();
    expect($rootScope.$on.calls.argsFor(0)[0]).toBe('$stateChangeSuccess');
    const callback = $rootScope.$on.calls.argsFor(0)[1];
    callback({...}, {...});
    // expect logic in callback
});
```

上面的测试并没有引入真正的`$rootScope`，它只验证`$on`被调用过（第4行），并且第一个参数是`$stateChangeSuccess`（第5行），然后通过jasmine的`argsFor(0)[1]`拿到callback（第6行），意思就是第1次调用时的第2个参数，显然，第2个参数就是我们要测的回调函数本身。拿到了回调函数，我们只要给定参数执行它，然后在expect里面的一些逻辑即可。在这个例子里，我们再次做到了关注分离，我们的关注点只放在了自己实现的回调上，而框架本身的事件机制我们选择忽略。

当然，有些情况下我们无法脱离Angular框架，必须要引入真实的service，我总结了一下大概有以下几种情况：

* 测试directive时需要`$compile`和`$rootScope`：
    * 用`$compile`来编译含有directive的HTML代码。
    * 用`$rootScope.$new()`来生成directive的scope（用于link函数里）。
* 测含有HTTP请求的service时需要使用`$httpBackend`：使用`$httpBackend.expectXXX(xxx)`来模拟HTTP请求的返回。
* 测promise的时候需要`$q`和`$rootScope`：（下一节会详细讲到）
    * 使用`const deferred = $q.defer()`来生成一个deferred的对象，然后在需要promise的地方使用`deferred.promise`代替。
    * 使用`$rootScope.$digest()`来使promise的结果生效。

### controller的测试里不要出现$httpBackend，测service时才需要它

这是我一开头就提到的问题，我们来看下面这个controller和service：

``` javascript
// user.service.js
class UserSerivce {
    login (email, password) {
        const self = this;
        const req = {
            email,
            password
        };
        return this.$http.post('api/user/login', req)
            .then((response) => {
                const data = response.data;
                if (response.status === 200 && data.code === 0) {
                    return data.result.user;
                }
                return self.$q.reject(data.message);
            })
            .catch((reason) => {
                return self.$q.reject(reason);
            });
    }
}
// login.controller.js
class LoginController {
    login (credential) {
        this.User.login(credential.email, credential.password)
            .then(...)
            .catch(...);
    }
}
```

在测试这个controller时，很多文章直接这样写：

``` javascript
$httpBackend.expectPOST('api/user/login').respond({code: 0, result: {user: 'user'}});
controller.login(...);
$httpBackend.flush();
expect(...) // check logic in controller then branch
```

然后到了测试service时，还是这么写：

``` javascript
$httpBackend.expectPOST('api/user/login').respond({code: 0, result: {user: 'user'}});
UserService.login(...);
$httpBackend.flush();
expect(...) // check logic in service then branch
```

请问有什么区别吗？service这么写无可厚非，因为它依赖`$http`，所以测试的时候拿`$httpBackend`去mock是合理的，可是controller的直接依赖是service而不是`$http`，为什么也要这么mock呢？这就有点越俎代庖的意思，你不mock你的直接依赖，而是去mock你的依赖的依赖。显然，这不是一种好的隔离。

那么回到controller来说，它的直接依赖是`User.login`这个函数，那么我们只需要去mock这个函数就好，那么这个函数返回的是什么呢？没错，是一个promise。那么测controller的时候我们只需要去mock这个promise就好了，看下面的代码（变量定义已省略，完整的代码见[这里](https://github.com/PinkyJie/angular1-webpack-starter/blob/master/source/app/components/login-form/login-form.controller.spec.js)）：

``` javascript
beforeEach(() => {
    angular.mock.inject((_$q_, _$rootScope_) => {
        $q = _$q_;
        $rootScope = _$rootScope_;
        UserAPI = jasmine.createSpyObj('UserAPI', ['login']);
        controller = new LoginController(UserAPI);
    });
});

describe('login function', () => {
    beforeEach(() => {
        deferred = $q.defer();
        UserAPI.login.and.returnValue(deferred.promise);
    });

    it('should success', () => {
        deferred.resolve(...);
        controller.login(...);
        $rootScope.$digest();
        expect(...) // check logic in then branch
    });
});
```

上面的测试中引入了真实的`$q`和`$rootScope`，这就是我们上一小节提到的测试promise时需要的。第12、13行我们使用`$q`构造一个promise对象，让mock后的login函数返回这个promise，然后在后面的测试中，我们直接控制这个promise是resolve还是reject即可（第17行）。注意，第19行的`$rootScope.$digest()`是框架本身的要求，我们需要这行代码让promise的结果生效。这样一来，controller的测试就与它背后的HTTP请求隔离开了，因为它只关心promise的resolve或reject，只有service才需要去关注HTTP的返回。

这里顺带一提，测试service的时候需要让`$httpBackend`模拟返回3个结果：

``` javascript
apiResponse.respond({code: 0, result: {user: 'user'}});
apiResponse.respond({code: 1, message: 'error'});
apiResponse.respond(() => {return [500];});
```

这是基于测试覆盖率的考量，因为可以看到service的实现逻辑中，2xx的返回会进入then分支，非2xx的返回进入catch分支，而在then分支中手动执行`$q.reject()`也会进入catch分支。而测试controller时，因为它关心的只是promise，所以只需要模拟resolve和reject两种结果即可。

### 让module/inject引入我们mock过的provider/service

有些情况下，我们必须使用inject来引入一些provider和service，比如在测试provider的时候，因为provider的初始化并不像controller或service那样可以直接new，它是由框架本身来初始化的，并且`.provider('RouterHelper', RouterHelperProvider)`的定义会同时得到一个`RouterHelperProvider`的provider和一个`RouterHelper`的service。所以在测provider的时候我们只能通过module来实现，但即便这样我们依然可以让module/inject加载我们已经mock过的provider和service。我们来看下面这个provider的实现：

``` javascript
class RouterHelperProvider {
    constructor ($locationProvider, $stateProvider, $urlRouterProvider) {
        Object.assign(this, {$locationProvider, $stateProvider, $urlRouterProvider});

        this.config = {
            mainTitle: '',
            resolveAlways: {}
        };
        this.$locationProvider.html5Mode(true);
    }

    configure (cfg) {
        angular.extend(this.config, cfg);
    }

    $get ($rootScope, $state, Logger, Resolve) {
        return new RouterHelper(
            this.config, this.$stateProvider, this.$urlRouterProvider,
            $rootScope, $state, Logger, Resolve);
    }
}

RouterHelperProvider.prototype.$get.$inject = [
    '$rootScope', '$state', 'Logger', 'Resolve'
];

RouterHelperProvider.$inject = ['$locationProvider', '$stateProvider', '$urlRouterProvider'];
```

对于RouterHelperProvider和RouterHelper的依赖我们都需要来mock，要实现这一点其实我们只要**在运行`angular.mock.module('xxx')`之前进行provider的mock**即可：

``` javascript
import RouterHelperProvider from './router-helper.provider';

beforeEach(() => {
    angular.module('test', [])
        .provider('RouterHelper', RouterHelperProvider);
});

beforeEach(() => {
    // function passed to module() does not get called until inject() does it's thing
    angular.mock.module(($provide) => {
        // provider needs to be mocked before module load
        $provide.provider('$location', jasmine.createSpyObj('$locationProvider', ['html5Mode', '$get']));
        $provide.provider('$urlRouter', jasmine.createSpyObj('$urlRouterProvider', ['otherwise', '$get']));
        $provide.provider('$state', jasmine.createSpyObj('$stateProvider', ['state', '$get']));
        $provide.value('$rootScope', jasmine.createSpyObj('$rootScope', ['$on']));
        $provide.value('Logger', jasmine.createSpyObj('Logger', ['warning']));
        $provide.value('Resolve', jasmine.createSpyObj('Resolve', ['login']));
    });
});

beforeEach(() => {
    angular.mock.module('test');
});

beforeEach(() => {
    angular.mock.module((_$locationProvider_, _$stateProvider_,
        _$urlRouterProvider_, _RouterHelperProvider_) => {
        $locationProvider = _$locationProvider_;
        $stateProvider = _$stateProvider_;
        $stateProvider.$get.and.returnValue(jasmine.createSpyObj('$get', ['get', 'go']));
        $urlRouterProvider = _$urlRouterProvider_;
        provider = _RouterHelperProvider_;
    });
});

beforeEach(() => {
    angular.mock.inject((_$rootScope_, _$state_, _Logger_, _Resolve_) => {
        $rootScope = _$rootScope_;
        $state = _$state_;
        Logger = _Logger_;
        Resolve = _Resolve_;
    });
});
```

上面的测试包含5块beforeEach，每一块都有自己的分工：

1. 定义我们自己的provider，我们把它定义在自己的`test`module上而不是真实的module上，这样可以更好的隔离。
2. mock需要的各种provider，我们知道provider是带有`$get`函数的特殊的class，所以我们只需要保证mock后的provider包含`$get`及我们要用的其他函数即可。
3. 加载这个`test`的module。
4. 引入我们mock过的provider依赖，注意第30行有些特殊。我们的目标是mock`$state`这个service里的`get`和`go`函数，但这个`$state`service是我们mock的`$stateProvider`这个provider的`$get`函数生成的，所以我们需要直接mock这个`$get`的返回值，让其继续返回一个可以spy的object。如果我们不这样做，而是直接在inject函数里尝试`spyOn($state, 'get')`的话就会报错，因为这里的`$state`是没有`get`这个函数的。
5. 引入我们mock过的service依赖。

这样，所有的依赖虽然都是通过module/inject引入进来的，但是它依然是我们mock过的。这里的顺序值得注意，对于provider的mock必须在调用`angular.mock.module('test');`（第3个beforeEach块）之前，否则得到的就不是你mock过的provider，而是框架自带的provider。如果是第3方的provider如`$stateProvider`，如果放在module加载之后再mock，加载module时就会报错：

```
Error: [$injector:modulerr] Failed to instantiate module test due to:
Error: [$injector:unpr] Unknown provider: $stateProvider
```

这是因为其实`$stateProvider`根本就不在`test`这个module里，我们必须在加载module之前mock它，所以必须保证自定义provider的其它provider依赖都mock好了才能初始化这个module。但是service的mock就不需要这么做，我们把第15-17行移动到第3个beforeEach后面也是可行的。

### 其它小tip

最后讲一些单元测试里其它的小tip。

#### 整合jquey插件的directive的测试流程

我们先来看一个简单的directive，它的link函数里只做一件事，调用jquery插件的dropdown函数：

``` javascript
function DropdownInitDirective () {
    return {
        restrict: 'A',
        link
    };

    function link (scope, element) {
        element.dropdown();
    }
}
DropdownInitDirective.$inject = [];
export default DropdownInitDirective;
```

再来看看如何测试它：

``` javascript
import DropdownInitDirective from './dropdown-init.directive';

describe('DropdownInit Directive', () => {
    let scope;

    beforeEach(() => {
        angular.module('test', [])
            .directive('aioDropdownInit', DropdownInitDirective);
        angular.mock.module('test');
    });

    beforeEach(() => {
        angular.mock.inject(($rootScope, $compile) => {
            scope = $rootScope.$new();
            spyOn($.fn, 'dropdown').and.callThrough();
            $compile('<a aio-dropdown-init></a>')(scope);
            scope.$digest();
        });
    });

    it('should call dropdown function when initialization', () => {
        expect($.fn.dropdown).toHaveBeenCalled();
    });
});
```

directive的一般测试流程是：

1. 使用$rootScope来生成新的scope作为directive的scope（第14行）
2. 使用$compile来编译HTML代码并绑定scope（第16行）
3. 执行`scope.$diget()`使绑定的scope生效

这里有一点特殊的地方就是第15行，实现代码里是调用的`element.dropdown`，但我们spy的却是jquery的`$.fn.dropdown`，这是因为element元素是$compile后才能返回的（第16行），而一旦返回，它的link函数就已经被立即调用了。所以，在第16行后去执行`spyOn(element, 'dropdown')`已经来不及，而在第16行之前执行呢，这时element还没有dropdown这个函数。所以对`$.fn`进行spy是一个折衷的方法，因为所有的jquey插件最终都是定义在`$.fn`上的。但是这么一来有些违反单元测试的原则，目前我也没想到更好的方案。

#### 将重复的测试代码重构成函数

这一点不用所说，将重用的代码提取出来是编程中最常见的重构。单元测试中也存在很多这样的重复代码，一个最常见的例子就是在测试service的时候，2xx和非2xx的返回都有可能进入reject分支，所以在验证reject分支逻辑的时候就需要写两次，这时就可以把这些重复的代码提出来变成函数。看下面这个例子：

``` javascript
describe('login function', () => {
    let apiResponse;
    beforeEach(() => {
        spyOn(User, '_setUser');
        spyOn(User, '_clearUser');
        apiResponse = $httpBackend.expectPOST('api/user/login');
    });

    function assertError (error) {
        return () => {
            expect(User._setUser).not.toHaveBeenCalled();
            expect($rootScope.$broadcast).not.toHaveBeenCalled();
            expect($q.reject).toHaveBeenCalledWith(error);
            expect(User._clearUser).toHaveBeenCalled();
            expect(AjaxErrorHandler.catcher).toHaveBeenCalledWith(error);
        };
    }

    it('should not login user when API returns error result', () => {
        apiResponse.respond({code: 1, message: 'error'});
        User.login('a', 'b').catch(assertError('error'));
        $httpBackend.flush();
    });

    it('should not login user when API returns 500', () => {
        apiResponse.respond(() => {
            return [500];
        });
        User.login('a', 'b').catch(assertError(null));
        $httpBackend.flush();
    });
});
```

#### 测试覆盖率

一旦开始写测试就一定要加入测试覆盖率的统计，它不光是一个“看自己到底写了百分之多少的测试”的提醒，更重要的是它绝壁是写测试的最大动力。想想游戏里的成就系统，有时为了拿100%物品搜集成就即便通关了也要重来一遍。单元测试里的覆盖率统计也有同样的作用，有的时候不为别的，就想看那个数字跑到100%。ES5中我们有[karma-coverage](https://github.com/karma-runner/karma-coverage)配合[istanbul](https://github.com/gotwarlost/istanbul)，ES6中karma-coverage依然有效，但是我们需要[isparta](https://github.com/douglasduteil/isparta)——一个针对ES6的代码覆盖工具。你需要做的就是把它配置到karma的配置文件中，具体可以参考angular1-webpack-starter项目的[karma.conf.js](https://github.com/PinkyJie/angular1-webpack-starter/blob/master/karma.config.js)。
