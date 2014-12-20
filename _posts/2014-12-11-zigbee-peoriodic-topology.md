---
layout: post
title: "Send periodic msg contains parent network address"
description: "Send periodic msg contains parent network address"
category: [zigbee]
tags: [zigbee]
---

---------------------------------------

####周期性数据包含内容
**发送周期**
所在文件：SampleApp.h

````define SAMPLEAPP_SEND_PERIODIC_MSG_TIMEOUT   5000     // Every 5 seconds``````

**子节点传送至协调器的数据结构**
父节点ID(16位)+温度+湿度

**协调器传送至后端服务器数据结构**
数据类型(4字节)+父节点ID(16位)+子节点ID(16位)+温度+湿度
