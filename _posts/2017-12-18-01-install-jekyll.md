---
layout: post
keywords: jekyll
description: install jekyll
title: "安装 Jekyll"
categories: [Jekyll, Blog]
tags: [jekyll blog]
group: Blog
icon: file-alt
date: 2017-12-18 20:30:00
---
# 安装`Ruby gems`

首先需要安装`Ruby`，请参看：[安装Ruby](http://daemon369.github.io/ruby/2017/12/14/01-install-ruby)

下面是维基百科中对`RubyGems`的解释：

```
RubyGems是Ruby的一个包管理器，提供了分发Ruby程序和函式库的标准格式“gem”，旨在方便地管理gem安装的工具，以及用于分发gem的服务器。这类似于Python的pip。RubyGems大约创建于2003年11月，从Ruby 1.9版起成为Ruby标准库的一部分。
```

<!--excerpt-->

安装`Ruby`后，使用如下命令查看`gem`：

    $ gem -v
    2.6.14

# 更新`gem`

    $ gem update --system

# 安装`Bundler`

`Bundler`是`gem`管理工具，安装如下：

    $ gem install bundler

查看`bundler`版本：

    $ bundler -v
    Bundler version 1.16.0

# 安装`Jekyll`

    $ gem install jekyll

查看`Jekyll`版本：

    $ jekyll -v
    jekyll 3.6.2

新建博客：

    $ jekyll new my-blog

在本地启动web服务查看博客：

    $ cd my-blog
    $ bundle exec jekyll serve
    Configuration file: /Users/user/my-blog/_config.yml
                Source: /Users/user/my-blog
           Destination: /Users/user/my-blog/_site
     Incremental build: disabled. Enable with --incremental
          Generating...
                        done in 0.412 seconds.
     Auto-regeneration: enabled for '/Users/user/my-blog'
        Server address: http://127.0.0.1:4000/
      Server running... press ctrl-c to stop.

使用浏览器打开网址`http://127.0.0.1:4000/`即可在本地查看博客内容。

# 参看

[RubyGems](https://zh.wikipedia.org/wiki/RubyGems)

[Transform your plain text into static websites and blogs](https://jekyllrb.com/)
