---
layout: post
title: "Periodcally send message to coodornator"
description: "Periodcally send message to coodornator"
category: [zigbee]
tags: [zigbee]
---

---------------------------------------

####Periodcally send message to coodornator


        /**************************************************************************************************

        **************************************************************************************************/

        /*********************************************************************
          This application isn't intended to do anything useful, it is
          intended to be a simple example of an application's structure.

          This application sends it's messages either as broadcast or
          broadcast filtered group messages.  The other (more normal)
          message addressing is unicast.  Most of the other sample
          applications are written to support the unicast message model.

          Key control:
            SW1:  Sends a flash command to all devices in Group 1.
            SW2:  Adds/Removes (toggles) this device in and out
                  of Group 1.  This will enable and disable the
                  reception of the flash command.
        *********************************************************************/

        /*********************************************************************
         * INCLUDES
         */
        #include <string.h>
        #include "OSAL.h"
        #include "ZGlobals.h"
        #include "AF.h"
        #include "aps_groups.h"
        #include "ZDApp.h"
        #include "ZDObject.h"
        #include "ZDProfile.h"

        #include "SampleApp.h"
        #include "SampleAppHw.h"

        #include "OnBoard.h"

        /* HAL */
        #include "hal_lcd.h"
        #include "hal_led.h"
        #include "hal_key.h"
        #include "hal_adc.h"
        #include "NLMEDE.h"

        #include "MT_UART.h"

        #include "MT.h"

        /*********************************************************************
         * MACROS
         */

        /*********************************************************************
         * CONSTANTS
         */

        /*********************************************************************
         * TYPEDEFS
         */

        /*********************************************************************
         * GLOBAL VARIABLES
         */

        // This list should be filled with Application specific Cluster IDs.
        const cId_t SampleApp_ClusterList[SAMPLEAPP_MAX_CLUSTERS] =
        {
          SAMPLEAPP_PERIODIC_CLUSTERID,
          SAMPLEAPP_FLASH_CLUSTERID
        };

        const SimpleDescriptionFormat_t SampleApp_SimpleDesc =
        {
          SAMPLEAPP_ENDPOINT,              //  int Endpoint;
          SAMPLEAPP_PROFID,                //  uint16 AppProfId[2];
          SAMPLEAPP_DEVICEID,              //  uint16 AppDeviceId[2];
          SAMPLEAPP_DEVICE_VERSION,        //  int   AppDevVer:4;
          SAMPLEAPP_FLAGS,                 //  int   AppFlags:4;
          SAMPLEAPP_MAX_CLUSTERS,          //  uint8  AppNumInClusters;
          (cId_t *)SampleApp_ClusterList,  //  uint8 *pAppInClusterList;
          SAMPLEAPP_MAX_CLUSTERS,          //  uint8  AppNumInClusters;
          (cId_t *)SampleApp_ClusterList   //  uint8 *pAppInClusterList;
        };

        // This is the Endpoint/Interface description.  It is defined here, but
        // filled-in in SampleApp_Init().  Another way to go would be to fill
        // in the structure here and make it a "const" (in code space).  The
        // way it's defined in this sample app it is define in RAM.
        endPointDesc_t SampleApp_epDesc;

        /*********************************************************************
         * EXTERNAL VARIABLES
         */

        /*********************************************************************
         * EXTERNAL FUNCTIONS
         */

        /*********************************************************************
         * LOCAL VARIABLES
         */
        uint8 SampleApp_TaskID;   // Task ID for internal task/event processing
                                  // This variable will be received when
                                  // SampleApp_Init() is called.
        devStates_t SampleApp_NwkState;

        uint8 SampleApp_TransID;  // This is the unique message ID (counter)

        afAddrType_t SampleApp_Periodic_DstAddr;
        afAddrType_t SampleApp_Flash_DstAddr;

        afAddrType_t Point_To_Point_DstAddr;//点对点通信

        aps_Group_t SampleApp_Group;

        uint8 SampleAppPeriodicCounter = 0;
        uint8 SampleAppFlashCounter = 0;

        uint8 asc_16[16]={'0','1','2','3','4','5','6','7','8','9','A','B','C','D','E','F'};

        // NodeType  NodeID   DataType
        //   ↓         ↓        ↓
        //   0         000       0000
        // NodeType：1-Coordinator,2-Router, 3-EndPoint
        // NodeID：Unique Id of a node
        // DataType: Use this to specify different data, eg:0001 means temperature
        uint8* SerialPrefix = "10010000";  //协调节点
        //uint8* SerialPrefix = "20020000";  //路由节点
        //uint8* SerialPrefix = "30030000";  //子节点
        //uint8* SerialPrefix = "30040000";    //子节点
        /*********************************************************************
         * LOCAL FUNCTIONS
         */
        void Charmyin_SerialCMD(mtOSALSerialData_t* cmdMsg);
        void SampleApp_HandleKeys( uint8 shift, uint8 keys );
        void SampleApp_MessageMSGCB( afIncomingMSGPacket_t *pckt );
        void SampleApp_SendPeriodicMessage( void );
        void SampleApp_SendFlashMessage( uint16 flashTime );
        void SampleApp_SendPointToPointMessage(void);
        void Charmyin_SendSensorInfo(mtOSALSerialData_t* cmdMsg);
        float getTemperature (void);
        float adc_start(void);
        void To_string(uint8 *dest,char* src,uint8 length);
        /*********************************************************************
        **Utils
        **/
        uint16 convertFrom8To16(uint8 dataFirst, uint8 dataSecond);
        uint8 *convertFrom16To8(uint16 dataAll);

        void GenericApp_ProcessZDOMsgs(zdoIncomingMsg_t *inMsg);
        //增加这个函数的目的是对ZDO_CB_MSG消息响应

        /*********************************************************************
         * NETWORK LAYER CALLBACKS
         */

        /*********************************************************************
         * PUBLIC FUNCTIONS
         */

        /*********************************************************************
         * @fn      SampleApp_Init
         *
         * @brief   Initialization function for the Generic App Task.
         *          This is called during initialization and should contain
         *          any application specific initialization (ie. hardware
         *          initialization/setup, table initialization, power up
         *          notificaiton ... ).
         *
         * @param   task_id - the ID assigned by OSAL.  This ID should be
         *                    used to send messages and set timers.
         *
         * @return  none
         */
        void SampleApp_Init( uint8 task_id )
        {
          SampleApp_TaskID = task_id;
          SampleApp_NwkState = DEV_INIT;
          SampleApp_TransID = 0;

          /**********串口初始化***************/
          MT_UartInit();//初始化
          MT_UartRegisterTaskID(task_id);//登记注册号


          ZDO_RegisterForZDOMsg( SampleApp_TaskID, IEEE_addr_rsp );
          ZDO_RegisterForZDOMsg( SampleApp_TaskID, NWK_addr_rsp );
          HalUARTWrite(0,"Hello World\n",12);



          // Device hardware initialization can be added here or in main() (Zmain.c).
          // If the hardware is application specific - add it here.
          // If the hardware is other parts of the device add it in main().

         #if defined ( BUILD_ALL_DEVICES )
          // The "Demo" target is setup to have BUILD_ALL_DEVICES and HOLD_AUTO_START
          // We are looking at a jumper (defined in SampleAppHw.c) to be jumpered
          // together - if they are - we will start up a coordinator. Otherwise,
          // the device will start as a router.
          if ( readCoordinatorJumper() )
            zgDeviceLogicalType = ZG_DEVICETYPE_COORDINATOR;
          else
            zgDeviceLogicalType = ZG_DEVICETYPE_ROUTER;
        #endif // BUILD_ALL_DEVICES

        #if defined ( HOLD_AUTO_START )
          // HOLD_AUTO_START is a compile option that will surpress ZDApp
          //  from starting the device and wait for the application to
          //  start the device.
          ZDOInitDevice(0);
        #endif

          // Setup for the periodic message's destination address
          // Broadcast to everyone
          SampleApp_Periodic_DstAddr.addrMode = (afAddrMode_t)AddrBroadcast;
          SampleApp_Periodic_DstAddr.endPoint = SAMPLEAPP_ENDPOINT;
          SampleApp_Periodic_DstAddr.addr.shortAddr = 0xFFFF;

          // Setup for the flash command's destination address - Group 1
          SampleApp_Flash_DstAddr.addrMode = (afAddrMode_t)afAddrGroup;
          SampleApp_Flash_DstAddr.endPoint = SAMPLEAPP_ENDPOINT;
          SampleApp_Flash_DstAddr.addr.shortAddr = SAMPLEAPP_FLASH_GROUP;

          // 点对点通讯定义
          Point_To_Point_DstAddr.addrMode = (afAddrMode_t)Addr16Bit;//点播
          Point_To_Point_DstAddr.endPoint = SAMPLEAPP_ENDPOINT;
          Point_To_Point_DstAddr.addr.shortAddr = 0x0000; //发给协调器


          // Fill out the endpoint description.
          SampleApp_epDesc.endPoint = SAMPLEAPP_ENDPOINT;
          SampleApp_epDesc.task_id = &SampleApp_TaskID;
          SampleApp_epDesc.simpleDesc
                    = (SimpleDescriptionFormat_t *)&SampleApp_SimpleDesc;
          SampleApp_epDesc.latencyReq = noLatencyReqs;

          // Register the endpoint description with the AF
          afRegister( &SampleApp_epDesc );

          // Register for all key events - This app will handle all key events
          RegisterForKeys( SampleApp_TaskID );

          // By default, all devices start out in Group 1
          SampleApp_Group.ID = 0x0001;
          osal_memcpy( SampleApp_Group.name, "Group 1", 7  );
          aps_AddGroup( SAMPLEAPP_ENDPOINT, &SampleApp_Group );

        #if defined ( LCD_SUPPORTED )
          HalLcdWriteString( "SampleApp", HAL_LCD_LINE_1 );
        #endif
        }

        /*********************************************************************
         * @fn      SampleApp_ProcessEvent
         *
         * @brief   Generic Application Task event processor.  This function
         *          is called to process all events for the task.  Events
         *          include timers, messages and any other user defined events.
         *
         * @param   task_id  - The OSAL assigned task ID.
         * @param   events - events to process.  This is a bit map and can
         *                   contain more than one event.
         *
         * @return  none
         */
        uint16 SampleApp_ProcessEvent( uint8 task_id, uint16 events )
        {
          afIncomingMSGPacket_t *MSGpkt;
          (void)task_id;  // Intentionally unreferenced parameter
          HalUARTWrite(0, "RV\n" ,3 );//换行
          HalUARTWrite(0,&asc_16[MSGpkt->hdr.event/4096],1);
          HalUARTWrite(0,&asc_16[MSGpkt->hdr.event%4096/256],1);
          HalUARTWrite(0,&asc_16[MSGpkt->hdr.event%256/16],1);
          HalUARTWrite(0,&asc_16[MSGpkt->hdr.event%16],1);
          if ( events & SYS_EVENT_MSG )
          {
            MSGpkt = (afIncomingMSGPacket_t *)osal_msg_receive( SampleApp_TaskID );
            while ( MSGpkt )
            {

              switch ( MSGpkt->hdr.event )
              {
                //Serial msg arrived
                case CMD_SERIAL_MSG: //串口收到数据后由 MT_UART 层传递过来的数据，用网蜂方法接收，编译时不定义 MT相关内容，
                  HalUARTWrite(0, "SD\n" ,3 );//换行
                  //Charmyin_SerialCMD((mtOSALSerialData_t*)MSGpkt);
                  //获取自己的地址
                  ZDP_IEEEAddrReq(0x0000,ZDP_ADDR_REQTYPE_EXTENDED,0,0);
                  break;

                // Received when a key is pressed
                case KEY_CHANGE:
                  SampleApp_HandleKeys( ((keyChange_t *)MSGpkt)->state, ((keyChange_t *)MSGpkt)->keys );
                  break;

                // Received when a messages is received (OTA) for this endpoint
                case AF_INCOMING_MSG_CMD:
                  SampleApp_MessageMSGCB( MSGpkt );
                  break;
              case ZDO_CB_MSG:
                   GenericApp_ProcessZDOMsgs((zdoIncomingMsg_t*)MSGpkt);
                  break;
                // Received whenever the device changes state in the network
                case ZDO_STATE_CHANGE:
                  SampleApp_NwkState = (devStates_t)(MSGpkt->hdr.status);
                  if ( //(SampleApp_NwkState == DEV_ZB_COORD)
                      (SampleApp_NwkState == DEV_ROUTER)
                      || (SampleApp_NwkState == DEV_END_DEVICE) )
                  {
                    // Start sending the periodic message in a regular interval.
                    osal_start_timerEx( SampleApp_TaskID,
                                      SAMPLEAPP_SEND_PERIODIC_MSG_EVT,
                                      SAMPLEAPP_SEND_PERIODIC_MSG_TIMEOUT );
                    //Send Sensor and topology Info
                    Charmyin_SendSensorInfo((mtOSALSerialData_t*)MSGpkt);
                  }
                  else
                  {
                    // Device is no longer in the network
                  }
                  break;

                default:
                  break;
              }

              // Release the memory
              osal_msg_deallocate( (uint8 *)MSGpkt );

              // Next - if one is available
              MSGpkt = (afIncomingMSGPacket_t *)osal_msg_receive( SampleApp_TaskID );
            }

            // return unprocessed events
            return (events ^ SYS_EVENT_MSG);
          }

          // Send a message out - This event is generated by a timer
          //  (setup in SampleApp_Init()).
          if ( events & SAMPLEAPP_SEND_PERIODIC_MSG_EVT )
          {
            // Send the periodic message
            //SampleApp_SendPeriodicMessage();
           // SampleApp_SendPointToPointMessage();
            // Setup to send message again in normal period (+ a little jitter)
            osal_start_timerEx( SampleApp_TaskID, SAMPLEAPP_SEND_PERIODIC_MSG_EVT,
                (SAMPLEAPP_SEND_PERIODIC_MSG_TIMEOUT + (osal_rand() & 0x00FF)) );
            //Send Sensor and topology Info
            Charmyin_SendSensorInfo( (mtOSALSerialData_t*)MSGpkt );
            // return unprocessed events
            return (events ^ SAMPLEAPP_SEND_PERIODIC_MSG_EVT);
          }

          // Discard unknown events
          return 0;
        }

        /*********************************************************************
         * Event Generation Functions
         */


        /*********************************************************************
         * @fn      Charmyin_SendSensorInfo
         *
         * @brief   Handles all serial events for this device.
         *
        * @param   Message arrived:datalength+data
         *
         * @return  none
         */
        void Charmyin_SendSensorInfo(mtOSALSerialData_t* cmdMsg)
        {
        //    HalLedSet(HAL_LED_2,HAL_LED_MODE_TOGGLE);
        //    HalLedSet(HAL_LED_1,HAL_LED_MODE_TOGGLE);
            //3004000uint8uint813.11

            uint16 temp = NLME_GetCoordShortAddr();
            /****将短地址分解，ASC码打印*****/
            HalUARTWrite(0,&asc_16[temp/4096],1);
            HalUARTWrite(0,&asc_16[temp%4096/256],1);
            HalUARTWrite(0,&asc_16[temp%256/16],1);
            HalUARTWrite(0,&asc_16[temp%16],1);
            HalUARTWrite(0,"\n",1);               // 回车换行
            uint8 *addressArray = convertFrom16To8(temp);

            uint8 output[15];
            for(int ii=0; ii<8; ii++){
              output[ii]  = SerialPrefix[ii];
            }

            //获取温度
            char i;
            float avgTemp;

            avgTemp = 0;
            for (i = 0; i <64; i++)
            {
              avgTemp += getTemperature();
              avgTemp = avgTemp/2; // per sample times, and the average of 1 times
            }
            output[10] = (unsigned char)(avgTemp) / 10 + 48; // ten
            output[11] = (unsigned char)(avgTemp)% 10 + 48; // bits
            output[12] = '.'; // decimal point
            output[13] = (unsigned char)(avgTemp * 10)% 10 +48; // decile
            output[14] = (unsigned char)(avgTemp * 100)% 10 +48; // percentile
            //output [5] = '\0'; // string terminator
            //HalUARTWrite(0,"tp",2); //提示接收到数据

            //填入地址信息
            output[8] = addressArray[0];
            output[9] = addressArray[1];
             HalUARTWrite(0,output,15);

            //uint8 data[10] = {0,1,2,2,3,4,6,2,2,9};
            Point_To_Point_DstAddr.addr.shortAddr = 0x0000;//发送给协调器
            if ( AF_DataRequest( &Point_To_Point_DstAddr, &SampleApp_epDesc,
                                 SAMPLEAPP_POINT_TO_POINT_CLUSTERID,
                                 15,
                                 output,//指针方式
                                 &SampleApp_TransID,
                                 AF_DISCV_ROUTE,
                                 AF_DEFAULT_RADIUS ) == afStatus_SUCCESS )
            {
            }
            else
            {
              // Error occurred in request to send.
            }
        }

        /*********************************************************************
         * @fn      Charmyin_SerialCMD
         *
         * @brief   Handles all serial events for this device.
         *
        * @param   Message arrived:datalength+data
         *
         * @return  none
         */
        void Charmyin_SerialCMD(mtOSALSerialData_t* cmdMsg)
        {
        //    HalLedSet(HAL_LED_2,HAL_LED_MODE_TOGGLE);
        //    HalLedSet(HAL_LED_1,HAL_LED_MODE_TOGGLE);
            //数据类型,目的地址,pin编号, 操作数值
            uint16 dataType, destDeviceAddr;//, pinCode, dataValue;
            uint8 i,len,*str=NULL;       //len 有用数据长度
            str=cmdMsg->msg;         //指向数据开头
            len=*str;                  //msg 里的第 1 个字节代表后面的数据长度
            //获取数据类型
            memcpy(&dataType,str+1,2);
            //获取目的地址
            memcpy(&destDeviceAddr,str+3,2);
            //获取pin编号
            //memcpy(&pinCode,str+7,2);
            //获取操作数值
            //memcpy(&dataValue,str+9,2);

            HalUARTWrite(0, (unsigned char *)&destDeviceAddr ,2 );
            //获取
            /********打印出串口接收到的数据，用于提示*********/
        //    for(i=0;i<=7;i++)
        //    HalUARTWrite(0,str+i,1 );
        //    HalUARTWrite(0, "\n" ,1 );//换行
        //    HalUARTWrite(0, "OK2\n" ,3 );//换行
            /*******传送点对点指令*********/
            //发送给指定节点
            Point_To_Point_DstAddr.addr.shortAddr = destDeviceAddr;

            if ( AF_DataRequest( &SampleApp_Periodic_DstAddr, &SampleApp_epDesc,
            dataType,//自己定义一个
            10,                    //  数据长度
            str,                        //数据内容
            &SampleApp_TransID,
            AF_DISCV_ROUTE,
            AF_DEFAULT_RADIUS ) == afStatus_SUCCESS )
            {
            }
            else
            {
              // Error occurred in request to send.
              HalUARTWrite(0, "error\n",6 );
            }
        }



        /*********************************************************************
         * @fn      SampleApp_HandleKeys
         *
         * @brief   Handles all key events for this device.
         *
         * @param   shift - true if in shift/alt.
         * @param   keys - bit field for key events. Valid entries:
         *                 HAL_KEY_SW_2
         *                 HAL_KEY_SW_1
         *
         * @return  none
         */
        void SampleApp_HandleKeys( uint8 shift, uint8 keys )
        {
          (void)shift;  // Intentionally unreferenced parameter

          if ( keys & HAL_KEY_SW_1 )
          {
            /* This key sends the Flash Command is sent to Group 1.
             * This device will not receive the Flash Command from this
             * device (even if it belongs to group 1).
             */
            SampleApp_SendFlashMessage( SAMPLEAPP_FLASH_DURATION );
          }

          if ( keys & HAL_KEY_SW_2 )
          {
            /* The Flashr Command is sent to Group 1.
             * This key toggles this device in and out of group 1.
             * If this device doesn't belong to group 1, this application
             * will not receive the Flash command sent to group 1.
             */
            aps_Group_t *grp;
            grp = aps_FindGroup( SAMPLEAPP_ENDPOINT, SAMPLEAPP_FLASH_GROUP );
            if ( grp )
            {
        //      ZDP_IEEEAddrReq
        //        NodePowerDescriptorFormat_t
        //          ZDP_BindReq
              // Remove from the group
              aps_RemoveGroup( SAMPLEAPP_ENDPOINT, SAMPLEAPP_FLASH_GROUP );
            }
            else
            {
              // Add to the flash group
              aps_AddGroup( SAMPLEAPP_ENDPOINT, &SampleApp_Group );
            }
          }
        }

        /*********************************************************************
         * LOCAL FUNCTIONS
         */

        /*********************************************************************
         * @fn      SampleApp_MessageMSGCB
         *
         * @brief   Data message processor callback.  This function processes
         *          any incoming data - probably from other devices.  So, based
         *          on cluster ID, perform the intended action.
         *
         * @param   none
         *
         * @return  none
         */
        void SampleApp_MessageMSGCB( afIncomingMSGPacket_t *pkt )
        {
          uint8 len;
          /*16进制转ASCII码表*/
          uint8 asc_16[16]={'0','1','2','3','4','5','6','7','8','9','A','B','C','D','E','F'};
          uint16 temp, tempParentAddr;
          uint8 outputSerialPrefix[8];
          uint8 *str=pkt->cmd.Data;

          //获取数据
          byte tranSeq;
          zAddrType_t dstAddr;
          byte status;
          byte IEEEAddrRemoteDev;
          byte reqType;
          uint16 nwkAddr;
          byte numAssocDev;
          byte startIndex;
          uint16 *NWKAddrAssocDevList;
          byte securitySuite;


          //数据类型,目的地址,pin编号, 操作数值
          uint16 dataType, destDeviceAddr;//, pinCode, dataValue;
          switch ( pkt->clusterId )
          {
          case IEEE_addr_rsp:
            {
             char buf[16];
             char changeline[2]={0x0A,0x0D};
              ZDO_NwkIEEEAddrResp_t* pRsp=ZDO_ParseAddrRsp((zdoIncomingMsg_t*)pkt);//对接受到的数据包进行解析，解析完后pRsp指向数据包的存放地址
               if(pRsp)
               {
                  if(pRsp->status==ZSuccess)//解析是否正确
                  {
                     To_string(buf,pRsp->extAddr,8);//把MAC地址 转换为16进制形式存放
                     HalUARTWrite(0,"Coordinator MAC:",osal_strlen("Coordinator MAC:"));
                     HalUARTWrite(0,buf,16);
                     HalUARTWrite(0,"\n",1);
                   }
                   osal_mem_free(pRsp);//调用该函数释放数据包缓冲区
                }
                break;
            }
            case SAMPLEAPP_POINT_TO_POINT_CLUSTERID_DEP:
             //HalLedSet(HAL_LED_2,HAL_LED_MODE_TOGGLE);
        //      HalLedSet(HAL_LED_1,HAL_LED_MODE_TOGGLE);
              //case SAMPLEAPP_PERIODIC_CLUSTERID:
              //HalUARTWrite(0, SerialPrefix ,8);
              //uint8 outputDateAll[8+2+4+2+4+7];
              //读出前8位头信息

              for(int ii=0; ii<8; ii++){
                outputSerialPrefix[ii] = pkt->cmd.Data[ii];
              }
              HalUARTWrite(0,outputSerialPrefix,8);
              //读出数据包的16位短地址
              temp=pkt->srcAddr.addr.shortAddr;
              tempParentAddr=convertFrom8To16(pkt->cmd.Data[8], pkt->cmd.Data[9]);//读取16位信号源节点的父节点短地址

              /****将短地址分解，ASC码打印*****/
              HalUARTWrite(0,"0x",2); //提示接收到数据
              HalUARTWrite(0,&asc_16[tempParentAddr/4096],1);
              HalUARTWrite(0,&asc_16[tempParentAddr%4096/256],1);
              HalUARTWrite(0,&asc_16[tempParentAddr%256/16],1);
              HalUARTWrite(0,&asc_16[tempParentAddr%16],1);
              //HalUARTWrite(0, "\n", 1);

              HalUARTWrite(0,"0x",2); //提示接收到数据

              /****将短地址分解，ASC码打印*****/
              HalUARTWrite(0,&asc_16[temp/4096],1);
              HalUARTWrite(0,&asc_16[temp%4096/256],1);
              HalUARTWrite(0,&asc_16[temp%256/16],1);
              HalUARTWrite(0,&asc_16[temp%16],1);
              //30040000uint8uint8tp13.11
              //读出温度信息
              uint8 output[7];
              output[0]='t';
              output[1]='p';
              for(int ii=2; ii<=6; ii++){
                output[ii] = pkt->cmd.Data[ii+8];
              }
               HalUARTWrite(0,output,7);
              //100000000x00000xE439tp49.45
              //HalUARTWrite(0,"\n",1);               // 回车换行
              break;
            case SAMPLEAPP_COM_CLUSTERID:
        //      HalLedSet(HAL_LED_2,HAL_LED_MODE_TOGGLE);
        //      HalLedSet(HAL_LED_1,HAL_LED_MODE_TOGGLE);

              len=pkt->cmd.Data[0];
              for(int i=0; i<len;i++)
              HalUARTWrite(0, &pkt->cmd.Data[i+1], 1);
              HalUARTWrite(0, "\n", 1);
              break;
            case PIN_CONTROL_CLUSTERID:
              memcpy(&dataType,str+3,2);
              //获取目的地址
              memcpy(&destDeviceAddr,str+5,2);
              //获取pin编号
              //memcpy(&pinCode,str+7,2);
              //获取操作数值
              //memcpy(&dataValue,str+9,2);
              HalLedSet(HAL_LED_2,HAL_LED_MODE_TOGGLE);
              HalLedSet(HAL_LED_1,HAL_LED_MODE_TOGGLE);
              break;
          }
        }

        /*********************************************************************
         * @fn      SampleApp_SendPeriodicMessage
         *
         * @brief   Send the periodic message.
         *
         * @param   none
         *
         * @return  none
         */
        void SampleApp_SendPeriodicMessage( void )
        {
          uint8 data[10] = {0,1,2,2,3,4,6,2,2,9};
          if ( AF_DataRequest( &SampleApp_Periodic_DstAddr, &SampleApp_epDesc,
                               SAMPLEAPP_PERIODIC_CLUSTERID,
                               10,
                               data,//指针方式
                               &SampleApp_TransID,
                               AF_DISCV_ROUTE,
                               AF_DEFAULT_RADIUS ) == afStatus_SUCCESS )
          {
          }
          else
          {
            // Error occurred in request to send.
          }
        }


        /*********************************************************************
         * @fn      SampleApp_SendPointToPointMessage
         *
         * @brief   Send the flash message to group 1.
         *
         * @param   flashTime - in milliseconds
         *
         * @return  none
         */
        void SampleApp_SendPointToPointMessage(void)
        {
          uint8 data[10]={0,1,2,3,4,5,6,7,8,9};
          if(AF_DataRequest( &Point_To_Point_DstAddr, &SampleApp_epDesc, SAMPLEAPP_POINT_TO_POINT_CLUSTERID,
          10, data, &SampleApp_TransID, AF_DISCV_ROUTE, AF_DEFAULT_RADIUS ) == afStatus_SUCCESS )
          {
          }
          else
          {
            // Error occurred in request to send.
          }
        }

        /*********************************************************************
         * @fn      SampleApp_SendFlashMessage
         *
         * @brief   Send the flash message to group 1.
         *
         * @param   flashTime - in milliseconds
         *
         * @return  none
         */
        void SampleApp_SendFlashMessage( uint16 flashTime )
        {
          uint8 buffer[3];
          buffer[0] = (uint8)(SampleAppFlashCounter++);
          buffer[1] = LO_UINT16( flashTime );
          buffer[2] = HI_UINT16( flashTime );

          if ( AF_DataRequest( &SampleApp_Flash_DstAddr, &SampleApp_epDesc,
                               SAMPLEAPP_FLASH_CLUSTERID,
                               3,
                               buffer,
                               &SampleApp_TransID,
                               AF_DISCV_ROUTE,
                               AF_DEFAULT_RADIUS ) == afStatus_SUCCESS )
          {
          }
          else
          {
            // Error occurred in request to send.
          }
        }


        void GenericApp_ProcessZDOMsgs(zdoIncomingMsg_t* inMsg)
        {
         char buf[16];
         char changeline[2]={0x0A,0x0D};
         switch(inMsg->clusterID)
         {
         case IEEE_addr_rsp:
           {
             ZDO_NwkIEEEAddrResp_t* pRsp=ZDO_ParseAddrRsp(inMsg);//对接受到的数据包进行解析，解析完后pRsp指向数据包的存放地址
             if(pRsp)
             {
                if(pRsp->status==ZSuccess)//解析是否正确
                {
                   To_string(buf,pRsp->extAddr,8);//把MAC地址 转换为16进制形式存放
                   HalUARTWrite(0,"Coordinator MAC:",osal_strlen("Coordinator MAC:"));
                   HalUARTWrite(0,buf,16);
                   HalUARTWrite(0,changeline,2);
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

        uint8 *convertFrom16To8(uint16 dataAll) {
            static uint8 arrayData[2] = { 0x00, 0x00 };

            *(arrayData) = (dataAll >> 8) & 0x00FF;
            arrayData[1] = dataAll & 0x00FF;
            return arrayData;
        }

        /**
        Get temperature
        **/
        float getTemperature (void)
        {
          unsigned int value;
          ADCCON3 = (0x3E); // select 1.25V reference voltage; 14-bit resolution; sampling on-chip temperature sensor
          ADCCON1 |= 0x30; // select the the ADC startup mode to manual
          ADCCON1 |= 0x40; // Start the AD conversion
          while(!(ADCCON1 & 0x80)); // Wait for the end of the ADC conversion
          value = ADCL >> 2;
          value |= (ADCH << 6); // Get the final conversion results stored in the value in
          return value * 0.06229-311.43; // According to the formula to calculate the temperature
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
