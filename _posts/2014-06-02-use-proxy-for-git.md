---
layout: post
keywords: git, proxy
description: user proxy for git
title: "git使用http/https代理"
categories: [Network]
tags: [git, proxy, Network]
group: Network
icon: file-alt
---
{% include JB/setup %}

git使用http/https代理的方法如下：

打开命令行，设置如下环境变量：

$ export http_proxy=http://127.0.0.1:8087
$ export https_proxy=http://127.0.0.1:8087

<!--excerpt-->