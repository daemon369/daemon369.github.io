---
layout: post
keywords: blog, jekyll
description: jekyll using excerpt in post
title: "Jekyll -- 使用摘要"
categories: [Blog]
tags: [Blog, jekyll]
group: Blog
icon: file-alt
date: 2014-01-16 00:00:00
---

摘要是博客的一个重要部分，有助于使读者对博文有个简要的直观认识。我看到很多使用Jekyll的博客，有的是自己定义变量并使用代码来提取摘要，也有使用其他方法的。这可能是早期的Jekyll没有提供摘要的缘故，现在这些已经是不必要的了。Jekyll从1.0版本开始，提供了两种使用摘要的方法：

<ol>
  <li>在_config.yml文件中设置摘要分割符</li>
  <li>在YAML Front-Matter中设置excerpt变量</li>
</ol>

<!--excerpt-->

***
#设置摘要分割符

在配置文件_config.yml文件中配置摘要分割符*excerpt_separator*，例如:

    excerpt_separator: <!--excerpt-->  #这里可以定义自己的摘要分割符

Jekyll会自动提取post内容从开始到摘要分割符(也就是设置的"&lt;!--excerpt--&gt;")第一次出现的地方之间的内容，将这部分内容设置为变量*post.excerpt*，这样你就可以在index索引页面中通过Liquid标签来使用摘要：

    <ul>
      {% raw %}{% for post in site.posts %}{% endraw %}
        <li>
          <a href="{% raw %}{{ post.url }}{% endraw %}">{% raw %}{{ post.title }}{% endraw %}</a>
          <p>{% raw %}{{ post.excerpt }}{% endraw %}</p>
        </li>
      {% raw %}{% endfor %}{% endraw %}
    </ul>

***
#excerpt变量

摘要也可以在每篇post的YAML Front-Matter中设置excerpt变量：

    ---
    layout: post
    title: A developers toolkit
    date: Friday 14 December, 2012
    excerpt: What text editor to use? Sass or plain old CSS? What on earth is Compass? Command line? I'm not touching that. Sound like you? Welcome, I was once like you and this is the guide I wish someone had given me.
    ---

这个变量会覆盖第一种方法提供的摘要，在index索引页面中使用的方法如上例。

***
#参考:

[Writing posts - Post excerpts](http://jekyllrb.com/docs/posts/#post_excerpts)

[How do I use markdownify in Jekyll to show an excerpt on the index](http://stackoverflow.com/questions/16422933/how-do-i-use-markdownify-in-jekyll-to-show-an-excerpt-on-the-index)