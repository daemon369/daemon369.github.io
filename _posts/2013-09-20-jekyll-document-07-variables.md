---
layout: post
keywords: blog, jekyll
description: jekyll variables
title: "Jekyll文档(七) -- Variables"
categories: [Blog]
tags: [Blog, jekyll]
group: Blog
icon: file-alt
date: 2013-09-20 00:00:00
---

Jekyll会遍历网站寻找可以处理的文件。每个包含[YAML Front Matter](http://jekyllrb.com/docs/frontmatter/)的文件都会被Jekyll处理。对于每个一文件，Jekyll都会通过[Liquid templating system](https://github.com/shopify/liquid/wiki/liquid-for-designers)来创建很多可供使用的数据。下面是一些可用的数据。

<!--excerpt-->

***
#全局变量

<table cellpadding="10">
  <col width="25%" />
  <col width="75%" />
  <tr>
    <th>变量</th>     
    <th>说明</th>
  </tr>
  <tr>
    <td>
      <p>site</p>
    </td>
    <td>
      <p><em>_config.yml</em>文件中配置的全网站范围的信息和配置项。更详细的内容参考下面。</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>page</p>
    </td>
    <td>
      <p>page专用的信息以及<a href="http://jekyllrb.com/docs/frontmatter/">YAML Front Matter</a>。用户在YAML front matter中设置的自定义变量也可以在这里使用。更详细的内容参考下面。</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>content</p>
    </td>
    <td>
      <p>在布局文件中，post和page的内容会被封装起来。这个变量在post和page文件中没有定义。</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>paginator</p>
    </td>
    <td>
      <p>这个变量在变量<em>paginate</em>变量被设置后才产生作用。详细内容参考<a href="http://jekyllrb.com/docs/pagination/">Pagination</a></p>
    </td>
  </tr>
</table>

***
#Site变量

<table cellpadding="10">
  <col width="25%" />
  <col width="75%" />
  <tr>
    <th>变量</th>     
    <th>说明</th>
  </tr>
  <tr>
    <td>
      <p>site.time</p>
    </td>
    <td>
      <p>当前时间(当你运行<em>jekyll</em>时)</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>site.pages</p>
    </td>
    <td>
      <p>所有pages的列表</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>site.posts</p>
    </td>
    <td>
      <p>所有posts的列表，按时间顺序倒序排列。</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>site.related_posts</p>
    </td>
    <td>
      <p>如果当前处理的页面是post，则这个变量包含了至多十个的相关的posts的列表。 By default, these are low quality but fast to compute. For high quality but slow to compute results, run the  jekyll command with the --lsi (latent semantic indexing) option.</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>site.categories.CATEGORY</p>
    </td>
    <td>
      <p>所有指定分类<em>CATEGORY</em>中的posts的列表。</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>site.tags.TAG</p>
    </td>
    <td>
      <p>所有包含标签<em>TAG</em>的posts的列表。</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>site.[CONFIGURATION_DATA]</p>
    </td>
    <td>
      <p>所有通过命令行以及通过<em>_config.yml</em>设置的变量都可以通过变量<em>site</em>来访问。例如，在配置文件中设置了<em>url:http://mysite.com</em>，那么在posts和pages中可以通过变量<em>site.url</em>来使用设置的变量值。在<em>watch</em>模式下，Jekyll并不会去解析<em>_config.yml</em>文件，只在Jeykll服务启动的时候才会解析，因此如果你改变了配置项的值，你需要重新启动Jekyll来使改变生效。</p>
    </td>
  </tr>
</table>

***
#Page变量

<table cellpadding="10">
  <col width="25%" />
  <col width="75%" />
  <tr>
    <th>变量</th>     
    <th>说明</th>
  </tr>
  <tr>
    <td>
      <p>page.content</p>
    </td>
    <td>
      <p>The un-rendered content of the Page.</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>page.title</p>
    </td>
    <td>
      <p>page的标题</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>page.excerpt</p>
    </td>
    <td>
      <p>The un-rendered excerpt of the Page.</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>page.url</p>
    </td>
    <td>
      <p>post的URL，不包含域名，以"/"开头。例如：<em>/2008/12/14/my-post.html</em></p>
    </td>
  </tr>
  <tr>
    <td>
      <p>page.date</p>
    </td>
    <td>
      <p>post指定的日期。这个变量可以在post的front matter中指定一个新的日期/时间来覆盖已有的日期，格式为：<em>YYYY-MM-DD HH:MM:SS</em>(假定是UTC世界标准时间)，或者：<em>YYYY-MM-DD HH:MM:SS +/-TTTT</em>(使用一个相对UTC的偏移量来指定时区，例如：<em>2008-12-14 10:30:00 +0900</em>）。</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>page.id</p>
    </td>
    <td>
      <p>post的唯一标识ID(对RSS feeds有用)。例如：<em>/2008/12/14/my-post</em>。</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>page.categories</p>
    </td>
    <td>
      <p>post所属的类别的列表。类别(categories)派生于<em>_posts</em>之上的目录结构。例如，一个设置<em>page.categories</em>变量为['work, 'code']的post将位于<em>/work/code/_posts/2008-12-24-closures.md</em>。这些可以在YAML Front Matter中进行设置。</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>page.tags</p>
    </td>
    <td>
      <p>post所属的tags的列表。可以在YAML Front Matter中进行设置。</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>page.path</p>
    </td>
    <td>
      <p>原生post或者page文件的路径。用法实例：链接到github上的page或者post的源文件。可以在YAML Front Matter中覆盖这个变量。</p>
    </td>
  </tr>
</table>

##使用自定义的front-matter

任何你设置的front matter都可以在*page*下使用。例如：如果你在page的front matter中设定了*custom_css: true*，那么你就可以使用变量*page.custom_css*。

***
#Paginator


<table cellpadding="10">
  <col width="25%" />
  <col width="75%" />
  <tr>
    <th>变量</th>     
    <th>说明</th>
  </tr>
  <tr>
    <td>
      <p>paginagor.per_page</p>
    </td>
    <td>
      <p>每个page页面包含的posts数目。</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>paginator.posts</p>
    </td>
    <td>
      <p>page页面可用的posts</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>paginator.total_posts</p>
    </td>
    <td>
      <p>posts的总数</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>paginator.total_pages</p>
    </td>
    <td>
      <p>pages的总数</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>paginator.page</p>
    </td>
    <td>
      <p>当前page页面的编号</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>paginator.previous_page</p>
    </td>
    <td>
      <p>前一个page页面的编号</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>paginator.previous_page_path</p>
    </td>
    <td>
      <p>前一个page页面的path路径</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>paginator.next_page</p>
    </td>
    <td>
      <p>后一个page页面的编号</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>paginator.next_page_path</p>
    </td>
    <td>
      <p>后一个page页面的path路径</p>
    </td>
  </tr>
</table>

##paginator变量的用途

这些变量只能用在index索引文件中。当然，索引文件可以放到子目录里，例如:*/blog/index.html*。

***
#PS:
文章翻译自jekyll官方文档(2013-09-20)：

[Variables](http://jekyllrb.com/docs/variables/)