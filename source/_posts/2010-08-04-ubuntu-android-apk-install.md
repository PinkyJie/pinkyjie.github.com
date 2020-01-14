date: 2010-08-04 18:15:21
title: Ubuntu下Android手机apk软件的安装
categories:
- 瞎折腾
tags:
- Android
- ssh
- Ubuntu
- 翻墙
- 软件安装
---

最新由于学术加装B的双重需要，有时候不得不用一下奔图，不过离开了Windows，给俺的G1安装软件就成了必须要面对的大问题，什么91手机助手，豌豆荚这一类的东西统统没有Linux版，根据中国的环境估计以后也不会有，那要管理手机只能寻求最原始的方式了，那就是Android SDK。

<!--more-->

### 下载Android SDK for Linux

本来这个下载SDK没啥可写的，但因为你我都在天朝，这东西就可以讲很多了，因为[Android Developer网站](http://developer.android.com/index.html)被墙了，俺用脚趾头想了个死去活来也没想通，最后只能用一句话来搪塞：因为这是在天朝。会翻Wall的朋友请狂击[本链接](http://dl.google.com/android/android-sdk_r06-linux_86.tgz)下载，然后跳过本节(囧)，不会的朋友没关系，我在这里简要写一下奔图下怎样使用ssh翻Wall，很有用的哦。

首先，到CJB.NET申请一个免费ssh帐号，狂击[注册地址](http://www.cjb.net/cgi-bin/shell.cgi?action=signup)，如下图：


![](/assets/images/ubuntu-android-apk-install-1.png)


填完资料后会发一封邮件到你的邮箱，点击邮箱中的地址激活帐号，如下图，画黑线那个地址。(我不禁觉着这教程太傻瓜了。。。)


![](/assets/images/ubuntu-android-apk-install-2.png)


激活后，又会发一封邮件给你，这封邮件包含的就是你的帐号和密码。


![](/assets/images/ubuntu-android-apk-install-3.png)


记住前面的三项就好，这就是我们的ssh的服务器地址和你的帐号密码。每次我们要翻Wall时都需要打开终端输一堆命令，为简单起见，我们新建一个空文档命名为Crosswall(名字随便起)，里面输入 ### ssh -D 6666 你的帐号@服务器地址 保存，其中6666是端口号，随便写一个就好，帐号和服务器地址来自邮件中。然后我们打开终端，切换目录到该文件夹，输入sh Crosswall就行了，稍候会提示你输入密码，不出以外的话就会出现下图，说明已经成功啦！


![](/assets/images/ubuntu-android-apk-install-4.png)


翻Wall的时候这个窗口不能关哦，翻累了就输入exit命令就会退出ssh了。配合firefox中大名鼎鼎的[AutoProxy插件](https://addons.mozilla.org/zh-CN/firefox/addon/11009/)，就可以自动实现翻墙啦，关于AutoProxy的介绍请Google。打开AutoProxy首选项，在“代理服务器”菜单中选择“编辑代理服务器菜单”，将ssh -D的端口改成你刚才指定的端口号，如下图：


![](/assets/images/ubuntu-android-apk-install-5.png)


至此，终于可以如愿下载SDK了！哎，感谢国家让我这一节可以写这么长，哦耶！


### 安装和配置SDK

说是安装，其实只要加压一下就好了，将下载下来的android-sdk_r06-linux_86.tgz解压到任意目录，然后讲子目录tools的路径添加到环境变量(环境变量是啥自行Google)中去，具体做法是先打开终端，输入 vim ~/.bashrc 命令，打开该配置文件，按Shift+G切到最后一行，按i进入插入模式，输入 ### export PATH=$PATH:/home/pinky/Download/android-sdk-linux_86/tools (以我的安装路径为例，请自行改动)，最后按Esc退出插入模式，按:进入冒号命令模式，输入wq保存退出，这样环境变量就添加完毕了。重启终端，输入adb命令，如果出现如下图的一大堆东西，就表示配置成功了。我们安装这个软件就靠这个被称之为Android Debug Bridge(安卓调试桥，很艺术阿)的命令了。


![](/assets/images/ubuntu-android-apk-install-6.png)


### 安装apk软件

手机连上电脑，在你的apk所在文件打开终端，当然你也可以打开终端切换到apk所在目录，然后输入 ### adb install apk文件名 命令回车即可安装了。如下图：

![](/assets/images/ubuntu-android-apk-install-7.png)

是不是很简单，这安装过程比那翻Wall过程容易多了，发现这篇文章大篇幅就花在翻Wall上了。。。另外adb还有很多其他的功能，有兴趣的话可以自己钻研滴！就到这吧～
