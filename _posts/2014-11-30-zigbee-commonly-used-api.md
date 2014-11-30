---
layout: post
title: "Z-Stcak协议栈中有常用的API函数"
description: "Z-Stcak协议栈中有大量的API函数，用起来非常方便可能有些朋友不太会用，在此送上实际使用时的程序代码，希望能对大家有所帮助。"
category: [zigbee]
tags: [zigbee]
---

---------------------------------------

###Z-Stcak协议栈中有常用的API函数

大家好！Z-Stcak协议栈中有大量的API函数，用起来非常方便可能有些朋友不太会用，在此送上实际使用时的程序代码，希望能对大家有所帮助。（我是采用串口指令的形式调用这些常用API函数的）

####进程注册和串口初始化程序：

      // Register the endpoint description with the AF
          afRegister( &ZG_Serial_Control_epDesc );

          ZDO_RegisterForZDOMsg( ZG_Serial_Control_TaskID, End_Device_Bind_rsp );
          ZDO_RegisterForZDOMsg( ZG_Serial_Control_TaskID, Match_Desc_rsp );
          ZDO_RegisterForZDOMsg( ZG_Serial_Control_TaskID, Bind_rsp );
          ZDO_RegisterForZDOMsg( ZG_Serial_Control_TaskID, Unbind_rsp );

          ZDO_RegisterForZDOMsg( ZG_Serial_Control_TaskID, IEEE_addr_rsp );
          ZDO_RegisterForZDOMsg( ZG_Serial_Control_TaskID, NWK_addr_rsp );
          ZDO_RegisterForZDOMsg( ZG_Serial_Control_TaskID, Node_Desc_rsp );
          ZDO_RegisterForZDOMsg( ZG_Serial_Control_TaskID, Device_annce );

          //Config UART0,38400-8-n-1   //lang
          uartConfig.configured           = TRUE;             // 2x30 don't care - see uart driver.
          uartConfig.baudRate             = HAL_UART_BR_38400;
          uartConfig.flowControl          = FALSE;
          uartConfig.flowControlThreshold = 64;               // 2x30 don't care - see uart driver.
          uartConfig.rx.maxBufSize        = 128;              // 2x30 don't care - see uart driver.
          uartConfig.tx.maxBufSize        = 128;              // 2x30 don't care - see uart driver.
          uartConfig.idleTimeout          = 6;                // 2x30 don't care - see uart driver.
          uartConfig.intEnable            = TRUE;             // 2x30 don't care - see uart driver.
          uartConfig.callBackFunc         = SerialApp_CallBack;
          HalUARTOpen (HAL_UART_PORT_0, &uartConfig);

