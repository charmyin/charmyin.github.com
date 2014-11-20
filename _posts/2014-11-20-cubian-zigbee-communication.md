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




###Set GPIO pins to UART pins

####Edit /boot/script.bin file

**script.bin info**
[Fex Guide](http://linux-sunxi.org/Fex_Guide)

[How to change uart](http://charmyin.github.io/cubieboard/2014/11/19/cubian-uart/)

**Get script.fex file**

        #bin2fex  script.bin script.fex
        #vi script.fex

**Make script.fex to script.bin**

        #fex2bin script.fex  script.bin
        #reboot

        [uart_para4]
        uart_used = 1
        uart_port = 4
        uart_type = 2
        uart_tx = port:PG10<4><1><default><default>
        uart_rx = port:PG11<4><1><default><default>

**Disabled used pin PG10 and PG11**

        [gpio_para]
        gpio_used = 1
        gpio_num = 67
        gpio_pin_1 = port:PG03<1><default><default><1>
        gpio_pin_2 = port:PB19<1><default><default><1>
        gpio_pin_3 = port:PB18<1><default><default><1>
        gpio_pin_4 = port:PG06<1><default><default><1>
        gpio_pin_5 = port:PG05<1><default><default><1>
        gpio_pin_6 = port:PG04<1><default><default><1>
        gpio_pin_7 = port:PG01<1><default><default><1>
        gpio_pin_8 = port:PG02<1><default><default><1>
        gpio_pin_9 = port:PG00<1><default><default><1>
        gpio_pin_10 = port:PH14<1><default><default><1>
        gpio_pin_11 = port:PH15<1><default><default><1>
        gpio_pin_12 = port:PI06<1><default><default><1>
        gpio_pin_13 = port:PI05<1><default><default><1>
        gpio_pin_14 = port:PI04<1><default><default><1>
        ---------------------下面删除---------------------
        gpio_pin_15 = port:PG11<1><default><default><1>
        gpio_pin_16 = port:PG10<1><default><default><1>
        ---------------------上面删除---------------------
        gpio_pin_17 = port:PG09<1><default><default><1>
        gpio_pin_18 = port:PG08<1><default><default><1>
        gpio_pin_19 = port:PG07<1><default><default><1>


###ATTENTION
The ttyS* is not the port 4 set upside; use ````dmesg | grep ttyS*```` to show which ttyS* is the corret name;

###Use follwing code to debug

````#vi uart_test.c````



        /********************************************************************
         **************************uart_test*********************************
         ********************************************************************/
        #include <stdio.h>
        #include <stdlib.h>
        #include <unistd.h>
        #include <sys/types.h>
        #include <sys/stat.h>
        #include <fcntl.h>
        #include <termios.h>
        #include <errno.h>
        #include <sys/time.h>
        #include <string.h>

        #define TRUE 1
        #define FALSE -1

        int speed_arr[] = {B115200, B38400, B19200, B9600, B4800, B2400, B1200, B300,
                B38400, B19200, B9600, B4800, B2400, B1200, B300, };

        int name_arr[] = {115200, 38400,  19200,  9600,  4800,  2400,  1200,  300,
                38400,  19200,  9600, 4800, 2400, 1200,  300, };

        void set_speed(int fd, int speed)
        {
          int i;
          int status;

          struct termios Opt;
          tcgetattr(fd,&Opt);

          for (i= 0;i<sizeof(speed_arr)/sizeof(int);i++)
          {
                if(speed == name_arr[i])
               {
                   tcflush(fd, TCIOFLUSH);


                cfsetispeed(&Opt, speed_arr[i]);

                cfsetospeed(&Opt, speed_arr[i]);


                status = tcsetattr(fd, TCSANOW, &Opt);

                if(status != 0)
                   perror("tcsetattr fd1");

                     return;
                 }
                tcflush(fd,TCIOFLUSH);
           }
        }

        int set_Parity(int fd,int databits,int stopbits,int parity)
        {
            struct termios options;
           if( tcgetattr( fd,&options)!= 0)
           {
                  perror("SetupSerial 1");

                   return(FALSE);
            }

           options.c_cflag &= ~CSIZE;


          switch(databits)
           {
                  case 7:
                  options.c_cflag |= CS7;
                  break;

               case 8:
                options.c_cflag |= CS8;
                break;

               default:
                fprintf(stderr,"Unsupported data size\n");

                return (FALSE);
            }

           switch(parity)
              {
                  case 'n':
                case 'N':
                options.c_cflag &= ~PARENB;    /* Clear parity enable */
                options.c_iflag &= ~INPCK;    /* Enable parity checking */
                options.c_iflag &= ~(ICRNL|IGNCR);
                options.c_lflag &= ~(ICANON );
                break;


            case 'o':

            case 'O':
                options.c_cflag |= (PARODD | PARENB);
                options.c_iflag |= INPCK;    /* Disnable parity checking */
                break;


            case 'e':

            case 'E':
                options.c_cflag |= PARENB;    /* Enable parity */
                options.c_cflag &= ~PARODD;
                options.c_iflag |= INPCK;    /* Disnable parity checking */
                break;


            case 'S':

            case 's':  /*as no parity*/
                options.c_cflag &= ~PARENB;
                options.c_cflag &= ~CSTOPB;
                break;

                 default:
                fprintf(stderr,"Unsupported parity\n");

                return (FALSE);
            }

           switch(stopbits)
              {

              case 1:

              options.c_cflag &= ~CSTOPB;

            break;


            case 2:

            options.c_cflag |= CSTOPB;

            break;


            default:

            fprintf(stderr,"Unsupported stop bits\n");


            return (FALSE);
            }

          /* Set input parity option */

           if(parity != 'n')
                  options.c_iflag |= INPCK;
                  options.c_cc[VTIME] = 150; // 15 seconds
               options.c_cc[VMIN] = 0;

                 tcflush(fd,TCIFLUSH); /* Update the options and do it NOW */

          if(tcsetattr(fd,TCSANOW,&options) != 0)
          {
                  perror("SetupSerial 3");

                return (FALSE);
            }

           return (TRUE);
        }

        int main(int argc, char **argv)
        {
            int fd;
            int nread;
            int nwrite;
            int n=0;
            int i=0;
            char buffer[15];
            char devname_head[10] = "/dev/";
            char dev_name[20];

             if(argc < 2)
            {
              printf("Please input './test_uart ttySx'\n");
              exit(1);
            }
            else
            {
                strcpy(dev_name, devname_head);
                strcat(dev_name, argv[1]);
            }

            fd = open(dev_name, O_RDWR);
            if(fd < 0)
            {
                perror("error to open /dev/ttySx\n");
                exit(1);
            }


            if (fd > 0)
            {
                set_speed(fd,115200);
            }
            else
            {
                printf("Can't Open Serial Port!\n");

                exit(0);
            }

              if (set_Parity(fd,8,1,'N') == FALSE)
              {
                printf("Set Parity Error\n");

                exit(1);
              }

            printf("\nWelcome to uart_test\n\n");

           memset(buffer,0,sizeof(buffer));

           char test[15] = "hello world";

           nwrite = write(fd,test,strlen(test));
            if(nwrite < 0)
            {
                printf("write error\n");
            }

           printf("Send test data------>%s\n",test);

              while(1)
              {
                nread = read(fd,&buffer[n],1);
                if(nread < 0)
                {
                    printf("read error\n");
                }

                printf("read char is -> %c \n",buffer[n]);

                   if (strlen(buffer) == strlen(test))
                   {
                       printf("Read Test Data finished,Read Test Data is------->%s\n",buffer);

                    memset(buffer,0,sizeof(buffer));
                    printf("Send test data again------>%s\n",test);

                    write(fd,test,strlen(test));
                    n=0;
                    sleep(1);

                    continue;
                }
               n++;
            }
        }

**Build by following shell script**

````#apt-get install gcc build-essential````
````#gcc uart_test.c -o uart_test````

