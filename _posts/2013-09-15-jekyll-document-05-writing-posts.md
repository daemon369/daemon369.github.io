---
layout: post
keywords: blog, jekyll
description: jekyll writing posts
title: "Jekyll文档(五) -- Writing posts"
categories: [Blog]
tags: [Blog, jekyll]
group: Blog
icon: file-alt
---
{% include JB/setup %}

jekyll是"blog aware"的，就是说写博客被设计成jekyll的固有功能。用户需要写博客以及发布博客到网络，只需要在本地管理一个文件夹，文件夹里包含一些文本文件即可。相比在数据库中配置保存博客以及使用基于网页的CMS系统来管理操作，这是个饱受欢迎的改变。

<!--excerpt-->

##Posts文件夹

前面介绍过，jekyll使用*\_posts*文件夹来保存posts。你可以使用*Markdown*或者*Textile*格式的文件，只需要在文件首部加上*YAML front-matter*，这样这些posts就会被jekyll转换成HTML网页，作为你的静态网站的一部分。

###创建Post文件

创建post只需要在*\_posts*目录下新建个文件即可，这个文件需要遵循一定的格式，这是jekyll要求的，格式如下：

    YEAR-MONTH-DAY-title.MARKUP

前面的三项是日期，其中年(YEAR)是4位的数字，月(MONTH)和日(DAY)都是两位的数字，MARKUP是文件的后缀，代表了文件的格式。下面是两个实例：

    2011-12-31-new-years-eve-is-awesome.md
    2012-09-12-how-to-write-a-blog.textile

###内容格式

所有的博客文件必须以*YAML front-matter*开头。然后剩余部分用户就可以自己选择使用哪种格式了。Jekyll提供了两种流行的内容标记格式：Markdown和Textile。每种格式都有自己的标记内容的方法，你需要熟悉这两种格式并选择你自己需要的格式。

##在文章中包含图片及其他资源

有时候用户需要在post中添加图片，下载文件或其他内容。因为语法的不同，Markdown和Textlie的链接的表达方式不同，因此存储图片，二进制文件等的位置也有所不同。

由于Jekyll的灵活性，有很多不同的解决方案来解决这个问题。通用的解决方案是在根目录创建一个文件夹，例如*assets*或者*downloads*，图片，下载文件和其他资源就可以放到这个目录下。这样，post中就可以以根目录为起始路径来链接到这些资源文件。当然，这要依赖与你的网站的（子）域名和路径的配置。下面给出一些利用*site.url*变量来实现的链接(使用Markdown)。

在post中添加图片资源：

    … which is shown in the screenshot below:
    ![My helpful screenshot]({% raw %}{{ site.url }}{% endraw %}/assets/screenshot.jpg)
    
在post中添加pdf文件的链接提供下载：

    … you can [get the PDF]({% raw %}{{ site.url }}{% endraw %}/assets/mydoc.pdf) directly.

###链接仅使用网站root URL

如果你明确知道你的网站只会展示到你的域名的根URL是，你可以省略掉*{% raw %}{{ site.url }}{% endraw %}*变量的使用。例如，你可以使用*/path/file.jpg*直接链接到图片。

##显示posts的索引

所有的posts都放到一个目录，如果不能提供一个posts的列表，那么这些blog都是无用的，因为无法访问。在一个单独的页面（或者在[template](http://jekyllrb.com/docs/templates/)中）创建一个posts的索引很简单，这要感谢[Liquid模板语言](http://wiki.shopify.com/Liquid)以及其标签。下面是一个创建posts链接的列表的基本例子：

    <ul>
      {% raw %}{% for post in site.posts %}{% endraw %}
        <li>
          <a href="{% raw %}{{ post.url }}{% endraw %}">{% raw %}{{ post.title }}{% endraw %}</a>
        </li>
      {% raw %}{% endfor %}{% endraw %}
    </ul>

当然，怎样显示和在哪显示你的posts，以及你的网站的结构是怎么样的，你都是可以自己控制的。想了解更多可以参看[how template work](http://jekyllrb.com/docs/templates/)

##摘要

每个post都会自动将文章的开始直到*excerpt_separator*第一次出现的地方之间的文本作为摘要，并设置变量*post.excerpt*。在上面给出的posts索引的例子，你或许想告诉浏览者每篇文章的大致内容，可以用每篇post的第一段作为摘要：

    <ul>
      {% raw %}{% for post in site.posts %}{% endraw %}
        <li>
          <a href="{% raw %}{{ post.url }}{% endraw %}">{% raw %}{{ post.title }}{% endraw %}</a>
          <p>{% raw %}{{ post.excerpt }}{% endraw %}</p>
        </li>
      {% raw %}{% endfor %}{% endraw %}
    </ul>

如果你不想使用post自动生成的摘要，可以在post的YAML front-matter中添加设置*excerpt*。完全禁止掉摘要可以设置*excerpt_separator*为*""*。

##代码段高亮

Jekyll内置支持使用Pygments进行代码段的高亮显示。在post中添加代码段高亮显示很简单，只要使用如下专用的Liquid标签：

    {% raw %}{% highlight ruby %}{% endraw %}
    def show
      @widget = Widget(params[:id])
      respond_to do |format|
        format.html # show.html.erb
        format.json { render json: @widget }
      end
    end
    {% raw %}{% endhighlight %}{% endraw %}

显示的效果如下：

{% highlight ruby %}
def show
  @widget = Widget(params[:id])
  respond_to do |format|
    format.html # show.html.erb
    format.json { render json: @widget }
  end
end
{% endhighlight %}

###显示行号
代码段也可以显示行号，添加*linenos*到高亮标签中即可，例如：*{% raw %}{% highlight ruby linenos %}{% endraw %}*。

了解这些基本的东西后，写post已经足够了。继续深入可以参看[customizing post permalinks](http://jekyllrb.com/docs/permalinks/)和[custom variables](http://jekyllrb.com/docs/variables/)，这些可用在posts中以及整个网站。

***
#PS:
文章翻译自jekyll官方文档(2013-09-15)：

[Writing posts](http://jekyllrb.com/docs/posts/)