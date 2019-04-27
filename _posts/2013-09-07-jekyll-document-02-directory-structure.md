---
layout: post
keywords: blog
description: 
title: "Jekyll文档(二) -- Directory structure"
categories: [Blog]
tags: [Blog, jekyll]
group: Blog
icon: file-alt
date: 2013-09-07 00:00:00
---

# 目录结构

jekyll是一个文本转换引擎，其原理为：

用户提供一些固定格式的文本，jekyll可以按设定好的规则将其转换为HTML文件等，然后就可以部署成网站给其他人观看；有很多种书写文本的格式可供选择，例如：Markdown，Textile，甚至是普通的HTML。用户还可以自己定义网站的布局，只要用户使用jekyll定义好的规则来编写布局文件，jekyll就会按照用户的设计生成对应的静态网页。

<!--excerpt-->

jekyll项目的基本组织结构一般如下：

    .
    ├── _config.yml
    ├── _drafts
    |   ├── begin-with-the-crazy-ideas.textile
    |   └── on-simplicity-in-technology.markdown
    ├── _includes
    |   ├── footer.html
    |   └── header.html
    ├── _layouts
    |   ├── default.html
    |   └── post.html
    ├── _posts
    |   ├── 2007-10-29-why-every-programmer-should-play-nethack.textile
    |   └── 2009-04-26-barcamp-boston-4-roundup.textile
    ├── _site
    └── index.html

***

# 功能

jekyll项目中的每一部分的功能如下：

<table cellpadding="10">
  <tr>
    <th>文件/目录</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>_config.yml</td>
    <td>jekyll的配置文件，jekyll的很多参数可以在输入命令行时直接输入，也可以写入配置文件，简化繁琐的命令输入过程。</td>
  </tr>
  <tr>
    <td>_drafts</td>
    <td>这个文件夹里面是用来存放草稿的，里面的文件不需要在文件名中加入固定格式的日期前缀。</td>
  </tr>
  <tr>
    <td>_includes</td>
    <td>这里面可以放置一些可重用的模块来加速开发，你可以方便的将里面的内容在多个文件里复用。譬如你也可以在多个页面中加入一块固定的模块，这个模块存放在文件_includes/file.ext，只需要在文件中插入liquid标签： {% raw %}{% include file.ext %}{% endraw %} 。</td>
  </tr>
  <tr>
    <td>_layouts</td>
    <td>这个目录存放的是一些布局文件，我们的post文章里面可以使用YAML前缀格式来设置我们使用的布局。我们可以使用liquid标签 {% raw %}{{ content }}{% endraw %} 在我们的web页中插入内容。</td>
  </tr>
  <tr>
    <td>_posts</td>
    <td>这里是用来放置你的文章的，里面的文件需要加入日期前缀，格式为： YEAR-MONTH-DAY-title.MARKUP 。前面的日期格式是固定的，jekyll可以自动提取出文件名中的日期前缀用来标识文件发布的日期，这个日期可以用来排序，分类，显示等。</td>
  </tr>
  <tr>
    <td>_site</td>
    <td>jekyll默认会将jekyll项目中的内容生成静态网页放到这个目录中，这个目录是可以自定义的，可以写到配置文件_config.yml中。默认这个目录会加入.gitignore文件中，里面的内容不会上传到github，github page会自动根据你上传的jekyll项目生成对应的网页，而不会直接展示你自己本地生成的网页。</td>
  </tr>
  <tr>
    <td>index.html</td>
    <td>默认的主页文件，可以在文件头部加入YAML区域，内容中也可以加入Markdown等格式的内容。这个文件也会被jekyll转换成对应网页，不会直接显示你定义的内容。文件后缀可以是：.html, .markdown, .md, .textile 等。</td>
  </tr>
  <tr>
    <td>其他</td>
    <td>除了上面列出的文件和目录，其他的文件和目录一般都会原样复制到生成的网站里。也有例外，譬如用户除了首页index.html，还创建了其他的页面，这些页面也可以用Markdown等格式编写，最终也会被转化为普通网页。</td>
  </tr>
</table>

***

# PS:
文章翻译自jekyll官方文档(2013-09-07)：

[Directory structure](http://jekyllrb.com/docs/structure/)
