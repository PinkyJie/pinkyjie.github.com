title: 使用AngularJS解决IE 8/9不支持placeholder的问题
date: 2014-08-06 21:23:25
categories:
- 前端开发
tags:
- IE兼容性
- AngularJS
- placeholder
---

> Curse You IE!

这句开场白是我司项目源码里的常见注释，我司的项目主要是海外市场，IE 6/7是不用折腾了，但是IE 8/9还是要整的，老版本的IE有多闹心我就不惜的说了，谁整谁知道！前段时间我被assign了一个bug要修，IE 8/9不支持input和textarea的placeholder属性，说体验很不好，让修复下。 大早上看到这个我整个人都不好了，这篇文章就记录一下我是如何解决这个问题的。

<!--more-->

### 方案1：用value属性来模拟placeholder

有问题，先google！最快捷的方式自然是一大堆的jQuery插件了，包一个JS/CSS文件，然后调一下`.placeholder()`函数就搞定了。但是，一个是为了这么个小功能去用一个插件感觉没啥必要，另外一个原因是jQuery插件要应用到AngularJS上就需要改写成directive，感觉也很麻烦。所以，最终的结论是还不如自己写一个directive来解决这个问题。

自己实现directive就需要搞清楚如何在IE上模拟placeholder的原理，网上比较常见的解决方案是直接用input或textarea的value属性来模拟placeholder，代码大致和下面的类似：

``` javascript ie-placeholder.js
$('[placeholder]').focus(function() {
    var input = $(this);
    if (input.val() == input.attr('placeholder')) {
        input.val('');
        input.removeClass('placeholder');
    }
}).blur(function() {
    var input = $(this);
    if (input.val() == '' || input.val() == input.attr('placeholder')) {
        input.addClass('placeholder');
        input.val(input.attr('placeholder'));
    }
}).blur();
```

从上面的代码可以看出，原理还是比较简单的。给所有包含placeholder的标签（input/textarea）的focus和blur事件绑定回调函数：

* 当focus事件发生时，即用户点击input框时，检查如果input的value属性等于它的placeholder属性，则认为用户还没有输入过任何东西，这时清空input框并移除class。
* 当blur事件发生时，即用户点击别的地方时，检查如果input的value属性为空或者等它的placeholder属性，则认为用户并没有输入任何东西，这时将placeholder的值设置为value属性并添加class。
* 最后先手动触发一下blur事件，好让input框初始化的状态是显示placeholder。
* 这里的class属性一般是用来添加字体颜色的样式，因为一般placeholder的颜色为灰色。

### 方案1需要考虑的问题

乍一看，这种模拟方式基本可以满足placeholder的需求。但是有几个问题需要解决：

1. type为password时placeholder也会以mask的方式显示。这个应该是最棘手的问题了，由于是使用value属性，而password的value默认是以圆点mask的方式显示的，这样给人很奇怪的感觉。
2. 无法准确判断input框是不是dirty的。如果你输入的内容恰好就和placeholder的内容一样的话，则一旦你点击input框，focus事件发生，输入的内容会被清掉。
3. 提交form表单时需要格外注意。这个问题其实和问题2差不多，由于value属性模拟placeholder，一个表单你即便什么也不填也都是有值的，这就要求提交表单的时候要对每个input框做检查，判断value属性是否与placeholder不同来判断这个form是否dirty。如果你的表单提交逻辑已经写好的情况下，则需要很多额外的修改工作。

抛开2、3两个问题不说，问题1是无法接受的，显然这种方案不太可行。

### 方案2：直接添加一个span标签模拟placeholder

既然从input框本身下手不行，那就需要借助其他的标签来模拟了。试想我们在每个input框同样的位置上放一个span标签来模拟placeholder，这个span标签和input框同样的大小，当点击这个span时隐藏自己并使input框focus，当input框blur时若input框为空则重新显示这个span，这样就可以模拟placeholder的功能了。带着这个思路我们可以写一个自定义的directive来模拟placeholder：

``` javascirpt myPlaceholderDirective.js
app.directive('myPlaceholder', ['$compile', function($compile){
    return {
        restrict: 'A',
        scope: {},
        link: function(scope, ele, attr) {
            var input = document.createElement('input');
            var isSupportPlaceholder = 'placeholder' in input;
            if (!isSupportPlaceholder) {
                var fakePlaceholder = angular.element(
                    '<span class="plcaeholder">' + attr['placeholder'] + '</span>');
                fakePlaceholder.on('click', function(e){
                    e.stopPropagation();
                    ele.focus();
                });
                ele.before(fakePlaceholder);
                $compile(fakePlaceholder)(scope);
                ele.on('focus', function(){
                    fakePlaceholder.hide();
                }).on('blur', function(){
                    if (ele.val() === '') {
                        fakePlaceholder.show();
                    }
                });
            }
        }
    };
}]);
```

