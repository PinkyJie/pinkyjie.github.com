date: 2010-07-28 12:43:30
title: elefant使用心得(一)
categories:
- 模式识别
tags:
- elefant
- nicta
- 入门
- 教程
- 机器学习
---

最近一周一直在搞这个称之为“大象”的工具箱，发现网上这方面的资料基本没有，并且这个工具箱的文档垃圾的一踏，遂决定把我这几天研究这个的心得写下来，供有兴趣的同学参考，也防止以后我也忘了。。。

<!--more-->

### 关于Elefant项目

{% img center-img http://elefant.developer.nicta.com.au/portal_skins/custom/ElefantLogo.png ElefantLogo %}

Elefant (大象)是为机器学习而开发的一个开源的库，免费，跨平台。官方的介绍就一句话：Efficient Learning, Large-scale Inference, and Optimisation Toolkit，它是有澳大利亚的NICTA公司开发的。有兴趣的可以去它的[官网](http://elefant.developer.nicta.com.au/)看看！(附官方特性介绍，呵呵～)


  * algorithms for machine learning utilising the power of  multi-core/multi-threaded processors/operating systems (Linux, WIndows,  Mac OS X)


  * a graphical user interface for users who want to quickly prototype machine learning experiments


关于Elefant的安装请参看官网的[Install Guide](http://elefant.developer.nicta.com.au/Getting_started/Install_guide)，讲的很细致的，总结起来也就是下载一个tar包解压，然后运行一堆命令。由于这个工具箱是基于Python的，所以需要您的机器上预先装好python，后续的安装命令都是需要Python解释器的，会自动下载Elefant运行所依赖的一些包(主要是Python的扩展包，包括numpy，scipy，matplotlab等等)。说到这个安装也有点奇怪，我在Windows上装过，一运行就报错`Python.exe已停止工作`，同学在Mac下尝试安装，也是莫名其妙的错误，为了搞这个我只好又装上了我的小奔图(Ubuntu 10.04)，在奔图下的安装进程是很顺利的，以下的所有操作都是在奔图下进行的。

### 安装及概览

安装了以后打开运行`Alt+F2`，输入`launchelefant.py`即可打开Elefant的GUI界面，界面如下:


![](/assets/images/elefant-introduction1-1.png)


我们看到Elefant实际上是由一些组件(Component)组成的，包括Data，用于处理数据的导入到处；Algorithms，包含各种算法；Modol Selections，解决模型选择的问题，Plots，画图，这就是Elefant基于组件的设计理念，其最大的好处就是可以自由扩充组建，这个后面我们会讲到。我们可以点击菜单下的一个组建试试，这里点击Algorithms>Support Vector Machine>Classification>Nu SVM Classification，这个算法实际上就是_New Support Vector Algorithms_ 这篇论文中提到的算法。我们看到点击后的截图如下:


![](/assets/images/elefant-introduction1-2.png)


先来关注一下界面中“不太重要”的也就是右边的元素：最上面的Help就是关于当前选中组件的帮助了，往下的Properties就是这个组件的一些属性，再往下Execute Flow中分上下两个字窗口，上面的是该组件的方法(Method)，下面的是整个算法的方法执行顺序。看完这些以后我们来看最重要的，也就是界面中央的东西，界面中央空白区出现了个矩形，它表示我们的算法，矩形上带有的小矩形则代表“端口(ports)”，这有点类似网络的端口概念：两个程序通过端口来进行通信，只要它们遵循同样的协议即可。同样地，在Elefant中，不同的组件间的通信也是靠这些各自提供的端口来实现的。这个我们下面会详细说，然后参照下图我们再拖三个组件上来，如下图:


![](/assets/images/elefant-introduction1-3.png)


从图中我们可以看到四个组件：`CSV Data Source`，`CSV Data Source 1`，`Nu SVM Classification`，`Two Dimensional Scatter Plot`。顾名思义，前两个是CSV格式的数据源，它们来自Data菜单下的Reader子菜单，我们的算法通过这两个组件获取数据，然后第四个是一个二维数据的Plot，也就是将二维数据以散点图方式画出来的组件，来自Plot菜单。### 每个组件有自己的端口，端口绑定着协议(Protocol)，协议之上绑定着数据(Data)，端口之间可以相连，只要它们的协议一样，就可以传递数据。这就是Elefant工作的核心理念。我们看到端口有红绿之分，红色代表着OUT_PORT，这种端口负责将数据传递出去；绿色则是IN_PORT，负责接受其他端口传来的数据。这样以来我们就明白上图的意思了吧。两个Data Source一个代表训练样本数据，一个代表测试样本数据，通过自己的端口将数据传递给算法组件的对应端口，算法通过自身的方法(Method)进行训练，测试，最后通过TestOut端口将测试得到的测试标签传给Plot组件，将结果以图形表示出来，当然，只限于二维数据才能做图。得到这个图以后我们怎么做呢，下回分解吧，这文章写长了。。。
