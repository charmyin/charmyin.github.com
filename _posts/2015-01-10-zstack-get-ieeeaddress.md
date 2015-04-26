---
layout: post
title: "Get coordinator ieeaddress and related device network address"
description: "Get coordinator ieeaddress and related device network address first"
category: [zigbee]
tags: [zigbee, topology]
---

---------------------------------------

####Get coordinator ieeaddress and related device network address

  1.注册请求响应事件
  ````ZDO_RegisterForZDOMsg( SampleApp_TaskID, IEEE_addr_rsp );````

  2.发出请求
  ````ZDP_IEEEAddrReq(0x0000,ZDP_ADDR_REQTYPE_EXTENDED,0,0);````

  3.接受对应的请求响应事件

        case ZDO_CB_MSG:
                 GenericApp_ProcessZDOMsgs((zdoIncomingMsg_t*)MSGpkt);
                break;

  4.解析数据打印

        void Charmyin_ZDO_CB_MSGs(zdoIncomingMsg_t* inMsg)
        {
         char buf[16];
         char childAdd[4];
         uint8 shortAddress[2] = {0x00,0x00};
         char changeline[2]={0x0A,0x0D};
         switch(inMsg->clusterID)
         {
         case IEEE_addr_rsp:
           {
             ZDO_NwkIEEEAddrResp_t* pRsp=ZDO_ParseAddrRsp(inMsg);//对接受到的数据包进行解析，解析完后pRsp指向数据包的存放地址
             uint16* itemList = (uint16*)pRsp->devList;
             if(pRsp)
             {
                if(pRsp->status==ZSuccess)//解析是否正确
                {
                   To_string(buf,pRsp->extAddr,8);//把MAC地址 转换为16进制形式存放
                   convertFrom16To8(pRsp->nwkAddr, shortAddress);
                   To_string(childAdd,shortAddress,2);//把NETWORK地址 转换为16进制形式存放
                   HalUARTWrite(0,"MAC-NET:",osal_strlen("MAC-NET:"));
                   HalUARTWrite(0,buf,16);
                   HalUARTWrite(0,"-",1);
                   HalUARTWrite(0,childAdd,4);
                   HalUARTWrite(0,changeline,2);
                   //遍历所有关联节点短地址，获取每个节点下的关联节点
                   for(int count =0; count<pRsp->numAssocDevs; count++){
                     //To_string(childAdd, (char *)&(pRsp->devList[count]),2);
                     //HalUARTWrite(0,"Child device 1:",osal_strlen("Child device 1:"));
                     //HalUARTWrite(0,childAdd,4);
                     //HalUARTWrite(0,changeline,2);
                     ZDP_IEEEAddrReq(pRsp->devList[count],ZDP_ADDR_REQTYPE_EXTENDED,0,0);
                   }

                 }
                  osal_mem_free(pRsp);//调用该函数释放数据包缓冲区
              }
            }
             break;
          }
        }
        /*********************************************************************
        *********************************************************************/
        uint16 convertFrom8To16(uint8 dataFirst, uint8 dataSecond) {
            uint16 dataBoth = 0x0000;

            dataBoth = dataFirst;
            dataBoth = dataBoth << 8;
            dataBoth |= dataSecond;
            return dataBoth;
        }
        uint8 *convertFrom16To8(uint16 dataAll, uint8 *arrayData) {
            //static uint8 arrayData[2] = { 0x00, 0x00 };

            *(arrayData) = (dataAll >> 8) & 0x00FF;
            arrayData[1] = dataAll & 0x00FF;
            //return arrayData;
        }
        //二进制书转化为十六进制数
        void To_string(uint8 *dest,char* src,uint8 length)
        {
          uint8* xad;
          uint8 i=0;
          uint8 ch;
          xad=src+length-1;
          for(i=0;i<length;i++,xad--)
          {
           ch=(*xad>>4)&0x0F;  //除以十六
           dest[i<<1]=ch+((ch<10)?'0':'7');
           ch=*xad&0x0F;
           dest[(i<<1)+1]=ch+((ch<10)?'0':'7');
          }
        }
