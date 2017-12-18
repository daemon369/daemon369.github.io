---
layout: post
keywords: ssh, ssh keys， ssh-keygen, ssh-add
description: generating ssh keys
title: "创建 SSH 密钥对"
categories: [SSH]
tags: [ssh, ssh keys， ssh-keygen, ssh-add]
group: SSH
icon: file-alt
date: 2015-03-08 00:00:00
---

SSH 密钥对可以让用户无需输入密码即可登录到 SSH 服务器中。由于登录的过程不需要密码，因此可以防止由于密码被拦截、破解造成的账户密码泄露。再加上密码短语(passphrase)的使用，使得 SSH 的安全性更高一层。

SSH 密钥对总是一把公钥、一把私钥的成对出现；公钥可以自由的添加到远程 SSH 服务器中用来验证用户是否合法；私钥相当于自己的身份认证，需要妥善保存不能泄露。

SSH 密钥的其使用原理很简单：用户将公钥添加到远程主机中，登录的时候，远程主机会向用户发送一段随即字符串，用户使用自己的私钥加密后，再发送到远程主机。远程主机使用本地存储的公钥进行解密，如果成功，证明用户时可信的，直接允许登录 shell ，不再要求密码。这样就保证了整个登录过程的安全，防止了中间人攻击。

<!--excerpt-->

#生成密钥对

## ssh-keygen 命令

我们可以使用 *ssh-keygen* 命令来生成密钥对：

    $ ssh-keygen -t ecdsa -b 521 -C "$(whoami)@$(hostname)-$(date -I)"
    Generating public/private ecdsa key pair.
    Enter file in which to save the key (/home/username/.ssh/id_ecdsa):
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved in /home/username/.ssh/id_ecdsa.
    Your public key has been saved in /home/username/.ssh/id_ecdsa.pub.
    The key fingerprint is:
    dd:15:ee:24:20:14:11:01:b8:72:a2:0f:99:4c:79:7f username@localhost-2015-03-08
    The key's randomart image is:
    +--[ECDSA  521]---+
    |     ..oB=.   .  |
    |    .    . . . . |
    |  .  .      . +  |
    | oo.o    . . =   |
    |o+.+.   S . . .  |
    |=.   . E         |
    | o    .          |
    |  .              |
    |                 |
    +-----------------+

其中可使用 *-t* 指定加密算法，使用 *-b* 自定生成密钥长度，使用 *-C* 添加密钥对的说明comment。生成的密钥对默认存储在用户目录下的 *.ssh* 目录中，私钥默认名称为 *id\_\*\*\** (即 id_ + 加密算法名称)。还可以使用 *-f* 指定生成的私钥存储的文件全路径名称；也可以不使用 *-f* 指定密钥文件路径，在密钥的创建过程中还会提示用户输入密钥文件全路径名称。私钥对应的公钥文件为*私钥文件全名称 + .pub*。

上面例子中创建了一对长度为512位的椭圆加密算法(ECDSA)加密的密钥对。创建 SSH 密钥对可选择多种加密算法，例如 *RSA* 、 *DSA* 、 *ECDSA* 等。

###密码短语(Passphras)

密码短语(passphras)是一连串的单词或文本组成，用来控制对电脑系统的访问。它的用法类似于密码(Password)，但是通常会比密码长度更长，这样就增加了破解的复杂度。密码短语不同于密码，它可以是有实际意义的一段话，便于用户记忆。

密码短语默认可以不创建，但是这会导致不安全性。私钥是未经加密存储在电脑上的，电脑遗失或被窃取后，任何人拿到你的私钥后都可以随意访问 SSH 服务器；另外，电脑的 *root* 用户有权限访问电脑上的任意文件，这也包括你的私钥文件。因此，为了提高安全性还是建议用户设置自己的密码短语。

已经生成的密钥对也可以修改密码短语。假设使用的是 RSA 加密的密钥对，存储到默认路径，输入以下命令即可：

    # ssh-keygen -f ~/.ssh/id_rsa -p

#SSH agent

SSH agent 是 OpenSSH 或其它 SSH 程序提供的一个程序，提供了存储私钥的安全方法。如果用户的私钥使用了密码短语来加密的话，那么每一次使用 SSH密钥进行登录时，都需要用户输入正确的的密钥短语。而 SSH agent 程序能够将已经解密的私钥缓存起来，在需要的时候提供给 SSH 客户端，这样用户只需要在将私钥加入 SSH agent 缓存的时候输入一次密码短语就可以了。

首先确保当前 SSH agent 可用：

    # start the ssh-agent in the background
    $ eval "$(ssh-agent -s)"
    Agent pid 29393

## ssh-add

添加 SSH 密钥到 SSH agent：

    $ ssh-add ~/.ssh/id_rsa
    Enter passphrase for /home/username/.ssh/id_rsa:
    Identity added: /home/username/.ssh/id_rsa (/home/username/.ssh/id_rsa)

##查看 SSH agent 缓存密钥列表：

    $ ssh-add -l
    2048 b9:a7:f0:44:a5:47:79:a5:ff:9d:14:5c:d3:78:04:65 /home/username/.ssh/id_rsa (RSA)

##测试连接

将 SSH 公钥添加到 SSH 服务端后，就可以使用 SSH 来连接远程主机了。下面以 GitHub为例测试连接：

    $ ssh -T git@github.com
    Hi username! You've successfully authenticated, but GitHub does not provide shell access.

这说明连接成功了。

#参考：

[Generating SSH keys](https://help.github.com/articles/generating-ssh-keys/ "generating ssh keys")

[Passphrase(维基百科)](http://en.wikipedia.org/wiki/Passphrase Passphrase)

[SSH Keys(简体中文)]("https://wiki.archlinux.org/index.php/SSH_Keys_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)" "SSH Keys(简体中文)" )

[ssh-agent](http://en.wikipedia.org/wiki/Ssh-agent ssh-agent)

[Git多帐号配置](http://yeungeek.com/2014/06/26/Git%E5%A4%9A%E5%B8%90%E5%8F%B7%E9%85%8D%E7%BD%AE/ Git多帐号配置)
