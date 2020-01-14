title: Promise，其实我真的不懂你
categories:
  - 前端开发
tags:
  - promise
  - 异步编程
  - AngularJS
  - generator-aio-angular
date: 2015-07-20 22:25:26
---

写过Angular的service的孩子肯定对Promise不陌生了吧，这玩意的出现可以说是极大改变了异步编程的写法，告别了让代码不停横向发展的[Callback Hell](http://callbackhell.com/)。一直以来自己都是按照网上流行的写法来写Promise的，并没有觉得有什么问题，直到看到这篇神一般的文章：[We have a problem with promises](http://pouchdb.com/2015/05/18/we-have-a-problem-with-promises.html)，以及文中提到的Bluebird的Wiki中关于[Promise的Anti-pattern](https://github.com/petkaantonov/bluebird/wiki/Promise-anti-patterns#the-deferred-anti-pattern)，顿时有一种醍醐灌顶的感觉，敢情自己以前根本就没理解Promise啊。这篇文章就来讲讲最近在Promise上的心得体会。

<!--more-->

开始前先放两张Callback Hell的搞笑图吧，虽然不是我们今天的主题，但是太搞笑啦！哈哈哈！
> ![图1](/assets/images/promise-actually-i-really-do-not-know-you-1.png)
>
> ![图2](/assets/images/promise-actually-i-really-do-not-know-you-2.png)

### Promise应该这么写

先来看一下原先我是怎么写controller和service里的Promise的。

``` javascirpt service-old.js
function login (email, password) {
    var d = $q.defer();
    var req = {
        email: email,
        password: password
    };
    $http.post('api/user/login', req)
        .success(_success)
        .error(_fail);
    return d.promise;

    function _success (response, status) {
        if (status === 200 && response.code === 0) {
            d.resolve(response.result.user);
        } else {
            _clearUser();
            d.reject(response.message);
        }
    }

    function _fail () {
        _clearUser();
        d.reject('$SERVER');
    }
}
```

``` javascirpt controller-old.js
userAPI.login(credential.email, credential.password)
    .then(_success, _error);

function _success (data) {
    // xxx
}

function _error (reason) {
    // xxx
}
```
乍一看没啥问题啊，就应该这么写的哇。先放着不管，我们再看看更改后的写法，有对比才有发现嘛：

``` javascirpt service-new.js
function login (email, password) {
    var req = {
        email: email,
        password: password
    };
    return $http.post('api/user/login', req)
        .then(_success)
        .catch(_fail);

    function _success (response) {
        var data = response.data;
        if (response.status === 200 && data.code === 0) {
            return data.result.user;
        } else {
            return $q.reject(data.message);
        }
    }

    function _fail (reason) {
        _clearUser();
        return $q.reject(reason || '$SERVER');
    }
}
```

``` javascirpt controller-new.js
userAPI.login(credential.email, credential.password)
    .then(_success)
    .catch(_error);

function _success (data) {
    // xxx
}

function _error (reason) {
    // xxx
}
```

看出区别了吗？我来捋一捋：
* 抛弃了`$q.defer()`，也就是不使用deferred对象
* 不使用then的第二个参数，而是直接`then().catch()`
* 文件`service-old.js`的第15行的那句`_clearUser()`在`service-new.js`中不见了，而且`_fail`函数的写法咋还改了呢
* 抛弃了$http的success和error方法

下面我们就一个一个来解释！

### 为啥不要用Deferred对象？

其实这个问题可以换个方式问：为啥要用Deferred对象？不靠谱版回答：你傻啊，因为很多教程里就是这么用的笨蛋！靠谱版回答：因为我们要指明这个异步API调用返回的两种状态啊，成功返回就resolve掉这个Promise，失败就reject掉。但你没有想过，`$http.post()`本来就是一个Promise，为啥还要用`$q.defer()`构造另一个Promise来再包一层呢？换句话讲，人家`$http.post()`本来就知道什么时候应该resolve，什么时候应该reject，你Deferred对象瞎操心啥玩意！人家`$http.post()`返回的本来就是一个Promise对象，何必返回`deferred.promise`这另一个Promise对象呢。那没有deferred对象怎么resolve和reject呢？很简单，**想要resolve就直接return你想resolve的值，想要reject就直接调用`$q.reject()`**。这就是为什么在`service-new.js`的第13行我们直接return想要返回的值。那这个值controller可以正常的接受到吗？Promise的链式调用保证了这种机制：

``` javascript
promise().then(handlerB).then(handlerC);
```

这里hanlderB会在promise结束或执行，并且参数是promise的return值，即`handlerB(returnValueOfPromise)`，同样，handlerC也是一样，即`handlerC(returnValueOfHandlerB)`。也就是说，**Promise链式调用始终返回上一个Promise中return的值**。Angular中的interceptor正是利用这个机制工作的。

那么有人要问了，什么时候才需要用Deferred对象呢？前面说了，deferred对象是又包了一层Promise，那显然，里面如果是非Promise的异步调用，用它包住就成为正常的Promise了！比如`setTimeout`：

``` javascript
function asyncGreet(name) {
    var deferred = $q.defer();
    setTimeout(function() {
        if (okToGreet(name)) {
            deferred.resolve('Hello, ' + name + '!');
        } else {
            deferred.reject('Greeting ' + name + ' is not allowed.');
        }
    }, 1000);
    return deferred.promise;
}
```
这个来自官方文档的例子就很好说明了问题。Deferred对象的意义在于将非Promise的异步流程包装成Promise。

### 为啥不让then里面传俩参数？

文档里说了，`catch(errorCallback) – shorthand for promise.then(null, errorCallback)`，你这`.then(callback1).catch(callback2)`就是一个简写嘛，我用`then(callback1, callback2)`为啥就不推荐呢？这里我要说：文档里面写错了！两种写法并不是完全等价的！区别就在于：
* `promise.then(callback1, callback2)`中，如果callback1抛错，callback2无法捕捉到。也就是说执行了callback1，就不会执行callback2。只有前面的promise抛错时才会进入callback2。
* 而`promise.then(callback1).catch(callback2)`中，如果callback1抛错，则callback2可以捕捉到。当然，promise如果抛错的话callback2自然也会被执行。那就是说，存在一种可能性，callback1和callback2都会被执行，那就是promise没抛错，但callback1抛错了。而这种情形常见吗？回答是非常常见，就拿我们上面的例子，server虽然正常返回了(promise没抛错)，但正常的返回中检查出code不等于0，这个时候我们就需要“抛错”。这里的抛错加了引号，因为这里的“抛错”不只包含显式的`throw`出一个错，而且包括reject。这样的好处非常明显，我的callback2不需要care这个“错”是上面哪个Promise（注意，因为`then()`同样返回一个新的Promise）抛出来的，是第一个promise中API请求因为网络原因（非200）挂了，还是第二个promise中发现账号密码不正确，这些我都不管，我希望在callback2中统一的处理这些异常。这也解释了我们为什么在`service-new.js`中的第15行删掉了`_clearUser()`调用，因为我们在`_success`中的分支中调用了`return $q.reject(data.message)`，这样就会进入后面的`_fail`函数，而里面有调用`_clearUser()`就可以了。并且，我们还给`_fail`加了参数，这个参数就用来接受callback1中`$q.reject()`传递过来的错误原因。

### $http的success和error方法咋不能用啊？

为啥不让用这个两个方便的函数呢，你看，它直接把`response`对象给你拆成四个参数，多方便啊！说实话，我也挺喜欢这个便利性的，但如果我们尝试将`service-new.js`中第7、8两行的`.then().catch()`替换成Angular的$http给我们提供的`.success().error()`会怎么样呢？嗒嗒！发现service是正常工作的，但是controller不正常了。在`controller-new.js`中的`_success`函数中，参数`data`并不是我们预想的在`service-new.js`中第13行返回的值，而是和`service-new.js`中`_success`函数的参数`response`一致。我香蕉你个疤瘌，博主，我读书少你怎么能骗我！你刚才不是说Promise在链式调用的时候总是拿到上一个Promise的返回值嘛，我controller里的`userAPI.login()`后面的then拿到的为什么不是service里`login`函数中返回的Promise的返回值呢！

我们再来捋一捋啊，倘若把service和controller写到一起，这个链式调用应该是这样的：`$http.post(xxx).success(_successInService).error(_fail).then(_successInController)`，这里最后的then里拿到的参数是第一个Promise，即`$http.post(xxx)`的返回结果。这是为什么呢？并不是博主骗人，问题就出在这个`success`和`error`上。我们可以看看这两个函数的实现。

``` javascript
promise.success = function(fn) {
    promise.then(function(response) {
        fn(response.data, response.status, response.headers, config);
    });
    return promise;
};
```

可以发现，success和error的实现并不符合Promise的标准，它们并没有返回一个新的Promise（`then(...)`会生成的新Promise），而是将前面原来的Promise给返回了！所以我们拿到的才会是第一个Promise的返回值！这就是我不建议使用success和error这两个函数的原因，不光是我，官方也注意到了，可以参看[Github上面Angular的这个issue](https://github.com/angular/angular.js/issues/10508)，有人表示后面的版本将考虑废弃这两个不正规的函数。

### 补充问题：以前的`deferred.resolve()`应该怎么整呢？

这个场景也是比较常见的，有些情况下，我们只是想resolve一个Promise，并不想具体的返回什么值。比如，一个API去请求用户的session还在不在，在的话返回0，不在的话返回1。我们的逻辑就是在的话我不管，不在的话才有所行动，比如登出用户什么的。这个时候我们就对`then(callback1).catch(callback2)`里的callback1不管兴趣了，那还需要手动的去写一句`return '';`或`return;`吗？答案是否定的，这种情形只需要写`.catch()`就好了，直接删掉`then()`。

### 还有问题？

在提别的问题之前，还是强烈建议阅读开篇推荐的那篇[We have a problem with promises](http://pouchdb.com/2015/05/18/we-have-a-problem-with-promises.html)，里面非常详细的讲解了各种关于Promise的误区！用醍醐灌顶来形容一点也不为过。

博主这整篇光列几行代码，也没有可运行的东西给看看有啥意思啊！有！完整的程序可以看我写的一个generator：[generator-aio-angular](https://github.com/PinkyJie/generator-aio-angular)。也可以看看我如何将原来的写法重构成推荐的写法，都在[这个commit](https://github.com/PinkyJie/generator-aio-angular/commit/18de9f01f73eff138ea1c42a08444e72b03bdfbb)里。
