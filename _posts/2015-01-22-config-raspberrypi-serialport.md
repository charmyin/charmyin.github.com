---
layout: post
title: "Config raspberry pi to communicated with serial port"
description: "Config raspberry pi to communicated with serial port"
category: [raspberryPi]
tags: [raspberryPi, serialPort]
---

---------------------------------------

####Problem description
**树莓派的串口原来已被占用，配置释放之后才能使用它来进行串口通信。**

**SerialPort of RP has already been used，need to release the serial port for other communication.**

####Solution


#####首先将 /boot/cmdline.txt里面的

#####First, find the line below in file "/boot/cmdline.txt"

````dwc_otg.lpm_enable=0 console=ttyAMA0,115200 kgdboc=ttyAMA0,115200 console=tty1 root=/dev/mmcblk0p2 rootfstype=ext4 elevator=deadline rootwait````

.改成
.change to the line below

````dwc_otg.lpm_enable=0 console=tty1 root=/dev/mmcblk0p2 rootfstype=ext4 elevator=deadline rootwait````

这样移除所有与ttyAMA0之间的关联

Then it releases the relation between ttyAMA0 and serial port.

之后进入文件 ````/etc/inittab````
In the end, fine file ````/etc/inittab````

注释掉最后一句
Comment the line below:

````#T0:23:respawn:/sbin/getty -L ttyAMA0 115200 vt100````
