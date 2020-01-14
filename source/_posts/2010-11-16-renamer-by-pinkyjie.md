date: 2010-11-16 12:14:39
title: “批量重命名”小工具
categories:
- 桌面开发
tags:
- nsis
- py2exe
- pyqt
- Python
- renamer
- 批量重命名
---

最近看了点Python的东西，加上前一段看的Qt，就写了这么一个“批量重命名”的工具。写下这篇文章，简单总结一下写代码的过程中遇到的一些乱七八糟的问题！

<!--more-->

### 小工具简介

这个小东西的编写主要基于Python，感觉Python很适合做类似的小东西。GUI界面使用的是PyQt框架，Qt的Python版本，两个东东都是跨平台的，我是在Windows下写的，不过这个东西在Linux下跑应该也是没问题的。我已经在Google Code([前往](http://code.google.com/p/renamer-by-pinkyjie))中建了项目，包含所有源代码和打包后的exe下载，以后我会继续完善这个东西，加一点我想要的功能。先来张截图吧，直观感觉一下。


![](/assets/images/renamer-by-pinkyjie-1.png)


界面就这一个啦，为啥用英文？为了zhuangbility~~~左边上面用来选择相应的文件夹路径，右边的Table显示文件名，可以使用扩展名来过滤，改后的文件名用蓝色表示。下面就是一些自定义的命名规则，可以自定义前缀，支持中文，下面可以选择递进的是字母还是数字。若果选字母，则“start with”框不可输入，只能下拉选择26个大小写字母，若选数字，则没有下拉，只能自己输入。最后的count用来表示递进部分占几位。配置好以后，点击Rename按钮即执行，结果限制在上面的Table中，左上角的路径选择也会同步更新。如果你的输入不符合要求，比如，选择了“Using 123”，而“Start with”却输入了别的乱七八糟的东西，会报错提示。使用起来比我介绍的药容易多了，Just Try it ！

### Python和PyQt的协作

PyQt是Qt的Python版本，Windows平台下直接可以exe安装的，下载可以参见[它的官网](http://www.riverbankcomputing.co.uk/software/pyqt/download)，使用exe安装后直接配备了Designer用来设计GUI界面的，接触过Qt的应该不陌生，无非就是拉拉控件神马的，当然，完全用代码也可以写，像我这比较懒的就直接拉吧。Designer会生成.ui的文件，如果你的窗体中使用了图标，比如上图中的可爱小西瓜，那么还会生成.qrc的资源文件。一个首要的问题就是Python怎么引用这么文件呢？一番摸索后发现，PyQt提供了两个小工具：pyuic4和pyrcc4，两个东西都已经配置好了环境变量，直接就可以使用，它们可以把ui文件和prc文件转化为py文件供Python源文件import使用。比如我们生成了rename.ui和rename.qrc：

``` bash
pyuic4 rename.ui -o rename_ui.py
pyrcc4 rename.qrc -o rename_rc.py
```

这里值得一说的就是，资源文件的命名最好采用.qrc文件名加_rc的方式，因为编译后的rename_ui.py文件最后一句通常是import资源文件，默认import的文件名就是rename_rc，所以如果不这样命名最后整合时会报错。生成py文件后，就可以通过import来引用了。打开rename_ui.py文件，发现里面有一个Ui_开头的类，Ui_后面跟的是你的主窗体名称。打开rename_rc.py文件，发现是将资源代码化了。所以在源文件中，我们可以通过这两句将其引用。

``` python
from rename_ui import Ui_mainWidget
import rename_rc
```

其实第二句也可以不要，忘了我们刚才说过，rename_ui.py文件最后一句已经将其引入了，不过Python不会出现重复引入模块的错误。引用了ui以后，我们需要在主类的init方法中，初始化ui。

``` python
self.ui = Ui_mainWidget()
self.ui.setupUi(self)
```

这样我们就可以通过self.ui来引用控件了，控件的名字和我们在Designer中的命名是一致的。

### 遇到的一系列问题

首先，我其实先用非GUI的方式实现了一个批量重命名的工具，算法很简单的，代码量也很短，因为没有各种容错代码。

``` python
import os
print '-'*50
print 'Image File Renamer v1.0'
print '-'*50
print 'Note: Put this file in the dirction you want to change!'
print '-'*50
pre = raw_input('Input the file name Prefix:')
start = int(raw_input('Input the start number:'))
count = int(raw_input('Input the count of the number\n[exp.1-&gt;0,2-&gt;00]:'))
fileList = os.listdir('.')
total = 0
for f in fileList:
    ext = os.path.splitext(f)[1]
    # print ext
    if ext in ('.jpg','.png','.bmp'):
        os.rename(f,pre + str(start).zfill(count) + ext)
        start += 1
        total += 1
print '-'*50
print '%d files changed!' % total
print 'your direction will be opened!'
os.system('explorer .')
```

这个是可以直接运行的，有兴趣的可以玩玩。但我把程序改成GUI时，很多乱七八糟的问题随之而来。给控件绑定事件处理是首先需要解决的，Qt中直接connect解决，但需要发出的SIGNAL的参数与接受的SLOT参数匹配，可以很多参数类型Python根本没有，被闹心的参数问题搞得晕头转向以后，终于发现了个简单的方法，如下：

``` python
self.ui.letterRadio.toggled.connect(self.letterToggled)
```

letterradio的toggled的事件可以直接通过connect函数连接自己编写的处理函数letterToggled，而在connect的过程中不需要考虑参数问题，让见鬼的参数都去屎吧~当然，实现处理函数的时候是要匹配相应参数的。

另外一个头疼的问题，就是中文编码问题，Python自己有string类型，而从GUI控件获取的类型是Qt的QString类型，这两种乱七八糟的掺和在一起，只处理英文当然很顺利，但一牵涉到中文处理就彻底乱了，后来发现了一篇奇文 [python String和PyQt QString的区别](http://www.scriptlearn.com/archives/1943)，才算搞清楚。高人都说，不想麻烦的话全部使用Unicode编码，这话果然没错。一开始我的程序，连碰到中文目录都会报错，更别说使用中文“批量命名”了。最后只得全部采用Unicode处理，简单的说，所有Qstring类型的变量都需要调用自身的`toLocal8Bit()`函数先转化为本地编码的8位格式，经验证，`toLocal8Bit()`在Windows上工作很好，但在Linux上会导致中文输入乱码，所以这里采用`toUtf8()`比较靠谱，然后再使用Python提供的工厂函数`Unicode()`转化为Unicode字符。举个简单例子：

``` python
prefix = unicode(self.ui.prefixEdit.text().toUtf8(),'utf8','ignore')
```

我们想将prefix控件(prefixEdit)的字符储存在prefix变量中，需要像上面这样做。这样就完美的解决了中文乱码的一系列问题。

### 一切为了zhuangbility

程序出来了怎么办，源代码传到Google Code上，这样B就装完了吗？NO~要编译成Windows上能用的exe才行啊，一番搜索(发现总是是一番搜索)之后，发现了Py2Exe这种东西，顾名思义啊，就是将Python程序转成exe用的，使用很简单的，编辑一个setup.py配置文件，一条命令`python setup.py py2exe`搞定，具体步骤可以参考其[官网](http://www.py2exe.org/)或google搜索。这样B就装好了吗？NO~NO~我只想说你在装B这方面too simple,sometimes naive！我们还要把它封装成Installer，让人家一双击就自动安装好，又一番搜索后发现了NSIS这个东西，强大的超乎想象，定制性很强，免费的东西，据说很多常用软件就用这个做的安装包，具体可参见其[官网](http://nsis.sourceforge.net/Main_Page)，这里推荐一个汉化的配置NSIS的辅助软件——NSIS eidt，再推荐一篇入门文章 [NSIS安装制作基础教程](http://www.360doc.com/content/08/0731/13/66250_1492542.shtml)，简直一学就会啊~OK，就到这里吧~
