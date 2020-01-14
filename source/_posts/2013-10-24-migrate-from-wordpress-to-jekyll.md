date: 2013-10-24 21:09
title: 从 Wordpress 迁移到 Jekyll
categories:

- 网站相关

tags:

- jekyll
- wordpress
- github
- github pages
- blog
- mathjax
- 多说

---

上周末闲着没事干突然想把博客从 Wordpress 迁移到 Github pages 上，于是周日花了一天时间做迁移，期间各种折腾，终于变成了现在这样还算满意的情况。虽然我已经好久不写博客了，但俗话说“书非借不能读也”，博客这一迁移也激起了我重新写博客的欲望。再说，这个博客还身兼重任，那就是——养媳妇！哈哈～还是说说迁移中遇到的乱七八糟的问题吧～

<!--more-->

### 搭建本地环境

首先，自然是在自己的 github 上面 new 一个 repository 了，名字必须是`yourname.github.com`，虽然现在 github pages 的地址已经改为`yourname.github.io`了，但我还没有尝试过 repository 用 io 结尾行不。
完成以后，就是各种本地的设置了，搭建 git 环境，搭建 ruby 环境啊一类的，这里就不详细说了，ruby 环境的搭建推荐使用[RVM](https://rvm.io/)，很方便的说，当然，貌似不支持 windows。。。如果在 widnows 下我推荐使用云 IDE，就好像浏览器里的虚拟机，`editor+terminal`，我常用`Nitrous.IO`，给个[我的推荐链接](https://www.nitrous.io/join/OoRQwt0SaLc)，有兴趣的可以试试。

然后就要在本地安装`Jekyll`了，一句话`gem install jekyll`搞定。[Jekyll](http://jekyllrb.com/)是一个 ruby 编写的静态站点生成器，在 github 上搭博客就靠它。但直接用 jekyll 貌似外观太朴素了，推荐直接使用[jekyll-bootstrap](http://jekyllbootstrap.com/)，从名字就可以看出，这玩意把来自 twitter 的优秀 CSS 框架`bootstrap`结合进来，很多漂亮的主题可以选择，而且内置了很多常用的功能。直接从`jekyll-bootstrap`起步吧，先 clone 下来，然后把 remote 的地址改成自己的～

```bash
git clone https://github.com/plusjade/jekyll-bootstrap.git yourname.github.com
cd yourname.github.com
git remote set-url origin git@github.com:yourname/yourname.github.com.git
```

然后`jekyll serve`就可以在本地`localhost:4000`看到博客的样子，当然，现在空空如也～

### 先换主题

在导入文章进行各种设置之前，推荐先选一个自己喜欢的主题换上，因为后面很多定制涉及到修改主题文件，为了避免 2 次修改，先换一个主题比较方便。`jekyll-bootstrap`提供了一个[主题预览网站](http://themes.jekyllbootstrap.com/preview/twitter/)，上面就有很多不错的主题。点击页面下面的主题名字进行预览，选择喜欢的以后，点击蓝色的`Install Theme`按钮，复制里面的代码执行，比如我选择的`hooligan`，代码如下：

```bash
rake theme:install git="https://github.com/dhulihan/hooligan.git"
rake theme:switch name="hooligan"
```

其中第一行将主题下载到`_theme_packages`里面，第二行切换到新主题。现在本地预览就可以看到新主题了。主题有两个相应的文件夹，一个是`_includes/themes/xxx`，这里放的是网页模板，另一个是 `assets/themes/xxx`，这里放的主题用到的 css/js 文件。我们要定制页面显示的内容通常修改第一个文件夹的东西，定制样式修改第二个，后面遇到了详细说。

### Wordpress 的文章迁移

现在可以导入原来的文章了，这个就比较闹心了，我的折腾时间有很大一部分花在这上面。[官方文档](http://jekyllrb.com/docs/migrations/)专门讲解了各个常用的博客系统怎么迁移过来，wordpress 的步骤也不复杂，从管理后台导出 xml，然后执行一条命令即可将所有文章转换成`.markdown`，将其拷入`_post`文件夹即可。这个时候你本地预览就可以看到文章，标签以及分类信息了。闹心的事情来了，为什么我预览报一堆错误？为什么我的显示跟预期的不一样？唉，一个一个说吧～

### 乱七八糟的`Could not find ref_id`

这个问题浪费了我不少时间，最后发现原来是字符转义的问题！`markdown`语法里中括号`[]`是特殊字符，如果你的文章里出现则需要转义，加`\`号即可。Too Simple!

### 公式的显示

以前研究学术的时候写过好几篇满是公式的博文，原先 wordpress 里貌似用的是一种插件，到这里显然行不通了。一番搜索后发现一个叫[MathJax](http://www.mathjax.org/)的牛逼东西，用 js 来渲染公式。在`_includes/themes/hooligan/default.html`里加入

```html
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>
```

即可，这样就可以用`$$`作为开始结束符来输入公式了。注意这里的`hooligan`是你的主题名称，你的主题如果是别的就改成相应别的。这样的公式是独占一行的，如果要输入 inline 的公式，则还需要添加

```html
<script type="text/x-mathjax-config">
    MathJax.Hub.Config({
      tex2jax: { inlineMath: [['$','$'], ['\\(','\\)']] }
    });
</script>
```

这样，使用`$`作开始结束符就可以输入行内的公式了。这里有个坑，有可能会遇到类似`Liquid Exception: Variable xxx was not properly terminated with regexp xxx`，这个我搜了半天，最后发现是公式内需要显示大括号时，原来会输入`\{\}`，但 jekyll 中`\`也需要转义，改成`\\{\\}`即可。同样的问题还有公式换行的时候，原先的`\\`改成`\\\\`才行。有时`[]`也需要转义。转义——多么痛的领悟。

### 代码高亮

把博客整到 github 上的应该多半都是程序员吧，博客自然少不了一堆公式。在 github 上直接用类似

{% codeblock %}

```python
code
```

{% endcodeblock %}

即可，但不知道为什么我的高亮一直没发生效，后来还是乖乖的用

{% raw %}

```ruby
{% highlight python %}
code
{% endhighlight %}
```

{% endraw %}

虽然写起来稍显复杂，但高亮效果不错，配合我的主题非常好看。另外，在写这篇文章的时候我又遇到一个问题，如何显示标签，比如像上面的代码里带有标签该如何转义书写呢，这个问题我也折腾了好久，很多人说用`"`进行嵌套，能解决问题，但是有点搞不清楚为啥这么写，后来发现，原来官方早就给答案了，把内容放到`{% raw %}{% raw %}{% endraw %}`和`{{ "{% endraw "}}%}`里面就不会进行 tag 解析了。

### 图片迁移

博客搬过来了，图片不多的话自然也搬过来吧～在`assets`文件夹下新建文件夹`images`和`files`，一个用来放图片，一个用来放文件附件。然后为了方便引用这两个路径，可以在`_config.yml`里定义变量：

```yaml
img_url: /assets/images
file_url: /assets/files
```

然后就可以在文章里像这样`![]({% raw %}assets/images{% endraw %}/picture.jpg)`插入图片了。其实这里可以发现，`_config.yml`里就是这样定义变量的，然后文章里就可用`site.xxx`来访问了，后面添加评论的时候还会用到。`_config.yml`里还有很多有用的配置，比如作者信息，博客的名字等等，不多说，自己研究吧。添加图片路径后，巨大的工作量来自——下载原博客里的图片，这里推荐一个 chrome 插件[Fatkun 图片批量下载](https://chrome.google.com/webstore/detail/fatkun-batch-download-ima/nnjjahlikiabnchcpehcpkdeckfgnohf?hl=zh-CN)，谁用谁知道。

### 评论系统

由于是静态网站，评论什么的就知道使用第三方的套件来实现了，`jekyll-bootstrap`里自带的是国外很有名的社交评论系统`disqus`，只要在`_config.yml`里配置自己的帐号即可。但入乡随俗嘛，我用`disqus 国内 山寨`为关键词 Google 了一把，立马发现了符合我朝国情的产品，类似[友言](http://www.uyan.cc/)，[多说](http://duoshuo.com/)等等，随便看了一下，最后决定多说吧。。。据说多说可以将原 wordpress 的评论导入，反正我是没成功。注册一个多说，填写相关的站点信息，就可以在`工具=>获取代码=>通用代码`里得到需要粘贴的代码，其中`<div class="ds-thread"></div>`是具体显示评论的代码，其他部分是 js 代码。仿照前面的，将 js 代码复制到`_includes/themes/hooligan/default.html`里。为了不写死代码，我们将`short_name`这一行修改为

```javascript
var duoshuoQuery = {
  short_name: {% raw %} "{{ site.JB.comments.duoshuo.short_name }}" {% endraw %}
};
```

后面我们会在`_config.yml`里配置这个`short_name`。然后另外的代码放哪里呢？首先，将你的多说信息配置到`_config.yml`里面，找到`_config.yml`里`comments`这一段，改成：

```yaml
comments:
  provider: duoshuo
  duoshuo:
    short_name: xxxx
```

其中的`short_name`就是多说的通用代码里的`short_name`值。然后根据`jekll-bootstrap`的结构，评论由`_includes/JB/comments`管理，打开发现里面是一个 ruby 的`switch-case`分支结构，在`{% raw %}{% endcase %}{% endraw %}`之前加上多说的判断即可：

```html
{% raw %}{% when "duoshuo" %}{% endraw %}
   {{ "{% include JB/comments-providers/duoshuo "}}%}
```

可以看到，实际的评论代码是放在`_includes/JB/comments-probiders`目录下的，在该目录下新建一个文件`duoshuo`，然后将刚才通用代码里的`<div class="ds-thread"></div>`复制进去即可。因为这个`_includes/JB/comments`会被很多模板自动引用，所以评论已经被添加到需要的地方去了。另外，如果你想定制评论，比如每篇博客右侧显示最近的评论和最近的来访者，可以修改`_includes/themes/hooligan/page.html`，这个是页面的默认模板，加一个`<div class="span4">`，熟悉`bootstrap`的孩子一定不陌生，这里就是右侧边栏，在这个 div 的最后面加入：

```html
<section>
  <h3>Latest Comments</h3>
  <ul class="ds-recent-comments" data-num-items="10" data-show-avatars="0" data-show-time="0" data-show-title="0" data-show-admin="0" data-excerpt-length="18"></ul>
</section>
<section>
  <h3>Recently Visitors</h3>
  <ul class="ds-recent-visitors" data-num-items="4" data-avatar-size="45" style="margin-top:10px;"></ul>
</section>
```

### 首页的定制：分页与摘要

到现在博客应该像模像样了吧，但唯一不爽的就是首页了，默认的首页`index.md`显示的是所有文章的存档，比较朴素，怎样改成像传统 wordpress 那样显示文章的摘要和合理分页呢？[官方文档](http://jekyllrb.com/docs/pagination/)同样专门有一节讲分页。首先，在`_config.yml`里配置分页，即添加`paginate: 5`，其中的`5`为每页的文章数，自己把握。然后，删除`index.md`，新建`index.html`，这一步很重要，我原来设置分页一直不成功就卡在这里，官方文档里也有讲，分页只对`html`生效！文档中有中规中矩的写法，大家可以参考，这里放出我修改过的版本。

{% raw %}

```html
---
layout: page
title: 进击的马斯特
tagline: Go Fighting! Master!
---

{% for post in paginator.posts %}
<div class="post-wrapper">
  <h2 class="h2 entry-title">
    <a href="{{ post.url }}">{{ post.title }}</a>
  </h2>
  <div class="entry-meta">
      <i class="icon-user"></i>  <span>{{ post.author }}</span> &nbsp;
      <i class="icon-calendar"></i>  <span>{{ post.date | date:"%Y-%m-%d" }}</span> &nbsp;
      {% unless post.categories == empty %}
        <i class="icon-bookmark"></i>
        {% for category in post.categories %}
          <a href="/categories.html#{{ category }}-ref" target="_blank">
            <span class="label post-category">{{ category }}</span>
          </a>
        {% endfor %}
      {% endunless %}
       &nbsp;
      {% unless post.tags == empty %}
        <i class="icon-tag"></i>
        {% for tag in post.tags %}
          <a href="/tags.html#{{ tag }}-ref" target="_blank">
            <span class="label post-tag">{{ tag }}</span>
          </a>
        {% endfor %}
      {% endunless %}
  </div>
  <br /><br />
  <div class="entry">
    {{ post.content | split: '<!--more-->' | first }}
    <p>
      <a href="{{ post.url }}/#more" class="more-link btn">
        <span class="readmore">阅读全文 &raquo;</span>
      </a>
    </p>
  </div>
</div>
<br />
{% endfor %}

<!-- Pagination links -->
<div class="pagination pagination-centered">
  <hr />
  <ul>
    <li class="disabled"><a>{{ paginator.page }} / {{ paginator.total_pages }}</a></li>
    <li><a href="/">&laquo;</a></li>
    {% if paginator.previous_page %}
      {% if paginator.previous_page == 1 %}
        <li><a href="/" class="current">&lt;</a></li>
      {% else %}
        <li><a href="/page{{paginator.previous_page}}/">&lt;</a></li>
      {% endif %}
    {% else %}
      <li class="disabled"><a>&lt;</a></li>
    {% endif %}
    {% for count in (2..paginator.total_pages) limit:6 %}
      {% if count == paginator.page %}
        <li class="active"><a>{{count}}</a></li>
      {% else %}
        <li><a href="/page{{count}}/">{{count}}</a></li>
      {% endif %}
    {% endfor %}
    {% if paginator.next_page %}
      <li><a href="/page{{paginator.next_page}}/">&gt;</a></li>
    {% else %}
      <li class="disabled"><a>&gt;</a></li>
    {% endif %}
    <li><a href="/page{{paginator.total_pages}}/">&raquo;</a></li>
  </ul>
</div>
<br />
<br />
```

{% endraw %}

头部就是主标题副标题，这个没啥说的，下面开始，`paginator.posts`里保存的是该页的所有文章信息，每篇文章的结构大致是`.post-wrapper > .entry-title > .entry-meta > .entry`的`div`嵌套，这些样式你可以在`assets/themes/hooligan/css/style.css`里自定义。其中可以看到`entry-meta`里会输出文章的分类信息和标签信息，`.entry`里输出摘要。摘要如何实现网上有千奇百怪的版本，我也试了很多，心力憔悴，最后发现居然一句话就能实现，网上还有很多为了这个自己写插件的。。。其实想想原理就不难，因为 wordpress 里使用`<!--more-->`来显示摘要，只要在`post.content`里查找`<!--more-->`的位置显示其前面的内容就是摘要啊！所以`{{ "{{ post.content | split: '<!--more-->' | first "}}}}`用这种类似管道的`filter`方式，先以`<!--more-->`调用`split`分割，然后用`first`取前面即可。其实我不太明白为什么直接写`{{"{{ post.content.split('!--more-->').first "}}}}`为啥不行。。。有懂的可以告诉我～后面的部分就是分页的页码了，我这里为了风格统一，套用了`bootstrap`的分页样式，具体可以看`bootstrap`的[分页文档](http://v2.bootcss.com/components.html#pagination)。这样“高大上”的首页就诞生了。

### 大段代码的显示

大段代码的显示推荐使用 github 的[gist](https://gist.github.com)，否则转义累死人啊！上面的大段代码就是用的 gist 显示的，配合了自定义的 gist 样式。这里值得说一下，gist 也是个坑！关于在 jekyll 嵌入 gist，可以使用官方的方案，直接写`<script src="https://gist.github.com/yourname/xxx.js"></script>`，也可以用 jekyll 的 gist 标签：`{% raw %}{% gist xxx %}{% endraw %}`。很不幸，这两种方案都有问题，本地预览时正常显示，但上传到 github 后发现，gist 后面的内容被截断，匪夷所思。经过一番算搜索发现，又是 jekyll 的标签解析惹得货啊。详细可以看这两篇文章：[Fixing your embedding gist snippets](http://mystcolor.me/post/22455268027/fixing-your-embedding-gist-snippets) 和 [Why don't self-closing script tags work?](http://stackoverflow.com/questions/69913/why-dont-self-closing-script-tags-work)。简单来说就是，这两种方式会生成`<script src="https://gist.github.com/yourname/xxx.js" />`这样的标签，而很多浏览器解析这样的“自关闭标签”有 bug，很奇怪吧，非常简单的问题居然有 bug！！那怎么破呢，非常简单，加个空格即可，如：`<script src="https://gist.github.com/yourname/xxx.js"> </script>`。

### push 到 github 时没效果

一切就绪，push 到 github 去吧，相信很多人遇到过这个问题：明明我本地预览正常，为啥 push 到 github 不生效呢？快去查查邮箱，有没有收到 github 给你发的`page build failure`邮件啊，我是收到过五六封。。。其实这是因为 github 为了安全考虑禁止使用第三方插件，所以首先删除你`_plugins`文件夹下除了`debug.rb`以外的其他文件，然后本地预览时使用`jekyll serve --safe --watch`命令，`--safe`确保满足 github 的安全要求，`--watch`可以不用重启 jekyll 使修改即时生效。

### 总结

好了，博客现在差不多了，但对美有要求的你应该不会停止。`jekyll-bootstrap`由于是以`bootstrap`为基础的，所以熟悉前端的孩子修改起来特别方便，比如首页加个侧边栏，各种样式修改，加入`font-awesome`图标等等，慢慢探索吧～用`bootstrap`的另外一大好处就是，这个框架本来就是响应式的，所以不用费心去像 wordpress 一样装个插件去优化手机版，各种尺寸的屏幕都能做到自适应，非常方便！快把你的博客也迁过来吧！
