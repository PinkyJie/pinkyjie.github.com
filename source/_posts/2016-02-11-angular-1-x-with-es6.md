title: ES6与Angular 1.x
categories:
  - 前端开发
tags:
  - AngularJS
  - ES6
  - ESLint
  - angular1-webpack-starter
date: 2016-02-11 21:17:20
---

ES6出来也很长时间了，把ES6应用在Angular 1.x的文章也不少，有了`class`这个语法糖Angular里的很多东西都可以写的比较规范了，但很多文章非要把Angular里的所有概念都写成class，这我就觉得没啥必要了。这篇文章就谈谈我自己在ES6和Angular 1.x上的一些实践。

<!--more-->

### Angular中的那些概念放在ES6中怎么写

#### controller

controller的作用是把一些变量和函数绑定在`$scope`上，但controllerAs的出现，改变了controller对`$scope`的强依赖（除非需要绑定事件），使得变量和函数可以绑定在`this`上了（如果对controllerAs不熟悉的可以看[这篇文章](2015/02/09/controller-as-vs-scope/)），这使得我们可以轻易的将controller写成class。一个controller肯定少不了依赖其他的service，在ES6中这些依赖自然是通过controller的构造函数传进来了。来看一个简单的例子（完整实现看[这里](https://github.com/PinkyJie/angular1-webpack-starter/blob/master/source/app/components/login-form/login-form.controller.js)）：

``` javascript
class LoginFormController {
    constructor (UserAPI, $state) {
        this.UserAPI = UserAPI;
        this.$state = $state;
    }

    login (credential) {
        if (this.loginForm.$invalid) {
            return;
        }
        this.UserAPI.login(credential.email, credential.password)
            .then(() => {...})
            .catch(() => {...});
    }
}
```

可以看到controller的构造函数首先要做的就是把这些service依赖全部挂在`this`上（line 3-4 ），这样一来，在别的函数中才能够使用这些service（line 11）。另外，ES6中的箭头函数也可以让我们写起匿名函数来更加的方便，不用每次都敲function这个关键字了（line 12-13）。从上面看，controller的定义就是一个普通的class，与Angular框架本身分开了，需要这个controller时直接调用`.controller(LoginFormController.name, LoginFormController)`即可。这给单元测试带来了非常大的方便，不用再去费劲mock一堆Angular自身的东西，只要单独的去测这个class即可。


另外还有一个问题也值得一说：代码压缩后参数改名导致依赖注入失效的情况。尽管有很多插件可以在build的时候自动做这个事，但我还是比较喜欢自己手写的。ES5中常用这样的写法：

``` javascript
.controller('LoginFormController', ['UserAPI', '$state', function(UserAPI, $state) {}])
```

这种称为“inline array annotation”的写法将controller的定义与实现绑在一起，直接套用在class定义上显得不太合适。我们需要将controller的依赖直接反映在controller的class定义上，这就需要`$inject`出马了。看下面这个例子：

``` javascript
LoginFormController.$inject = ['UserAPI', '$state'];
```

我们可以直接把这句话写在class定义的下面，一目了然。关于`$inject`，可以看[官方文档](https://docs.angularjs.org/guide/di#-inject-property-annotation)里的讲解。

#### service/factory

service就是天生的class，在ES5中我们也是直接把它作为构造函数来进行类比的。service的写法与上面的controller是类似的，看这个例子（完整实现看[这里](https://github.com/PinkyJie/angular1-webpack-starter/blob/master/source/app/pages/phone/phone.service.js)）：

``` javascript
class PhoneService {
    constructor ($http, $q, AjaxError) {
        this.$http = $http;
        this.$q = $q;
        this.AjaxError = AjaxError;
    }

    getPhones () {
    }

    getPhoneDetail (id) {
    }

    addNewPhone (phone) {
    }

    updatePhone (id, phone) {
    }

    removePhone (id) {
    }
}

PhoneService.$inject = ['$http', '$q', 'AjaxErrorHandler'];
```

和controller的套路是完全一致的。说完了service来说说factory，我们知道factory和service的区别就是它的定义需要返回一个对象。如果拿构造函数来类比service的话，那么factory就是一个需要返回一个普通对象的函数，也就是说，在真正实例化的时候，内部是类似这样的实现：

``` javascript
var a = factoryA();
var b = new serviceB()
```

从这个意义上来讲，factory它本质上就不是一个类，虽然有很多文章用一些根本不直观的方法在ES6中把factory封装成class，我认为是根本没必要的。更进一步讲，我认为没必要使用factory，我没有觉得哪个场景是必须使用factory而不能使用service的，两者在除了定义时的方式不同，使用起来没有任何的区别。所以我个人的建议是在ES6中统统使用service。

#### provider

provider与service的区别是，它有一个`$get()`函数，这个函数会返回一个service的实例（即`new serviceB()`过后的实例）。除此之外，它还提供一些函数用来配置这个返回的service。显然，它也是一个class，一个必须带有`$get()`方法的class。我们来看一个provider的例子（完整实现看[这里](https://github.com/PinkyJie/angular1-webpack-starter/blob/master/source/app/components/_common/services/router-helper.provider.js)）：

``` javascript
class RouterHelperProvider {
    constructor ($locationProvider, $stateProvider, $urlRouterProvider) {
        this.$locationProvider = $locationProvider;
        this.$stateProvider = $stateProvider;
        this.$urlRouterProvider = $urlRouterProvider;

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

这时一个典型的provider的实现：

* 它依赖其他三个provider，通过构造函数传入。
* 它内部有一个`config`对象用来存储一些配置。
* 它有一个`configure()`函数暴露给外面，这样在app的config阶段可以传入object进行配置。
* 它有一个`$get()`函数，这个函数返回一个实例，这个实例是由另一个称为`RouterHelper`的class定义好的，并且在初始化这个实例的时候，将自己的config对象传入，这样config对象就可以应用在实例化后的结果上了。

同样我们要使用`$inject`来处理依赖注入的问题，但这里有一点不一样的地方，对于`$get()`函数我们也要处理依赖注入，因为真正返回的service实例的依赖是通过这里传入的，而不是通过`RouterHelper`这个class定义时候注入的，所以在第25行我们需要使用`prototype.$get`去拿到这个函数，然后定义上面的`$inject`属性。然后在定义`RouterHelper`这个class时我们不需要进行依赖注入，因为它的构造函数里的参数都是`$get()`函数里传给它的。

``` javascript
class RouterHelper {
    constructor (config, $stateProvider, $urlRouterProvider,
        $rootScope, $state, Logger, Resolve) {
        ...
    }
}
```

进一步，这个class根本没必要暴露给外面，因为Angular框架到时调用的是provider里的`$get()`来生成这个实例的，而不是直接调用这个class，我们只需要将provider的定义暴露给外面即可，最终在定义这个provider时只需要调用`.provider('RouterHelper', RouterHelperProvider)`。这样一句，我们便同时拥有了`RouterHelperProvider`这个provider和`RouterHelper`这个service，两者分别在config阶段和run阶段进行注入，这是由Angular框架自己来保证的。

#### directive

directive是返回一个键值对配置的函数，显然，它也不是一个class，同样没必要花心思把它封装成class。来看一个例子（完整实现看[这里](https://github.com/PinkyJie/angular1-webpack-starter/blob/master/source/app/components/phone-form/phone-form.directive.js)）：

``` javascript
import PhoneFormController from './phone-form.controller';
import phoneFormHtml from './phone-form.jade';

function PhoneFormDirective () {
    return {
        restrict: 'AE',
        scope: {},
        controller: PhoneFormController.name,
        controllerAs: 'form',
        bindToController: {
            phone: '=',
            state: '@',
            submit: '&',
            cancel: '&'
        },
        template: phoneFormHtml
    };
}
```

directive里的逻辑可以放在它的controller里实现，可以看到它也是支持controllerAs语法的。注意这里的`bindToController`选项，它是1.3引入的用来解决[传入`=`绑定时属性不更新的问题](http://blog.thoughtram.io/angularjs/2015/01/02/exploring-angular-1.3-bindToController.html)。很多人对它的认识是传一个true即可，其实从1.4开始它是支持传入一个对象的，这个对象和传给`scope`配置的对象是一样的，用来指定directive的属性是什么绑定的。这样，`scope: {}`配置只需要传空就可以。这样的写法更加直观，因为这些属性本来就是绑定在controller的`this`上的，而不是`scope`上，这样写更有意义。

当然，如果使用link而不使用controller的话，可以看这个例子（完整实现看[这里](https://github.com/PinkyJie/angular1-webpack-starter/blob/master/source/app/components/_common/directives/datepicker-init.directive.js)）：

``` javascript
function DatepickerInitDirective () {
    return {
        require: 'ngModel',
        restrict: 'A',
        link
    };

    function link (scope, element, attrs, ngModelCtrl) {
        ...
    }
}

DatepickerInitDirective.$inject = [];
```

注意这种情况下需要处理依赖注入，而使用controller的话，依赖注入在controller上解决即可。另外，这里的第5行使用的是ES6中的新语法，即如果对象的key和value名字一样的话，可以使用只写key的简写形式。

### ES6的小tip

最后讲一些写ES6时的tip，有些问题可能在ES5中不是问题，但ES6中反而成了闹心的问题了。

#### 闹心的构造函数参数赋值

可以看到，上面所有class定义的构造函数中，第一步都是要将传入的参数全部赋值到`this`上来。既然这是一种必需，那有没有简单的方法来处理这种繁琐的写法呢？答案是肯定有的哇！那就是ES6中新引入的`Object.assign`方法，比如：

``` javascript
// method 1
Object.assign(this, {UserAPI, $state});
// method 2
this.UserAPI = UserAPI;
this.$state = $state;
```

上面两种写法是等价的。`Object.assign`会把第二个参数传入的对象属性赋值在第一个参数上。另外，这里同样使用了上面提到的object简写方法，因为key和value是一样的。

#### 能用const就用const

ES6中引入了`let`和`const`关键字来定义变量和常量，根据个人经验，90%以上的变量定义其实都是常量。这里的常量说的是变量本身，比如一个object是常量，你照样是可以修改这个object上面的属性的。

#### this不是万能的

箭头函数可以保持上下文this不变，这样可以让我们少写很多`const self = this;`这样的语句。但注意，只有箭头函数能达到这样的效果，在很多别的地方还是需要把this赋值留到后面待用的。比如：

``` javascript
class UserSerivce {

    getProductSummary () {
        const self = this;
        return this.$http.get('api/user/products')
            .then(_success)
            .catch(this.AjaxError.catcher.bind(this.AjaxError));

        function _success (response) {
            const data = response.data;
            if (response.status === 200 && data.code === 0) {
                return data.result.summary;
            }
            return self.$q.reject(data.message);
        }
    }
}
```

可以看到，在class的一个方法中定义函数，这里的this是无法保持的。所以在方法中调用函数需要传入其他函数时，要么通过提前保存this这种做法，要么就要使用bind函数来手动的改变context。

#### 私有变量/函数的实现

ES6的语法中并没有给出私有变量或函数的实现，没有了private关键字，class里不同方法需要共享变量就必须把变量绑定在this上，而一旦绑在this上，外部就可以访问，这还是有些不便的。一种很自然的想法是，把不想让外部访问的变量和函数写在class外：

``` javascript
let _foo;
function _bar () {}
class ClassA {}
export default ClassA;
```

上面的变量`_foo`和函数`_bar()`只有本文件里的`ClassA`才能访问的到，由于export出去的只是`ClassA`，外部是没法访问到这两个东西的，这在一定程度上实现了私有。当然，这种实现显得有些强求，不太面向对象。

另一种方法就是借助ES6中的`Symbol()`函数，`Symbol()`函数会生成一个唯一的随机字符串，我们可以用这个随机字符串来当做私有变量和私有函数的名字。看这个例子：

``` javascript
const [foo, bar] = [Symbol(), Symbol()];
class ClassA {
    constructor () {
        this[foo] = 'aaa';
        this[bar] = () => {};
    }
}
export default ClassA;
```

注意第一行使用了ES6中的“赋值展开”。可以看到，变量`foo`和`bar`虽然都定义在this上，但由于外部（本文件以外）都不知道这两个变量代表的字符串是什么，所以外部是无法访问的。这相当于，外部根本不知道私有变量和私有函数的变量名和函数名，自然无法访问的到了，这种实现方法更加面向对象一点。

#### 使用ESLint

在ES5阶段我们有JSHint和JSCS来进行静态检查和规范代码风格，但两者对于ES6的支持都不是那么的完善。而[ESLint](http://eslint.org/)对ES6的支持非常完善，你甚至可以指定只用ES6的代码风格来写（比如只使用`let`和`const`而不使用`var`）。很多编辑器都有ESLint的[插件](http://eslint.org/docs/user-guide/integrations)，可以在书写的时候就进行检查和警告。ESLint有[非常多的Rule](http://eslint.org/docs/rules/)可供使用，基本可以涵盖JSHint和JSCS的所有规范。如果你觉得ESLint的Rule比较多，不知道每个都干啥用的，那么我有一个方法：把所有Rule都打开，按自己的风格写代码，一旦你的风格和Rule冲突就会报错，这个时候你再选择是否关掉这个Rule。ESLint可以帮助我们做很多静态检查，比如上面提到的const的问题，一旦ESLint发现你定义了一个`let`变量但没有改变其值，就会提示你将`let`改成`const`。
