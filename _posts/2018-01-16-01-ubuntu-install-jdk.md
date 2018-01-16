---
layout: post
keywords: Ubuntu
description: ubuntu install jdk
title: "Ubuntu中安装JDK"
categories: [Ubuntu]
tags: [Ubuntu, JDK, PPA]
group: Ubuntu
icon: file-alt
date: 2018-01-16 23:41:00
---

在`Ubuntu`中安装`JDK`主要有`PPA`安装和手动安装两种方式：

# `PPA`安装

## [添加`PPA`](https://launchpad.net/~webupd8team/+archive/ubuntu/java)

    $ sudo add-apt-repository ppa:webupd8team/java
    $ sudo apt-get update

<!--excerpt-->

## 安装`JDK8`

    $ sudo apt-get install oracle-java8-installer

## 设置默认JDK

    $ sudo update-java-alternatives -s java-8-oracle

`update-java-alternatives`工具依赖`java-common`，在`oracle-java8-installer`安装时已经自动安装，如果没有的话，也可以自行安装：

    $ sudo apt-get install java-common

# 手动安装

首先去[`Oracle`官网](http://www.oracle.com/technetwork/java/javase/downloads/index.html)下载`JDK`安装包。

目前`JDK6`及`JDK7`属于已归档、不再提供安全更新的版本，已经无法通过官方源或`PPA`直接下载安装，需要手动从官网下载，且需要注册`Oracle`账户才能下载。下载地址：

[Java SE 6 Downloads](http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase6-419409.html)

[Java SE 7 Archive Downloads](http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html)

以[`JDK8`](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)为例：

首先进入下载页面，勾选`Accept License Agreement`单选框，选择`Linux`版本安装包，点击下载。

下载完成后，安装：

    $ tar zxvf jdk1.8.0_152
    # 如果/usr/lib/jvm目录不存在，创建目录
    $ sudo mkdir -p /usr/lib/jvm
    $ sudo mv jdk1.8.0_152 /usr/lib/jvm/java-8-oracle

    $ sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/java-8-oracle/bin/java 1
    $ sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/java-8-oracle/bin/javac 1

选择默认`java`版本：

    $ sudo update-alternatives --set java /usr/lib/jvm/java-8-oracle/bin/java
    update-alternatives: using /usr/lib/jvm/java-8-oracle/bin/java to provide /usr/bin/java (java) in manual mode.

    $ sudo update-alternatives --set javac /usr/lib/jvm/java-8-oracle/bin/javac
    update-alternatives: using /usr/lib/jvm/java-8-oracle/bin/javac to provide /usr/bin/javac (javac) in manual mode.

