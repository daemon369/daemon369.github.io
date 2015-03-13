---
layout: post
keywords: ubuntu,chinese folder name
description: change chinese folder name for ubuntu
title: "修改 Ubuntu 用户根目录下中文文件夹名称"
categories: [Ubuntu]
tags: [ubuntu, chinese folder name]
group: Ubuntu
icon: file-alt
---
{% include JB/setup %}

安装中文版 Ubuntu 后，我们可以看到在我们的用户目录下，有一些中文名称的文件夹：

    $ ls ~/
    examples.desktop  公共的  模板  视频  图片  文档  下载  音乐  桌面

这些中文命名的文件夹国人看起来会比较顺眼一些；但是对于习惯用命令行的用户，每次输入目录名称，切换中英文也是件麻烦的事情。我习惯的会把这些文件夹重新修改为英文目录。

这些目录可以在如下配置文件找到：

<!--excerpt-->

    $ cat ~/.config/user-dirs.dirs
    # This file is written by xdg-user-dirs-update
    # If you want to change or add directories, just edit the line you're
    # interested in. All local changes will be retained on the next run
    # Format is XDG_xxx_DIR="$HOME/yyy", where yyy is a shell-escaped
    # homedir-relative path, or XDG_xxx_DIR="/yyy", where /yyy is an
    # absolute path. No other format is supported.
    #
    XDG_DESKTOP_DIR="$HOME/桌面"
    XDG_DOWNLOAD_DIR="$HOME/下载"
    XDG_TEMPLATES_DIR="$HOME/模板"
    XDG_PUBLICSHARE_DIR="$HOME/公共的"
    XDG_DOCUMENTS_DIR="$HOME/文档"
    XDG_MUSIC_DIR="$HOME/音乐"
    XDG_PICTURES_DIR="$HOME/图片"
    XDG_VIDEOS_DIR="$HOME/视频"

我们在这里将对应的中文目录修改为英文，同时将用户主目录下的对应文件夹名称也修改为英文：

    $ cat ~/.config/user-dirs.dirs
    XDG_DESKTOP_DIR="$HOME/Desktop"
    XDG_DOWNLOAD_DIR="$HOME/Download"
    XDG_TEMPLATES_DIR="$HOME/Template"
    XDG_PUBLICSHARE_DIR="$HOME/Public"
    XDG_DOCUMENTS_DIR="$HOME/Document"
    XDG_MUSIC_DIR="$HOME/Music"
    XDG_PICTURES_DIR="$HOME/Picture"
    XDG_VIDEOS_DIR="$HOME/Video"

最新版的 Ubuntu 系统，在文件浏览器中直接按 *F2* 按键修改这几个特殊目录的名称，配置文件 *~/config/user-dirs.dirs* 中的对应目录也会被自动修改；但是使用 *mv* 命令修改目录名称，却不会影响配置文件。
