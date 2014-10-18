---
layout: post
title: "Cubieboard设定RTC"
description: "Cubieboard设定RTC, DS1307"
category: [cubieboard]
tags: [cubieboard, rtc, ds1307]
---


[linux-sunxi](https://github.com/linux-sunxi)

###Cubieboard和DS1307接线

	VCC -> 5.0V VCC
	GND-> GND
	SCK-> PB18
	SDA-> PB19

**如图**

![1]({{ site.url }}/assets/images/2014-10-18/2-1.jpg)

###调试步骤

**1.安装i2c-tools包**

	#apt-get install i2c-tools

**2.执行命令查看i2c设备**

	sudo i2cdetect -y -a 1 //查看设备
	i2cdump 1 0x68 //读取设备数据

注意：如果没有找到68，此处可以将“1”换成“0”

**3.开机后，输入su -，切换为root登录，并执行**

	modprobe i2c-dev
	echo ds1307 0x68 > /sys/class/i2c-adapter/i2c-1/new_device

注意：如果这里报错，请修改/sys/class/i2c-adapter/i2c-1/new_device的权限

	sudo chmod 777 /sys/class/i2c-adapter/i2c-1/new_device

**4.设定rtc时间**

	# sudo hwclock --systohc -D --noadjfile --utc
	# sudo hwclock --set --date "06/05/13 23:00:00" //如果无法设置，请修改提示文件的权限
	# sudo hwclock --show

**5.读取，写入RTC时钟**

	hwclock -r //显示RTC时间
	hwclock -w //将RTC时间设定为系统时间 set the hardware clock from the current system time
	hwclock –s //将系统时间设定为RTC时间 set the system time from the hardware clock,


