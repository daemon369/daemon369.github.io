---
layout: post
keywords: blog, jekyll
description: jekyll templates
title: "Jekyll文档(九) -- Templates"
categories: [Blog]
tags: [Blog, jekyll]
group: Blog
icon: file-alt
---
{% include JB/setup %}

Jekyll使用[Liquid](http://wiki.shopify.com/Liquid)模板语言来处理模板，它支持Liquid的所有[标签tags](http://wiki.shopify.com/Logic)和[过滤器filters](http://wiki.shopify.com/Filters)。Jekyll还增加了一些自身独有的标签和过滤器，用来简化通用任务。

<!--excerpt-->

***
#过滤器(Filters)

<table cellpadding="10">
  <col width="45%" />
  <col width="65%" />
  <tr>
    <th>描述</th>     
    <th>过滤器(FILTER)和输出(OUTPUT)</th>
  </tr>
  <tr>
    <td>
      <p>Date to XML Schema</p>
      <p>将日期转化为XML格式(ISO 8601)</p>
    </td>
    <td>
      <p>{% raw %}{{ site.time | data_to_xmlschema }}{% endraw %}</p>
      <p>2008-11-17T13:07:54-08:00</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>Date to RFC-822 Format</p>
      <p>将日期转换为RFC-822格式，在RSS订阅中有用</p>
    </td>
    <td>
      <p>{% raw %}{{ site.time | data_to_rfc822 }}{% endraw %}</p>
      <p>Mon, 17 Nov 2008 13:07:54 -0800</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>Date to String</p>
      <p>将日期转换为短字符串格式</p>
    </td>
    <td>
      <p>{% raw %}{{ site.time | data_to_string }}{% endraw %}</p>
      <p>17 Nov 2008</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>Date to Long String</p>
      <p>将日期转换为长字符串格式</p>
    </td>
    <td>
      <p>{% raw %}{{ site.time | data_to_long_string }}{% endraw %}</p>
      <p>17 November 2008</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>XML Escape</p>
      <p>将一些文本转义，用在XML中</p>
    </td>
    <td>
      <p>{% raw %}{{ page.content | xml_escape }}{% endraw %}</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>CGI Escape</p>
      <p>将字符串进行CGI转义，用在URL中。将每个特殊字符用对应的%XX来替换。</p>
    </td>
    <td>
      <p>{% raw %}{{ "foo,bar;baz?" | cgi_escape }}{% endraw %}</p>
      <p>foo%2Cbar%3Bbaz%3F</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>URI Escape</p>
      <p>将字符串进行URI转义</p>
    </td>
    <td>
      <p>{% raw %}{{ “'foo, bar \\baz?'” | uri_escape }}{% endraw %}</p>
      <p>foo,%20bar%20%5Cbaz?</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>Number of Words</p>
      <p>计算文本中包含的单词个数</p>
    </td>
    <td>
      <p>{% raw %}{{ page.content | number_of_words }}{% endraw %}</p>
      <p>1337</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>Array to Sentence</p>
      <p>将数组转换为句子，即将数组中的多个元素组合成一个字符串。在显示列出所有标签时比较有用。</p>
    </td>
    <td>
      <p>{% raw %}{{ page.tags | array_to_sentence_string }}{% endraw %}</p>
      <p>foo, bar, and baz</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>Textilize</p>
      <p>通过RedCloth将Textile格式的字符串转换为HTML</p>
    </td>
    <td>
      <p>{% raw %}{{ page.excerpt | textilize }}{% endraw %}</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>Markdownify</p>
      <p>将Markdown格式的字符串转换为HTML</p>
    </td>
    <td>
      <p>{% raw %}{{ page.excerpt | markdownify }}{% endraw %}</p>
    </td>
  </tr>
</table>

***
#标签(Tags)

###包含(Includes)

如果你有一些页面布局希望能在多个地方复用，可以使用*include*标签。

    {% raw %}{% include footer.html %}{% endraw %}

Jekyll默认将所有include文件放到源码根目录下的子目录*\_includes*中。上面的语句会将*&lt;source&gt;/_includes/footer.html*文件中的内容嵌入到调用这个语句的文件中。

还可以给一个include传入参数：

    {% raw %}{% include footer.html param="value" %}{% endraw %}

这些参数可以在include中通过Liquid来调用：

    {% raw %}{{ include.param }}{% endraw %}

###代码片段高亮化

Jekyll有内置的[Pygments](http://pygments.org/)语法高亮支持，可以支持[超过100种语言](http://pygments.org/languages/)的语法高亮。使用Pygments需要安装Python并在配置文件中设置*pygments*为*true*。

若要使代码片段语法高亮，使用以下语句将代码片段包裹起来：

    {% raw %}{% highlight ruby %}{% endraw %}
    def foo
      puts 'foo'
    end
    {% raw %}{% endhighlight %}{% endraw %}

*highlight*标签的参数(例如上例中的*ruby*)是语言的标识。在[Lexers page](http://pygments.org/docs/lexers/)中可以找到需要高亮显示的编程语言对应的"短名称"。

#####行号

*highlight*标签还有第二个参数：*linenos*，这是个可选项。使用参数*linenos*可以强制高亮的代码段前面显示行号。例如，下面的代码段每一行前面都会包含行号：

    {% raw %}{% highlight ruby linenos %}{% endraw %}
    def foo
      puts 'foo'
    end
    {% raw %}{% endhighlight %}{% endraw %}

#####语法高亮样式单(Stylesheets for syntax highlighting)

语法的高亮显示需要包含包含高亮样式单。这里有个样式单的例子：[syntax.css](http://github.com/mojombo/tpw/tree/master/css/syntax.css)。还有很多Github使用的样式单，你可以在你的网站中自由使用。如果使用了行号参数*linenos*，你可能需要在*syntax.css*中包含一个额外的CSS类的声明：*.lineno*，这样可以将行号和高亮代码区分开来。

###文章链接地址(Post URL)

如果你希望包含你的网站中的某个post的链接，可以使用*post_url*标签，它会指定post的正确的永久链接。

    {% raw %}{% post_url 2010-07-21-name-of-post %}{% endraw %}

使用*post_url*标签时，不需要加上文件的后缀名。

你也可以在Markdown中使用这个标签来创建某个post的链接：

    {% raw %}[Name of Link]({% post_url 2010-07-21-name-of-post %}){% endraw %}

###Gist

使用*gist*标签可以轻易的将一个GitHub Gist嵌入到你的网站中：

    {% raw %}{% gist 5555251 %}{% endraw %}

你也可以在gist中指定显示的文件名，这是可选的：

    {% raw %}{% gist 5555251 result.md %}{% endraw %}

*gist*标签也可以使用私有的gists，需要有gist的拥有者的github用户名：

    {% raw %}{% gist parkr/931c1c8d465a04042403 %}{% endraw %}

私有gist语法也支持指定文件名。

***
#PS:
文章翻译自jekyll官方文档(2013-10-09)：

[Templates](http://jekyllrb.com/docs/templates/)