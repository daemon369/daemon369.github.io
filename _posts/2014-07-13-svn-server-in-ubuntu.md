---
layout: post
keywords: ubuntu, svn server
description: install and config svn server in ubuntu 14.04
title: "Ubuntu14.04下SVN服务器的安装配置"
categories: [Ubuntu]
tags: [ubuntu, svn server]
group: Ubuntu
icon: file-alt
date: 2014-07-13 00:00:00
---

***

# 1、安装SVN

    $ sudo apt-get install subversion

***

# 2、添加svn管理用户及subversion组

    $sudo adduser svnuser
    $sudo addgroup subversion
    $sudo addgroup svnuser subversion

<!--excerpt-->

***

# 3、创建项目目录

    $ sudo mkdir /home/svn
    $ cd /home/svn
    $ sudo mkdir myproject  //创建项目目录
    $ sudo chown -R root:subversion myproject
    $ sudo chmod -R g+rws myproject
    $ sudo svnadmin create /home/svn/myproject //创建svn文件仓库

***

# 4、访问权限设置

svn账户的配置文件在/home/svn/myproject/conf目录下，主要有svnserve.conf、passwd、authz三个文件。

编辑svnserve.conf文件，注意行首不能留空。把以下几行取消注释

    [general]
    anon-access = read #匿名用户(anonymous users)的访问权限，设置为可读，设置为none可以拒绝匿名用户访问
    auth-access = write #授权用户(authenticated users)的访问权限
    password-db = passwd #密码数据库文件的位置，这里指向同级目录下的passwd文件
    authz-db = authz #用户授权规则文件的位置，这里指向同级目录下的authz文件

现在来配置用户账户及密码，编辑/home/svn/myproject/conf/passwd文件：

    [users]
    user1 = 123456
    user2 = 123456
    test1 = 123456

配置用户权限：

    [groups]
    admin = user1,user2
    test = test1
    [/]
    @admin=rw
    *=r

这里配置了三个用户user1,user2及test1，其中user1和user2属于admin组，有读写权限，test1属于test组，有只读权限。

***

# 4、启动SVN服务

    $ svnserve -d -r /home/svn

-d 表示svn服务以守护进程模式运行
-r 指定了svn版本库的根目录

***

# 5、客户端访问

假设svn服务运行在服务器192.168.1.11上，局域网内客户端访问方式为：

    svn checkout svn://192.168.1.11/myproject --username user1 --password 123456 /home/myuser1/svn/myproject

***

# 6、问题

因为版本库的创建使用的是root权限，因此客户端在commit时可能会报告db/txn-current-lock: 权限不够 或者 txn-current-lock : Permission denied 之类的信息，这种情况下需要修改版本库中db目录下文件的访问权限：

    $ cd /home/svn/myproject
    $ sudo chmod 777 -R db/

***

# 参考

[ubuntu下SVN服务器安装配置](http://www.mobibrw.com/?p=336)
