---
layout: post
title: "Cubieboard zigbee connection"
description: "Cubieboard zigbee connection"
category: [cubieboard]
tags: [cubieboard, zigbee]
---

---------------------------------------
1、首先用杜邦线连接Zigbee模块的VCC,GND,TX,RX，另一端连接CB的这几个引脚，CB的串口我使用的是UART3，
通过这个页面http://linux-sunxi.org/Cubieboard，知道UART3对应的引脚分别为PG6->TX、PG7->RX。

        2、连线连接好了后，修改fex文件，具体修改方法，可以参照WinLand大神的帖子（[教程]如何修改script.bin/script.fex
http://cn.cubieboard.org/forum.php?mod=viewthread&tid=141&fromuid=221）

       3、修改一下几个项：（对于这些项的修改，我还有疑问，uart_type不知道数字是代表什么意思，还有就是功能分配）
             [uart_para3]
             uart_used = 1
             uart_port = 3
             uart_type = 8
             uart_tx = port:PG06<4><1><default><default>
             uart_rx = port:PG07<4><1><default><default>
             uart_rts = port:PG08<4><1><default><default>
             uart_cts = port:PG09<4><1><default><default>

         Port:端口+组内序号<功能分配><内部电阻状态><驱动能力><输出电平状态>
         注意：我在实验的过程中，发现uart_type,功能分配，貌似不能跟其它已经使用的串口，使用相同的数字，如果相同，打开串口就会出现too much work for irq4的错误。


通过上述步骤，结合WinLand大神的帖子，就能使用cb的串口进行通信了。

补充：uart_type数字含义：2 (2 wire), 4 (4 wire), 8 (8 wire, full function)
