---
layout: post
keywords: ubuntu, numlockx
description: Ubuntu开机自动开启数字小键盘
title: "Ubuntu12.04开机自动开启数字小键盘"
categories: [Ubuntu]
tags: [ubuntu, numlockx]
group: Ubuntu
icon: file-alt
---
{% include JB/setup %}

Ubuntu12.04开启后数字小键盘默认是关闭的，不方便使用。可以使用numlocxk程序在Ubuntu启动后自动将小键盘打开：

numlockx程序有3个可用命令：

    numlockx on          打开数字小键盘
    numlockx off         关闭数字小键盘
    numlockx toggle      开关数字小键盘

<!--excerpt-->

安装numlockx：

    sudo apt-get install numlockx

设置Ubuntu启动时执行命令：

    sudo vim /etc/lightdm/lightdm.conf

打开lightdm.conf 文件后在文件最后一行加入：

    greeter-setup-script=/usr/bin/numlockx on

保存，退出就可以解决问题