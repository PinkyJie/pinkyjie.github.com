date: 2011-04-09 09:40:43
title: 在Amazon的EC2上部署nginx+web.py
categories:
- 网站相关
tags:
- amazon
- ec2
- nginx
- spawn-fcgi
- web.py
---

一直想弄个VPS玩，可动辄五六百一年的money目前还是承受不起，昨天又看到一个网站在推广Amazon的免费一年的EC2，就尝试申请了下。

<!--more-->

### 申请Amazon的EC2

免费申请的条件有两个，首先你得是新用户，另外需要一张能够提供外币支付的信用卡。整个注册过程大概会扣2刀，网上说是测试信用卡是否可用，到底这2刀是否归还我也不清楚。点击 [http://aws.amazon.com/free/](http://aws.amazon.com/free/) ，点右面的sign up进行申请，申请的步骤跟普通的差不多，就是填填姓名啊地址邮箱神马的，需要注意的是留信用卡的地方，信息一定要正确。另外，申请过程中需要打电话确认，一定要使用真实的电话号码，打电话之前会让你重新填写电话的，所以先前注册时随便填电话也行。输完电话号码点call me now，就会收到一个+1开头的号码打来的电话，一堆非常机械的女声洋文以后，会请你输入电脑屏幕闪的四个数字。不要急，输完就行，电话自动挂断后就可以继续注册。我当时就是太紧张了，导致打了两次电话才完成认证。认证完成后收到电邮确认，然后登陆 [https://console.aws.amazon.com/ec2/home](https://console.aws.amazon.com/ec2/home) ，这就是你的控制台了，在Region里选US west，离我们国家近一点的机房，然后点Launch Instance就可以配置操作系统一类的了。配置中有几点需要注意，首先，选操作系统的时候右面加五星的才是免费的，默认的只有Amazon定制的可选，是基于Fedora的，如果想选Ubuntu，可以点第三个标签Community AMIs，在Viewing里选择Free tier eligible，这里会有很多带五星的系统可以选，有Ubuntu啊，Cent OS一类的可以选，而且都是预装了一些服务器程序或Wordpress程序的，如下图：

![](/assets/images/build-nginx_webpy_on_amazon_ec2-1.png)

在select以后进入的下一步中，注册Instance Type选第一项Micro，这个才是免费的。另外，后面配置security的时候要开几个端口，SSH，HTTP和HTTPS都要开，因为要搞webpy，我直接开了8000-9500所有的端口，参见下图：

![](/assets/images/build-nginx_webpy_on_amazon_ec2-2.png)

其他详细步骤可以看[千百度记忆角落的这篇《Amazon EC2免费一年申请使用图文教程》](http://www.baidu.com.ru/archives/556.html)，非常的详细。所有步骤完成后，就可以在自己的机器上SSH登陆这个VPS了，涉及到公钥证书神马的，因为我是直接用msys在windows上虚拟了一个unix环境中使用ssh的，一条命令搞定，如果你想用putty，SecureCRT一类的SSH客户端，请参见[千百度记忆角落的这篇《Windows下如何用putty连接Amazon EC2实例图文教程！》](http://www.baidu.com.ru/archives/573.html)。

### 搭建nginx+web.py

登陆上主机以后，我们开始配置环境吧。首先下好各种软件，以Ubuntu环境为例。

``` bash
sudo apt-get install nginx   # 安装nginx
sudo apt-get install python-setuptools
# python默认有，不用装，这一步是为了装好easy_install
sudo easy_install web.py  # 安装web.py
sudo apt-get install flup # 发布用
sudo apt-get install spawn-fcgi # 执行py需要的cgi
```

这样，所有需要的软件就都安装好了，Amazon的下载速度是很快的。首先，先测试一下web.py，按照官网给的例子写一个code.py。

``` python
import web

urls = (
    '/(.*)', 'hello'
)
app = web.application(urls, globals())

class hello:
    def GET(self):
        return 'Hello, web.py'

if __name__ == "__main__":
    app.run()
```

然后运行`python code.py`，如果出现`http://0.0.0.0:8080/`则表明`web.py`工作正常，按`Ctrl +Ｃ`结束即可。下面我们要做的就是三步走，第一，修改nginx的配置文件；第二，开启一个`spawn-fcgi`进程；最后，打开nginx服务器。在这之前，我们还需要修改下`code.py`，以便顺利完成发布，首先修改`code.py`为可执行，即执行`chmod +x code.py`，然后在`app.run()`这句之前加上一句：

``` python
web.wsgi.runwsgi = lambda func, addr=None: web.wsgi.runfcgi(func, addr)
```

### 修改nginx的配置文件

nginx默认被安装在`/etc/nginx`目录中，其中的`nginx.conf`就是默认的配置文件，为了避免一开始就修改坏，我们将其拷贝出来一份，比如放在和刚才的`code.py`相同的文件夹下，这里以`~/pytest/`为例。打开后我们发现这个文件由http模块和注释掉的mail模块组成，我们要修改的就是http模块。这里配置文件的具体参数我也不大懂，放上我最终修改好的以供参考。

``` bash
user www-data;
worker_processes  1;

error_log  /var/log/nginx/error.log;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
    # multi_accept on;
}

http {
    include       /etc/nginx/mime.types;

    access_log  /var/log/nginx/access.log;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;
    tcp_nodelay        on;

    gzip  on;
    gzip_disable "MSIE [1-6]\.(?!.*SV1)";

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;

# 新添加的部分
    server {
        listen 80;    # 监听80端口，也就是HTTP对应的端口
        server_name 127.0.0.1:9001; #这里端口可以随便填，但确定先前配置系统的时候已开通
        # 处理静态文件
        location /static/ {
            if (-f $request_filename) {
                rewrite ^/static/(.*)$ /static/$1 break;
            }
        }
        # 重点部分
        location / {
             fastcgi_pass 127.0.0.1:9001;
             fastcgi_param SERVER_NAME $server_name;
             fastcgi_param SERVER_PORT $server_port;
             fastcgi_param SERVER_PROTOCOL $server_protocol;
             fastcgi_param PATH_INFO $fastcgi_script_name;
             fastcgi_param REQUEST_METHOD $request_method;
             fastcgi_param QUERY_STRING $query_string;
             fastcgi_param CONTENT_TYPE $content_type;
             fastcgi_param CONTENT_LENGTH $content_length;
             fastcgi_pass_header Authorization;
             fastcgi_intercept_errors off;
        }
    }
}
```

主要就是添加了server块的部分，nginx的详细参数意义可以参加[官方的Wiki](http://wiki.nginx.org/NginxChsHttpCoreModule)。

### 开启spawn-fcgi进程

执行如下命令：

``` bash
sudo spawn-fcgi -d ~/pytest/ -f ~/pytest/code.py -a 127.0.0.1 -p 9001
```

这里，-d表示py文件所在的目录，-f表示py文件的绝对路径，-a是地址，-p是端口，这里要与`nginx.conf`里的设置保持一致。如果你的运气好，就会出现`spawn-fcgi: child spawned successfully: PID: 1517`，这说明进程已经成功开启，ID号为1517。但是，通常是会报错的，比如`spawn-fcgi: child exited with: xxx`。这时需要检查几点，首先刚才的chmod命令有没有执行，即`code.py`的可执行属性有没有加上。如果加上以后还不行，那么先把`code.py`最后加上的`web.wsgi.runwsgi = lambda func...`这句代码加#号注释掉，然后执行`python code.py`看是否出现刚才说的`http://0.0.0.0:8080`，如果代码报错，如`[Errno 98] Address already in use`，说明刚才python可能没有正常结束，还占用着这个地址，直接`killall python`，或者用`ps aux | grep “python”`查出python对应的PID，通过`kill PID`来结束。如果这步还不行，那只能重启试试了。

### 打开nginx服务器

在成功开启一个spawn-fcgi进程后，我们就可以打开nginx了：

``` bash
sudo nginx -t -c ~/pytest/nginx.conf
# 测试自己指定的配置文件是否正确，若正确则执行
sudo nginx -c ~/pytest/nginx.conf
```

同样，如果运气好的话，什么也不显示代表正常启动了。否则会报错，如`[emerg]: bind() to 0.0.0.0:80 failed (98: Address already in use)`，这个错误一方面可能是上面的spawn-fcgi启动不正常导致，先检查下，使用命令`ps aux | grep “code.py”`看是否存在两个结果(因为一个进程是grep查询进程)，如果正常，则可能是nginx重复启动，自己占用了自己的端口。通过命令 `ps aux | grep “nginx”`看下是否有nginx的进程，通常是一个master主进程，一个worker进程，如果有，同样几下它们的PID，用kill命令杀掉进程再试，有必要的话再重启下nginx的服务，执行`service nginx restart`。一般的话就正常了。如果启动正常，那通过浏览器访问你Public DNS地址，就会出现 `hello web.py`的网页了。如果此时出现的还是welcome to nginx的默认网页，有可能是nginx的默认配置所致，需要检查下你的`/etc/nginx/sites-enabled`目录下，是否有一个default的文件，有的话移走，随便移动到哪都行，然后再试，应该就正常了。如果还不行肿么办，因为上面的情况都是我自己遇到的，其他没遇到的那我也没办法了，你只能查看下nginx的`error log`文件，看有没有神马蛛丝马迹，这个错误日志文件默认在`/var/log/nginx/`目录下，`error.log`文件，看看里面记录了什么再进一步排查了。
