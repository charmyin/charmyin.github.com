---
layout: post
title: "Arduino设定RTC"
description: "Arduino设定RTC, DS1307"
category: [arduino]
tags: [arduino, rtc, ds1307]
---

[http://www.dfrobot.com.cn/community/thread-2636-1-1.html](http://www.dfrobot.com.cn/community/thread-2636-1-1.html)
   Arduino有各种各样的模块，其中有一种模块叫“I2C模块”，淘宝上搜索“I2C模块”就出来好多。所谓I2C模块，是指通过I2C总线，在一个小板上挂接了好几个I2C设备的模块。


下面介绍其中的一种模块，在这里可以下载到本帖中所要用到的所有资料（包括库文件、源代码、芯片数据手册、配套教程）：http://pan.baidu.com/share/link?shareid=580350052&uk=3358736498
![1]({{ site.url }}/assets/images/2014-10-18/1.jpg)


记得在开始使用代码文件时，先把库文件放到Arduino IDE的“libraries”目录下，所有例程在Arduino 1.0.3下编译通过。

先上图，新入手的I2C模块：
![2]({{ site.url }}/assets/images/2014-10-18/2.jpg)
![3]({{ site.url }}/assets/images/2014-10-18/3.jpg)


这个I2C模块在I2C总线上挂接了三个IC，分别是实时时钟芯片（DS1307），EEPROM存储芯片（AT24C32），温度侦测芯片（LM75）。

一、先来说说实时时钟DS1307的使用！

DS1307是一款十分常用的实时时钟芯片，它可以记录年、月、日、时、分、秒等信息，提供至2100年的记录。可使用电池供电，也就是说，即使Arduino 在断电状态下，时钟芯片仍然是在运行的。它使用十分常用的两线式串行总线(I2C)，只要两根线即可和Arduino 通信。

接线图：
![4]({{ site.url }}/assets/images/2014-10-18/4.jpg)

运行Arduino IDE，打开以下文件：

![5]({{ site.url }}/assets/images/2014-10-18/5.jpg)

可以看到代码：


		#include <Wire.h>
		#include <RTClib.h>

		void printDateTime(DateTime dateTime);

		//创建实例
		RTC_DS1307 RTC;

		void setup (void){
		  Serial.begin(9600);
		  //初始化总线
		  Wire.begin();
		  //初始化实时时钟
		  RTC.begin();

		}

		void loop() {
		  if (Serial.available() > 0) {

		    int instruct = Serial.read();

		    switch (instruct) {
		    case 'D': {
		      //获取当前日期和时间
		      DateTime now = RTC.now();
		      //通过串口传送当前的日期和时间
		      printDateTime(now);
		      break;
		    } case 'S':
		      //设置成6月
		      RTC.set(RTC_MONTH, 6);
		      //设置成16点
		      RTC.set(RTC_HOUR, 16);
		      break;
		    }
		  }
		}

		void printDateTime(DateTime dateTime) {
		    //传送年份
		    Serial.print(dateTime.year(), DEC);
		    Serial.print('/');
		    //传送月份
		    Serial.print(dateTime.month(), DEC);
		    Serial.print('/');
		    //传送月份中的第几天
		    Serial.print(dateTime.day(), DEC);
		    Serial.print(' ');
		    //传送小时
		    Serial.print(dateTime.hour(), DEC);
		    Serial.print(':');
		    //传送分钟
		    Serial.print(dateTime.minute(), DEC);
		    Serial.print(':');
		    //传送秒
		    Serial.print(dateTime.second(), DEC);
		    Serial.println();
		}


编译并上传到Arduino，打开串口监控器，当输入字符“D”时，会返回当前的时间：
![6]({{ site.url }}/assets/images/2014-10-18/6.jpg)


而当输入字符“S”时，则会设置RTC为代码中指定的时间。输入“S”后再输入“D”，显示为代码中指定的时间了：
![7]({{ site.url }}/assets/images/2014-10-18/7.jpg)


下面我们来看一下代码：

````#include <Wire.h>````

Arduino 与实时时钟通讯是通过I2C总线，为了使用这个总线，必须告知编译器加入这个库。 #include 是一种预编译指令，它告知编译器，在编译代码时加入Wire.h 这个文件，而这个文件则包含了我们使用I2C总线的所有函数，就是所谓的库。而紧接着的 #include <RTClib.h> 也是一样的意思。

````Wire.begin();````

begin() 则是Wire实例的一个方法，作用是初始化芯片的I2C总线，这个初始化只要一次就行，即使你有多个I2C设备也是如此，所以我们把它放在setup()里面。紧接着的RTC.begin() 也是一样的意思。


````RTC_DS1307 RTC;````

这是本章比较难理解的地方，RTC_DS1307 是实时时钟库的一个类，而后面的RTC，则是创建一个RTC_DS1307类的实例，这个实例包括一些相关的函数和变量。例如adjust() 是RTC_DS1307的一个方法，你可以通过实例来调用它，即RTC.adjust()

````DateTime now =RTC.now();````

now() 是RTC_DS1307 的另一个方法，它返回一个DateTime实例，保存着当前的日期时间信息，运行这条语句之后我们可以通过now.month() 获知当前的月份， 通过now.minute() 获取当前的分钟数，如此类推year、day、hour、second。

````printDateTime(now);````

printDateTime() 是一个把当前时间通过串口传送回电脑的函数，它接收一个Datatime 类型的参数。

````RTC.set(RTC_MONTH, 6);````

这条语句的作用是设置实时时钟的月份为6月，set()是一个专门设置实时时钟的函数，它接收两个参数，一个是设置的类别，这里的RTC_MONTH 表示月份，相应的还有 RTC_YEAR、RTC_DAY、RTC_MINUTE、RTC_SECOND。第二个参数这是设置的值、这里的6表示6月。

````Serial.print(dateTime.year(),DEC);````

这条语句的意思是把月份的数值通过串口传送回电脑。 print 是Serial(串口)实例的一个方法，它接收两个参数，第一个是传送的值，第二个是传送的格式，DEC表示10进制，还有OCT 表示8进制，HEX 被表示16进制。

第一部分就到这里。
