---
layout: post
keywords: blog, jekyll
description: jekyll blog migrations
title: "Jekyll文档(八) -- Blog migrations"
categories: [Blog]
tags: [Blog, jekyll]
group: Blog
icon: file-alt
date: 2013-09-25 00:00:00
---

如果你想从其他博客系统转换到Jekyll，Jekyll的导入功能能够对你有所帮助。下面列出的一些方法都需要你对你的旧的博客系统的数据库有访问权限，这样Jekyll才能够从这些数据库中获取数据产生posts。每种方法都会根据外部系统入口产生*.markdown* posts存放到*\_posts*文件夹中。

<!--excerpt-->

***

# 博客迁移前的准备

因为所有的导入系统都有其单独的依赖群，因此我们需要一个特殊的叫做[jekyll-import](https://github.com/jekyll/jekyll-import)的gem来使得我们的导入系统能够正常工作。你只需要安装这个gem，这些导入系统就会作为Jekyll标准命令行接口的一部分来提供我们需要的功能。

    $ gem install jekyll-import --pre

现在，所有的导入系统都已经设置完善可以使用了。如果依然无法产生作用，可以参看每个导入系统的帮助信息：

    $ jekyll help import          # => See list of importers 查看导入系统列表
    $ jekyll help import IMPORTER # => See importer specific help 查看导入系统详细帮助信息

这里*IMPORTER*是特定导入系统的名称。

### 仔细核查导入的内容

Jekyll的导入系统可能无法分清需要发布的posts和私人的posts，因此你需要仔细的核查Jekyll导入的内容是否是你希望展示的效果。

***

# WordPress

### WordPress的导出文件

如果你没有安装hepricot，那么你需要运行*gem install hpricot*来安装。然后，你需要使用WordPress导出工具来导出你的博客。假定你已经导出了博客并保存到文件*wordpress.xml*，运行下面的命令：

    $ ruby -rubygems -e 'require "jekyll/jekyll-import/wordpressdotcom";
          JekyllImport::WordpressDotCom.process({ :source => "wordpress.xml" })'

### WordPress.com导出工具

如果你从WordPress.com账户中迁移，可以从以下地址访问导出工具：*https://YOUR-USER_NAME.worpress.com/wp-admin/export.php*。

### 直连WordPress的MySQL服务器

如果你希望能够直接连接WordPress的MySQL服务器来导入博客，方法如下：

    $ ruby -rubygems -e 'require "jekyll/jekyll-import/wordpress";
        JekyllImport::WordPress.process({:dbname => "database", :user => "user", :pass => "pass"})'

如果你在使用Webfaction并且需要设置[SSH tunnel](http://docs.webfaction.com/user-guide/databases.html?highlight=mysql#starting-an-ssh-tunnel-with-ssh)，需要明确制定hostname*(127.0.0.1)*，否则MySQL可能阻塞你的访问，因为在其认证系统上*localhost*和*127.0.0.1*并不等同：

    $ ruby -rubygems -e 'require "jekyll/jekyll-import/wordpress";
        JekyllImport::WordPress.process({:host => "127.0.0.1", :dbname => "database", :user => "user", :pass => "pass"})'

###更深入的WordPress博客迁移工具

上面提供几种方法无法很好的导入通常存储在posts和pages中的metadata元数据。如果你需要导入pages，标签，自定义域，图片附件等，下面的资源可能对你有用：

* [Exitwp](https://github.com/thomasf/exitwp) 是一个使用Python
编写的可配置的工具，可以用来迁移WordPress博客到Jekyll(Markdown)的格式，并能够保持其metadata元数据的可用性。Exitwp还能够下载附件和pages页面.
* 文章[Migrating from WordPress to Jekyll](http://vitobotta.com/how-to-migrate-from-wordpress-to-jekyll/) 提供了从WordPress博客到Jekyll迁移的一步步的引导步骤，能够保持源博客的很多的结构和metadata元数据。
* [wpXml2Jekyll](https://github.com/theaob/wpXml2Jekyll)是一个运行在windows下的可执行程序，可以从你的WordPress的xml文件中创建Markdown的posts。

***

# Drupal

如果你是从[Drupal](http://drupal.org/)迁移博客，根据你的Drupal的版本的不同，下面有两个工具可供使用：

* [Drupal 6](https://github.com/jekyll/jekyll-import/blob/v0.1.0.beta1/lib/jekyll/jekyll-import/drupal6.rb)
* [Drupal 7](https://github.com/jekyll/jekyll-import/blob/v0.1.0.beta1/lib/jekyll/jekyll-import/drupal7.rb)

命令如下：

    $ ruby -rubygems -e 'require "jekyll/jekyll-import/drupal6";
        JekyllImport::Drupal6.process("dbname", "user", "pass")'
    # ... or ...
    $ ruby -rubygems -e 'require "jekyll/jekyll-import/drupal7";
        JekyllImport::Drupal7.process("dbname", "user", "pass")'

如果你连接到其他的主机(host)或者需要给你的数据库指定一个表格前缀(table prefix)，下面两个可选的参数可以添加到Drupal迁移工具执行的末尾

    $ ruby -rubygems -e 'require "jekyll/jekyll-import/drupal6";
        JekyllImport::Drupal6.process("dbname", "user", "pass", "host", "table_prefix")'
    # ... or ...
    $ ruby -rubygems -e 'require "jekyll/jekyll-import/drupal7";
        JekyllImport::Drupal7.process("dbname", "user", "pass", "host", "table_prefix")'

***

# Movable Type

从Movable Type中导入posts：

    $ ruby -rubygems -e 'require "jekyll/jekyll-import/mt";
        JekyllImport::MT.process("database", "user", "pass")'

***

# Typo

从Typo中导入posts：

    $ ruby -rubygems -e 'require "jekyll/jekyll-import/typo";
        JekyllImport::Typo.process("database", "user", "pass")'

以上代码只在Typo版本4+中测试过

***

# TextPattern

从TextPattern中导入posts：

    $ ruby -rubygems -e 'require "jekyll/jekyll-import/textpattern";
        JekyllImport::TextPattern.process("database_name", "username", "password", "hostname")'

你需要在你的*\_import*目录的父目录中运行以上命令。例如你的*\_import*目录在*/path/source/_import*，你需要在*/path/source*中运行上述命令。主机名(hostname)默认为*localhost*，其他变量都是必须的。你可以需要调整上述命令来过滤某些入口(extries)。否则，会试图获取所有的live或者sticky的入口(entries)。

***

# Mephisto

从Mephisto中导入posts：

    $ ruby -rubygems -e 'require "jekyll/jekyll-import/mephisto";
        JekyllImport::Mephisto.process("database", "user", "password")'

如果你的数据存储在Postgres数据库中，则运行下面的命令：

    $ ruby -rubygems -e 'require "jekyll/jekyll-import/mephisto";
        JekyllImport::Mephisto.postgres({:database => "database", :username=>"username", :password =>"password"})'

***

# Blogger(Blogspot)

导入Blooger中的posts，可以参看文章[Migrate from blogger to jekyll](http://blog.coolaj86.com/articles/migrate-from-blogger-to-jekyll.html)。如果导入不成功，你可以尝试一下方法：

* [@kennym](https://github.com/kennym)按照上面给出的文章操作不成功，因此他写了一个[迁移脚本](https://gist.github.com/1115810)。
* [@ngauthier](https://github.com/ngauthier)创建了另外的[importer](https://gist.github.com/1506614)，可以通过blogger的归档文件而不是RSS feed来导入评论。
* [@juniorz](https://github.com/juniorz)创建了另外的[importer](https://gist.github.com/1564581)，对[Octopress](http://octopress.org/)有效。这个比较类似[@ngauthier的版本](https://gist.github.com/1506614)，不同的是将草稿和posts文章分离开来，并且导入tags标签和静态链接。

***

# Posterous

导入Posterous博客：

    $ ruby -rubygems -e 'require "jekyll/jekyll-import/posterous";
        JekyllImport::Posterous.process("my_email", "my_pass")'

对于你账户上的其他Posterous博客，你需要指定博客的*blog_id*：

    $ ruby -rubygems -e 'require "jekyll/jekyll-import/posterous";
        JekyllImport::Posterous.process("my_email", "my_pass", "blog_id")'

还有一个[可选的迁移工具](https://github.com/pepijndevos/jekyll/blob/patch-1/lib/jekyll/migrators/posterous.rb)可以保持静态链接并试图导入图片。

***

# Tumblr

导入Tumblr的posts：

    $ ruby -rubygems -e 'require "jekyll/jekyll-import/tumblr";
        JekyllImport::Tumblr.process(url, format, grab_images, add_highlights, rewrite_urls)'
    # url    - String: your blog's URL
    # format - String: the output file extension. Use "md" to have your content
    #          converted from HTML to Markdown. Defaults to "html".
    # grab_images    - Boolean: whether to download images as well. Defaults to false.
    # add_highlights - Boolean: whether to wrap code blocks (indented 4 spaces) in a Liquid
    #                  "highlight" tag. Defaults to false.
    # rewrite_urls   - Boolean: whether to write pages that redirect from the old Tumblr paths
    #                  to the new Jekyll paths. Defaults to false.

***

# 其他系统

如果你在使用一个目前不存在迁移器的博客系统，可以考虑自己写一个并发送一个推送请求给[Jekyll](https://github.com/jekyll/jekyll-import)。

***

# PS:
文章翻译自jekyll官方文档(2013-09-25)：

[Blog migrations](http://jekyllrb.com/docs/migrations/)
