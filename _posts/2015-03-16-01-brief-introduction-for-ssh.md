---
layout: post
keywords: ssh
description: brief introduction for ssh
title: "SSH 简介"
categories: [SSH]
tags: [ssh]
group: SSH
icon: file-alt
---
{% include JB/setup %}

SSH(即 Secure Shell)，是一项创建在应用层和传输层基础上的安全协议，为计算机 Shell 提供安全的传输和使用环境。

传统的网络服务程序，如FTP、POP、Telnet等本质上并不安全；因为它们在网络上用明文传送数据、用户帐号和用户口令，很容易受到中间人（man-in-the-middle）攻击方式的攻击。就是存在另一个人或者一台机器冒充真正的服务器接收用户传给服务器的数据，然后再冒充用户把数据传给真正的服务器。

而SSH是目前较可靠，专为远程登录会话和其他网络服务提供安全性的协议。利用SSH协议可以有效防止远程管理过程中的信息泄露问题。通过SSH可以对所有传输的数据进行加密，也能够防止DNS欺骗和IP欺骗。

SSH之另一项优点为其传输的数据可以是经过压缩的，所以可以加快传输的速度。SSH有很多功能，它既可以代替Telnet，又可以为FTP、POP、甚至为PPP提供一个安全的“通道”。

<!--excerpt-->

# 安装 SSH

最初的 SSH 协议由芬兰一家公司的研究员Tatu Ylönen于1995年设计开发，但是由于版权和加密算法的等等的限制，很多人转而使用开源的自由软件 OpenSSH。

客户端安装 openssh-client 用以登录远程主机：

    $ sudo apt-get install openssh-client

服务端安装 openssh-server 用以提供客户端登录：

    $ sudo apt-get install openssh-server

# 安全认证

SSH 提供了两种级别的安全认证，基于密码的安全认证和基于密钥的安全认证：

## 基于密码的安全认证

基于密码的安全认证，登录的时候需要提供账号和密码；远程主机将自己的公钥分发给登录客户端，客户端访问主机使用该公钥加密；远程主机使用自己的私钥解密数据。

登录的流程如下：

1. 远程主机收到用户登录请求，将自己的公钥发给用户
2. 用户通过远程主机公钥的指纹确认主机的真实性，然后使用远程主机公钥将登录密码加密后，发送回远程主机
3. 远程主机使用自己的私钥解码登录密码，验证密码正确后，允许用户登录

### 用法

假设需要以用户名 user 登录远程主机 host：

    $ ssh user@host

如果本地用户名与远程用户名一致，可以省略用户名：

    $ ssh host

SSH 默认端口号22，可以使用 p 参数来指定端口号：

    $ ssh -p 2345 user@host

第一次登录到远程主机时，系统会出现如下提示：

    $ ssh user@host
    The authenticity of host 'host (***.***.***.***)' can't be established.
    RSA key fingerprint is 98:2e:d7:e0:de:9f:ac:67:28:c2:42:2d:37:16:58:4d.
    Are you sure you want to continue connecting (yes/no)?

这段话提示用户无法确认远程主机的真实性，只知道 RSA 公钥的指纹，询问用户是否继续。

我们使用 ssh-keygen 工具可以生成 SSH 密钥对，其中公钥的长度可以很长，对用户来说不方便直接对比验证，因此对其进行了 MD5 计算，生成了一个128的指纹，这样再进行比较就比较容易了。

那么这里就要求我们事先知道远程主机的公钥指纹，才可以确认主机的真实性。

用户确认主机的真实性，输入 yes 继续连接：

    Warning: Permanently added 'host,***.***.***.***' (RSA) to the list of known hosts.

然后输入密码：

    Password: (enter password)

密码正确，即可登录成功。

当第一次登录成功后，远程主机的公钥会被保存到文件 $HOME/.ssh/known_hosts 中，下次再连接这台主机就会跳过警告，直接提示输入密码。

每个SSH用户都有自己的known_hosts文件，此外系统也有一个这样的文件，通常是 /etc/ssh/ssh_known_hosts ，保存一些对所有用户都可信赖的远程主机的公钥。

### 中间人攻击

基于密码的安全认证无法避免中间人攻击：

网络提供者(ISP、公共 wifi 提供者等，或其它形式拦截者)，拦截用户的登录请求，用自己的公钥伪造远程主机的公钥发送给用户，然后获取用户加密后的密码，用自己的私钥解密已获取用户密码，这样用户的账号密码就被盗取了。

## 基于密钥的安全认证

基于密钥的安全认证，客户端将将公钥上传到服务器。登录的时候，客户端向服务器发送登录请求；服务器收到请求后，向用户发送一段随机字符串；用户用自己的私钥加密后，再发送回服务器；服务器使用事先存储的公钥进行解密，如果解密成功，证明用户可信，允许登录。

这种方式，在登录服务器的过程中，不需要上传密码，增加了安全性。

密钥的生成可参看[创建 SSH 密钥对][1]。

#  authorized_keys 文件

我们上传公钥到服务端，即将公钥内容附加到服务器用户目录下的 *$HOME/.ssh/authorized_keys* 文件中：

服务端首先需要安装 openssh-server 程序用以提供 ssh 登录服务，在服务器(Ubuntu 14.04 LTS)上查看服务是否打开：

    $ service ssh status
    ssh start/running, process 1201

检查 ssh 服务配置项

    RSAAuthentication yes
    PubkeyAuthentication yes
    AuthorizedKeysFile .ssh/authorized_keys

是否开启：

    $ cat /etc/ssh/sshd_config | grep RSAAuthentication
    RSAAuthentication yes
    $ cat /etc/ssh/sshd_config | grep PubkeyAuthentication
    PubkeyAuthentication yes
    $ cat /etc/ssh/sshd_config | grep AuthorizedKeysFile
    AuthorizedKeysFile %h/.ssh/authorized_keys

上传公钥：

    $ ssh-copy-id user@host

重启远程主机 ssh 服务：

    $ ssh user@host 'service ssh restart'
    # ubuntu

    $ ssh user@host '/etc/init.d/ssh restart'
    # debian

也可以使用更复杂的命令：

    $ ssh user@host 'mkdir -p .ssh && cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub

这个命令可以清晰的看到公钥的上传过程：

1. 在远程主机用户目录下创建目录：~/.ssh
2. 将本地主机文件 ~/.ssh/id_rsa.pub 拷贝到远程主机的文件 ~/.ssh/authorized_keys ，追加到文件末尾

然后重启服务即可

# 参考

[Secure Shell](http://zh.wikipedia.org/wiki/Secure_Shell "Secure Shell")

[SSH原理与运用（一）：远程登录](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html "SSH原理与运用（一）：远程登录")

[1]: {% post_url 2015-03-08-generating-ssh-keys %} "创建 SSH 密钥对"
