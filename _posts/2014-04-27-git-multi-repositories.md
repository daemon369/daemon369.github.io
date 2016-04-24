---
layout: post
keywords: git, push, multi-repositories
description: push code to multiply git repositories
title: "Git推送代码到多个远程仓库"
categories: [Git]
tags: [git, push]
group: Git
icon: file-alt
date: 2014-04-27 00:00:00
---

github是使用最为广泛的git远程仓库管理平台，很多优秀的开源代码使用github进行托管。但是由于网络或地域的问题，不是每个人都能享受到它的便捷性。当你的代码托管到github，急需使用时却突然无法访问github，这将是致命的灾难。因此将代码托管到多个平台以做备份就很有必要了。中国目前也有许多git托管平台，包括[开源中国社区代码托管](http://git.oschina.net/)、[CSDN代码托管平台](http://code.csdn.net/)等，可以为国内开发者提供代码托管服务。

***

#添加多个仓库

打开github项目下的.git/config文件，在[remote "origin"]节点下增加新的url，这个url必需是先到对应的托管平台创建repository之后才有用。也可以创建新的remote节点，譬如[remote "all"]，在新的节点下添加url。

<!--excerpt-->

    [remote "origin"]
    	url = git@github.com:daemon369/daemon369.github.com.git
    	url = git@git.oschina.net:Daemon369/github_blog.git
    	fetch = +refs/heads/*:refs/remotes/origin/*

也可以使用命令行来添加：

    git remote set-url --add origin git@git.oschina.net:Daemon369/github_blog.git

***

#参考:

[Git一键推送多个远程仓库](http://my.oschina.net/chinesedragon/blog/81483)

[推送git项目到多个远程仓库](http://blog.codepiano.com/2013/07/03/push-multi-remote-repositories/)