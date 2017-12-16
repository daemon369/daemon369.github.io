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

常见的`Ruby`版本管理工具有`rbenv`、`RVM`等，这里介绍在`macOs`及`Ubuntu 16.04LTS`中使用轻量的`rbenv`来安装`Ruby`的方法。

# 安装`rbenv`

## Github方式

### 安装`rbenv`

采用`Github`上将`rbenv` checkout 到本地的方式，这样就不需要系统范围的安装，而且能够及时获取最新版本：

    $ git clone https://github.com/rbenv/rbenv.git ~/.rbenv
    $ cd ~/.rbenv && src/configure && make -C src

<!--excerpt-->

添加脚本配置：

    $ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
    $ echo 'eval "$(rbenv init -)"' >> ~/.bash_profile

重新打开终端窗口使配置生效，或者运行以下命令：

    $ . ~/.bash_profile

### 更新`rbenv`

    $ cd `rbenv root`
    $ git pull

### 安装`ruby-build`

    $ mkdir -p `rbenv root`/plugins
    $ git clone https://github.com/rbenv/ruby-build.git `rbenv root`/plugins/ruby-build

### 更新`ruby-build`

    $ cd `rbenv root`/plugins/ruby-build
    $ git pull

### 关闭`rbenv`

若要关闭`rbenv`，使其对`Ruby`的版本管理失效，只需要将`~/.bash_profile`(macOs)或`~/.bashrc`(Ubuntu)文件中加入的`rbenv init`行删除，然后重启`shell`即可。

### 卸载

1. 删除`~/.bashrc`中添加的配置行
2. 删除`rbenv`安装目录：

    $ rm -rf `rbenv root`

注意：以上设置，在`Ubuntu`中将`~/.bash_profile`替换为`~/.bashrc`，如果使用`zsh`，替换为`~/.zshrc`。

## macOS Homebrew

### 安装`rbenv`及`ruby-build`

在`macOS`中使用`Homebrew`安装`rbenv`：

    $ brew install rbenv

`rbenv`安装时会自动安装其依赖的`ruby-build`。

添加脚本配置：

    $ echo 'eval "$(rbenv init -)"' >> ~/.bash_profile

重新打开终端窗口使配置生效，或者运行以下命令：

    $ . ~/.bash_profile

### 更新`rbenv`：

    $ brew update && brew upgrade rbenv ruby-build

### 卸载

    $ brew uninstall rbenv
    $ brew uninstall ruby-build

# 安装`Ruby`

在`Ubuntu 16.04LTS`中首先需要安装依赖：

    $ sudo apt install -y libssl-dev libreadline-dev zlib1g-dev

## 查看可安装的版本列表：

    $ rbenv install -l
    Available versions:
      1.8.5-p52
      1.8.5-p113
      1.8.5-p114
      ...
      2.4.3
      ...

当前最新版本为`2.4.3`

## 安装：

    $ rbenv install 2.4.3

## 查看当前`Ruby`版本：

    $ rbenv version
    2.4.3 (set by /Users/user/.rbenv/version)

## 查看本地安装所有`Ruby`版本：

    $ rbenv versions
      system
      2.4.2
    * 2.4.3 (set by /Users/user/.rbenv/version)

# 切换版本

安装多个版本`Ruby`时，可以自如的切换需要使用的版本。`rbenv`中有三个不同的作用域：

* 全局(global)
* 本地(local)
* 当前终端(shell)

多个作用域同时设置时，查找版本的优先级为：shell > local > global

## 全局设置

全局作用域设置后针对全局，没有设置本地或终端作用域的`Ruby`版本时，全局设置将会生效：

    $ rbenv global 2.4.3

## 本地设置

本地作用域主要针对各个不同的项目，在项目根目录设置本地作用域时，会通过在项目根目录下生成的`.rbenv-version`文件来管理`Ruby`版本：

    $ rbenv local 2.4.3

这样对`Ruby`版本的设置就只有在这个项目中才生效

## 当前终端设置

当前终端设置只针对当前终端，退出当前终端或打开新的终端时就会失效：

    $ rbenv shell 2.4.3

## 取消设置

    $ rbenv global --unset
    $ rbenv local --unset
    $ rbenv shell --unset

## 使用系统`Ruby`

如果通过其他方式安装了`Ruby`且设置了环境变量，我们可以通过以下命令使得系统`Ruby`生效：

    $ rbenv global system

# 卸载`Ruby`：

直接删除`Ruby`安装目录：

    $ rm -rf ~/.rbenv/versions/2.4.3

还可以使用命令：

    $ rbenv uninstall 2.4.3

# 参看

[rbenv](https://github.com/rbenv/rbenv)

[rbenv-install-and-using](https://gist.github.com/sandyxu/8aceec7e436a6ab9621f)

[How To Install Ruby on Rails with rbenv on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-ruby-on-rails-with-rbenv-on-ubuntu-16-04)

[ruby-build](https://github.com/rbenv/ruby-build)
