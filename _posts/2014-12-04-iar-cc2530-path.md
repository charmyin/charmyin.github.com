---
layout: post
title: "Config CC2530 Development file path in IAR"
description: "Config CC2530 Development file path in IAR"
category: [zigbee]
tags: [zigbee]
---

---------------------------------------

####打开

 ````project->options->linker->config->override default````

####选择文件

````C:\Program Files (x86)\IAR Systems\Embedded Workbench 6.0 Evaluation\8051\config\devices\Texas Instruments\lnk51ew_cc2530F256_banked.xcl````

####打开

````project->options->General Options->Target````

####选择文件

````C:\Program Files (x86)\IAR Systems\Embedded Workbench 5.4\8051\config\devices\Texas Instruments\CC2530F256.i51````


###Problem

 在IAR 8.1中，基于CC2530F256芯片对Z-STACK进行builder,如果对IAR环境设置不当会出现：

 ````Fatal Error[e72]: Segment ZIGNV_ADDRESS_SPACE must be defined in a segment definition option (-Z, -b or -P)````

###解决方法如下：

####1、在General options设置：

选Target, 在Device information下的Device选

````C:\Program Files\IAR Systems\Embedded Workbench 6.0 for 8051 Evaluation\8051\config\devices\Texas Instruments````
下的：````CC2530F256.i51````

###2、Linker设置
  1）选config页面，在link configuration file下选择上override default选项，点击选择上f8w2530.xcl文件（缺省），即：
````C:\cc2530_example\link_long_slice_cc2530\20131205\cord\ZStack-CC2530- 2.3.0-1.4.0\Projects\zstack\Tools\CC2530DB\f8w2530.xcl````

  2）在output页面
    在output file选择上选项override default,并填写修改为:coord.hex文件

###3、Debug设置

Setup页面，Driver下选择````Texas Instruments````
