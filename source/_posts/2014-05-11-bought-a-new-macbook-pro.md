date: 2014-05-11 19:00
title: 入手Macbook Pro
categories:
- 生活点滴
tags:
- Mac
- MacbookPro
- 苹果团
- dotfiles
- Alfred
- Homebrew
---

入手Macbook Pro大概一个多月了，早就想写篇文章记录一下，一直懒得动笔拖到现在。。。这篇文章就记录下从购买到使用的一些细枝末节的小事吧。

<!--more-->

### 买啥型号，从哪买

买之前就琢磨这回事来着，尺寸方面原来其实很中意15寸的，觉得13寸的做开发有点小，后来去Apple Store真机体验了下，15寸的太尼玛大了，最后还是选了13寸。配置方面考虑到价格因素还是选了中配的ME865，低配感觉硬盘太小，高配是挺好但是买不起。至于从哪买，本着经济实惠的考虑当然首选港行网购了，先前就知道[苹果团](http://appletuan.com)这么一家，觉得这模式还挺有意思的，价格每天跟随华强北变动，然后在其报价基础上透明加价，感觉混v2ex的好多都在那边买过，质量什么的也挺有保障的。后来关注了一段时间，发现类似的站也蛮多的，比如前端大神[sofish](https://twitter.com/sofish)的妹妹开的[小闷的水果店](http://appled.cc)，哥哥的名气和信用在那放着，应该也挺靠谱的。不过最后还是犯了中国人“爱凑热闹”的坏毛病，选了苹果团，晚上下单第二天下午就到了，顺丰的物流非常给力。开箱随便看了下电池循环，3次还算正常吧，不过感觉这玩意也就是个心理安慰。网上还有根据序列号查保修的文章，不过我这个到官网上以后让我手动输入购买日期，也不知道算咋回事。

### 装机

既然买了Macbook，我的想法就是坚决不装Windows，实在有需要就来个虚拟机凑合一下就行了。其实以前就有过一些Mac系统的使用经验，我觉得Mac综合了Windows和Linux的优点，看着好看，然后又有各种命令行工具。首先必装的就是“Mac上的apt-get”——[Homebrew](http://brew.sh/)，一些常用的curl、wget等工具都可以用这个来安装，非常方便。另外还有它的好兄弟——[Homebrew-Cask](https://github.com/caskroom/homebrew-cask)，这玩意可以非常方便的安装应用软件。虽然Mac上的好多软件类似Windows上的绿色软件，下载dmg格式的安装文件，双击打开，图标拖入Applications文件夹即可。但是拖这一步感觉还是很闹心，而Homebrew-Cask就是解决这个问题的，可以使用命令行轻松的安装应用软件并自动软链接到Applications目录中，有了这个，自动装机不是梦啊。如果用Mac来做开发的话，命令行是每天都要接触的东西，一个好用的shell必不可少。[zsh](http://www.zsh.org/)和[oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)是一个不错的选择，好看的主题，各种实用的扩展让你赏心悦目。另外，使用命令行就不可避免的要同各种dotfiles打交道，使用Git来维护这些文件是个很棒的创意，比如[我的dotfiles仓库](https://github.com/PinkyJie/dotfiles)。详细的装机设置可以参考[这篇文章](http://www.yangzhiping.com/tech/mac-dev.html)。

### 常用软件

说到常用软件，可以参考下[BestApp的Mac软件推荐](https://github.com/hzlzh/Best-App/blob/master/README.md)，不过很多都是付费的，感觉用了Mac少不了要花点钱买软件了。。。说说我现在常用的吧：

* [Alfred](http://www.alfredapp.com/) - 绝对神器，类似于启动器，方便启动各种App，不过其最牛逼的功能还要数workflow了，可以轻松的实现查天气、翻译等各种实用的功能。不过workflow这项功能是需要付费的，这也是我目前唯一买的一个软件了。
* [Spectacle](http://spectacleapp.com/) - 窗口管理工具，在Windows上可以非常方便的将窗口居左居右居上居下，而这个工具就是通过快捷键在Mac上实现类似的功能，简单使用。
* [aria2](http://aria2.sourceforge.net/) - 一个命令行下载工具，强烈推荐，非常好用，配合[Yaaw](https://github.com/binux/yaaw)可视化工具可以轻松的监视其下载进程。另外还有其[百度云盘插件](https://chrome.google.com/webstore/detail/mblmc%E8%BF%85%E9%9B%B7%E7%A6%BB%E7%BA%BFqq%E6%97%8B%E9%A3%8E%E7%99%BE%E5%BA%A6%E7%BD%91%E7%9B%98360%E4%BA%91%E7%9B%98%E7%AD%89ar/iamaphkapjbdhhpdapkalhanifedeged)，安装以后会在云盘的网站上显示一个按钮，调用aria2下载百度云盘的文件。亲身体验，家里20M的宽带，速度可达2.5M左右，非常快。
* [MplayerX](http://mplayerx.org/) - 全能播放器。
* [Dash](http://kapeli.com/dash) - 整合各种开发文档，方便速查，除了图标磕碜点其他没啥缺点。
* [The unarchiver](http://wakaba.c3.cx/s/apps/unarchiver) - 解压缩工具，基本够用吧。网上有说兼容性不好，不过我目前没碰到不能解压的情况。

其他Windows常用的也基本都一样，比如QQ、旺旺、evernote一类的。值得一提的就是这些软件都可以通过Homebrew-Cask安装哦。

### iOS开发

买了Macbook自然是想学点iOS开发了，目前在跟着斯坦福的公开课学，[网易公开课](http://open.163.com/special/opencourse/ios7.html)有字幕翻译，感觉讲的很到位，课后的作业也很有挑战性。如果遇到作业不会做，还可以参考[这个网站](http://cs193p.m2m.at/)，有代码有讲解，非常到位。我们的目标是：像Flappy Bird作者那样日入5万刀！














