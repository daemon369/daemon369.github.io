---
layout: post
keywords: android,桌面,launcher 
description: 使用第三方桌面替换Android手机预装的系统桌面
title: "替换Android系统桌面"
categories: [Android]
tags: [Android, launcher]
group: archive
icon: file-alt
date: 2013-08-06 00:00:00
---

~~使用a.apk桌面程序替换手机中系统桌面b.apk~~

~~# 1.如果系统只读，运行命令行~~

<!--excerpt-->

    ~~$ adb remount~~

~~2.将a.apk放入手机~~

    ~~$ adb push d:/a.apk /system/app~~

~~3. 更改a.apk权限~~

    ~~$ adb shell chmod 644 /system/app/a.apk~~

~~4.删除原系统桌面~~

    ~~$ adb shell rm /system/app/b.apk~~

~~5 .重启手机~~

    ~~$ adb reboot~~
