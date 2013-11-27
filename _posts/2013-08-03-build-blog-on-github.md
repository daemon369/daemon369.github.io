---
layout: post
keywords: blog,github,jekyll
description: How to build my blog on github
title: "在github上搭建博客"
categories: [Blog]
tags: [Blog, github pages, jekyll]
group: Blog
icon: file-alt
---
{% include JB/setup %}

最开始是看到[阮一峰](http://www.ruanyifeng.com/blog/)的博客[搭建一个免费的，无限流量的Blog----github Pages和Jekyll入门](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html)，才知道可以使用github来搭建个人博客的。因为刚在github上面创建了一个repository用来放一些自己的学习过程中写的demo，那么再创建个博客来记录自己的学习过程，也是个不错的事情。因此开始折腾这个博客。

<!--excerpt-->

我对github不是很熟悉，对Jekyll，Markdown也是之后查资料才知道的，虽然开始很麻烦，但是能够接触点新东西来充实自己，也是一件愉快的事情。

最开始，我是跟着阮一峰的[文章][1]一步步做的，新手可以照着这个步骤跟着做，做完后对整个流程也就有了大致的了解了。

自己一步步写的终究比较慢，而且我对这方面没什么功底，还是直接clone别人的模板，替换上自己的文章，然后在此基础上进行修改比较快些。对于我这种菜鸟来说，这样能最迅速的搭建一个比较漂亮丰富的博客。

下面给出我目前的博客搭建的简要过程。当然，实际我折腾的时候，费了很多力气，走了不少弯路。

首先申请个github帐号，这个很简单。我的账户名为：daemon369，因此我的个人首页为：<a href="https://github.com/daemon369">https://github.com/daemon369</a>。

创建一个repository，名称为：daemon369.github.com，这样创建的博客直接通过网址http://daemon369.github.com可以直接访问，每个用户只可以创建一个。还有一种方式是与repository相关连的博客或主页，每个项目可以关联一个，访问地址需要在前面的网站的基础上加上repository名称。我选择的是第一种方式。阮一峰的[文章][1]中介绍的方法使用的是第二种方式。

然后clone一个现有的模板，我使用了codepiano的博客作为模板，链接见文章末尾。

    git clone https://github.com/codepiano/codepiano.github.com.git

然后拷贝本地codepiano.github.com文件夹下除了.git文件夹以外的所有文件到我的本地新建目录daemon369.github.com下。

可以现在本地看看是否能够正常显示：

    $ cd daemon369.github.com
    $ jekyll serve

打开浏览器，输入网站：http://localhost:4000，就可以本地查看网站的显示情况。如果显示正常就可以上传github，有问题就可以实时修改。

很多博客给出的是旧版本的命令：jekyll --server，目前我使用的1.1.2版本的jeykll已经废弃了旧的命令。

上传到github：

    $ cd daemon369.github.com
    $ git init
    $ git add .
    $ git commit -a -m "upload my blog"
    $ git push -u origin master

当前直接clone的别人的博客，很多地方显示的都是不符合个人要求的，网址链接有可能也是链到原作者的博客，所以后面需要自己修改，_posts目录下面的文章也要都替换成自己的文章。

参考：
[1]:[搭建一个免费的，无限流量的Blog----github Pages和Jekyll入门](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html)

[2]:[Github Pages极简教程](http://yanping.me/cn/blog/2012/03/18/github-pages-step-by-step/)

[3]:[使用Github Pages建独立博客](http://beiyuu.com/github-pages/)

我的博客初始模板：

博客：[http://codepiano.github.io/](http://codepiano.github.io/)

github：[https://github.com/codepiano/codepiano.github.com](https://github.com/codepiano/codepiano.github.com)
