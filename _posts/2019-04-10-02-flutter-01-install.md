---
layout: post
keywords: Flutter
description: Flutter
title: "Flutter 01 - 安装"
categories: [Flutter]
tags: [Flutter]
group: Flutter
icon: file-alt
date: 2019-04-10 22:11:27
---

# 系统需求

* 操作系统：`macOS(64位)`
* 磁盘空间：安装完成IDE/工具等，仍需要至少700M空间
* 工具：
    * bash
    * curl
    * git(2.x)
    * mkdir
    * rm
    * unzip
    * which

# 下载`Flutter SDK`

下载`Flutter SDk`[安装包](https://flutter.io/docs/development/tools/sdk/archive?tab=macos)，当前最新稳定版本[1.2.1](https://storage.googleapis.com/flutter_infra/releases/stable/macos/flutter_macos_v1.2.1-stable.zip)。

下载完成后，解压到本地：

    $ cd ~/develop
    $ unzip ~/Downloads/flutter_macos_v1.0.0-stable.zip

添加`flutter`工具到`PATH`环境变量中：

    $ export PATH=~/develop/flutter/bin:$PATH

# 运行`flutter doctor`

    $ flutter doctor
    Doctor summary (to see all details, run flutter doctor -v):
    [✓] Flutter (Channel stable, v1.0.0, on Mac OS X 10.14.2 18C54, locale zh-Hans-CN)
    [!] Android toolchain - develop for Android devices (Android SDK 28.0.3)
        ! Some Android licenses not accepted.  To resolve this, run: flutter doctor --android-licenses
    [✗] iOS toolchain - develop for iOS devices
        ✗ Xcode installation is incomplete; a full installation is necessary for iOS development.
          Download at: https://developer.apple.com/xcode/download/
          Or install Xcode via the App Store.
          Once installed, run:
            sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
        ✗ libimobiledevice and ideviceinstaller are not installed. To install with Brew, run:
            brew update
            brew install --HEAD usbmuxd
            brew link usbmuxd
            brew install --HEAD libimobiledevice
            brew install ideviceinstaller
        ✗ ios-deploy not installed. To install with Brew:
            brew install ios-deploy
        ✗ CocoaPods not installed.
            CocoaPods is used to retrieve the iOS platform side's plugin code that responds to your plugin usage on the Dart side.
            Without resolving iOS dependencies with CocoaPods, plugins will not work on iOS.
            For more info, see https://flutter.io/platform-plugins
          To install:
            brew install cocoapods
            pod setup
    [✓] Android Studio (version 3.3)
        ✗ Flutter plugin not installed; this adds Flutter specific functionality.
        ✗ Dart plugin not installed; this adds Dart specific functionality.
    [!] IntelliJ IDEA Community Edition (version 2018.3.3)
        ✗ Flutter plugin not installed; this adds Flutter specific functionality.
        ✗ Dart plugin not installed; this adds Dart specific functionality.
    [!] Connected device
        ! No devices available
    
    ! Doctor found issues in 4 categories.
