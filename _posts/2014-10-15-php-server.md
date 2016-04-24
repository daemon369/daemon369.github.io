---
layout: post
keywords: goagent, php, liunx
description: 免费PHP空间的申请及使用
title: "免费PHP空间的申请及使用"
categories: [Network]
tags: [Linux, PHP]
group: Network
icon: file-alt
date: 2014-10-15 01:00:00
---

***

#一. 申请PHP空间

###1. EcVPS

首先进入[EcVps申请页面](http://www.ecvps.com/client/cart.php?gid=3 "EcVps申请页面")：

![](/image/2014-10-15/1.png)

<!--excerpt-->

点击右下角*Order Now*，进入域名设置页面：

![](/image/2014-10-15/2.png)

选择最后一项并输入空间的二级域名，然后点击*Click to Continue*，进入资料编辑页面：

![](/image/2014-10-15/3.png)

输入自己详细的信息，勾选接收服务条款，然后选择*Complete Order*，完成注册。如果注册成功，则注册填写的邮箱会收到注册成功的邮件。

注册成功后，进入[My Products & Services](http://www.ecvps.com/client/clientarea.php?action=products)页面，在这里可以查看到之前填写的二级域名，点击右侧详情按钮进入详情页面，可以在底部看到用户名密码：

![](/image/2014-10-15/4.png)

***

#二. 上传文件

空间申请成功后，我们可以进入网页版管理员页面来上传文件，也可以用ftp客户端来管理上传文件。这里我使用的是linux下的ftp工具FileZilla：

![](/image/2014-10-15/5.png)

输入主机地址、用户名、密码，点击快速连接，进入文件管理，上传自己的网页等文件。

***

#参考

[EcVPS免费空间goagent之绝配](http://www.faith.ga/2913.html)

[1]: {% post_url 2014-06-01-use-goagent %} "使用GoAgent代理"