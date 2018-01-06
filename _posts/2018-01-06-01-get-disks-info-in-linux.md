---
layout: post
keywords: Linux
description: get disks info in linux
title: "Linux中查看硬盘信息"
categories: [Linux]
tags: [Linux, fdisk, parted, df, lsblk, lshw, blkid]
group: Linux
icon: file-alt
date: 2018-01-06 17:37:00
---

一台电脑中可以安装多块硬盘，下面我们来研究下在`Linux`中如何查看所有硬盘信息。系统中添加了两块硬盘，第二块没有格式化也没有挂载。

# df

`df`命令是用来查看文件系统中硬盘的使用状况的，也可以用来列出系统中挂载的硬盘，使用`-h`选项可以以人类可读的格式输出硬盘使用状况：

    ~$ df -h
    文件系统        容量  已用  可用 已用% 挂载点
    /dev/sda2        55G  3.7G   49G    8% /
    udev            2.0G  4.0K  2.0G    1% /dev
    tmpfs           394M  776K  394M    1% /run
    none            5.0M     0  5.0M    0% /run/lock
    none            2.0G  220K  2.0G    1% /run/shm
    /dev/sda1       487M  3.3M  483M    1% /boot/efi

<!--excerpt-->

`df`命令无法显示未挂载的硬盘。

# lsblk

`lsblk`命令是用来查看块设备的：

    $ lsblk 
    NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sda      8:0    0    60G  0 disk 
    ├─sda1   8:1    0   487M  0 part /boot/efi
    ├─sda2   8:2    0  55.5G  0 part /
    └─sda3   8:3    0     4G  0 part [SWAP]
    sdb      8:16   0    50G  0 disk 
    sr0     11:0    1  1024M  0 rom  

不带参数时会列出所有硬盘，根节点代表硬盘，二级节点代表磁盘上的分区。使用`-d`选项可以只列出硬盘，不列出分区信息。

# lshw

`lshw`可以打印硬件的详细信息：

    $ sudo lshw -class disk
      *-disk:0
           description: SCSI Disk
           physical id: 0.0.0
           bus info: scsi@32:0.0.0
           logical name: /dev/sda
           size: 60GiB (64GB)
           capabilities: gpt-1.00 partitioned partitioned:gpt
           configuration: guid=ff481b67-ace2-47b9-a7e8-a50d4e6a6e55
      *-disk:1
           description: SCSI Disk
           physical id: 0.1.0
           bus info: scsi@32:0.1.0
           logical name: /dev/sdb
           size: 50GiB (53GB)
      *-cdrom
           description: DVD-RAM writer
           physical id: 0.0.0
           bus info: scsi@3:0.0.0
           logical name: /dev/cdrom
           logical name: /dev/cdrw
           logical name: /dev/dvd
           logical name: /dev/dvdrw
           logical name: /dev/sr0
           capabilities: audio cd-r cd-rw dvd dvd-r dvd-ram
           configuration: status=open

# blkid

`blkid`命令可以打印块设备的一些信息：

    $ sudo blkid 
    /dev/sda1: UUID="AB45-3BA0" TYPE="vfat" 
    /dev/sda2: UUID="802daf3d-fe98-4f0c-a9a8-b02e6fa83f2d" TYPE="ext4" 
    /dev/sda3: UUID="e313a026-1e9b-4b5d-87ca-f604199984c4" TYPE="swap"

# fdisk

`fdisk`是一个用来格式化硬盘、分区等的常用的分区表操纵工具，可以用来打印硬盘信息：

    $ sudo fdisk -l
    
    WARNING: GPT (GUID Partition Table) detected on '/dev/sda'! The util fdisk doesn't support GPT. Use GNU Parted.
    
    
    Disk /dev/sda: 64.4 GB, 64424509440 bytes
    255 heads, 63 sectors/track, 7832 cylinders, total 125829120 sectors
    Units = 扇区 of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x00000000
    
       设备 启动      起点          终点     块数   Id  系统
    /dev/sda1               1   125829119    62914559+  ee  GPT
    
    Disk /dev/sdb: 53.7 GB, 53687091200 bytes
    255 heads, 63 sectors/track, 6527 cylinders, total 104857600 sectors
    Units = 扇区 of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x00000000
    
    Disk /dev/sdb doesn't contain a valid partition table

`fdisk`工具不支持`GPT`分区表，可以使用`GNU Parted`即下面的`parted`工具替代。

# parted

`parted`也是一个分区表操纵工具，目前只能在`GNU/Linux`及`GNU/Hurd`下运行：

    $ sudo parted -l
    Model: VMware, VMware Virtual S (scsi)
    磁盘 /dev/sda: 64.4GB
    Sector size (logical/physical): 512B/512B
    分区表：gpt
    
    数字  开始：  End     大小    文件系统        Name  标志
     1    1049kB  512MB   511MB   fat32                 启动
     2    512MB   60.1GB  59.6GB  ext4
     3    60.1GB  64.4GB  4293MB  linux-swap(v1)
    
    
    错误: /dev/sdb：未确认磁盘标签

# /proc/partitions

通过查看`/proc/partitions`文件内容可以查看当前硬盘及分区的一些信息：

    $ cat /proc/partitions 
    major minor  #blocks  name
    
       8        0   62914560 sda
       8        1     498688 sda1
       8        2   58221568 sda2
       8        3    4192256 sda3
       8       16   52428800 sdb
      11        0    1048575 sr0

# lsscsi

`lsscsi`工具可以打印`SCSI`硬盘信息，这个工具在`Ubuntu12.04`中默认没有安装，需要自行安装：

    $ lsscsi 
    [3:0:0:0]    cd/dvd  NECVMWar VMware SATA CD01 1.00  /dev/sr0
    [32:0:0:0]   disk    VMware,  VMware Virtual S 1.0   /dev/sda
    [32:0:1:0]   disk    VMware,  VMware Virtual S 1.0   /dev/sdb

# 参看

[how to list all hard disks in linux from command line](http://www.lostsaloon.com/technology/how-to-list-disks-in-linux/)

[GNU Parted](https://zh.wikipedia.org/wiki/GNU_Parted)

[Platforms on which GNU Parted runs](https://www.gnu.org/software/parted/manual/parted.html#Supported-Platforms)