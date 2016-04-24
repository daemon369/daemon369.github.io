---
layout: post
keywords: ubuntu, root, passwd
description: use root account in ubuntu
title: "Ubuntu 中启用 root 账号"
categories: [Ubuntu]
tags: [ubuntu, passwd, root]
group: Ubuntu
icon: file-alt
date: 2015-03-13 00:00:00
---

Ubuntu 安装的时候并没有设置 root 用户的密码，第一次登录也无法使用 root 用户登录。如果我们需要启用 root 用户，首先需要设置 root 用户密码：

    $ sudo passwd root
    [sudo] password for username:
    # 输入用户密码
    输入新的 UNIX 密码：
    # 输入需要设置的 root 密码
    重新输入新的 UNIX 密码：
    # 重新输入 root 密码

<!--excerpt-->

设置完 root 密码后，就可以在终端切换到 root 用户了：

    $ su
    # 输入 root 密码切换 root 用户

我们可以锁定已经启用的 root 用户：

    $ sudo passwd -l root
    $ su
    密码：
    su：认证失败

锁定后的 root 用户无法再登入

对应的解锁方法为：

    $ sudo passwd -u root
