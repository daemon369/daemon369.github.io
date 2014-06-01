---
layout: post
keywords: goagent
description: use goagent
title: "使用GoAgent代理"
categories: [Network]
tags: [GoAgent, agent, Network, Linux]
group: Network
icon: file-alt
---
{% include JB/setup %}

***

#一.关于GoAgent

GoAgent 是使用跨平台语言 Python 开发的代理软件，利用 Google App Engine 的服务器充当代理，帮助我们浏览一些被封锁或被过滤的内容。由于GFW等的封锁，很多的技术资料网站或某些资源网站都无法访问或访问困难。免费的GoAgent在这方面给予了很大的帮助。

<!--excerpt-->

***

#二.GoAgent原理

下面是GoAgent开发者提供的GoAgent原理图：

![](/image/2013-08-11/9.png)

简单来说：我们发送的数据发送到本地GoAgent代理处进行了封装；然后封装的数据发送到运行于GAE的代理数据中转处；GAE代理数据中转处帮助我们获取到我们需要的数据，封装后再发送给我们本地的GoAgent代理处；最终本地运行的GoAgent代理解包后得到我们需要的数据。

***

#三.创建GAE应用

1.首先申请Google Appengine并创建appid。打开网页[ https://appengine.google.com/start](https://appengine.google.com/start)注册或使用Google账号登录。

![](/image/2013-08-11/1.png)

2.点击Create Application按钮

![](/image/2013-08-11/2.png)

3.然后需要进行身份验证。输入手机号码并点击发送验证码，输入接收到的验证码。

![](/image/2013-08-11/3.png)

4.输入appid，这里appid不能和别人重复，点击Check Avaliablity可以检查你输入的appid是否可用；接受google协议然后点击create application。这样你的appid就创建好了

每个账号最多可以创建10个GAE应用，每个应用每天有1G的免费流量，一个账号每天总共有10G的免费流量，这已经可以满足大部分人的需求了。


***

#四.安装GoAgent依赖环境

以Ubuntu14.04为例安装GoAgent：

###依赖

1. 必选
* python2(建议安装python2.7，如需在Linux上传或安装gevent需先安装python-dev)
2. 可选
* gevent 1.0(提升多线程性能，强烈建议安装)
* greenlet (gevent的依赖，一般安装gevent会自动安装)
* python-vte(基于GTK的简单GUI所需)
* python-openssl(生成证书所需，强烈建议安装，如删除了goagent自动证书则必须安装)
* pycrypto(RC4加密所需，建议安装)
* python-appindicator(Unity桌面下的托盘组件，其他桌面不必安装)

###安装依赖

    sudo apt-get install python-dev python-greenlet python-gevent python-vte python-openssl python-crypto python-appindicator

###安装gevent

需要在安装 python-dev 之后才能正确安装gevent和上传server，安装gevent需要安装了 gcc(Linux/Unix)。

    sudo apt-get install python-dev python-pip && sudo pip install gevent --upgrade

也可以手动编译安装：

如果greenlet版本低于0.4.0会导致gevent装不上，请先使用以下命令安装greenlet（0.4.2）

    wget http://mirrors.aliyun.com/pypi/packages/source/g/greenlet/greenlet-0.4.2.zip && unzip greenlet-0.4.2.zip && cd greenlet-0.4.2 && sudo python setup.py install

安装gevent（1.0）

    wget http://mirrors.aliyun.com/pypi/packages/source/g/gevent/gevent-1.0.tar.gz && tar xvzpf gevent-1.0.tar.gz && cd gevent-1.0 && sudo python setup.py install

如果不想安装gevent可以下载[gevent-1.0-py2.7-linux.egg](https://github.com/binyuj/gevent-egg/raw/egg/gevent-1.0-py2.7-linux.egg)放local文件夹

###下载GoAgent

到[https://code.google.com/p/goagent/](https://code.google.com/p/goagent/)或[https://github.com/goagent/goagent/releases](https://github.com/goagent/goagent/releases)下载最新GoAgent程序压缩包，解压安装包到本地。

打开local/proxy.ini文件，修改\[gae\]下面的appid=你自己申请的appid，多个appid使用“\|”来分隔，保存文件并退出。

###上传服务端

如果需要设置密码，先修改server/gae/gae.py文件，设置：

    __password__ = '密码'

进入server目录，运行命令：

    $ python uploader.zip

输入appid，多个appid同样使用“\|”来分隔，一次只能上传同一个Google账号下的appid，回车确认。然后根据提示依次输入Gmail邮箱和对应密码(注意：如果开启了两步验证，密码应为16位的应用程序专用密码而非谷歌帐户密码，否则会出现AttributeError: can\'t set attribute错误），填完按回车。如果要上传多个谷歌帐户下的appid，先上传一个账号的，传完一个账号后删除uploader.bat同目录下的.appcfg_cookies文件再传另一个。

###运行GoAgent

进入目录local/，运行python脚本，第一次运行需要管理员权限来导入证书：

    $ cd goagent-goagent-c8524c5/local/
    $ sudo python goagent-gtk.py

***

#五.浏览器中使用GoAgent代理

###在google chrome浏览器中使用goagent

首先安装[SwitchySharp插件](https://chrome.google.com/webstore/detail/dpplabbmogkhghncfbfdeeokoefdjegm)

![](/image/2013-08-11/7.png)

然后导入配置文件[https://goagent.googlecode.com/files/SwitchyOptions.bak](https://goagent.googlecode.com/files/SwitchyOptions.bak)或GoAgent附带的配置文件local/SwitchyOptions.bak。

![](/image/2013-08-11/8.png)

点击chrome地址栏右部SwitchySharp选项菜单，选择自动切换模式，即可使用GoAgent代理。

![](/image/2014-06-01/1.png)

但是，使用GoAgent时会出现一个问题，那就是在查看https网站时，会出现SSL错误，提示“该网站的安全证书不受信任！“：

![](/image/2014-06-01/2.png)

这时就需要自己在浏览器中添加GoAgent的证书：

从chrome的设置里找到HTTPS/SSL项，打开证书管理：

![](/image/2014-06-01/3.png)

![](/image/2014-06-01/4.png)

导入证书local/CA.crt：

![](/image/2014-06-01/5.png)

勾选上3个信任选项：

![](/image/2014-06-01/6.png)

重启浏览器即可。

###在firefox中使用goagent代理

首先在firefox中安装autoproxy插件。通过菜单项：工具->AutoProxy首选项，打开AutoProxy配置。

![](/image/2013-08-11/4.png)

选择菜单：代理服务器->编辑代理服务器，然后添加如下：

![](/image/2013-08-11/5.png)

点击确定后，在AutoProxy首选项窗口选择菜单：代理服务器->选择代理服务器：

![](/image/2013-08-11/6.png)

设置新建的代理服务器为默认代理服务器。

***

#六.Tips：

1.不是每次升级goagent都需要重新上传服务端。只有你升级的版本带[是]的，或者你的升级跨越了多个版本，其中某个版本带[是]，这样你就需要重新上传服务端。

2.每个帐号可以创建10个appid，在proxy.ini文件里[gae]写的appid字段后面添加所有appid，中间用"|"分隔，就可以使用10个appid来负载均衡，而且每个appid每天有1G免费流量，10个appid保证你能每天使用10G流量。

3.别人若知道了你的appid，是可以使用你的帐号的，而每个帐号每天只能有1G的免费流量。因此，你可以设置密码来保护你的流量不被盗用。解决办法就是给你的appid加上密码。在server/python/wsgi.py文件中找到__password__字段，在后面的单引号里填入你的密码，然后重新上次服务端。然后在local/proxy.ini文件中，在[gae]下的password字段后面输入你的密码，不需要加引号。重启GoAgent客户端即可。

4.Linux下，如果提升CA证书不受信任，可以通过如下方式将GoAgent提供的CA加到系统信任列表里：

    $ sudo cp goagent/local/CA.crt /usr/share/ca-certificates/goagent.crt
    $ sudo chmod a+r /usr/share/ca-certificates/goagent.crt
    $ sudo dpkg-reconfigure ca-certificates

在弹出的图形节目里，勾选上GoAgent的CA就可以了。

***

#七.参考：

[GaAgent FAQ](https://code.google.com/p/goagent/wiki/FAQ)

[goagent GAE平台部署教程](https://code.google.com/p/goagent/wiki/InstallGuide)

[GoAgent_Linux](https://code.google.com/p/goagent/wiki/GoAgent_Linux)