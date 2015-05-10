---
layout: post
keywords: maven, nexus
description: build private maven repositories using nexus
title: "使用Nexus搭建maven仓库"
categories: [Maven]
tags: [maven, nexus]
group: Maven
icon: file-alt
---
{% include JB/setup %}

搭建个人 maven 仓库有以下好处：

1. 直接从 Maven 中央仓库下载构件，由于国内坑爹网络的问题，速度会很慢；而通过搭建个人仓库来代理中央仓库，则可以将构件缓存到自己的服务器上，一次下载，重复使用，多人使用更能提高速度；对于某些要求员工不能直连外网的公司，也可以使用一台可连接外网的服务器提供远程仓库的代理，员工的电脑只需要连接到这台服务器即可
2. 我们可以将个人或公司项目打包为 maven 构件上传到个人仓库上，供公司或团队内部使用，即保证了代码安全，也享受了 maven 依赖管理的便利

下面是我的 maven 仓库的搭建过程，使用的是 nexus：

<!--excerpt-->

# 一. 下载 nexus oss

首先去 sonatype 官网下载 [nexus oss](http://www.sonatype.org/nexus/go/ "nexus oss")。当前最新版本为：2.11.2-06。

![](/image/2015-05-10/nexus01.png)

下载后的压缩包解压到本地：~/nexus-2.11.2-06-bundle

# 二. 运行 nexus

    # 进入可执行程序目录
    $ cd ~/nexus-2.11.2-06-bundle/nexus-2.11.2-06/bin

    # 启动nexus
    $ ./nexus start
    Starting Nexus OSS...
    Started Nexus OSS.

    # 查看nexus运行状态
    $ ./nexus status
    Nexus OSS is running (5532).

    # 结束运行
    $ ./nexus stop
    Stopping Nexus OSS...
    Stopped Nexus OSS.

## 注意

这里我遇到了个问题，由于我本地默认java环境使用的是 java6，导致 nexus 无法正常启动，切换到 java7 就好了

# 三. nexus管理

1.启动 nexus oss 后，打开浏览器，进入：http://127.0.0.1:8081/nexus/

![](/image/2015-05-10/nexus02.png)

2.点击右上角"log in"按钮登录，初始管理员帐号admin，密码admin123

![](/image/2015-05-10/nexus03.png)

3.登录成功后，点击右上角帐号名称->profile，可进入资料编辑页面

![](/image/2015-05-10/nexus04.png)

在资料编辑页面可修改用户姓名、邮箱、修改密码等

4.也可以在左侧打开 Security -> Users，在右侧栏选择你的帐号，在右下侧栏里修改用户信息

![](/image/2015-05-10/nexus05.png)

# 四.仓库类型

maven 仓库有以下几种常用类型：

* hosted，本地仓库，通常用来部署自己的构件到这一类型的仓库。
* proxy，代理仓库，它们被用来代理远程的公共仓库，如maven中央仓库。
* group，仓库组，用来合并多个hosted/proxy仓库，通常我们配置maven依赖仓库组。

# 五. 添加构件

以添加第三方构件到 3rd party 仓库为例讲下添加构件到 maven 仓库的流程：

##1.选择左侧栏 Views/Repositories -> Repositories

![](/image/2015-05-10/nexus06.png)

##2.在右侧打开仓库列表

##3.选择 3rd party

##4.打开右下方 Artifact Upload选项卡

##5.GAV Definition 选择 GAV Parameters

##6.填写构件相关信息，包括构件的组、构件ID、版本号、packaging等信息。

##7.点击Select Artifact(s) to Upload 按钮，打开需要上传的构件 jar 包文件

##8.选择jar包后，点击Add Artifact按钮将 jar 包添加到构件

![](/image/2015-05-10/nexus07.png)

多个构件可以使用相同的组、构件ID、版本号信息，它们通过指定不同的 Classifier 字段来区分；例如可添加可执行 jar、javadoc jar、source jar 等多个构件；每个构件的 classifier 都不能重复；可以存在一个不指定 classifier 的 jar包，这种特殊 jar 只能有一个。

##9.添加构件后，点击Upload Artifact(s)按钮上传构件到maven仓库

![](/image/2015-05-10/nexus08.png)

##10.打开Browse Index 选项卡，点击刷新按钮，查看已添加到maven仓库的构件的信息

![](/image/2015-05-10/nexus09.png)

# 六. 项目中使用个人仓库构件

搭建maven私有仓库后，可在在项目中添加对第三方jar包的依赖

## 1.pom.xml

{% highlight java linenos %}
<repositories>
    <repository>
        <id>nexus</id>
        <name>My Nexus 3rd Party Repository</name>
        <url>http://127.0.0.1:8081/nexus/content/repositories/thirdparty/</url>
    </repository>
</repositories>

<pluginRepositories>
    <pluginRepository>
        <id>nexus</id>
        <name>My Nexus 3rd Party Repository</name>
        <url>http://127.0.0.1:8081/nexus/content/repositories/thirdparty/</url>
    </pluginRepository>
</pluginRepositories>

<dependencies>
    <dependency>
        <groupId>com.maventest</groupId>
        <artifactId>testartifact</artifactId>
        <version>1.0</version>
    </dependency>
</dependencies>
{% endhighlight %}

## 2. gradle 

build.gradle文件中添加：

{% highlight java linenos %}
allprojects {
    repositories {
//        jcenter()
          maven{ url 'http://127.0.0.1:8081/nexus/content/repositories/thirdparty/' // 个人仓库
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.maventest:testartifact:1.0' // 有 classifier 属性时，添加到最后，例如：'com.maventest:testartifact:1.0:javadoc'
}
{% endhighlight %}

# 参考：

[Maven学习 (四) 使用Nexus搭建Maven私服](http://www.cnblogs.com/quanyongan/archive/2013/04/24/3037589.html "Maven学习 (四) 使用Nexus搭建Maven私服")

[Nexus入门指南（图文）](http://juvenshun.iteye.com/blog/349534 "Nexus入门指南（图文）")