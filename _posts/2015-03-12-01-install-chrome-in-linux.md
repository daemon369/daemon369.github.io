---
layout: post
keywords: linux, google chrome
description: install chrome for linux
title: "Linux 中安装 Google Chrome 浏览器"
categories: [Linux]
tags: [linux, google chrome]
group: Linux
icon: file-alt
---
{% include JB/setup %}

我们在 Ubuntu 的软件中心无法找到 Google Chrome 浏览器，使用 *apt-get  install* 命令也无法直接安装 chrome。要在　Linux　中安装 chrome ，首先需要安装谷歌 Linux 软件仓库签名 Key。

可以在如下链接下载签名 key：

    https://dl-ssl.google.com/linux/linux_signing_key.pub

也可以直接使用命令安装：

    wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -

安装完谷歌签名 key 后，添加谷歌软件源：

    $ sudo sh -c 'echo "deb http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list'

然后安装 chrome：

    $ sudo apt-get update
    $ sudo apt-get install google-chrome-stable

#参考：

[Google Linux Software Repositories](https://www.google.com/linuxrepositories/ "Google Linux Software Repositories")

[How to Install Google Chrome 41 in Ubuntu 14.10, 14.04 & 12.04 and LinuxMint](http://tecadmin.net/install-google-chrome-in-ubuntu/ "How to Install Google Chrome 41 in Ubuntu 14.10, 14.04 & 12.04 and LinuxMint")
