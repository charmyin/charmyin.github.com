---
layout: post
title: "Reload network when coordinator restarted"
description: "Reload network when coordinator restarted"
category: [zigbee]
tags: [zigbee, topology]
---

---------------------------------------

####Problem description
**协调器重启后，系统无法正常工作**

####Solution

**终端节点在父节点丢失以后会自动搜寻网络加入，而路由节点加入到协调器以后就会在这个网络中正常工作，即使协调器断电仍会继续工作。**

**如果协调器断电，路由节点仍然在工作，其他路由节点或终端节点仍可加入该组建好的网络当中。**

**如果想让协调器重启后加入到该网络，在条件编译选项添加NV_RESTORE便可以保存协调器的建网PANID。**
![Picture]({{ site.url }}/assets/images/2015-01-18/resetcoordinator.jpg)

**如果想让路由节点重启，协议栈中有指令复位功能，需要手动添加。搜索reset关键词即可查到。复位原理：关闭所有终端，打开看门狗，进入死循环。。然后复位。**
