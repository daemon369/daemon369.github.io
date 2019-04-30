---
layout: post
keywords: blog, jekyll
description: jekyll creating pages
title: "Jekyll文档(六) -- Creating pages"
categories: [Blog]
tags: [Blog, jekyll]
group: Blog
icon: file-alt
date: 2013-09-17 00:00:00
---

很多网站默认启动服务会找HTML文件*index.html*作为网站的首页来展示。Jekyll也一样，除非你自己在web服务器上配置了其他文件，否则Jekyll也会默认将*index.html*文件作为首页。

<!--excerpt-->

### 在首页中使用布局layouts

Jekyll项目中的任何HTML文件都可以使用布局(layouts)和包含文件(includes)，包括首页。一些通用的元素，例如头部和尾部，可以抽取出来做到布局中，这样方便开发，网站的风格也能统一。

***

# 额外的页面放置在哪？

在哪里放置HTML页面文件取决与你想让你的页面如何工作。有两种主要的创建页面的方式：

<ol>
    <li>
      放置HTML文件到网站根目录，每个页面单独命名。
    </li>
    <li>
      在网站根目录为每个页面创建一个单独命名的文件夹，每个文件夹里放置一个名为<em>index.html</em>的文件
    </li>
</ol>

这两种方式都能很好的工作，也可以混合使用，唯一的不同就是最终访问路径URL的不同。

## 命名HTML文件

最方便的创建页面的方式就是在根目录创建HTML文件并给予合适的文件名。下面给出一个包含主页，关于页，联系页的网站的组织情况：

    .
    |-- _config.yml
    |-- _includes/
    |-- _layouts/
    |-- _posts/
    |-- _site/
    |-- about.html    # => http://yoursite.com/about.html
    |-- index.html    # => http://yoursite.com/
    └── contact.html  # => http://yoursite.com/contact.html

## 包含index HTML文件的命名目录

上面一种组织方式是没有任何问题的，但是由于有的人喜欢他们的页面的URL不包含文件扩展名等一些东西。你可以为每一个顶层页面创建一个目录，然后在每个目录中放置*index.html*文件，这样就可以让你的URL保持干净整洁。这样页面的URL就会以目录名为结尾，访问时web服务器会提供各自的*index.html*文件。示例如下：

    .
    ├── _config.yml
    ├── _includes/
    ├── _layouts/
    ├── _posts/
    ├── _site/
    ├── about/
    |   └── index.html  # => http://yoursite.com/about/
    ├── contact/
    |   └── index.html  # => http://yoursite.com/contact/
    └── index.html      # => http://yoursite.com/

这种方法或许并不适合每个人，但是适合希望URL保持干净整洁的人。最终的选择取决与你的决定。

***

# PS:

文章翻译自jekyll官方文档(2013-09-17)：

[Creating pages](http://jekyllrb.com/docs/pages/)
