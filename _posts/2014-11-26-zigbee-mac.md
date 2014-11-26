---
layout: post
title: "How to get mac address of zigbee node"
description: "How to get mac address of zigbee node"
category: [zigbee]
tags: [zigbee]
---

---------------------------------------

##How to get mac address of zigbee node

查阅资料可以知道，德州仪器公司对cc2530芯片mac地址做了约定，主要是以下几项：从Z-stack的NV中读取、从Second IEEE 的位置中寻找、在Primary IEEE 的位置寻找、由随机数产生器产生一个临时IEEE地址。

需要知道的是，CC2530芯片在TI出厂时已经预先烧写了 Primary IEEE address，并且不同的芯片基本可以保证MAC地址不同。所以对于一般应用，只需要从Primary IEEE地址中读取即可。
参考ZStack协议栈，只需要添加以下代码，就可以读取IEEE 地址：

      uint8 *macaddrptr = (uint8 *)(P_INFOPAGE+HAL_INFOP_IEEE_OSET);

      for(int i=0;i<8;i++)
      {
        devmacaddr[i] = macaddrptr[i];
      }

其中

    //device mac address
    uint8 devmacaddr[8];    //保存设备MAC地址数组

如何以只读方式读取cc2530 mac 地址
下面简要分析一下代码，其中

    /* Pointer to Start of Flash Information Page */
    #define P_INFOPAGE  PXREG( 0x7800 )


可以知道P_INFOPAGE是指向Flash信息存储页的指针，也就是信息存取区的首页地址。
如何以只读方式读取cc2530 mac 地址
其次，可以看到如下定义

    #define HAL_INFOP_IEEE_OSET        0xC

这个就是IEEE地址在Flash信息存储页中的地址偏移量，通过加运算就可以计算出cc2530芯片的MAC（IEEE 地址）在Flash存储位置，这样子只需要通过赋值运算就可以读取到cc2530芯片64-bit的IEEE扩展地址。
如何以只读方式读取cc2530 mac 地址


###打印IEEE地址

      uint8 *macaddrptr = (uint8 *)(P_INFOPAGE+HAL_INFOP_IEEE_OSET);
      uint8 asc_16[16]={'0','1','2','3','4','5','6','7','8','9','A','B','C','D','E','F'};
      uint8 devmacaddr[8];    //保存设备MAC地址数组

      for(int i=0;i<8;i++)
      {
        devmacaddr[i] = macaddrptr[i];
        HalUARTWrite(0,&asc_16[devmacaddr[i]%256/16],1);
        HalUARTWrite(0,&asc_16[devmacaddr[i]%16],1);
      }
      HalUARTWrite(0,"\n",1);
