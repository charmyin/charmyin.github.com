---
layout: post
title: "Cubieboard uart usage"
description: "Cubieboard uart usage"
category: [cubieboard]
tags: [cubieboard, uart]
---

---------------------------------------
##How To Use The Uart

About this Article

Author: allen — allen@cubietech.com — 2014/03/26 16:50
Copyrights: CC Attribution-Share Alike 3.0 Unported
Contributors: Cubieboard Community :


[http://docs.cubieboard.org/tutorials/common/how_to_use_the_uart](http://docs.cubieboard.org/tutorials/common/how_to_use_the_uart)

###Abstract

Here is the guide to teach that how to enable the uart port on cubiboard and how to test the uart port is available .

###Cubieboard pin

Cubietruck http://docs.cubieboard.org/a20-cubietruck_gpio_pin

Cubieboard 1 2 http://docs.cubieboard.org/cubieboard1_and_cubieboard2_gpio_pin

Fex Guide http://linux-sunxi.org/Fex_Guide

###Open the port

        #mount /dev/nanda /mnt
        #cd /mnt
        #bin2fex  script.bin script.fex
        #vi script.fex

Modify the script.fex to open the uart3 ，uart4 as below ：

        [uart_para3]
        uart_used = 1
        uart_port = 3
        uart_type = 4
        uart_tx = port:PG06<4><1><default><default>
        uart_rx = port:PG07<4><1><default><default>
        uart_rts = port:PG08<4><1><default><default>
        uart_cts = port:PG09<4><1><default><default>

        [uart_para4]
        uart_used = 1
        uart_port = 4
        uart_type = 2
        uart_tx = port:PG10<4><1><default><default>
        uart_rx = port:PG11<4><1><default><default>
        Save the modification.

        #fex2bin script.fex  script.bin
        #reboot

After the reboot ,the uart3 and uart4 are available.

remember save as script.bin an