这里我们定义了一个叫做myPlaceholder的directive，按照directive的定义指定这个directive只能作为属性存在，并且它的scope是独立的，然后就是最重要的link函数，主要的逻辑都放在这里面。先看6、7两行，这两行的作用主要是为了判断当前的浏览器是否支持placeholder属性，这个神一般的方法来自于[StackOverflow](http://stackoverflow.com/questions/3937818/how-to-test-if-the-browser-supports-the-native-placeholder-attribute)，当然这里也可以使用[Modernizr](http://modernizr.com/)这样的第三方库来判断。而我们后面的逻辑都是发生在不支持placeholder的前提下的，如果支持的话就什么也不做了。第9-16行是添加span的过程，先定义一个span标签，内容就是input框的placeholder属性，然后为其绑定click事件，当用户点击这个假的placeholder时，触发input框的focus事件。最后将这个假的placeholder插入到DOM树中去。这里值得注意的有三点：

1. 为什么使用span标签？这可不是随便拿来用的，其实这个场景使用label标签非常合适，语义上比较符合，但杯具的是label标签在IE 8中不支持click事件。
2. 第12行为何停止事件的冒泡？防止对其他部分产生影响，因为这个span标签是硬塞进来的，一旦click事件往上传播可能会影响现有的逻辑。
3. 第16行的作用？这是AngularJS里特有的问题，在link函数里对DOM树进行更改后只有使用`$compile`函数这个更改才能生效。当然你也可以将DOM更改放入compile函数中去。

接下去的17-23行就跟方案1差不多了，focus事件发生时隐藏假的placeholder，blur事件发生时若input框内容为空则重新显示假的placeholder。不同之处就在于这里我们不用管input框的内容是否与placeholder属性相不相同了。

### 方案2需要考虑的问题

那么到现在为止这个方案2能工作吗？答案是能，但是有很多问题！下面一个一个来分析：

_1. 位置问题_

这个恐怕是最首当其冲的问题，方案1由于使用就是input框本身，所以根本不存在错位的问题，而方案2是新插入一个标签，所以必须让新的span标签与原来的input框位置重合，这样才能以假乱真。首先，一些必要的样式是要有的，比如

``` css
.placeholder {
    position: absolute;
    color: #aaa;
    cursor: text;
    z-index: 1;
}
```

绝对定位可以保证span可以叠加在input框之上，同样`z-index`样式也是为了保证这个，而`cursor`样式是为了让鼠标更逼真，普通的input框当鼠标移上去时显示的就是text的样式。这些样式仅仅是一个开始，要知道，input框的位置在页面渲染好以后也可能是会发生变化的。举个例子，有两个input框上下相邻，如果第一个输入的错误的信息，则两个input框之间可能会需要一条报错提示，这样，下面的input框的位置就会发生变化。所以，必须时刻根据input框的位置调整span的位置。而AngularJS里恰好有这么一个特性：`$watch`，我们要做的就是watch这个input框的位置，一旦发生变化立马调整span的位置：

``` javascript
scope.getElementPosition = function() {
    return ele.position();    
};
scope.$watch(scope.getElementPosition, function(){
    fakePlaceholder.css({
        'top': ele.position().top + 'px',
        'left': ele.position().left + 'px'
    }); 
}, true);
```

jQuery的`position()`函数可以获取元素的位置，一旦原始input框的top、left发生变化，则将span元素的top、left属性与其保持一致。注意`$watch`函数的第三个参数，默认`$watch`判断是否发生变化使用的是`===`，这样的话两个object是永远不可能相等的，而加上第三个参数并给true，则会使`$watch`函数比较object的各个property是否相等。

_2. 大小和样式问题_

解决了位置问题会发现，有些情况下还是会有错位的问题，比如span的内容要比input框偏左一点，span框的行高与input框不一致等等，这些牵涉到span的大小和样式必须要与input框一致。同样我们可以用`$watch`来解决：

``` javascript
scope.getElementHeight = function() {
    return ele.outerHeight();  
};
scope.$watch(scope.getElementHeight, function(){
    fakePlaceholder.css('line-height', ele.outerHeight() + 'px'); 
});
if (ele.css('font-size')){
    fakePlaceholder.css('font-size', ele.css('font-size'));
}
if (ele.css('text-indent')){
    fakePlaceholder.css('text-indent', 
        parseInt(ele.css('text-indent')) + 
        parseInt(ele.css('border-left-width'))
    );
}
if (ele.css('padding-left')){
    fakePlaceholder.css('padding-left', ele.css('padding-left'));
}
if (ele.css('margin-top')){
    fakePlaceholder.css('margin-top', ele.css('margin-top'));
}
```

可能还有很多样式没有考虑到，这就要看具体的情况了，发现有问题再去看是哪个样式影响的，再进行添加。

_3. 随input框的隐藏而隐藏_ 

这个应该是AngularJS特有的问题吧，因为有`ng-show`的影响，即便页面渲染完成后很多input框的显示与隐藏也是会变化的，所以，这个自然也需要`$watch`了。

``` javascript
scope.isElementVisible = function(){
    return ele.is(':visible'); 
};
scope.$watch(scope.isElementVisible, function(){
    var displayVal = ele.is(':visible') ? 'block' : 'none';
    fakePlaceholder.css('display', displayVal);
    if (displayVal === 'blcok' && ele.val()) {
        fakePlaceholder.hide();
    }
});
```

这里同样使用jQuery的工具函数来做判断，将input框的显示状态赋值给span。注意7、8两行，如果input框在隐藏的过程中突然有值了，这时再显示的时候就要隐藏假的placeholder了。

_4. 若input框有值则隐藏_

这个同样是AngularJS的问题吧，因为有`ng-model`属性，有时即便你不手动输入，input框的内容也是会变化的，所以，这个也需要`$watch`。

``` javascript
scope.hasValue = function(){
    return ele.val(); 
};
scope.$watch(scope.hasValue, function(){
    if (ele.val()) {
        fakePlaceholder.hide();
    }
});
```

好了，到这里为止，我们才可以说这个假的placeholder比较稳定了。当然，实际情况很复杂，具体情况具体分析，无非是善用`$watch`了。



