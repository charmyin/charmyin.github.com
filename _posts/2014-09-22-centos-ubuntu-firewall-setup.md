---
layout: post
title: "Firewall setup under centos and ubuntu"
description: "Firewall setup under centos and ubuntu"
category: [linux]
tags: [linux, firewall]
---


**Run in terminal：**

  ````#/sbin/iptables -I INPUT -p tcp --dport 80 -j ACCEPT````
  ````#/sbin/iptables -I INPUT -p tcp --dport 22 -j ACCEPT````
  ````#/sbin/iptables -I INPUT -p tcp --dport 3306 -j ACCEPT````

然后保存：

  ````#/etc/rc.d/init.d/iptables save````

查看打开的端口：

  ````#/etc/init.d/iptables status````
