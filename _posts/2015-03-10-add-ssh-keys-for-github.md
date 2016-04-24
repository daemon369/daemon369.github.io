---
layout: post
keywords: git, github, ssh keys
description: add SSH keys for github
title: "GitHub 添加 SSH keys"
categories: [Git]
tags: [git, github, SSH keys]
group: Git
icon: file-alt
date: 2015-03-10 00:00:00
---

***
[上文][1]中介绍了 SSH 密钥对的生成，下面介绍下如何在 GitHub 中使用 SSH 密钥对。

***

#一. 查看已有的 SSH 密钥对

首先查看我们目前已有的 SSH 密钥对：

    $ ls -al ~/.ssh
    # 列出我们电脑上已有的所有 SSH 密钥对文件。

默认公钥文件名为以下形式：

* id_dsa.pub
* id_ecdsa.pub
* id_ed25519.pub
* id_rsa.pub

我们可以使用本地已有的密钥对，也可以[建立新的密钥对][1]。

<!--excerpt-->

#二. 添加 SSH key 到 GitHub 账户

首先，复制 SSH 公钥到剪切板。可以自己打开文件复制，也可使用 *xclip* 工具：

    $ sudo apt-get install xclip
    $ xclip -sel clip < ~/.ssh/id_rsa.pub

打开 GitHub 首页，点击右上角设置按钮：

![](/image/2015-03-10/1.png)

在设置页面中，选择"SSH keys":

![](/image/2015-03-10/2.png)

点击右上角的"Add SSH key"按钮，展开编辑框，在"Title"中输入 SSH Key 的标题，在"Key"中粘贴之前复制的 SSH 公钥内容：

![](/image/2015-03-10/3.png)

点击左下方按钮"Add key",稍后片刻，添加成功：

![](/image/2015-03-10/4.png)

下面测试下连接：

    $ ssh -T git@github.com
    Hi username! You've successfully authenticated, but GitHub does not provide shell access.

连接成功， SSH 密钥添加完成。

***

#相关文章

[创建 SSH 密钥对][1]

***

#参考

[Generating SSH keys](https://help.github.com/articles/generating-ssh-keys/ "Generating SSH keys")

[1]: {% post_url 2015-03-08-generating-ssh-keys %} "创建 SSH 密钥对"
