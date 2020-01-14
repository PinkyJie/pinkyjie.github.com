date: 2010-09-01 11:21:28
title: MyTexPoint——在PowerPoint中使用Tex
categories:
- 瞎折腾
tags:
- latex
- MyTexPoint
- PowerPoint
- 公式编辑
---

昨天在写wordpress上写日志时，发现要编辑许多公式，一番搜索之下发现了Using Latex这个插件(详情请[猛击这里](http://www.dutor.net/index.php/2010/05/wordpress-using-latex/))，Latex在编辑公式方面我觉得是无可替代的王者。今天上午做PPT的时候，又被很多公式弄的心烦，MathType实在太不好使了，自带的就更不用说了。我突发奇想，是不是PowerPoint里面也可以使用Latex来制作公式呢？Google之，果然有！一个叫TexPoint的收费东东，本着不用收费软件的宗旨(画外音：放P！你是没找到破解版吧~)，继续搜索之下发现了这个好东东——MyTexPoint([官网](http://thd.pnpi.spb.ru/~gromov/mytexpoint.html))，人家的介绍就是Free simplified version of TeXPoint，免费简化版的TexPoint，嘿嘿，针对性多强啊~

<!--more-->

下面就由我带领大家领略一下MyTexPoint的风骚吧！猛击[这里](http://thd.pnpi.spb.ru/~gromov/ccount.php?zip)从官方下载，下载下来是个压缩包，绿色版的，随便解压个地方就好。值得注意的是本软件需要你的电脑上已经安装好了[MiKTex](http://miktex.org/) 和 [ghostscript](http://www.ghostscript.com/)，猜想可能需要调用他俩进行编译，安装过[CTex](http://www.ctex.org/HomePage)的朋友这两个东西默认都是装好的。

由于该软件并不像MathType那样与Office集成，所以无法在PowerPoint中直接调用它。每次使用的时候，我们需要先打开PowerPoint(本文以2003为例，因为俺觉得07和10都没有03快~官方声明是兼容07和10版的)，然后打开MyTexPoint.exe文件，发现出现了这个一个小窗口，不仔细找还真发现不了。。。


![](/assets/images/mytexpoint-powerpoint-tex-1.jpg)


点击右边的新建按钮，新建一个公式，此时，会同时出现两个东东，如下图①和②：


![](/assets/images/mytexpoint-powerpoint-tex-2.jpg)


①即为出现在PPT上的公式，以图片的形式存在，②即为新打开的Latex编辑窗口，熟悉Latex的孩子对此应该不陌生吧，我们需要更改的只是Color这一行和下面一行的公式部分，下面我随意输入一个公式。根据我们的PPT背景，黄色的公式比较鲜明一些，首先，我们将默认的blue改为yellow，然后添加一句公式，如下：


![](/assets/images/mytexpoint-powerpoint-tex-3.jpg)


这行代码将生成一个简单的方差公式，编辑完成后，点击绿色的运行按钮，此时将调用你机器的上两个软件进行编译，会出现dos窗口，就跟平时我们用Latex的编译窗口一样的，编译完成后，PPT原来的图片new equation就变成了我们想要的公式了。如图，公式是以普通图片形式 出现的，我们可以随意移动调整大小。


![](/assets/images/mytexpoint-powerpoint-tex-4.jpg)


公式可能有些大，可以自行调整，若需要修改公式，在保证MyTexPoint打开的情况下，点一下PPT上的公式即可激活代码编辑窗口。

经过一下午的使用，我觉得还是很方便的，像这样一个公式如果使用MathType输入的话要键盘鼠标并用，还要寻找，而使用MyTexPoint只需熟悉相关的命令即可，维基百科上有一个Tex的[数学公式命令速查表](http://zh.wikipedia.org/zh-cn/Help:%E6%95%B0%E5%AD%A6%E5%85%AC%E5%BC%8F)，当工具很不错哦~当然，这个软件的唯一一个小缺憾就是不能有效地和PowerPoint集成，每次使用时需要自己打开，希望以后能改进吧，毕竟人家是免费的~
