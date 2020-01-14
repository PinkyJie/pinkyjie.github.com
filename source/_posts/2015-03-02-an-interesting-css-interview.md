title: 一道有(bian)趣(tai)的CSS面试题
categories:
  - 前端开发
tags:
  - CSS
  - CodePen
  - box-shadow
  - linear-gradient
  - 多层边框
date: 2015-03-02 22:27:50
---

今天在登录CodePen的时候看到一个弹窗，推荐一些网站的用法，其中[有一篇](http://blog.codepen.io/2015/01/22/interviewing-front-end-hires-codepen/)说很多公司通过CodePen来面试前端工程师，网站支持合作模式，可以实时观察被面试者敲代码，也可以实时沟通，觉得挺有意思，国外就是高端！点进去发现一道用来面试CSS的题目，因为一直觉得自己CSS不是特别好，就想我也做做试试。初看感觉应该不难吧，实际上做下来苦不堪言啊。。。

<!--more-->

<script async src="//assets.codepen.io/assets/embed/ei.js"></script>
<p data-height="268" data-theme-id="12085" data-slug-hash="GtqKj" data-default-tab="css" data-user="mobify" class='codepen'>See the Pen <a href='http://codepen.io/mobify/pen/GtqKj/'>CSS Test — Button</a> by Mobify (<a href='http://codepen.io/mobify'>@mobify</a>) on <a href='http://codepen.io'>CodePen</a>.</p>

题目的HTML部分只有一个a标签，链接的文本是Checkout，然后要求是不能修改HTML的内容，纯用CSS模仿出给出的图片按钮。

![图1](/assets/images/an-interesting-css-interview-1.png)

题目给出的要求是15分钟内完成，不必太在意完美的匹配，可以随便google但是要能够讲得出原理。下面是我最终的答案，为了方便对比我在HTML中加了一个img标签，把原始图片放在下面了。

<p data-height="280" data-theme-id="12085" data-slug-hash="raKxPz" data-default-tab="result" data-user="pinkyjie" class='codepen'>See the Pen <a href='http://codepen.io/pinkyjie/pen/raKxPz/'>raKxPz</a> by Pinky Jie (<a href='http://codepen.io/pinkyjie'>@pinkyjie</a>) on <a href='http://codepen.io'>CodePen</a>.</p>

题目要求是15分钟，可是，我花了好几个小时还没搞定我会乱说。O(∩_∩)O~ 下面就跟着我的节奏来一步一步看看具体咋模拟这个button。每一步我都会先列出代码和效果，然后讲讲这一步都可以问出哪些CSS的知识吧。

### 调位置，调大小

``` css
a {
  display: block;
  width: 250px;
  height: 62px;
  margin: 28px;
}
```

效果如下（图中黄色区域为margin）：

![图2](/assets/images/an-interesting-css-interview-2.png)

第一步当然是调调位置和大小了。首先，必须给a标签先加上`display:block;`的样式，否则如果直接给a标签加类似width和height的属性是没效果的，why？那么第一个问题来了，**问题1：什么是行内元素？什么是块级元素？两者有什么特点和区别？**那么首先，a标签属于行内元素，行内元素不像div这种块级元素可以独占一行，并且对于宽高也有一定的限制：
* 宽度：只能是内容的宽度，不能改变。
* 高度：只能是内容的高度，不能改变，类似line-height这种样式也是没用的。
* margin：只有左右的margin才生效。
* padding：上下左右都生效，但上下的padding比较特殊。以padding-bottom举例，如果行内元素后面紧跟一个块级元素（比如div），则块级元素会与padding的部分重合，即padding-bottom并不能使下面的行下移，如下图（图中绿色区域为padding）：

![图3](/assets/images/an-interesting-css-interview-3.png)

也就是说，想让一个行内元素有固定的宽度和高度，必须先改变其display样式使其变为块级元素才行。

### 设置字体

``` css
a {
  font-family: arial;
  font-size: 24px;
  font-weight: bold;
  color: #535253;
  text-decoration: none;
  text-transform: uppercase;
}
```

效果如下：

![图4](/assets/images/an-interesting-css-interview-4.png)

第二步我们来设置字体，这些属性都很常见，如果你想问我怎么识别图中的字体，两个痣：猜的。。。至于字体大小，用Chrome的Dev Tools调吧。另外两个跟文字有关的样式也比较容易理解，text-decoration去掉a标签默认的下划线样式，text-transform可以指定文字为全部大写。这个可以作为**问题2：怎样设置文字的样式，如大小写以及上/下划线？**值得一说的时颜色的模拟，如何得知图中的颜色到底是什么值吗？大家肯定第一时间想到“取色器”，以前也用过一些插件，不过Chrome的Dev Tools自带的取色器就很好用。写的时候可以随便给color属性一个值，然后点击属性值左边的小色块，即可调出取色器，然后移动光标到想取色的图片上即可出现放大镜，定位到指定的像素上时点击鼠标，取色完成。如下图：

![图5](/assets/images/an-interesting-css-interview-5.png)

### 文字的位置

``` css
a {
  background: #ccc;
  text-indent: 52px;
  line-height: 63px;
}
```

效果如下：

![图6](/assets/images/an-interesting-css-interview-6.png)

这一步要做的就是调整文字的位置了，为了方便对比，先随便设一个背景色。为了调整文体的位置，这里使用了text-indent来使文字向右偏移，使用line-height来达到使文字向下偏移的效果。也许有人会问，怎么不用padding-left和padding-top呢？那么问题来了，**问题3：怎么设置字体的偏移？text-indent/line-height和padding-left/padding-top有什么区别呢？**那么我们尝试将代码中的text-indent改为padding-left，line-height改为padding-top，此时的效果如下图：

![图7](/assets/images/an-interesting-css-interview-7.png)

可以看到，绿色区域为padding，使用padding会使整个a标签的宽度和高度增大，使得与原始图片的宽高已经不一致了，此时就需要重新调整宽度和高度。那么问题来了，**问题4：元素在页面中实际占用的宽度和高度如何计算？**，**问题5：什么是CSS中的盒子模型？**这两个问题常常被放在一起提问，因为不同的盒子模型决定不同的宽高计算方式。总共有两种盒子模型，IE的和W3C标准的。。。具体可以看[WIKI](http://zh.wikipedia.org/wiki/IE%E7%9B%92%E6%A8%A1%E5%9E%8B%E7%BC%BA%E9%99%B7)。这里只说结论：
* IE盒模型：*实际占用的宽（高）度* = 内容宽（高）度[CSS值]
* 标准盒模型：*实际占用的宽（高）度* = 内容宽（高）度[CSS值] + 左（上）padding + 右（下）padding + 左（上）border + 右（下）border

可以看到，在IE盒模型中，不管pading和border设置多大，元素在页面中实际占用的宽高度都与CSS设置的保持一致。在我们这个场景下，显然希望不管padding怎么变化，a标签在页面中的实际宽高都能始终等于样式里设置的width和height。可以看到恰巧IE的盒模型可以满足我们的要求，有史以来第一次觉得IE的实现很好。。。但是就没有办法在非IE浏览器解决了吗？非也！试试box-sizing属性吧。那么问题来了，**问题6：box-sizing属性是干什么用的？**。试着将text-indent和line-height注释掉，添加box-sizing样式，并适当调整padding的值：

``` css
a {
  box-sizing: border-box;
  padding-left: 60px;
  padding-top: 18px;
}
```

这样得到的效果如下图：

![图8](/assets/images/an-interesting-css-interview-8.png)

可以看到，外观是一样的，除了绿色的padding。我们给box-sizing样式赋了一个`border-box`，意思就是使用类似IE的盒模型来计算宽高。除此之外，它还可以取`content-box`，就是W3C默认的标准盒模型了。正是由于这个便利，可以发现很多CSS框架（如bootstrap）都会有这么一条样式`* {box-sizing: border-box}`，即将所有的元素都按IE的盒模型处理，因为这样给我们的布局带来很多便利，避免不必要的像素计算。关于box-sizing还有很多有意思的面试题，比如在CodePen上面我还发现一个有意思的[布局题目](http://codepen.io/chriscoyier/pen/ClGcF)，要求就是将右边的侧栏恢复正常。显然，最简单有效的方式就是给section加一条样式：`box-sizing: border-box`即可。

总之，为了实现文字的偏移，两种方案都是可行的，根据自己的习惯选用即可。

### 简单效果：文字阴影，圆角边框

``` css
a {
  border-radius: 10px;
  text-shadow: 0 1px #eee;
}
```

效果如下：

![图9](/assets/images/an-interesting-css-interview-9.png)

这一步就没啥讲的了，如果几年前圆角边框还算新鲜的话，现在早已经烂大街了。至于弯几个像素，开着Dev Tools可劲调吧，调到觉得像为止。至于text-shadow，用来模拟图中CHECKOUT文字下边缘的一丝灰色。参数也很好理解，第一个是阴影向右偏移的像素，第二个是向下偏移的像素，第三个颜色照例取色器搞定。

### 加星星

``` css
a:before, a:after {
  font-size: 27px;
  color: #888;
  content: "★";
  text-shadow: 0 1px #fff;
}

a:before {
  margin-left: -30px;
}

a:after {
  margin-left: 10px;
}
```

效果如下：

![图10](/assets/images/an-interesting-css-interview-10.png)

这一步给按钮加上了左右两个小星星，由于不能改变HTML结构，那么能使用的方法也就只剩`:before`和`:after`了。那么。。。**问题7：before和after是干什么用的？**根据字面意思理解，其实就是在指定标签的内容前面和后面插入内容，而插入的内容呢是由content属性来定义的。先来说说这个content，肯定要有人问了，这个五角星是怎么打出来呢？用Mac的同学自然不用着急，所有编辑器的“编辑”菜单下面都有这么一项子菜单：特殊字符。里面有各种字符，各种五角星。如下图：

![图11](/assets/images/an-interesting-css-interview-11.png)

那对于非Mac系统的同学怎么弄呢？各种输入法应该提供特殊字符的输入。除此之外，还可以使用Unicode编码来代表五角星，五角星的CSS编码是`\272D`，所以我们将content属性替换为`content: "\272D"`即可。关于如何在HTML/CSS/JS中使用Unicode编码可以参考[这篇文章](http://www.qianduan.net/html-special-characters-daquan.html)。除了content属性，其他属性就没有太多可说的了，设置字体颜色阴影位置等等。有一点值得一说，除了使用margin-left以外，还可以将before、after的position属性设置为absolute，使用top和left属性来随意调整。

### 多层边框

``` css
a {
 box-shadow: 0 1px 2px 1px #656565,0 0 0 6px #CACACA,
            0 0 0 8px #fff, 0 0 0 10px #696969,
            0 2px 3px 11px #CBCBCB;
}
```

效果如下：

![图12](/assets/images/an-interesting-css-interview-12.png)

这一步应该是最复杂的一步了，起初看到这个按钮图，可能就只能注意到最外面的一个边框。其实，如果在不改变HTML的情况，最外面的几层都只能通过边框来实现，通过下面这个放大后的图我们应该就可以非常清晰的看到这四层边框。

![图13](/assets/images/an-interesting-css-interview-13.png)

图中用红色数字标出的1、2、3、4就是上面所说的四层边框。那么。。。**问题8：如何实现多层边框？**显然，使用border属性是无法实现的。其实在做这道题的时候我也不会，就搜索呗，谷歌搜“multiple border”第一条就是[这篇文章](https://css-tricks.com/snippets/css/multiple-borders/)（什么？搜不到？再用百度就剁手了啊）。这里我们使用的就是文章里介绍的box-shadow的方法。box-shadow属性和text-shadow属性是一样的，前面两个参数还是向右和向下的阴影偏移，第三个和第四个参数都是可选的，所以我们在前面的text-shadow里没有用这两个参数。第三个参数是阴影模糊的距离，第四个参数是阴影的宽度。讲到这里，你应该就可以想到，要想实现边框，那么只要前两个偏移给0，然后第四个参数给值就可以了，事实也正是如此。我们一层一层边框来分析，按上面截图的顺序，由内向外：
* 第1层：最里面的边框，这里纵向偏移我们给了1px，是因为图中下边框有一点偏黑的阴影。至于模糊的距离怎么确定，三个字：慢慢调。。。
* 第2层：最宽的阴影，这个没啥讲的，只给了宽度这个参数，慢慢调吧。
* 第3层：白色边框，同样只给了宽度这个参数。
* 第4层：最外面的边框，也是只给了宽度。

慢着，怎么还有一个参数，有第5层边框吗？仔细观察，可以发现最外面边框的下面有一层很淡很模糊的阴影，为了逼真，我们就也用一层边框来模拟。还有一点，这里之所以从最里面的边框开始写，是因为前面的阴影“优先级”更高，也就是越靠前越在最上面，这样才能显示出层次感。如果直接将宽度为11的写在最前面，那后面的都会被盖住的。

### 分段背景

``` css
a {
  background: linear-gradient(rgba(219, 219, 219, 0.9) 48%, rgba(169, 169, 169, 0.6) 48%);
}
```

效果如下：

![图14](/assets/images/an-interesting-css-interview-14.png)

最后一步了，只剩下按钮的背景了，很显然，背景分为两层，那么很自然的会想到需要使用渐变的背景来实现。常见的渐变是给定两种颜色，从前者过渡到后者去，但是我们效果图中并没有看到渐变，而是泾渭分明的两种颜色。这时候就需要紧跟颜色后面的百分比这个参数来是实现了。那么。。。**问题9：颜色渐变的时候这个百分比参数是干嘛用的？**在做这个题之前，我对这个问题的认识是很模糊的。百分比参数最常见的用法就是`background: linear-gradient(red 0%, green 100%)`，也就是最上面是红色，最下面是绿色，中间渐变。那这里第一个百分比不是0%代表什么呢？其实可以理解为其前面有一个同样颜色的0%。以上面的CSS代码为例，在前面加一个`rgba(219, 219, 219, 0.9) 0%`的效果与现在是一致的。同样，最后一个百分比不是100%的话，就相当于在后面加一个同样颜色的100%。这样一来就好解释了，从0%到48%是颜色1到颜色1的渐变（显然，颜色渐变区间为0，无渐变效果），从48%到48%是颜色1到颜色2的渐变（显然，距离渐变区间为0，无渐变效果），最后，从48%到100%是颜色2到颜色2的渐变（也是无渐变效果）。通过这种方式，就实现了颜色分界的效果。那么最后一个问题来了，**问题10：如果颜色分3层怎么实现呢？**类推一下就可以写出来，比如从上到下分别为红绿蓝的话，CSS为：`background: linear-gradient(red 33%, green 33%, green 66%, blue 66%)`，自己试试就明白啦。

### 总结与不足

到这里，大致的效果已经被还原了（当然，如果考虑浏览器兼容性，很多样式都要多写几条的，你懂得！）。之所以说大致还原，是因为对于效果图还有很多不足：很多颜色不够准确，阴影的范围也不逼真，还有一点关键的就是：分界背景的下半段的两边是有一丝弧度的，这个我目前为止还是不知道怎么实现（有更新）。。。最最变态的是，这道题的要求是15分钟，我估计做了好几个小时了吧O(∩_∩)O~ 在CSS的不归路上还有很多路要走啊。。。

PS: 要问为啥截图这么大？我用麦克不克瑞缇娜！

<hr>

### Update at 2015.03.17

把这篇文章分享到[v2ex](https://v2ex.com/t/177397)后，得到了很多高手的反馈，关于背景弧度的实现，大家提出了两种解决方案：
* 一种是只用before来实现两个五角星，然后用after来实现弧度的背景，可以看[这个方案](http://codepen.io/anon/pen/NPEwJp)。诀窍就是content属性里面用空格隔开两个五角星。还有一位提到了用box-shadow来实现第二个五角星，即第二个五角星是第一个五角星的阴影。两种方法的原理都是只使用before来实现五角星，这样就省下了after属性，可以用after来实现带弧度的阴影。
* 第二种是使用box-shadow来模拟这个背景，这个想法简直是巧夺天工！box-shadow有一个inset的值可以将由外投射的阴影改为由内，所以可以用这部分由内投射的阴影来模拟上半部分的背景，原来的背景就被挤到下面，而原来的背景正是有弧度的。直接上代码吧，将box-shadow和background改为：

``` css
a {
  box-shadow: 0 29px rgba(222, 222, 222, 1) inset,
    0 1px 2px 1px #656565,0 0 0 6px #CACACA, 0 0 0 8px #fff,
    0 0 0 10px #696969, 0 2px 3px 11px #CBCBCB;
  background: rgba(182, 182, 182, 0.6);
}
```

也就是说，现在的background的颜色是背景的下半部分，颜色值同样取色器搞定，而上半部分用box-shadow的第一个值来实现，比较特殊的就是inset参数了。阴影改为inset后，由向外投影改为向内投影，向右偏移为0，向下偏移为29，这样就模拟了背景的上半部分了。

