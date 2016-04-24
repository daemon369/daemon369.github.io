---
layout: post
keywords: blog, jekyll
description: jekyll front-matter
title: "Jekyll文档(四) -- Front-matter"
categories: [Blog]
tags: [Blog, jekyll]
group: Blog
icon: file-alt
date: 2013-09-13 00:00:00
---

jekyll项目中，包含YAML front matter代码区块的所有文件都会被jekyll进行特殊处理。Front-matter能够使得jekyll真正的变酷起来。文件中需要把front-matter到文件的最前端，格式是上下两行三个短横线，中间包裹合法的YAML代码。下面给出一个简单的例子：

<!--excerpt-->

    ---
    layout: post
    title: Blogging Like a Hacker
    ---

三段横线之间可以设置预定义的变量，也可以自己定义变量。这些变量能够使用Liquid标签来使用。你可以在这个文件中，也可以在这个文件依赖的布局文件或include文件中使用。

###!!UTF-8编码需要注意
如果使用UTF-8编码，需要注意文件中不能包含*BOM*头。否则jekyll会产生很大的问题。尤其是在windows系统中使用jekyll中更需要注意。

###Front Matter变量是可选项
如果你希望在文件中使用Liquid标签和变量，而不需要在front-matter设置变量等内容，那么front-matter可以留空。两行三短横线中留空也会被jekyll特殊处理。(在CSS或者RSS中比较有用)

***
#预定义的全局变量

jekyll中预先定义了一些全局变量，用户可以在page或post的front-matter中进行设置。

<table cellpadding="10">
  <col width="25%" />
  <col width="75%" />
  <tr>
    <th>变量</th>     
    <th>描述</th>
  </tr>
  <tr>
    <td>
      <p>layout</p>
    </td>
    <td>
      <p>设置这个变量可以指定使用某个指定的布局文件。布局文件不需要加文件名后缀。布局文件需要放到<em>_layouts</em>文件夹中。</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>permalink</p>
    </td>
    <td>
      <p>发布的post的url默认会是<em>/year/month/day/title.html</em>的格式，用户可以设置这个变量来设置静态URL。</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>published</p>
    </td>
    <td>
      <p>post设置这个变量为false，那么这个post在产生网页时不会发布。</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>category</p>
      <p>categories</p>
    </td>
    <td>
      <p>Instead of placing posts inside of folders, you can specify one or more categories that the post belongs to. When the site is generated the post will act as though it had been set with these categories normally. 类别(Categories)可以使用YAML列表的形式或者使用空格分隔的字符串形式。</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>tags</p>
    </td>
    <td>
      <p>与分类(categories)类似，可以定义一个或多个标签(tags)，同样的也可以用YAML列表形式或空格分隔的字符串形式</p>
    </td>
  </tr>
</table>

***
#自定义变量

Front-matter中定义的所有非预定义变量在jekyll转换文件时，都会和数据一起发送给Liquid模板引擎。譬如，如果你设置了变量*title*，那么你就可以在你的布局文件中用这个变量来设置你的page标题：

    <!DOCTYPE HTML>
    <html>
      <head>
        <title>{% raw %}{{ page.title }}{% endraw %}</title>
      </head>
      <body>
        ...

***
#为Posts预定义的变量

These are available out-of-the-box to be used in the front-matter for a post.


<table cellpadding="10">
  <col width="25%" />
  <col width="75%" />
  <tr>
    <th>变量</th>     
    <th>描述</th>
  </tr>
  <tr>
    <td>
      <p>date</p>
    </td>
    <td>
      <p>设置变量可以覆盖post名称中定义的日期，可以用来保证posts的排序的正确性。</p>
    </td>
  </tr>
</table>

***
#PS:
文章翻译自jekyll官方文档(2013-09-13)(有很多地方能够理解却不好翻译，或者不太理解，先留英文原文，后面理解透彻了再改)：

[Configuration](http://jekyllrb.com/docs/frontmatter/)