date: 2010-10-19 07:18:34
title: Ubuntu下Opencv与Python的协作
categories:
- 模式识别
tags:
- OpenCV
- OpenCV+Python
- Python
- Ubuntu
---

暑假时就想写一篇关于Ubuntu下配置Opencv与Python协作的文章，苦于当时自己也不是搞得特别清楚，正好最近重装了个Ubuntu 10.10，重新配置了一遍，才算真正明白。网上关于这个的文章虽然一大把，但不是太复杂就是根本不管用，这个配置还是有点闹心的。

<!--more-->

### Opencv的编译和安装

Linux下貌似只能下载Opencv的源码了，来到[Opencv的中文官网](http://www.opencv.org.cn/index.php/Download)，最新版本是2.1，点击[OpenCV for Linux](http://www.opencv.org.cn/download/OpenCV-2.1.0.tar.bz2)，下载下来的是一个压缩包，随便解压到你喜欢的位置，以我的机器为例，解压到 `/MySoft`下，最终的目录结构为： `/home/pinky/MySoft/OpenCV-2.1.0/`。由于下载得到的是源码，需要进行编译，官网推荐的编译方式是使用cmake，这里我是用的是cmake的gui版本，即`cmke-qt-gui`，首先我们需要使用终端安装它。打开终端，运行命令：

``` bash
sudo apt-get install cmake-qt-gui
```

安装完成之后，按理说应该可以正常编译了，但是我的机器直接编译会报错，提示需要安装libgtk2.0-dev这个东东，不知是我的机器问题还是都需要这么做，这个软件具体干嘛的我也不知道，推荐先安装一下，终端运行以下命令：

``` bash
sudo apt-get install aptitude
sudo aptitude install libgtk2.0-dev
```

aptitude 可以说是apt-get的一个GUI界面，不过这里用它可能是因为它可以自动安装所有的依赖包吧。这个安装以后，为了后面和Python协作，还需要检查一下你机器上面的Python相关依赖，终端运行命令：

``` bash
apt-get install libpython2.6 python-dev python2.6-dev
```

这样以后，编译前所做的工作就基本结束了，从“应用程序”中的“编程”子菜单中打开cmake，Source code这一栏中填写你的OpenCV解压的路径，第二行是编译后的可执行文件放在哪里，可以在OpenCV的路径下新建一个文件夹，如release文件夹，则编译结果会放到其中去，点击Configure按钮，接着出现选择generator的界面，使用默认选项，即Unix Makefiles，点击finish，如果一切顺利的话就会出现以下界面：


![](/assets/images/ubuntu-opencv-python-1.png)


正常的话上图中底下的白色框中是有信息的，我的截图是在编译过后截的，所以是空白，接着选择编译参数，这里我们只需要注意以下几个参数有没有被选中，其他的默认即可：


> BUILD-EXAMPLES 生成例子
BUILD-NEW-PYTHON-SUPPORT 生成新的python支持
OPENCV-BUILD-3RDPARTY-LIBS 生成第三方的库
INSTALL-C-EXAMPLES 安装C的例子
INSTALL-PYTHON-EXAMPLES安装python的例子


然后，点击Generate按钮，就可以生成可执行的文件了，可以看到release目录下已经有很多文件，下面我们要对这些文件进行make。打开终端，进入release所在文件夹：

``` bash
cd  /home/pinky/MySoft/OpenCV-2.1.0/
make
```


make过程大概需要几分钟左右，过程如下图这样：


![](/assets/images/ubuntu-opencv-python-2.png)


![](/assets/images/ubuntu-opencv-python-3.png)

最终出现的cv.so文件很重要，它就是OpenCV和Python协作的关键，然后再执行install步骤，终端继续运行：

``` bash
sudo make install
```


过程大概如下图：


![](/assets/images/ubuntu-opencv-python-4.png)


这样以后，OpenCV的安装步骤宣告结束。

### 代码测试

下面我们用代码测试一下安装是否正确。首先是一段C代码：

``` c
#include "cv.h"
#include "highgui.h"

void main()
{
    IplImage* img;
    img = cvLoadImage("test.jpg",1);
    cvNamedWindow("ShowImage",1);
    cvShowImage("ShowImage",img);
    cvWaitKey(0);
}
```


使用gcc进行编译链接，终端输入命令：

``` bash
gcc `pkg-config --cflags --libs opencv` -o testCV testCV.c
```


注意上面的引号是“反单引号”，在数字1的左面，cflags和libs前面都是两个“连接线”，其中pkg-config用来定位OpenCV的头文件和库文件，testCV为生成的可执行文件，testCV.c为源文件名称。如果编译报错，可能是pkg-config的配置还没有生效，在终端中运行sudo ldconfig使之生效。编译完成后，输入 ./testCV 运行，test.jpg文件就被显示出来了。

C代码测试成功之后，我们来看看Python。在终端打开输入python打开python的编译环境，输入import cv，看看是否能成功。这里可能会报"No module named cv"错误，原因就是python找不到我们刚才提到的cv.so文件，我们进入路径/usr/local/lib/python2.6中发现有两个文件夹，一个是dist-package，一个是site-package，python默认只在前者里搜索，而我们生成的cv.so文件则在后者的文件夹里(如果你的两个文件夹都没有这个文件，则前面的make步骤出问题了，可能需要重新走一遍整个安装过程)，我们只需要简单的将它移动到dist-package文件夹即可。终端运行命令：

``` bash
sudo mv /usr/local/lib/python2.6/site-packages/cv.so /usr/local/lib/python2.6/dist-packages/cv.so
```

执行之后，再import一下试试，就可以了，下面写一段简单的python代码：

``` python
import cv

if __name__ == '__main__':
    img = cv.LoadImageM ("test.jpg")
    cv.NamedWindow ("ShowImage")
    cv.ShowImage ("ShowImage", img)
    cv.WaitKey (0)
```


像普通python代码那样运行它即可，即python testCVPY.py，图片就能显示出来了。至此，尽情的使用python享受Opencv吧！
