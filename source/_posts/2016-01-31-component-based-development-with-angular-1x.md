title: 用 Angular 1.x 做组件式开发
categories:

- 前端开发

tags:

- AngularJS
- 组件式开发
- Webpack
- ES6
- directive
- angular1-webpack-starter

date: 2016-01-31 15:51:58

---

自打去年参加了 JSConf 后一直断断续续看了很多 React 的资料，不得不承认 React 确实在很多方面给人耳目一新的感觉，尤其是组件化的思想给我留下了很深的印象。JSX 的语法虽然一开始很难让人接受，但 React 将函数式编程的思维用在组件化上，确实值得借鉴。作为一个 Angular 死忠粉，自然很想把这种好的思维方式也用在 Angular 上了，于是就有了[angular1-webpack-starter](https://github.com/PinkyJie/angular1-webpack-starter)这个项目。这篇文章就来记录下用 Angular 来做纯组件式开发的一些尝试吧。

<!--more-->

### 文件夹结构

文件夹结构怎么安排是每次新开一个项目必定要纠结的问题，社区在这个问题也有过很多思考，比如最初的按照文件类型划分，到后来按照 feature 来划分。

```bash 按文件类型区分文件夹
app/
-- controllers/
---- homeController.js
---- loginController.js
-- services/
---- userService.js
-- directives/
---- inputDirective.js
-- other/
---- xxx.js
-- app.js
-- routes.js
images/
-- xxx.png
-- ...
styles/
-- xxx.css
-- ...
views/
-- index.html
-- ...
```

```bash 按feature区分文件夹
app/
-- core/
---- userService.js
---- inputDirective.js
-- home/
---- homeController.js
---- homeRoute.js
---- home.html
---- home.css
-- login/
---- loginController.js
---- loginRoute.js
---- login.html
---- login.css
-- app.js
assets/
---- images/
```

为什么文件夹结构让人如此纠结？一个好的文件夹结构可以迅速让开发者明白项目的整体结构！更重要的是，在实际开发的过程中，好的文件夹结构可以迅速的让开发者将实际页面与页面对应的源文件对应起来，这可以极大的提高开发效率。从这个评判标准出发，我们可以发现最初的“按文件类型区分”显得稍显粗暴。项目小还好，一旦项目大起来，很难快速的找到与某个页面对应的相应源文件。并且，一个页面是由 JS、CSS、HTML 共同组成的，那么我要改一个页面，就需要从 3 个文件夹里找，稍显低效。一旦项目大了，参加项目的人一多，文件命名一乱，找起文件来非常繁琐。为了解决这个痛点，社区又提出了第二种方法：按 feature 来区分，将属于同一 feature（一般是一个页面或一组页面）的 JS、CSS、HTML 放在一起，将公用的文件放在单独的文件夹里。这样一来，结构分明，而且找起文件来也相对轻松。所以，目前这种文件夹结构及类似的衍生成为社区的主流。但这种结构是否真的适合组件式开发呢？通常一个 feature（页面）是由多个组件组成的，显然组件的粒度要比 feature 细的多。一种直接的想法就是我们将 feature 层级直接替换成组件，按组件区分文件夹，但这样一来，页面与组件的关系又很难从文件夹结构中反映出来。所以，我们需要这样一种新的文件夹组织方式：

```bash
app/
-- components/
---- _core/
---- _layout/
---- home-hero/
---- login-form/
-- pages/
---- home/
---- login/
index.html
index.js
```

整个结构分为 pages 和 components，一目了然，属于一个组件的资源放在一起，而页面只是将这些组件“组合”起来。这个新的组织方式可以说就是为组件化而生，开发者一看就知道是怎么回事。关于这种文件夹结构的详细解释，可以参看[这篇文章](https://github.com/fouber/blog/issues/10)。

### 用函数式思维实现组件

有了文件夹结构，剩下的就是如何实现组件和页面了。React 的一大思想就是将函数式编程的思维引入到组件开发上来：`UI = f(states)` 一个组件就是一个函数。函数有三个要素：输入参数，实现逻辑和返回值。对应到组件上来说，这三要素就变成：

- 输入参数`states` => 将组件渲染到页面上需要的数据，使用这个组件（模仿函数的概念可以称之为“调用者”）需要将数据源传入组件。
- 实现逻辑`f()` => 组件内部的交互逻辑。
- 返回值`UI` => 组件渲染后的页面。

具体到 Angular 框架上来讲，组件有其对应的概念——`directive`，那么我们就来看看这三个要素要怎么应用到 directive 上来。我们来思考一个具体的例子，比如我们有一个`phone-form`的表单组件，它允许用户查看、编辑、新增一款手机的型号、品牌、价格、系统等参数，如下图所示。

![图1](/assets/images/component-based-development-with-angular-1x-1.PNG)

为了让一个组件具备这三种功能，那么它的输入参数需要点什么呢？

- `state`变量，指明当前表单处于什么状态：查看、编辑还是新增。
- `phone`变量，充当传入的 model，在查看和编辑状态下需传入当前的 phone，新增状态下传空。
- `cancel()`函数和`submit()`函数，作为组件本身它不知道这两个 button 需要干什么，需要外部调用者来告诉组件。

关于参数的传入，directive 上又有很多讲究了。首先，为了让我们的 directive 尽可能的通用和可重用，必须使用独立的 scope（[isolate scope](https://docs.angularjs.org/guide/directive#isolating-the-scope-of-a-directive)），也就是定义 directive 的时候使用`scope: {}`配置。独立 scope 可以让 directive 的 scope 脱离当前的父 scope（这里称为父 scope 并不是说独立 scope 派生自它，而是说 DOM 树关系），防止不小心对父 scope 的数据进行更改。确立了使用独立 scope，我们还需要思考使用哪种绑定方式来传值。Angular 提供了`=`、`@`、`&`三种方式，关于三者的区别可以参见 StackOverflow 上的[这个回答](http://stackoverflow.com/questions/14050195/angularjs-what-is-the-difference-between-and-in-directive-scope)，这里简单解释一下：

- `=`: 双向绑定，建立起父 scope 和 directive 独立 scope 的双向联系，如果 directive 中需要更改传入的变量，并且这个更改要传回父 scope，则使用这种绑定。
- `@`: 字符串绑定，它是将父 scope 的字符串传入独立 scope，比如`aaa="bbb"`则传入 bbb 这个字面字符串，{% raw %}`aaa={{bbb}}`{% endraw %}则传入 bbb 变量在父 scope 上的值，总之它的结果是字符串。很多地方将其解释为单向绑定是不对的，因为它只能绑定字符串。
- `&`: 表达式绑定，简单来说就是将传入的表达式在父 scope 上执行，一般用于传入函数表达式，它将一个函数绑定在独立 scope 上。也可以将`&`用于单向绑定，如`aaa=bbb`则会将一个`aaa()`函数绑定在独立 scope 上，它的返回值是父 scope 上的 bbb 属性值（相当于一个 getter 函数），但 aaa()函数返回的仅仅是初始化 directive 时计算得到的 bbb 的值，后续父 scope 对 bbb 的更改不会反应到独立 scope 上。

具体到我们的例子上，`state`变量只是告诉当前组件应处于`view/edit/new`三种状态中的哪一种，完全可以使用`@`绑定。而`phone`变量仅需要将 phone 从父 scope 传入 directive 的 scope，directive 里对 phone 的更改不需要传回父 scope，所以是单向绑定，可以使用`&`。最后，两个函数自然是使用`&`绑定了。从“输入参数”这个层面上来看，一旦要“调用”某个组件，你就必须负责提供这个组件需要的各种数据，这些数据有些可能必须通过异步请求才能拿到。所以在将组件组合成页面时，这些数据一般由页面的 controller 准备好然后传递给组件。

说完了“输入参数”，来说说后两项。“实现逻辑”自然是放在 directive 的 controller 里实现了（用于 UI 组件时建议不要使用 link 函数，使用 controller 在语义上更好），看似没啥可说的，但其实这一块也有很多讲究。函数式编程中一个重要的概念就是“纯函数”（[pure function](https://en.wikipedia.org/wiki/Pure_function)）。简单的说，就是纯函数不会产生很多副作用，比如说它不能更改传入的参数本身，它不能依赖一些全局变量，不能有异步的 HTTP 请求或 DB 请求。举个例子，JS 中操作数组的`slice`和`splice`，前者就是个纯函数，而后者不是，因为它会改变调用者本身。那么把这种理念用在组件开发上，就要求组件在自己的 controller 实现逻辑中，要注意一下几点：

1. 即便采用`=`绑定了也不能直接修改传入的值，否则就会影响父 scope 的值，函数不纯了，或者干脆永远不要使用`=`绑定。
2. 不能有异步的 HTTP 调用，意味着组件不能调用 service 进行 HTTP 请求。
3. 想要改变数据，只能 fire 事件。

所以，组件里如果需要改变 model 的值就需要 fire 一个 change 事件，service 里响应事件后更新 model，然后 fire 一个 update 事件，父 scope 响应这个事件，再更新自己的 model。这也正是[Flux 架构](https://facebook.github.io/flux/)提出的原因，没有了副作用，整个程序的状态改变就变得可控，出现问题后也比较好追溯。当然，它也确实增加了编写程序的复杂度，是否需要全盘应用它还是视具体的项目而定了。

最后，“返回值”这项就不用说了，直接在 HTML 里“调用”这个组件即可，组件就可以渲染到最终的页面上去了。

> （整个组件的具体实现可以[参见 Github](https://github.com/PinkyJie/angular1-webpack-starter/tree/master/source/app/components/phone-form)，值得一提的是，实际项目中`phone-form`组件上 phone 的绑定并没有使用`&`，而是使用了`=`，因为父 scope 中的 phone 是异步 service 得到的，初始化 directive 时得到的 phone 是空值。）

### 将组件“组合”成页面

采用上述的文件夹结构后，一个很自然的想法是：我们可以模拟 node 的 commonJS 模块那样，每个 component 文件夹有一个`index.js`来将这个组件的各个资源粘合在一起（需配合 Webpack），然后将 component 这个模块暴露出去。来看看上面的`phone-form`这个组件的`index.js`的实现（采用 ES6）：

```javascript index.js for phone-form component
import angular from "angular";

import PhoneFormController from "./phone-form.controller";
import PhoneFromDirective from "./phone-form.directive";

const phoneForm = angular
  .module("app.components.phoneForm", [])
  .controller(PhoneFormController.name, PhoneFormController)
  .directive(`aioPhoneForm`, PhoneFromDirective);

export default phoneForm;
```

那么在 phone-detail 页面我们需要“调用”这个 component 时怎么写呢？同样，每个 page 文件夹里也有一个`index.js`来将这个页面需要的各个组件引用进来，然后将这个页面暴露出去给整个程序的入口`index.js`来引用。来看看`phone-detail`这个页面的`index.js`的实现（采用 ES6）：

```javascript index.js for phone-detail component
import angular from "angular";

import PhoneDetailController from "./detail/phone-detail.controller";
import phoneForm from "../../components/phone-form";

export default angular
  .module("app.pages.phone-detail", [phoneForm.name])
  .controller(PhoneDetailController.name, PhoneDetailController);
```

可以看到，页面只需要将组件所在的 Angular module 声明为依赖就可以了。这样在页面的 HTML 中就可以直接使用这个组件了：

{% raw %}

```jade phone-detai.jade
.phone-detail-view.full-width
    .card
        .card-header.p2
            h5.title {{::vm.phone.model}}
            button.btn-floating.blue(ng-show="vm.state === 'view'", ng-click="vm.beginEdit()")
                i.mdi-image-edit
        .card-content
            aio-phone-form(
                phone="vm.phone",
                state="{{vm.state}}",
                submit="vm.updatePhone(phone)",
                cancel="vm.cancelUpdate()"
            )
```

{% endraw %}

注意上面在传递`submit`的时候，参数`phone`有些奇怪，它没有挂在`vm`下，说明它不是父 scope 上的变量，那这个传递进去的 phone 是什么呢？要解决这个疑问，我们就要来分析分析组件间的通信了。

### 组件间通信

说到组件间通信，我们可以简单的分为：

- 外部向组件内传递数据（父 scope => directive）
- 组件向外部传递数据（directive => 父 scope）
- 组件与组件间传递数据（directive A <=> directive B）

显然，我们还是可以像 Angular 中不同 controller 间的通信方法一样，使用 service 作为中间层来交换数据，或者使用事件来传递，这两种方法在此不再赘述。我们来看看其他两种方式。

#### 通过属性传参（即前面提到的“输入参数”）

前面有讲到，父 scope 给 directive 的独立 scope 传参时有三种绑定方式，采用`=`绑定时可以在父 scope 和独立 scope 之间建立双向通信，`@`只是单向的向 directive 中传递字符串。`&`就比较有意思了，一方面，前面讲过，它可以实现单向绑定，将一个 getter 函数绑定在 directive 的独立 scope 上，这相当于将父 scope 的变量单向传递给独立 scope。另一方面，它允许 directive 调用外部定义好的函数，通过这个函数我们其实可以**将 directive 的独立 scope 里的变量传递给父 scope**。我们回到上部分最后提到的问题，使用`submit="vm.updatePhone(phone)"`传入函数时，这里的 phone 其实并不是父 scope 的变量（其实也没必要显式的传入父 scope 的变量，因为这个函数定义在父 scope 上，里面你是可以任意访问父 scope 的变量和其他函数的），它只是一个**形参**，而这个形参的具体值可以由 directive 中调用时来填充。有了这个形参，我们就可以在 directive 的 controller 中将独立 scope 上的变量传递给父 scope 的函数调用。但注意在 directive 中的 controller 调用 submit 方法时，也需要采取特殊的语法：`this.submit({phone: phoneData})`，我们需要传入一个 object，key 就是形参名，value 就是我们要传入的真正值。通过这么两种`&`的不同用法我们可以看到，`&`是可以用来建立双向通信的，下面这个小 demo 能帮助你更好的理解这种双向通信：

<a class="jsbin-embed" href="http://jsbin.com/xucexi/embed?html,js,console">JS Bin on jsbin.com</a><script src="http://static.jsbin.com/js/embed.min.js?3.35.9"></script>

上面的代码中，属性`getVar`和`func`采用的都是`&`绑定，属性`getVar`将一个 getter 函数绑定到 directive 上，directive 通过这个函数获得从父 scope 单向传递过来的变量，而属性`func`将另一个函数绑定到 directive 上，directive 通过这个函数将自己的变量传递出去。

#### 通过 directive 的`require`配置

`require`配置相信大家都见到过，一般在使用 directive 进行自定义表单校验时，肯定会接触到`require: 'ngModel'`：在带有`ng-modal`属性的 input 标签上定义 directiveA，并且在在 directiveA 的配置中加上`require: 'ngModal'`，那么在 directiveA 的 link 函数中就可以获得第 4 个参数，这个参数通常命名为`ngModelController`，也就是定义在`ng-model`这个框架自带的 directive 上的 controller 函数。这表明，通过`require`配置可以在 directiveA 的 link 函数中访问到另一个 directiveB 的 controller 函数，也就是说 directiveA 的实现依赖 directiveB，这对于实现类似 Tab 这种 container 的 UI 组件非常有用。另外，require 还支持很多前缀，可以指定要依赖的 controller 的搜索路径，可以参考[这篇文章](https://demisx.github.io/angularjs/directives/2014/11/25/angular-directive-require-property-options.html)。值得注意的是，一旦使用`require`，directiveA 的定义只能使用 link 函数，directiveB 只能使用 controller 函数，相当于 directiveB 通过 controller 函数将自己的 API 暴露出去了。

### 总结

可以看到，Angular 作为一个不是那么 opinionated 的框架，写法有很多，我们完全可以把 React 倡导的一些概念给融合进来取长补短。最后再安利一下[angular1-webpack-starter](https://github.com/PinkyJie/angular1-webpack-starter)这个项目，后面应该还会围绕这个项目的实践写一些其他文章吧。
