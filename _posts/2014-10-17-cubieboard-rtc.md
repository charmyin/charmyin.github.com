---
layout: post
title: "树莓派设定RTC"
description: "树莓派设定RTC, DS1307"
category: [cubieboard]
tags: [cubieboard, raspberrypi, rtc, ds1307]
---

[http://www.dfrobot.com.cn/community/thread-2636-1-1.html](http://www.dfrobot.com.cn/community/thread-2636-1-1.html)


树莓派自带的I2C为我们连接众多的I2C的外设提供了方便，下面咱们试试树莓派上连接一个 DFRobot 推出的 Raspberry Pi meet Arduino 扩展板，在这个扩展板上已经有一个DS1307 RTC实时时钟它就是I2C的设备。
  

![DS1307]({{ site.url }}/assets/images/2014-10-17/rtc.jpg)


首先我们需要修改树莓派的配置文件使能I2C，通过nano编辑器修改raspi-blacklist.conf 文件内容

	pi@raspberrypi ~ $ sudo nano /etc/modprobe.d/raspi-blacklist.conf

修改为如下，开启I2C功能

	# blacklist spi and i2c by default (many users don't need them)

	blacklist spi-bcm2708
	blacklist i2c-bcm2708


打开/etc/modules ，在文件结尾加上 i2c-dev

	pi@raspberrypi ~ $ sudo nano /etc/modules

	# /etc/modules: kernel modules to load at boot time.
	#
	# This file contains the names of kernel modules that should be loaded
	# at boot time, one per line. Lines beginning with "#" are ignored.
	# Parameters can be specified after the module name.

	snd-bcm2835
	i2c-bcm2708
	i2c-dev

 

更新一次包列表

	pi@raspberrypi ~ $ sudo apt-get update
 

安装 i2c-tools工具与python-smbus

	pi@raspberrypi ~ $ sudo apt-get install i2c-tools python-smbus
 

重启树莓派

	pi@raspberrypi ~ $ sudo reboot
 
重启后重新通过ssh 登录到树莓派 通过刚才安装的i2c-tools对i2c设备进行探测
输入以下命令，得到如下结果，说明检测到一个地址为0x68的I2C设备就是板上的DS1307。

	pi@raspberrypi ~ $ sudo i2cdetect -y 1
	     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
	00:          -- -- -- -- -- -- -- -- -- -- -- -- -- 
	10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
	20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
	30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
	40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
	50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
	60: -- -- -- -- -- -- -- -- 68 -- -- -- -- -- -- -- 
	70: -- -- -- -- -- -- -- --

检看 DS1307的数据手册，发现DS1307的7位地址的确是0x68。

下面我们通过i2c-tools来测试下DS1307的功能，并将DS1307这个掉电不丢失的时钟用于树莓派系统的时钟。
下面的测试必须在root权限下测试，如何进入root可以参考前面的文章

	pi@raspberrypi ~ $ su
	Password: 
	root@raspberrypi:/home/pi# modprobe i2c-dev
	root@raspberrypi:/home/pi# echo ds1307 0x68 > /sys/class/i2c-adapter/i2c-1/new_device
	root@raspberrypi:/home/pi# hwclock -r
	Sat Jan  1 00:00:12 2000  -0.120717 seconds
	root@raspberrypi:/home/pi#


将系统时钟设定为当前时间，然后将系统时钟写入到DS1307硬件时钟里面

	root@raspberrypi:/home/pi# date 062120352014.00     备注：6月21日  20点35分 2014年
	Sat Jun 21 20:35:00 UTC 2014
	root@raspberrypi:/home/pi# hwclock -w
	root@raspberrypi:/home/pi# date
	Sat Jun 21 20:35:21 UTC 2014


编辑启动文件

	sudo nano /etc/rc.local


l将以下内容加入“exit 0”行之前

	modprobe i2c-dev
	echo ds1307 0x68 > /sys/class/i2c-adapter/i2c-1/new_device
	hwclock -r
	hwclock –s

ctrl+x退出

过几分钟拔掉网线，重启树莓派，输入“date”看看时间是否正确。

	root@raspberrypi:/home/pi# date
	Sat Jun 21 20:55:00 UTC 2014
	root@raspberrypi:/home/pi# hwclock -r
	Sat Jun 21 20:56:30 2014  -0.181549 seconds
	root@raspberrypi:/home/pi# 

以上将系统时钟和DS1307硬件时钟读出来发现 系统时钟比DS1307时钟慢了1分多。可能是在重启后DS1307写入系统后，系统需要等待1分多才开始计时。

至此，以后要用到实时时间就可以简单的使用 date 这个命令来获取。
在python编程中可以使用以下代码读取实时时间。

	import datetime                                                        #导入系统时钟
	now = datetime.datetime.now()                              #读取当前时间并保存到now变量
	timeString = now.strftime("%Y-%m-%d%H:%M")   #把now中的时间按指定格式转换成字符串