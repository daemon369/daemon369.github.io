---
layout: post
keywords: adb,android
description: ADB命令无法连接手机的解决方法
title: "ADB命令无法连接手机的解决方法"
categories: [Android]
tags: [Android, adb]
group: archive
icon: file-alt
---
{% include JB/setup %}

我的海信E860手机使用qq手机管家、360等工具都能连接并管理，但是使用adb命令行或者eclipse的android插件都无法找到设备。解决了方法如下：

一. Windows：

1.安装手机驱动（使用官方驱动或者打开qq手机管家等工具自动安装）

2.在电脑上%USERPROFILE%文件夹下建一个".android"文件夹，并在文件夹下新建文件"adb_usb.ini"(如果不存在的话)。

3.打开文本编辑器，在"adb_usb.ini"文件中加一行(海信设备厂商ID)：

	0x109B

4.打开命令行，输入

	$ adb kill-server

	$ adb start-server

	$ adb devices

这样就看到自己的设备了。

设备ID的获取方法：

打开设备管理器，找到手机设备，点击右键查看属性，点开详细信息，在设备范例ID下看到VID后面的4个16进制数据就是了

![图片](/image/2013-08-05/win.png)

二. ubuntu：
terminal工具下输入lsusb，就可以看到手机设备ID

![图片](/image/2013-08-05/ubu.png)

adb_usb.ini文件在目录~/.andrdoid下

很多国产手机使用ADB工具无法连接，都可以使用这种方法解决。
