---
layout: post
keywords: git， github, email， git config
description: setting email in git
title: "设置 Git 账户及邮箱"
categories: [Git]
tags: [git, github, email, git config]
group: Git
icon: file-alt
date: 2015-03-11 02:00:00
---

当我们在`GitHub`中提交修改时，`GitHub`通过我们本地`git`配置文件中配置的邮箱地址，与我们的`GitHub`账户相关联，这样`GitHub`提交记录就可以关联提交者的`GitHub`账户。

那么我们怎么在本地配置我们的`git`邮箱呢？

#全局 git 配置

我们可以使用`git config`命令来修改本地`git`配置。设置全局用户及邮箱：

    $ git config --global user.name gitaccount
    $ git config --global user.email gitaccount@example.com

<!--excerpt-->

其中`gitaccount`是我们的`git`账户，`gitaccount@example.com`是我们的`git`邮箱。

修改配置后，可以查看本地`git`配置文件：

    $ cat ~/.gitconfig
    [user]
        name = gitaccount
        email = gitaccount@example.com

也可以使用命令来查看修改后的配置：

    $ git config --global user.name
    gitaccount
    $ git config --global user.email
    gitaccount@example.com

这里修改的是全局的`git 配置项，配置完成后，我们在所有的代码仓库中提交的修改，默认都将关联全局配置的账户及邮箱，除非我们为每个代码仓库单独配置账户及邮箱。

# 代码仓库`git`配置

我们可以为每个代码仓库配置单独配置账户及邮箱。

取消全局配置：

    $ git config --global --unset user.name
    $ git config --global --unset user.email
    $ git config --global user.name
    #全局配置账户已经移除
    $ git config --global user.email
    #全局配置邮箱已经移除

进入代码仓库目录，修改配置：

    $ cd git-repository/
    $ git config user.name anothergitaccount
    $ git config user.email anothergitaccount@example.com

修改后的配置可以使用命令查看：

    $ git config user.name
    anothergitaccount
    $ git config user.email
    anothergitaccount@example.com

也可以在代码仓库目录配置文件查看：

    $ cat .git/config
    [core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
    [remote "origin"]
        url = https://github.com/username/repository.git
        fetch = +refs/heads/*:refs/remotes/origin/*
    [branch "master"]
        remote = origin
        merge = refs/heads/master
    [user]
        name = anothergitaccount
        email = anothergitaccount@example.com

#Troubleshooting

##GitHub 提交记录无法关联账户

如果 GitHub 的提交记录没有关联本地设置的邮箱，可能是因为你没有在 GitHub 的邮箱设置项中添加你的邮箱。

添加的方法为：

首先在 GitHub 网页右上角点击设置按钮：

![](/image/2015-03-11/01.png)

打开 *Emails* 页，在右侧添加你的邮箱：

![](/image/2015-03-11/02.png)

修改完本地 git 账户、邮箱后，再次提交的代码会自动关联到设置的邮箱，但是之前的提交依然会关联之前设置的账户、邮箱。

##提交记录没有关联正确的邮箱

如果你本地 git 设置正确，但是 GitHub 上的提交记录仍然没有关联正确的邮箱，这有可能是设置的邮箱被环境变量覆盖了。查看以下环境变量是否设置：

    $ echo $GIT_COMMITTER_EMAIL
    # 打印环境变量 GIT_COMMITTER_EMAIL
    $ echo $GIT_AUTHOR_EMAIL
    # 打印环境变量 GIT_AUTHOR_EMAIL

如果本地设置了这两个环境变量，且设置的值不是我们想要设置的邮箱，重新设置环境变量：

    $ GIT_COMMITTER_EMAIL=gitaccount@example.com
    $ GIT_AUTHOR_EMAIL=gitaccount@example.com

#参考

[Setting your email in Git](https://help.github.com/articles/setting-your-email-in-git/ "Setting your email in Git")

[Adding an email address to your GitHub account](https://help.github.com/articles/adding-an-email-address-to-your-github-account/ "Adding an email address to your GitHub account")
