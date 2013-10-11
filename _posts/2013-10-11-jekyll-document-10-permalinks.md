---
layout: post
keywords: blog, jekyll
description: jekyll permalinks
title: "Jekyll文档(十) -- Permalinks"
categories: [Blog]
tags: [Blog, jekyll]
group: Blog
icon: file-alt
---
{% include JB/setup %}

Jekyll允许你灵活定义你的URL。你可以通过[配置项](http://jekyllrb.com/docs/configuration/)或者在每篇post的[YAML Front Matter](http://jekyllrb.com/docs/frontmatter/)中指定网站的静态链接。Jekyll有一些URL的内置样式可选，你也可以自己定义。默认的样式是日期样式*date*。

Jekyll的静态链接是由设定的或默认的URL模板构建的，其中的动态元素是由以冒号为前缀的关键词给出的。例如，默认的*date*样式的静态链接定义如下：

    /:categories/:year/:month/:day/:title.html

<!--excerpt-->

***
#模板变量(Template variables)

<table cellpadding="10">
  <col width="25%" />
  <col width="75%" />
  <tr>
    <th>变量</th>     
    <th>描述</th>
  </tr>
  <tr>
    <td>
      <p>year</p>
    </td>
    <td>
      <p>post文件名中的年份</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>month</p>
    </td>
    <td>
      <p>post文件名中的月份</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>i_month</p>
    </td>
    <td>
      <p>post文件名中的月份，去除前面的0</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>day</p>
    </td>
    <td>
      <p>post文件名中的日期</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>i_day</p>
    </td>
    <td>
      <p>post文件名中的日期，去除前面的0</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>title</p>
    </td>
    <td>
      <p>post文件名中的标题title</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>categories</p>
    </td>
    <td>
      <p>post指定的分类，Jekyll会自动将类别解析出来使用放到双斜线中作为URL的一部分。如果没有设置类别，这部分就会被略过。</p>
    </td>
  </tr>
</table>

***
#内置静态链接样式(Tags)

<table cellpadding="10">
  <col width="25%" />
  <col width="75%" />
  <tr>
    <th>静态链接样式</th>     
    <th>URL模板</th>
  </tr>
  <tr>
    <td>
      <p>date</p>
    </td>
    <td>
      <p>/:categories/:year/:month/:day/:title.html</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>pretty</p>
    </td>
    <td>
      <p>/:categories/:year/:month/:day/:title/</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>none</p>
    </td>
    <td>
      <p>/:categories/:title.html</p>
    </td>
  </tr>
</table>

***
#静态链接样式示例

给定一个post名为：*/2009-04-29-slap-chop.textile*

<table cellpadding="10">
  <col width="50%" />
  <col width="50%" />
  <tr>
    <th>静态链接设置</th>     
    <th>静态链接URL</th>
  </tr>
  <tr>
    <td>
      <p>没有指定或者指定为：<em>permalink: date</em></p>
    </td>
    <td>
      <p>/2009/04/29/slap-chop.html</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>permalink: pretty</p>
    </td>
    <td>
      <p>/2009/04/29/slap-chop/index.html</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>permalink: /:month-:day-:year/:title.html</p>
    </td>
    <td>
      <p>/04-29-2009/slap-chop.html</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>permalink: /blog/:year/:month/:day/:title</p>
    </td>
    <td>
      <p>/blog/2009/04/29/slap-chop/index.html</p>
    </td>
  </tr>
</table>


***
#PS:
文章翻译自jekyll官方文档(2013-10-11)：

[Permalinks](http://jekyllrb.com/docs/permalinks/)