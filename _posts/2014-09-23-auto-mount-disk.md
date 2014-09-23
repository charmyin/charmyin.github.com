---
layout: post
title: "Linux分区格式化，自动挂载"
description: "Linux分区格式化，自动挂载"
category: [linux]
tags: [linux, fdisk]
---


###1. 给硬盘分区

　　**在slackware下有两个分区软件fdisk和cfdisk**

　　**例如我们已经有一个硬盘了，现在添加另一个硬盘到系统**

　　**那么我们根据命名规则知道这个新添加的硬盘应该是hdb。我们用下面命令给硬盘分区**

　　````fdisk /dev/hdb````

　　**你也可以用cfdisk来分区，命令如下**

　　````cfdisk /dev/hdb````

###1. 格式化硬盘

　　**格式化成ext3格式**

　　````mkfs.ext3 /dev/hdb1````

　　**格式化成reiserfs的格式**

　　````mkfs.reiserfs /dev/hdb1````

###1. 让硬盘启动自动挂载

  **配置文件包含以下几项:**

  ````<file system> <mount point>   <type>  <options>       <dump>  <pass>````

  ````<file system>```` ：分区定位，可以给磁盘号，UUID或LABEL，例如：````/dev/sda2````，UUID=6E9ADAC29ADA85CD或LABEL=software

  ````<mount point> ````: 具体挂载点的位置，例如：````/media/C````

  ````<type>```` : 挂载磁盘类型，linux分区一般为ext4，windows分区一般为ntfs

  ````<options>```` : 挂载参数，一般为defaults

  ````<dump>```` : 磁盘备份，默认为0，不备份

  ````<pass>```` : 磁盘检查，默认为0，不检查


  **例如挂载````/dev/hdb1````分区到````/mnt/hd````目录下**
　**用vi编辑````/etc/fstab````文件，加入如下内容**

　　````/dev/sdb1      /home                ext4     defaults       0         0````

　　````/dev/sdb5      /yammy/appSpace      ext4     defaults       0         0````
　　

　　````/dev/sdb6      /yammy/documents     ext4     defaults       0         0````

