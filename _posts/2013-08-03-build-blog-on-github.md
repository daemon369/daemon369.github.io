---
layout: post
keywords: blog,github,jekyll
description: How to build my blog on github
title: "在github上搭建博客"
categories: [Blog]
tags: [Blog, github pages, jekyll]
group: Blog
icon: file-alt
date: 2013-08-03 00:00:00
---

最开始是看到[阮一峰](http://www.ruanyifeng.com/blog/)的博客[搭建一个免费的，无限流量的Blog----github Pages和Jekyll入门][1]，才知道可以使用github来搭建个人博客的。因为刚在github上面创建了一个repository用来放一些自己的学习过程中写的demo，那么再创建个博客来记录自己的学习过程，也是个不错的事情。因此开始折腾这个博客。

<!--excerpt-->

首先申请个`GitHub`帐号，这个很简单。我的账户名为：`daemon369`，因此我的个人首页为：<https://github.com/daemon369>。

然后就是创建`GitHub Pages`。有两种类型：用户/组织页面、项目页面，这两种类型基本上是一致的，只有部分区别：用户/组织页面每个账户只能创建一个，可以使用`http://<username>.github.io`链接直接访问；项目页面是针对项目，每个项目可以创建一个对应的项目页面，通过`http://github.com/<projectname>`链接访问。

创建一个repository，名称为：daemon369.github.com，这样创建的博客直接通过网址<http://daemon369.github.com>可以直接访问。

然后clone一个现有的模板，我使用了[codepiano的博客](http://codepiano.github.io/)作为模板：

    git clone https://github.com/codepiano/codepiano.github.com.git

然后拷贝本地`codepiano.github.com`文件夹下除了.git文件夹以外的所有文件到我的本地新建目录`daemon369.github.com`下。

可以现在本地看看是否能够正常显示：

    $ cd daemon369.github.com
    $ jekyll serve -w

打开浏览器，输入网站：<http://localhost:4000>，就可以本地查看网站的显示情况。如果显示正常就可以上传github，有问题就可以实时修改。

上传到github：

    $ cd daemon369.github.com
    $ git init
    $ git add .
    $ git commit -a -m "upload my blog"
    $ git push -u origin master

当前直接clone的别人的博客，很多地方显示的都是不符合个人要求的，网址链接有可能也是链到原作者的博客，所以后面需要自己修改，`_posts`目录下面的文章也要都替换成自己的文章。

# 参考

1.[搭建一个免费的，无限流量的Blog----github Pages和Jekyll入门][1]

2.[Github Pages极简教程][2]

3.[使用Github Pages建独立博客][3]

[1]:http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html "搭建一个免费的，无限流量的Blog----github Pages和Jekyll入门"

[2]:http://yanping.me/cn/blog/2012/03/18/github-pages-step-by-step/ "Github Pages极简教程"

[3]:http://beiyuu.com/github-pages/ "使用Github Pages建独立博客"
