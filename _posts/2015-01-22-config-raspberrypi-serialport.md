---
layout: post
title: "Reload network when coordinator restarted"
description: "Reload network when coordinator restarted"
category: [zigbee]
tags: [zigbee, topology]
---

---------------------------------------

####Problem description
**树莓派的串口原来已被占用，配置释放之后才能使用它来进行串口通信。**

####Solution


#####首先将 /boot/cmdline.txt里面的

````dwc_otg.lpm_enable=0 console=ttyAMA0,115200 kgdboc=ttyAMA0,115200 console=tty1 root=/dev/mmcblk0p2 rootfstype=ext4 elevator=deadline rootwait````

.改成

````dwc_otg.lpm_enable=0 console=tty1 root=/dev/mmcblk0p2 rootfstype=ext4 elevator=deadline rootwait````
这样移除所有与ttyAMA0之间的关联

这样移除所有与ttyAMA0之间的关联

之后进入文件 ````/etc/inittab````

注释掉最后一句

````#T0:23:respawn:/sbin/getty -L ttyAMA0 115200 vt100````
