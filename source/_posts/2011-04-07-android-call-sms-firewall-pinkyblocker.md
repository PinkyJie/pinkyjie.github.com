date: 2011-04-07 12:08:39
title: PinkyBlocker -- 简易的 Android 来电短信防火墙
categories:

- Android 开发

tags:

- Android
- PinkyBlocker
- 拦截
- 来电防火墙
- 短信防火墙

---

最近研究了下 Android 下的开发，搞出了这么一个东西。

软件名称：PinkyBlocker (以后咱开发的东西都以 Pinky 开头好了)

软件版本：v 1.0 (测试版本)

适用系统：Android 1.6 及以上

<!--more-->

---

### Update 2011-4-12

添加从通讯录和通话记录中选择号码的功能(仅当选择的规则类型为“特定号码”时才会出现)，截图如下：

![](/assets/images/android-call-sms-firewall-pinkyblocker-1.png)

![](/assets/images/android-call-sms-firewall-pinkyblocker-2.png)

---

PinkyBlocker 是一款简单实用的来电和短信防火墙，通过简单的规则设置即可轻松拦截短信，拒接电话，让你的世界从此清静！

### 功能特性：

- 支持两种方式添加来电拦截：

1. 【以某几个数字开头的号码来电】 添加区号即可拦截对应城市的市话

2) 【特定号码的来电】 即传统的黑名单

- 支持三种方式添加短信拦截：

1. 【以某几个数字开头的号码发来的短信】

2) 【特定号码发来的短信】

3. 【包含某些内容的短信】即传统的关键字内容过滤

- 拦截后可以设置状态栏提醒和声音提醒

* 退出后依然可以正常拦截

### 软件截图：

![](/assets/images/android-call-sms-firewall-pinkyblocker-3.png)

{% blockquote %}
软件安装界面和关于界面，这里的“拨打电话”权限是为了拦截电话，本软件不会自动往外拨电话。
{% endblockquote %}

![](/assets/images/android-call-sms-firewall-pinkyblocker-4.png)

{% blockquote %}
软件主界面，由两个 tab 主成，一个是拦截规则，一个是拦截记录。
{% endblockquote %}

![](/assets/images/android-call-sms-firewall-pinkyblocker-5.png)

{% blockquote %}
添加来电拦截规则的页面，有两种选择，拦截特定的号码或者拦截以某些数字开头的号码。
{% endblockquote %}

![](/assets/images/android-call-sms-firewall-pinkyblocker-6.png)

{% blockquote %}
添加短信拦截规则的页面，有三种选择，前两种和来电的一样，最后还可以拦截包含某些字词的短信。
{% endblockquote %}

![](/assets/images/android-call-sms-firewall-pinkyblocker-7.png)

{% blockquote %}
软件的主菜单，以及长按规则列表出现的上下文菜单。哥有退出菜单，看到没，没有退出菜单和一按返回键就退出神马的最讨厌啦~
{% endblockquote %}

![](/assets/images/android-call-sms-firewall-pinkyblocker-8.png)

{% blockquote %}
拦截后会出现状态栏提醒，点击后直接进入拦截日志界面。
{% endblockquote %}

![](/assets/images/android-call-sms-firewall-pinkyblocker-9.png)

{% blockquote %}
软件的设置界面，通过设置菜单可以设置是否打开拦截服务，拦截后是否出现状态栏提醒，是否有拦截声音等。
{% endblockquote %}

![](/assets/images/android-call-sms-firewall-pinkyblocker-10.png)

{% blockquote %}
退出程序时的提示，按返回键或退出菜单都会出现该提示，退出后依然可以正常的拦截。
{% endblockquote %}

### 操作视频：

我用模拟器录制了一段操作视频，不太清晰，大家凑合看看吧，一看就懂的！

<p>
<embed src="http://player.youku.com/player.php/sid/XMjU2NzU4MzE2/v.swf" allowFullScreen="true" quality="high" width="480" height="400" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash"></embed>
</p>

### 已知问题：

由于手头只有 G1 和模拟器，测试范围很窄，模拟器的效果参见上面的视频，各项功能还是很正常的。至于在我的 G1 上的测试结果，有以下几个问题：

- 来电拦截时有延迟，会出现像响一声才被挂断的情况，貌似是 Android 上的通病

* 来电拦截的时候，有时会出现拦截一个电话记录多次，提醒多次的情况，不知是我的 G1 太慢还是什么别的原因

总之，基本不影响使用吧！

### 想添加的功能：

在清除 bug 的同时，近期还想添加一些实用的功能：

- <del>添加号码时可以选择从通话记录或联系人添加</del>（已添加）

* 考虑实现同时满足多个规则时才拦截

### 最后放上下载链接 [点我下载](/assets/files/PinkyBlocker.apk1)，有兴趣的可以用下试试，有问题向我反馈！

> 下载后请自行修改扩展名为 apk

### （刚刚 fix 了一个 bug，大分辨率的机器伤不起啊，增加了大图标支持高分辨率机器，排版不错乱了~）
