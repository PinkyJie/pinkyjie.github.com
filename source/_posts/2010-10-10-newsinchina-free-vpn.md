date: 2010-10-10 03:01:19
title: 参与牧师公益计划，免费发放VPN
categories:
- 瞎折腾
tags:
- Twitter
- VPN
- VPS
- 公益
- 牧师
- 翻墙
---

### 2010.12.20 Update：停止发放，由于赞助时间只有三个月，所有VPN将在1月7日左右失效~


> ** 请大家不要重复留言，第一次留言需要审核，所以不能立即显示出来！！另外，发送VPN以后也不在博客一一回复，请直接查您的邮箱！有什么问题直接发邮件，不要在这里留言，谢谢配合！**


国庆假期期间牧师[@NewInChina](http://twitter.com/newsinchina)发表了一篇博文[消息：本人将出资1000元为无收入的国内学生购买VPS，便于他们提高IT技术](http://tuitui.info/archives/newsinchina-vps.html)，本人经过申请有幸参与了这项公益计划(No.4那个就是我了~)。经过几天的研究，现在已基本掌握了VPS的配置技巧，已经可以让她生出VPN啦，吼吼~现按照牧师的愿望，现在无偿分享一些VPN供大家使用，由于对VPN的流量消耗情况还不太清楚，首期准备初步放出30个，待对流量消耗有了统计后会继续免费发放。**需要说明的是本次购买的VPS是由牧师@NewInChina出资付费的，根据估算能撑三个月左右，三个月后账号就会自动失效！** 经过测试，本VPN的速度不是很快，看Youtube有点悬，不过上个Twitter，Facebook，查个资料什么的还是没问题的，严禁用于P2P！需要申请VPN的同学请在本文后留言，留下您的邮箱，无收入的大学生优先(虽然无法考证)~另外，由于本人也是刚接触VPS，可能设置啊稳定性方面有很多问题，大家在使用VPN过程中有什么问题及时跟我反馈~

<!--more-->

### VPN知识普及

1.我们为什么需要VPN？


> VPN(Virture Private Network，虚拟专用网)，关于VPN的一些概念大家可以查询[维基百科](http://zh.wikipedia.org/zh-cn/%E8%99%9B%E6%93%AC%E7%A7%81%E4%BA%BA%E7%B6%B2%E8%B7%AF)。简单地说，就是对通信进行了加密。所以，VPN不是天然用来翻墙的，它能用于翻墙其原因就是，连上了VPN相当于你的电脑被加入了一个远程的局域网，而这个局域网是可以翻墙的(因为VPS一般架设在国外)。那有的同学就会问，墙是啥，我为啥要翻墙，这个问题就比较复杂了，不过我可以简单通过一下几个方面说说~

> ①更广阔更及时的新闻来源。今年诺贝尔和平奖得主你知道是谁吗？对，小波，你认识他吗？为什么你一Google他的名字就重置呢，甚至你一搜“刘”这个字就重置呢？作为一个想了解一下诺贝尔奖得主的人，为什么连诺贝尔官网都上不去呢~如果你有这些疑问，无疑你需要翻墙，这还不够，想更及时的了解新闻，你需要上Twitter，各种突然事件的直播等着你。(关于Twiiter的普及知识，可以看看李笑来老师的[非官方指南](http://www.lixiaolai.com/index.php/archives/8993.html))

> ②科研学习的需要。听起来挺扯淡，是吧？那说明你不是个爱研究的孩子，嘿嘿。比如我想搞Android开发，当然得先上[Android开发官网](http://developer.android.com/)了，什么，上不去？为啥？你问我我还想问你为啥呢！我想学Python，那咱得先下个Python吧，好吧，来到[Python下载网站](http://www.python.org/download/)，什么，又TM没连上，国外这网站不靠谱吧？你自己想想为啥吧~


2.拿到了VPN账号后，怎么使用呢？


> 下面我以Win7的设置为例，截图说明如何连上VPN，XP的同学自行Google吧，Linux的同学想必不需要教程吧~

首先，左键单击任务栏最右边的网络连接图标，然后选择“打开网络和共享中心”，如下图，围观我们宿舍牛逼的无线网络名称吧~颤抖吧~

![](/assets/images/newsinchina-free-vpn-1.jpg)

在打开后的界面，点击“设置新的连接或网络”，如图：

![](/assets/images/newsinchina-free-vpn-2.jpg)

在点击后出现的界面中选择“连接到工作区”，如图：

![](/assets/images/newsinchina-free-vpn-3.jpg)

接下来，选择上面一项“使用我的Internet连接”，如图：

![](/assets/images/newsinchina-free-vpn-4.jpg)

继续来，下面出现的页面就需要输入VPN的服务器地址了，如图，在“Internet地址”这一栏输入分配给你的地址(申请成功的同学通过邮箱发送)：

![](/assets/images/newsinchina-free-vpn-5.jpg)

接下来，就进入了用户名密码界面，输入分配给你的用户名密码即可，“域”这一项可以留空，这一步就不截图了吧，呵呵，突然觉得这教程太傻瓜了点，如果一切都顺利的话，将会自动连接VPN：

![](/assets/images/newsinchina-free-vpn-6.jpg)

设置成功后，以后这个VPN选项就会出现在第一步的菜单中，当你连上了Internet以后，点击连接该VPN即可。

当然，VPN也可以使用在移动设备上，本次发放的VPN类型为PPTP，如果你的手机支持，也可以在手机上使用，但是我们宿舍的G1和ipod touch都没有连接成功，原因不明~另外，有的路由器可能会屏蔽PPTP的端口，导致连接失败报691错误，请检查你的路由器。已经证实我们实验室的有线网络无法使用该VPN。


最后，希望大家Fuck GFW愉快，再一次说明，由于我是第一次架设VPN，技术和经验都不足，难免很多问题，出现使用问题请跟我反馈，不要乱发脾气~
