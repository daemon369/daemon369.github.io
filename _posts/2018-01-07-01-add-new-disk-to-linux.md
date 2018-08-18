---
layout: post
keywords: Linux
description: add new disk to linux system
title: "Linux中添加硬盘"
categories: [Linux]
tags: [Linux, parted, mkfs, fstab]
group: Linux
icon: file-alt
date: 2018-01-07 19:59:00
---

下面研究下怎么在`Linux`系统中增加新的硬盘。

当前环境如下：

    Host    : macOs High Sierra(10.13.6)
    VM      : VMWare Fusion 10.1.3 (9472307)
    Guest   : Ubuntu 12.04.5LTS

查看已有硬盘：

    $ lsblk  -d
    NAME MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sda    8:0    0    60G  0 disk
    sr0   11:0    1  1024M  0 rom

<!--excerpt-->

当前只有一块硬盘`sda`

# 添加硬盘

在`VMWare Fusion`中为虚拟机`Ubuntu 12.04.5LTS`增加一块新的硬盘，然后重启操作系统。

再次查看已有硬盘：

    $ lsblk  -d
    NAME MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sda    8:0    0    60G  0 disk
    sdb    8:16   0    50G  0 disk
    sr0   11:0    1  1024M  0 rom

`sdb`就是新添加的硬盘。

# 创建分区

在新添加的硬盘上用`parted`工具创建一个分区：

    $ sudo parted -s /dev/sdb mklabel gpt mkpart primary ext4 0% 100%
    $ lsblk
    NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sda      8:0    0    60G  0 disk
    ├─sda1   8:1    0   487M  0 part /boot/efi
    ├─sda2   8:2    0  55.5G  0 part /
    └─sda3   8:3    0     4G  0 part [SWAP]
    sdb      8:16   0    50G  0 disk
    └─sdb1   8:17   0    50G  0 part
    sr0     11:0    1  1024M  0 rom

# 创建文件系统

在新创建的分区`/dev/sdb1`上创建文件系统`ext4`：

    $ sudo mkfs -t ext4 /dev/sdb1
    mke2fs 1.42 (29-Nov-2011)
    文件系统标签=
    OS type: Linux
    块大小=4096 (log=2)
    分块大小=4096 (log=2)
    Stride=0 blocks, Stripe width=0 blocks
    3276800 inodes, 13106688 blocks
    655334 blocks (5.00%) reserved for the super user
    第一个数据块=0
    Maximum filesystem blocks=4294967296
    400 block groups
    32768 blocks per group, 32768 fragments per group
    8192 inodes per group
    Superblock backups stored on blocks:
    	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
    	4096000, 7962624, 11239424

    Allocating group tables: 完成
    正在写入inode表: 完成
    Creating journal (32768 blocks): 完成
    Writing superblocks and filesystem accounting information: 完成

# 挂载硬盘

## 创建挂载点

首先在系统中新建目录作为挂载点：

    $ sudo mkdir /media/mynewdrive

## 手动挂载

    $ sudo mount /dev/sdb1 /media/mynewdrive
    $ df -h
    文件系统        容量  已用  可用 已用% 挂载点
    /dev/sda2        55G  3.8G   49G    8% /
    udev            2.0G  8.0K  2.0G    1% /dev
    tmpfs           394M  784K  394M    1% /run
    none            5.0M     0  5.0M    0% /run/lock
    none            2.0G  152K  2.0G    1% /run/shm
    /dev/sda1       487M  3.3M  483M    1% /boot/efi
    /dev/sdb1        50G   52M   47G    1% /media/mynewdrive

## 自动挂载

也可以配置让系统启动后自动挂载硬盘到挂载点。

首先获取硬盘的`UUID(universally unique identifier)`：

    $ sudo blkid
    /dev/sdb1: UUID="a9e94538-f436-4773-9d1b-5390c1b2e018" TYPE="ext4"
    /dev/sda1: UUID="AB45-3BA0" TYPE="vfat"
    /dev/sda2: UUID="802daf3d-fe98-4f0c-a9a8-b02e6fa83f2d" TYPE="ext4"
    /dev/sda3: UUID="e313a026-1e9b-4b5d-87ca-f604199984c4" TYPE="swap"

修改配置文件`/etc/fstab`：

    $ sudo vim /etc/fstab

在配置文件添加一行配置项：

    UUID=a9e94538-f436-4773-9d1b-5390c1b2e018 /media/mynewdrive ext4 defaults 0 2

如果需要配置生效，重启操作系统，或者使用命令：

    $ sudo mount -a

# 配置写入权限

挂载的硬盘拥有者`root`用户，当前用户没有写权限，无法写入内容：

    $ ls -ld /media/aosp/
    drwxr-xr-x 3 root root 4096 Aug 16 10:00 /media/mynewdrive/

需要使用户可写入内容，有两种方式：

1. 修改挂载点所属用户与组为当前用户与组
2. 使用`plugdev`组

## 修改用户与组

将挂载点目录的用户、组修改为当前用户：

    $ sudo chown -R USERNAME:USERNAME /media/mynewdrive

这种方式不够灵活，不利于多用户使用，推荐使用`plugdev`组

## 使用`plugdev`组

一般桌面用户或可挂载移动硬盘的用户都会添加到`plugdev`组中，这个组的用户可以在挂载的移动硬盘上创建文件、目录，写入内容。

修改挂载点的组：

    $ sudo chgrp plugdev /media/mynewdrive

赋予组的写权限：

    $ sudo chmod g+w /media/mynewdrive

添加`sticky bit`，这样即使用户对文件有写权限，也只能删除他们自己创建的文件与目录，无法删除其他用户的文件与目录：

    $ sudo chmod +t /media/mynewdrive

查看修改结果：

    $ ls -ld /media/mynewdrive/
    drwxrwxr-t 4 root plugdev 4096 Aug 16 11:41 /media/mynewdrive/

# 参看

[InstallingANewHardDrive](https://help.ubuntu.com/community/InstallingANewHardDrive)

[Adding a New Disk Drive to an Ubuntu Linux System](http://www.techotopia.com/index.php/Adding_a_New_Disk_Drive_to_an_Ubuntu_Linux_System)

[Parted User's Manual](https://www.gnu.org/software/parted/manual/)

[Create partition aligned using parted](https://unix.stackexchange.com/questions/38164/create-partition-aligned-using-parted#49274)

[How to align partitions for best performance using parted](https://rainbow.chard.org/2013/01/30/how-to-align-partitions-for-best-performance-using-parted/)
