---
layout: post
keywords: SSH
description: SSH
title: "SSH无法连接：Too many authentication failures"
categories: [SSH]
tags: [ssh]
group: SSH
icon: file-alt
date: 2017-12-23 18:24:00
---
# 问题

今天使用`ssh`连接另外一台主机，提示以下错误

    $ ssh serverusername@xxx.xxx.xxx
    Received disconnect from xxx.xxx.xxx.xxx port 22:2: Too many authentication failures for user
    Disconnected from xxx.xxx.xxx.xxx port 22

# 分析

使用`-v`选项来查看`ssh`登录时的详细输出信息：

<!--excerpt-->

    $ ssh -v serverusername@xxx.xxx.xxx.xxx
    ...
    debug1: Authenticating to xxx.xxx.xxx.xxx:22 as 'serverusername'
    ...
    debug1: Found key in /Users/username/.ssh/known_hosts:16
    ....
    debug1: Offering public key: RSA SHA256:xxx /Users/username/.ssh/a_id_rsa
    ...
    debug1: Offering public key: RSA SHA256:xxx /Users/username/.ssh/b_id_rsa
    ...
    debug1: Offering public key: RSA SHA256:xxx /Users/username/.ssh/id_rsa
    ...
    debug1: Trying private key: /Users/username/.ssh/id_dsa
    debug1: Offering public key: ECDSA SHA256:xxx /Users/username/.ssh/id_ecdsa
    Received disconnect from xxx.xxx.xxx.xxx port 22:2: Too many authentication failures
    Disconnected from xxx.xxx.xxx.xxx port 22

由上面的详细输出，即可分析连接失败的原因，是由于本地有多个`ssh key`导致的问题。

我们知道，使用`ssh`连接服务器时，会自动用本地`~/.ssh`目录下的`ssh key`文件进行登录授权校验。服务端在收到客户端过多的`ssh key`校验请求时，就会拒绝客户端的连接，并报错：`Too many authentication failures`

# 解决

## 指定`ssh key`文件

在连接服务器时，如果需要使用指定的`ssh key`文件，需要排除不相干的`ssh key`文件的干扰。

### 命令行参数方式

可以使用命令行参数指定`ssh key`文件：

    $ ssh -i some_id_rsa -o 'IdentitieseOnly yes' serverusername@xxx.xxx.xxx

或者：

    $ ssh -i some_id_rsa -o IdentitiesOnly=yes serverusername@xxx.xxx.xxx

### 配置文件方式

可以在`ssh`配置文件`~/.ssh/config`中针对服务器地址进行配置。如果文件不存在，创建空配置文件。

针对服务器地址，增加配置或修改现有配置为：

    Host xxx.xxx.xxx.xxx
      IdentityFile ~/.ssh/key_for_somehost_rsa
      IdentitiesOnly yes
      Port 22

## 密码登录

如果需要采用密码登录方式，可以选择不使用`ssh key`来进行登录校验

### 命令行参数方式

    $ ssh -o 'PubkeyAuthentication no' serverusername@xxx.xxx.xxx

或：

    $ ssh -o PubkeyAuthentication=no serverusername@xxx.xxx.xxx

### 配置文件方式

增加配置或修改现有配置如下：

    Host xxx.xxx.xxx.xxx
      PubkeyAuthentication no
      Port 22

# 参看

[Too many authentication failures for *username*](https://superuser.com/questions/187779/too-many-authentication-failures-for-username)