####串口回调函数：

      /*********************************************************************
      * @fn      void SerialApp_CallBack(uint8 port, uint8 event)
      *
      * @brief   Process response messages
      *
      * @param   none
      *
      * @return  none
      */
      static void SerialApp_CallBack(uint8 port, uint8 event)
      {
      #ifndef ZG_ENDDEVICE
          rtgItem_t *rtTable;
          byte rtnum;
          uint8 *pRtBuf = NULL;
      #endif
      #ifndef ZG_COORDINATOR
          neighborLqiItem_t *NeighborTable;
          byte Neighnum;
          uint8 *pNeighBuf = NULL;
          uint8 My_CoordExtAddr[8];
          uint16 My_CoordShortAddr;
      #endif
          zAddrType_t destAddr, devAddr, BindAddr;
          BindingEntry_t *pBindingTable;
          uint16 ZG_Serial_Control_ClusterList[ZG_Serial_Control_MAX_CLUSTERS]={ZG_Serial_Control_CLUSTERID};
          uint8 index;

          uint8 Bind_SourceAddr[8];
          uint8 Uart_buf[20];
          uint8 My_ExtAddr[8];
          uint16 My_ShortAddr;
          uint16 Dest_ShortAddr;

          if ((event & (HAL_UART_RX_FULL | HAL_UART_RX_ABOUT_FULL | HAL_UART_RX_TIMEOUT)) &&
      #if SERIAL_APP_LOOPBACK
              (SerialApp_TxLen < SERIAL_APP_TX_MAX))
      #else
              !SerialApp_TxLen)
      #endif
          {
              SerialApp_TxLen=HalUARTRead(HAL_UART_PORT_0,Uart_buf,20);
              if(SerialApp_TxLen)
              {
                  HalUARTWrite(HAL_UART_PORT_0,Uart_buf,SerialApp_TxLen);
                  SerialApp_TxLen=0;
                  switch(Uart_buf[0])
                  {
                    case 0x11:        //返回设备IEEE地址
                      osal_cpyExtAddr( My_ExtAddr, (NLME_GetExtAddr()));
                      HalUARTWrite(HAL_UART_PORT_0,My_ExtAddr,8);
                      break;
                    case 0x22:        //返回设备16位网络地址
                      My_ShortAddr=NLME_GetShortAddr();
                      HalUARTWrite(HAL_UART_PORT_0,(uint8*)&My_ShortAddr,1);
                      HalUARTWrite(HAL_UART_PORT_0,((uint8*)&My_ShortAddr+1),1);
                      break;
                  #ifndef ZG_COORDINATOR
                    case 0x33:        //返回设备父节点IEEE地址
                      NLME_GetCoordExtAddr(My_CoordExtAddr);
                      HalUARTWrite(HAL_UART_PORT_0,My_CoordExtAddr,8);
                      break;
                    case 0x44:        //返回设备父节点网络地址
                      My_CoordShortAddr=NLME_GetCoordShortAddr();
                      HalUARTWrite(HAL_UART_PORT_0,(uint8*)&My_CoordShortAddr,1);
                      HalUARTWrite(HAL_UART_PORT_0,((uint8*)&My_CoordShortAddr+1),1);
                      break;
                  #endif
                    case 0x55:        //通过16位网络地址查询设备的IEEE地址，输入网络地址为小端存储方式
                      Dest_ShortAddr=BUILD_UINT16( Uart_buf[1], Uart_buf[2] );
                      ZDP_IEEEAddrReq(Dest_ShortAddr,ZDP_ADDR_REQTYPE_SINGLE,0,0);
                      break;
                    case 0x66:        //通过64位IEEE地址查询设备的网络地址，输入IEEE地址为小端存储方式
                      osal_cpyExtAddr( My_ExtAddr, &Uart_buf[1]);
                      ZDP_NwkAddrReq(My_ExtAddr,ZDP_ADDR_REQTYPE_SINGLE,0,0);
                      break;
                  #ifndef ZG_ENDDEVICE
                    case 0x77:        //返回设备的路由表
                      NLME_GetRequest(nwkNumRoutingTableEntries,0,&rtnum);
                      pRtBuf =osal_mem_alloc((short)rtnum* sizeof(rtgItem_t));
                      if(pRtBuf!=NULL)
                      {
                          rtTable = (rtgItem_t *)pRtBuf;
                          for(index=0;index<rtnum;index++)
                          {
                              NLME_GetRequest(nwkRoutingTable,index,(void *)rtTable);
                              HalUARTWrite(HAL_UART_PORT_0,(uint8 *)rtTable,4);
                              rtTable++;
                          }
                      }
                      break;
                  #endif
                  #ifndef ZG_COORDINATOR
                    case 0x88:        //返回设备的邻居表
                      NLME_GetRequest(nwkNumNeighborTableEntries,0,&Neighnum);
                      pNeighBuf =osal_mem_alloc((short)Neighnum* sizeof(neighborLqiItem_t));
                      if(pNeighBuf!=NULL)
                      {
                          NeighborTable = (neighborLqiItem_t *)pNeighBuf;
                          for(index=0;index<Neighnum;index++)
                          {
                              NLME_GetRequest(nwkNeighborTable,index,(void *)NeighborTable);
                              HalUARTWrite(HAL_UART_PORT_0,(uint8 *)NeighborTable,10);
                              NeighborTable++;
                          }
                      }
                      break;
                  #endif
                    case 0x99:        //请求网络设备的设备描述符,节点逻辑类型。000：协调器；001：路由器；010：终端设备
                      destAddr.addrMode = Addr16Bit;
                      destAddr.addr.shortAddr = BUILD_UINT16( Uart_buf[1], Uart_buf[2] );
                      Dest_ShortAddr = BUILD_UINT16( Uart_buf[1], Uart_buf[2] );
                      ZDP_NodeDescReq( &destAddr, Dest_ShortAddr, 0 );
                      break;
                    case 0xAA:        //由第三方设备对另外两个设备进行绑定
                      HalLedSet( HAL_LED_2, HAL_LED_MODE_ON );
                      destAddr.addrMode = Addr16Bit;
                      destAddr.addr.shortAddr = BUILD_UINT16( Uart_buf[1], Uart_buf[2] ); //目标设备网络地址
                      osal_cpyExtAddr(Bind_SourceAddr,&Uart_buf[3]);

                      devAddr.addrMode = Addr64Bit;
                      osal_cpyExtAddr( devAddr.addr.extAddr, &Uart_buf[11] );

                      ZDP_BindReq( &destAddr,
                                   Bind_SourceAddr,
                                   ZG_Serial_Control_ENDPOINT,
                                   ZG_Serial_Control_CLUSTERID,
                                   &devAddr,
                                   ZG_Serial_Control_ENDPOINT,
                                   0 );
                      break;
                    case 0xBB:        //由第三方设备对另外两个设备进行解绑定
                      HalLedSet( HAL_LED_2, HAL_LED_MODE_ON );
                      destAddr.addrMode = Addr16Bit;
                      destAddr.addr.shortAddr = BUILD_UINT16( Uart_buf[1], Uart_buf[2] ); //目标设备网络地址
                      osal_cpyExtAddr(Bind_SourceAddr,&Uart_buf[3]);

                      devAddr.addrMode = Addr64Bit;
                      osal_cpyExtAddr( devAddr.addr.extAddr, &Uart_buf[11] );

                      ZDP_UnbindReq( &destAddr,
                                     Bind_SourceAddr,
                                     ZG_Serial_Control_ENDPOINT,
                                     ZG_Serial_Control_CLUSTERID,
                                     &devAddr,
                                     ZG_Serial_Control_ENDPOINT,
                                     0 );
                      break;
                    case 0xCC:        //初始化绑定表
                      InitBindingTable();
                    case 0xDD:        //在绑定表中添加一条绑定，但目前只能绑定其父节点
                      BindAddr.addrMode = Addr64Bit;
                      osal_cpyExtAddr( BindAddr.addr.extAddr, &Uart_buf[1] );
                      bindAddEntry(ZG_Serial_Control_ENDPOINT,
                                   &BindAddr,
                                   ZG_Serial_Control_ENDPOINT,
                                   ZG_Serial_Control_MAX_CLUSTERS,
                                   ZG_Serial_Control_ClusterList);


                      break;
                    case 0xEE:        //在绑定表中删除一条绑定
                      BindAddr.addrMode = Addr64Bit;
                      osal_cpyExtAddr( BindAddr.addr.extAddr, &Uart_buf[1] );
                      pBindingTable = bindFindExisting( ZG_Serial_Control_ENDPOINT,
                                                        &BindAddr,
                                                        ZG_Serial_Control_ENDPOINT );

                      bindRemoveEntry( pBindingTable );
                      break;
                    default:
                      break;
                  }
              }
          }
      }

