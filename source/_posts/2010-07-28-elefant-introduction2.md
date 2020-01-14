date: 2010-07-28 12:51:15
title: elefant使用心得(二)
categories:
- 模式识别
tags:
- elefant
- nicta
- 入门
- 教程
- 机器学习
---

书接上回，我们得到[上篇文章](http://pinkyjie.com/2010/07/28/elefant使用心得一/)中的图以后，来看一个简单的小例子。

<!--more-->

### 一个简单的例子

我们来举一个简单的比较完整的例子。按[上篇文章](http://pinkyjie.com/2010/07/28/elefant使用心得一/)拖动组件并链接端口后，我们需要在右边的窗口中配置一下各个组件的属性，首先是两个CSV数据源组件，点击相应组件，我们看到数据源有如下4个属性和一个方法:


![](/assets/images/elefant-introduction2-1.png)


属性分别是数据的分隔符，如逗号，空格一类的；然后是数据源的文件路径，也可以是网址，官方提供了很多各种格式的[在线数据源](http://elefant-svn.developer.nicta.com.au/elefant/data/)，本例我们使用在线数据([训练](http://elefant-svn.developer.nicta.com.au/elefant/data/csv/train_cl.csv)，[测试](http://elefant-svn.developer.nicta.com.au/elefant/data/csv/test_mu.csv))\[PS1  本人为了找这些个网址操碎了心，因为官方帮助文档给的地址是错的，再次Fuck一下官方文档\]\[PS2  注意，本人的电脑上这个属性无法修改，即修改了也不生效，可能是兼容性问题，如果你的也是这样，不要急，我们一会还有别的方法\]；第三项是数据文件的那一 列是标签，这个选项可以是first，last，None，选None就表示该数据源没有标签；最后一项则是读取的行数，填-1就代表全部读进来。而 LoadData()方法顾名思义了，就是读取数据啦。

分别配置好两个数据源之后，我们点击算法组件，看到它有两个属性和两个方法:


![](/assets/images/elefant-introduction2-2.png)


Kernel属性就是配置核的相关参数，可以选择使用什么核，核的大小及参数都可以轻松配置，下面的Nu自己就是本算法独有的参数了(详细了解该算 法请看上文提到的论文呢)，另外两个方法也很好懂，Train()和Test()，训练和测试。最后我们来配置Plot画图组件，它的属性和方法都比较容 易，一个是画图标题，一个是Plot()做图方法:

![](/assets/images/elefant-introduction2-3.png)

配置好属性后，我们配置Excute Flow中的执行流程，也就是添加各个组件的方法并调整执行顺序，可以点击各个组件，找到对应方法，点击Add按钮即可，最终只行顺序如下:

![](/assets/images/elefant-introduction2-4.png)

一切就绪，点击Execute按钮，算法开始执行，如果一切正常，就会出现如下图的结果:


![](/assets/images/elefant-introduction2-5.png)


如果你和我一样无法修改文件路径的话，那请继续看下一节的内容吧。
