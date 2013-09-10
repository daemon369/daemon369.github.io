---
layout: post
keywords: blog, jekyll
description: jekyll configuration
title: "jekyll文档(三) -- Configuration"
categories: [Blog]
tags: [Blog, jekyll]
group: Blog
icon: file-alt
---
{% include JB/setup %}

***
#jekyll配置

jekyll有两种配置参数的方法：

1. 通过命令行输入

2. 通过编辑配置文件_config.yml

***
#全局配置项

下面列出jekyll的配置项，前面加上-或者--的就是我们在执行jekyll命令时输入的参数；没有的自然就是用在配置文件中的配置项了。

<table cellpadding="10">
  <col width="75%" />
  <col width="25%" />
  <tr>
    <th>功能</th>     
    <th>配置项</th>
  </tr>
  <tr>
    <td>
      <p>Site Source</p>
      <p>jekyll可以从设置的源目录读取文件</p>
    </td>
    <td>
      <p>source: DIR</p>
      <p>-s, --source DIR</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>Site Destination</p>
      <p>jekyll将生成的静态网页放置到设定的目标目录</p>
    </td>
    <td>
      <p>destination: DIR</p>
      <p>-d, --destination DIR</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>Safe</p>
      <p>为了安全考虑可以禁止第三方的插件</p>
    </td>
    <td>
      <p>safe: BOOL</p>
      <p>--safe</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>Exclude</p>
      <p>不转换排除列表中的目录/文件</p>
    </td>
    <td>
      <p>exclude: [DIR, FILE, ...]</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>Include</p>
      <p>强制转换包含列表中的目录/文件，例如.htaccess文件，默认.开头的文件是被排除不进行转换的，可以强制包含进行转换</p>
    </td>
    <td>
      <p>include: [DIR, FILE, ...]</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>Time Zone</p>
      <p>生成网站时设置时区。设置此项会设置TZ环境变量，Ruby可以使用这个环境变量进行时间和日期的各种操作。设置值可以是IANA时区数据库(IANA Time Zone Database)中的可用值，例如：America/New_York。默认会设置成操作系统中的本地时区</p>
    </td>
    <td>
      <p>timezone: TIMEZONE</p>
    </td>
  </tr>
</table>
***
#Build命令选项

即命令 *jekyll build* 后面可以使用的选项

<table cellpadding="10">
  <col width="75%" />
  <col width="25%" />
  <tr>
    <th>功能</th>     
    <th>配置项</th>
  </tr>
  <tr>
    <td>
      <p>Regeneration</p>
      <p>当文件被修改过后，自动重新生成新的网站</p>
    </td>
    <td>
      <p>-w, --watch</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>Configuration</p>
      <p>jekyll默认配置文件是_config.yml文件，也可以自己指定其他文件作为配置文件。可指定多个文件，相同的配置项后面的文件覆盖前面的文件。</p>
    </td>
    <td>
      <p>--config FILE1[, FILE2, ...]</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>Drafts</p>
      <p>Process and render draft posts.</p>
    </td>
    <td>
      <p>--drafts</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>Future</p>
      <p>可以设置一个将来的时间点，文章会延迟到这个时间点发布。</p>
    </td>
    <td>
      <p>future: BOOL</p>
      <p>--future</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>LSI</p>
      <p>给有关联的文章产生序号</p>
    </td>
    <td>
      <p>lsi: BOOL</p>
      <p>--lsi</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>Limit Posts</p>
      <p>限制解析或发布的文章的数量</p>
    </td>
    <td>
      <p>limit_posts: NUM</p>
      <p>--limit_posts NUM</p>
    </td>
  </tr>
</table>
***
#Serve命令选项

下面给出 *jekyll serve* 命令的选项列表。另外，*jekyll serve* 适用上面列出的所有 *jekyll build* 命令的参数选项。

<table cellpadding="10">
  <col width="75%" />
  <col width="25%" />
  <tr>
    <th>功能</th>     
    <th>配置项</th>
  </tr>
  <tr>
    <td>
      <p>Local Server Port</p>
      <p>监听指定端口</p>
    </td>
    <td>
      <p>port: PORT</p>
      <p>--port PORT</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>Local Server Hostname</p>
      <p>监听指定主机名</p>
    </td>
    <td>
      <p>host: HOSTNAME</p>
      <p>--host HOSTNAME</p>
    </td>
  </tr>
  <tr>
    <td>
      <p>Base URL</p>
      <p>使用指定的基本URL来启动网站服务</p>
    </td>
    <td>
      <p>baseurl: URL</p>
      <p>--baseurl URL</p>
    </td>
  </tr>
</table>

***
###!!注意：配置文件里不要使用TAB制表符

使用TAB会导致错误，或者导致jekyll会使用默认的设置。一般使用空格来代替TAB

***
#默认设置

jekyll有很多默认的设置项。除非用户在命令行或者配置文件中显示的改变了这些选项，否则jekyll都是按照默认设置项来提供服务。

###!!有两个kramdown选项目前还不支持
这两个选项是：*remove_block_html_tags* 和 *remove_span_html_tags* 。因为kramdown HTML转换器目前还不包含这两个选项。

    source:      .
    destination: ./_site
    plugins:     ./_plugins
    layouts:     ./_layouts
    include:     ['.htaccess']
    exclude:     []
    keep_files:  ['.git','.svn']
    timezone:    nil

    future:      true
    show_drafts: nil
    limit_posts: 0
    pygments:    true

    relative_permalinks: true

    permalink:     date
    paginate_path: 'page:num'

    markdown:      maruku
    markdown_ext:  markdown,mkd,mkdn,md
    textile_ext:   textile

    excerpt_separator: "\n\n"

    safe:        false
    watch:       false    # deprecated
    server:      false    # deprecated
    host:        0.0.0.0
    port:        4000
    baseurl:     /
    url:         http://localhost:4000
    lsi:         false

    maruku:
      use_tex:    false
      use_divs:   false
      png_engine: blahtex
      png_dir:    images/latex
      png_url:    /images/latex

    rdiscount:
      extensions: []

    redcarpet:
      extensions: []

    kramdown:
      auto_ids: true
      footnote_nr: 1
      entity_output: as_char
      toc_levels: 1..6
      smart_quotes: lsquo,rsquo,ldquo,rdquo
      use_coderay: false

     coderay:
        coderay_wrap: div
        coderay_line_numbers: inline
        coderay_line_numbers_start: 1
        coderay_tab_width: 4
        coderay_bold_every: 10
        coderay_css: style

    redcloth:
      hard_breaks: true

***
#PS:
文章翻译自jekyll官方文档(2013-09-10)：

[Configuration](http://jekyllrb.com/docs/configuration/)