####进程回调处理函数：

      /*********************************************************************
      * @fn      ZG_Serial_Control_ProcessZDOMsgs()
      *
      * @brief   Process response messages
      *
      * @param   none
      *
      * @return  none
      */
      static void ZG_Serial_Control_ProcessZDOMsgs( zdoIncomingMsg_t *inMsg )
      {
        ZDO_NwkIEEEAddrResp_t *pAddrRsp;//网络地址、IEEE地址结构体声明

        static uint16 NWKAddr_rsp;
        static uint8 LogicalType_rsp;

        switch ( inMsg->clusterID )
        {
          case End_Device_Bind_rsp:
            if ( ZDO_ParseBindRsp( inMsg ) == ZSuccess )
            {
              HalLedSet ( HAL_LED_2, HAL_LED_MODE_OFF );
            }
            break;
          case Match_Desc_rsp:
            {
              ZDO_ActiveEndpointRsp_t *pRsp = ZDO_ParseEPListRsp( inMsg );
              if ( pRsp )
              {
                if ( pRsp->status == ZSuccess && pRsp->cnt )
                {
                  ZG_Serial_Control_DstAddr.addrMode = (afAddrMode_t)Addr16Bit;
                  ZG_Serial_Control_DstAddr.addr.shortAddr = pRsp->nwkAddr;
                  // Take the first endpoint, Can be changed to search through endpoints
                  ZG_Serial_Control_DstAddr.endPoint = pRsp->epList[0];

                  // Light LED
                  HalLedSet( HAL_LED_2, HAL_LED_MODE_OFF );
                }
                osal_mem_free( pRsp );
              }
            }
            break;
      #ifdef ZG_COORDINATOR
          case Device_annce:  //设备入网声明回调函数
            ZDO_ParseDeviceAnnce( inMsg, pDeviceAnnce );

            LCD_New_EndDevice[9]=Character_transfor(((pDeviceAnnce->nwkAddr>>12)&0x000F));
            LCD_New_EndDevice[10]=Character_transfor(((pDeviceAnnce->nwkAddr>>8)&0x000F));
            LCD_New_EndDevice[11]=Character_transfor(((pDeviceAnnce->nwkAddr>>4)&0x000F));
            LCD_New_EndDevice[12]=Character_transfor((pDeviceAnnce->nwkAddr&0x000F));

            HalLcdWriteString( LCD_New_EndDevice, HAL_LCD_LINE_4 );
            break;
      #endif
          case Bind_rsp:
            if ( ZDO_ParseBindRsp( inMsg ) == ZSuccess )//绑定成功
            HalLedSet( HAL_LED_2, HAL_LED_MODE_OFF );
            break;
          case Unbind_rsp:
            if ( ZDO_ParseBindRsp( inMsg ) == ZSuccess )//绑定成功
            HalLedSet( HAL_LED_2, HAL_LED_MODE_OFF );
            break;
          case IEEE_addr_rsp:
            pAddrRsp = ZDO_ParseAddrRsp( inMsg );//将接受到的数据解析输出到相应结构体中
            if(pAddrRsp)
            {
              if( pAddrRsp->status == ZSuccess )
              {
                  HalUARTWrite(HAL_UART_PORT_0,pAddrRsp->extAddr,8);//输出响应回来的IEEE地址
              }
              osal_mem_free( pAddrRsp );//释放内存
            }
            break;
          case NWK_addr_rsp:
            pAddrRsp = ZDO_ParseAddrRsp( inMsg );//将接受到的数据解析输出到相应结构体中
            if(pAddrRsp)
            {
              if( pAddrRsp->status == ZSuccess )
              {
                  NWKAddr_rsp=pAddrRsp->nwkAddr;
                  HalUARTWrite(HAL_UART_PORT_0,(uint8*)&NWKAddr_rsp,1);//输出响应回来的网络地址
                  HalUARTWrite(HAL_UART_PORT_0,((uint8*)&NWKAddr_rsp+1),1);
              }
              osal_mem_free( pAddrRsp );//释放内存
            }
            break;
          case Node_Desc_rsp:
            ZDO_ParseNodeDescRsp( inMsg, pNodeDescRsp);
            LogicalType_rsp=pNodeDescRsp->nodeDesc.LogicalType;   //节点逻辑类型。000：协调器；001：路由器；010：终端设备
            HalUARTWrite(HAL_UART_PORT_0,&LogicalType_rsp,1);     //输出响应回来的设备类型
            break;
          default:
            break;
        }
      }
      复制代码
