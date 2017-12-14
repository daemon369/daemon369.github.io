---
layout: post
keywords: ruby
description: ruby
title: "安装 Ruby"
categories: [Ruby]
tags: [ruby rbenv]
group: Ruby
icon: file-alt
date: 2017-12-14 21:11:00
---

常见的`Ruby`版本管理工具有`rbenv`、`RVM`等，这里介绍轻便的`rbenv`来安装`Ruby`:

# 安装`rbenv`

## macOS

在`macOS`中使用`brew`安装`rbenv`：

    $ brew install rbenv

在`~/.bash_profile`文件中加入：

    export PATH=$HOME/.rbenv/bin:$PATH
    eval "$(rbenv init -)"

重新打开终端或者输入：

    $ . ~/.bash_profile

更新`rbenv`：

    $ brew upgrade rbenv ruby-build

## Ubuntu

TODO

# 安装`Ruby`

## 查看可安装的版本列表：

    $ rbenv install -l
    Available versions:
      1.8.5-p52
      1.8.5-p113
      1.8.5-p114
      ...
      2.4.2
      ...

当前最新版本为`2.4.2`

## 安装：

    $ rbenv install 2.4.2

## 查看当前`Ruby`版本：

    $ rbenv version
    2.4.2 (set by /Users/user/.rbenv/version)

## 查看本地安装所有`Ruby`版本：

    $ rbenv versions
    yc-mac-2:~ yc$ rbenv versions
      system
      2.4.1
    * 2.4.2 (set by /Users/user/.rbenv/version)

# 切换版本

安装多个版本`Ruby`时，可以自如的切换需要使用的版本。`rbenv`中有三个不同的作用域：

* 全局(global)
* 本地(local)
* 当前终端(shell)

多个作用域同时设置时，查找版本的优先级为：shell > local > global

## 全局设置

全局作用域设置后针对全局，没有设置本地或终端作用域的`Ruby`版本时，全局设置将会生效：

    $ rbenv global 2.4.2

## 本地设置

本地作用域主要针对各个不同的项目，在项目根目录设置本地作用域时，会通过在项目根目录下生成的`.rbenv-version`文件来管理`Ruby`版本：

    $ rbenv local 2.4.2

这样对`Ruby`版本的设置就只有在这个项目中才生效

## 当前终端设置

当前终端设置只针对当前终端，退出当前终端或打开新的终端时就会失效：

    $ rbenv shell 2.4.2

## 取消设置

    $ rbenv global --unset
    $ rbenv local --unset
    $ rbenv shell --unset

## 使用系统`Ruby`

如果通过其他方式安装了`Ruby`且设置了环境变量，我们可以通过以下命令使得系统`Ruby`生效：

    $ rbenv global system

# 卸载`Ruby`：

直接删除`Ruby`安装目录：

    $ rm -rf ~/.rbenv/versions/2.4.2

还可以使用命令：

    $ rbenv uninstall 2.4.2

# 卸载`rbenv`

## 关闭`rbenv`

若要关闭`rbenv`，使其对`Ruby`的版本管理失效，只需要将`~/.bash_profile`文件中加入的`rbenv init`行删除即可

## 完全卸载

### 手动

1. 删除`~/.bash_profile`中添加的配置行
2. 删除`rbenv`安装目录：

    $ rm -rf `rbenv root`

### macOs

如果使用`brew`安装的`rbenv`，可以使用命令：

    $ brew uninstall rbenv

### Ubuntu

TODO

# 参看

[rbenv](https://github.com/rbenv/rbenv)

[rbenv-install-and-using](https://gist.github.com/sandyxu/8aceec7e436a6ab9621f)