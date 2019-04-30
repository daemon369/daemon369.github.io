---
layout: post
keywords: blog, jekyll
description: jekyll pagination
title: "Jekyll文档(十一) -- Pagination"
categories: [Blog]
tags: [Blog, jekyll]
group: Blog
icon: file-alt
date: 2013-11-24 00:00:00
---

对于很多网站来说，尤其是对于博客，我们很常见的将我们的posts的很长的列表分成多个小的列表并分多页进行展示。Jekyll有内置的分页，我们可以自行为分页的列表产生合适的文件和目录。

### 分页只能适用于HTML文件

Jekyll的分页并不适用于Jekyll网站中的Markdown或者Textile的文件，只适用于HTML文件。一般我们都是使用分页来展示posts，因此这个问题基本没有影响。

<!--excerpt-->

***

# 开启分页(Enable pagination)

在博客中开启分页功能需要在*\_config.yml*文件中指定每页需要显示的项数：

    paginate: 5

这个项数就是每页显示的posts的最大数目。

可以指定分页的页面放置的目标位置：

    paginate_path: "blog/page:num"

这里*:num*是分页的页号，从*2*开始。Jekyll会将首页写到*blog/index.html*中，将每个分页作为*paginator*并写到*blog/page:num*中。如果网站有12篇文章(posts)，指定*paginate:5*，Jekyll就会将开始的5篇写到*blog/index.html*中，之后的5篇写到*blog/page2/index.html*中，最后的2篇写到*blog/page3/index.html*中。

***

# 可用的Liquid属性

分页插件提供的*paginator*liquid对象有以下属性：

<table cellpadding="10">
  <col width="25%" />
  <col width="75%" />
  <tr>
    <th>属性</th>     
    <th>描述</th>
  </tr>
  <tr>
    <td>
      <p>page</p>
    </td>
    <td>
      <p>当前页数</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>per_page</p>
    </td>
    <td>
      <p>每页文章数量</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>posts</p>
    </td>
    <td>
      <p>当前页面中文章的列表</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>total_posts</p>
    </td>
    <td>
      <p>网站中所有文章的数目</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>total_pages</p>
    </td>
    <td>
      <p>所有分页的页数</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>previous_page</p>
    </td>
    <td>
      <p>前一个分页的页号，如果前面没有分页就是<em>nil</em></p>
    </td>
  </tr>
  <tr>
    <td>
      <p>previous_page_path</p>
    </td>
    <td>
      <p>上一个分页的路径，如果没有就是<em>nil</em></p>
    </td>
  </tr>
  <tr>
    <td>
      <p>next_page</p>
    </td>
    <td>
      <p>下一个分页的页号，如果前面没有分页就是<em>nil</em></p>
    </td>
  </tr>
  <tr>
    <td>
      <p>next_page_path</p>
    </td>
    <td>
      <p>下一个分页的路径，如果没有就是<em>nil</em></p>
    </td>
  </tr>
</table>

### 分页不支持标签(tags)和类别(categories)

Pagination pages through every post in the posts variable regardless of variables defined in the YAML Front Matter of each. It does not currently allow paging over groups of posts linked by a common tag or category.

***

# 展示分页的文章列表(Render the paginated Posts)

下一步要做的就是使用*paginator*变量来真正的展示你的文章列表，一般我们会在网站主页面的其中一页来显示文章列表。下面是一个简单的将分页的文章render到一个HTML文件中的例子：

    ---
    layout: default
    title: My Blog
    ---
    
    <!-- This loops through the paginated posts -->
    {% raw %}{% for post in paginator.posts %}{% endraw %}
      <h1><a href="{% raw %}{{ post.url }}{% endraw %}">{% raw %}{{ post.title }}{% endraw %}</a></h1>
      <p class="author">
        <span class="date">{% raw %}{{ post.date }}{% endraw %}</span>
      </p>
      <div class="content">
        {% raw %}{{ post.content }}{% endraw %}
      </div>
    {% raw %}{% endfor %}{% endraw %}
    
    <!-- Pagination links -->
    <div class="pagination">
      {% raw %}{% if paginator.previous_page %}{% endraw %}
        <a href="/page{% raw %}{{ paginator.previous_page }}{% endraw %}" class="previous">Previous</a>
      {% raw %}{% else %}{% endraw %}
        <span class="previous">Previous</span>
      {% raw %}{% endif %}{% endraw %}
      <span class="page_number ">Page: {% raw %}{{ paginator.page }}{% endraw %} of {% raw %}{{ paginator.total_pages }}{% endraw %}</span>
      {% raw %}{% if paginator.next_page %}{% endraw %}
        <a href="/page{% raw %}{{ paginator.next_page }}{% endraw %}" class="next">Next</a>
      {% raw %}{% else %}{% endraw %}
        <span class="next ">Next</span>
      {% raw %}{% endif %}{% endraw %}
    </div>

### !!注意页号为1的边界条件

Jekyll不会产生‘page1’的目录，所以上面的代码在*/page1*链接产生后不起作用。可以参考下面的方法处理这个问题。

下面的HTML代码片段处理了第一页的情况，同时提供了除了当前页的其他所有页面的链接列表。

    {% raw %}{% if paginator.total_pages > 1 %}{% endraw %}
    <div class="pagination">
      {% raw %}{% if paginator.previous_page %}{% endraw %}
        <a href="{% raw %}{{ paginator.previous_page_path | prepend: site.baseurl | replace: '//', '/' }}{% endraw %}">&laquo; Prev</a>
      {% raw %}{% else %}{% endraw %}
        <span>&laquo; Prev</span>
      {% raw %}{% endif %}{% endraw %}

      {% raw %}{% for page in (1..paginator.total_pages) %}{% endraw %}
        {% raw %}{% if page == paginator.page %}{% endraw %}
          <em>{% raw %}{{ page }}{% endraw %}</em>
        {% raw %}{% elsif page == 1 %}{% endraw %}
          <a href="{% raw %}{{ '/index.html' | prepend: site.baseurl | replace: '//', '/' }}{% endraw %}">{% raw %}{{ page }}{% endraw %}</a>
        {% raw %}{% else %}{% endraw %}
          <a href="{% raw %}{{ site.paginate_path | prepend: site.baseurl | replace: '//', '/' | replace: ':num', page }}{% endraw %}">{% raw %}{{ page }}{% endraw %}</a>
        {% raw %}{% endif %}{% endraw %}
      {% raw %}{% endfor %}{% endraw %}
    
      {% raw %}{% if paginator.next_page %}{% endraw %}
        <a href="{% raw %}{{ paginator.next_page_path | prepend: site.baseurl | replace: '//', '/' }}{% endraw %}">Next &raquo;</a>
      {% raw %}{% else %}{% endraw %}
        <span>Next &raquo;</span>
      {% raw %}{% endif %}{% endraw %}
    </div>
    {% raw %}{% endif %}{% endraw %}

***

# PS:

文章翻译自jekyll官方文档(2013-10-11)：

[Permalinks](http://jekyllrb.com/docs/permalinks/)
