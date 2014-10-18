---
layout: post
title: "Raspberry Pi 透過 I²C 讀取 eeprom "
description: "Raspberry Pi 透過 I²C 讀取 eeprom"
category: [raspberrypi]
tags: [raspberrypi, i2c, eeprom]
---


[[Raspberry Pi] 透過 I²C 讀取 eeprom](http://coldnew.github.io/blog/2013/06/19_e5bcf.html)
Raspberry Pi 對於剛接觸嵌入式系統開發的人而言，無疑是一個非常好的管道，除了購買開發板僅需要約 NT $1350 以外，更重要的是，他保留了 SPI 與 I²C 這一類的常用通訊接口。

本篇文章將講述如何使用 Raspberry Pi 進行讀/寫 EEPROM，以及 i2c-tool 的基本使用方式。

使用設備

要完成本篇文章所描述的部份，你需要以下幾種器材

	1. Raspberry Pi
	2. EEPROM 24c02
	3. 麵包板
	4. 單蕊線

硬體線路

下面的硬體線路使用 Fritzing 軟體來繪製，連線到 EEPROM 的線路很簡單，只要將 I²C 所需要使用的線路連接就好，其中 EEPROM 的 A0~A2 為位址線，這邊將其全部接地，因此此設備在 I²C 上的位址為 0x50 。(下載設計檔案)

麵包板連線

	rsp_24c04_bb.png

電路連接

	rsp_24c04_schem.png

讓 Raspberry Pi 可以讀取 i2c 設備

1. 將 i2c 模組從黑名單中移除
雖然我不清楚為什麼 i2c 模組並不會預設被 Raspberry Pi 載入，但是如果沒有將這個模組從黑名單中移除的話，你是無法使用 modeprobe 這個命令載入 i2c 模組的，移除的方式如下，首先編輯

	/etc/modprobe.d/raspi-blacklist.conf

將裏面的資訊變成如下

	# blacklist spi and i2c by default (many users don't need them)

	blacklist spi-bcm2708
	# blacklist i2c-bcm2708

完成後先進行重新啟動

2. 載入 i2c 模組

	Raspberry Pi 預設沒有載入 i2c 模組，因此我們必須手動載入他

	 pi@raspberrypi:/home/pi$
	 sudo modprobe i2c-

如果你覺得每次都要手動載入很麻煩，可以修改 /etc/modules，將 i2c-dev 加入到檔案裏面，這樣重開之後，Raspberry Pi 會自動載入該載入的模組。

載入好模組後，你會看到 /dev 下面多增加了 i2c-0 以及 i2c-1 兩個設備節點

	pi@raspberrypi:/home/pi$
	 ls /dev/i2c*
	/dev/i2c-0  /dev/i2c-1

安裝 i2c-tools

我們在這邊使用最常用的 i2c-tools，因為這個套件並沒有被預先安裝，因此你必須自己安裝

pi@raspberrypi:/home/pi$
 sudo apt-get install i2c-tools
安裝完後，你會增加以下幾個命令

		i2cdetect  i2cdump    i2cget     i2cset

這些命令的用途如下:

i2cdetect – 用來列舉 I2C bus 和上面所有的裝置
i2cdump – 顯示裝置上所有暫存器 (register) 數值
i2cget – 讀取裝置上某個暫存器值
i2cset – 修改裝置上的暫存器數值
使用 i2cdetect 察看目前有多少個 i2c bus

你可以使用以下命令來察看目前的系統有多少個 i2c bus，以我手上的 Raspberry Pi 為例

		pi@raspberrypi:/home/pi$
		 sudo i2cdetect -l

會得到

	i2c-0   i2c             bcm2708_i2c.0                           I2C adapter
	i2c-1   i2c             bcm2708_i2c.1                           I2C adapter

在 rev.1 版本的 Raspberry Pi 上，i2c bus 是使用 i2c-0，而在現在販售的 rev.2 版本，則都改成使用 i2c-1 作為 i2c bus。

使用 i2cdetect 察看目前掛在 i2c bus 上的設備

知道你要查詢的 I²C bus 後，我們可以使用

	pi@raspberrypi:/home/pi$
	 sudo si2cdetect -y 1

來查詢 i2c-1 bus 上的所有設備，所得到的結果如下

		root@raspberrypi:/home/pi#
		 i2cdetect -y 1
		    0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
		00:          -- -- -- -- -- -- -- -- -- -- -- -- --
		10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
		20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
		30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
		40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
		50: 50 -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
		60: -- -- -- -- -- -- -- -- UU -- -- -- -- -- -- --
		70: -- -- -- -- -- -- -- --

這樣代表共有兩個裝置掛在 i2c-1 上，其中標示為 UU 的代表該設備有被偵測到並正在被 kernel driver 使用著，而在這邊顯示 0x50 的就是我們所使用的 EEPROM。

使用 i2cdump 查詢設備內所有暫存器

我們現在知道 EEPROM 是掛在 i2c-1 上的 0x50，若想知道 EEPROM 裏面的資訊，則可以使用 i2cdump 來獲得，i2cdump 的使用方式如下

		Usage: i2cdump [-f] [-y] [-r first-last] I2CBUS ADDRESS [MODE [BANK [BANKREG]]]
		I2CBUS is an integer or an I2C bus name
		ADDRESS is an integer (0x03 - 0x77)
		MODE is one of:
		  b (byte, default)
		  w (word)
		  W (word on even register addresses)
		  s (SMBus block)
		  i (I2C block)
		  c (consecutive byte)
		  Append p for SMBus PEC

因此我們取得 i2c-1 上的 0x50 資訊，就使用

i2cdump -y 1 0x50
你會得到

		root@raspberrypi:/home/pi#
		 i2cdump -y 1 0x50
		No size specified (using byte-data access)
		     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f    0123456789abcdef
		00: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		10: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		20: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		30: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		40: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		50: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		60: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		70: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		80: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		90: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		a0: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		b0: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		c0: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		d0: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		e0: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		f0: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................

這邊 EEPROM 內的資訊都是 0xFF ，這是出廠時的預設狀況，我們可以使用 i2cset 來修改他的數值。

使用 i2cset 修改設備暫存器數值

如果我們想修改 EEPROM 裏面的數值，那要怎麼辦呢？這時候可以使用 i2cset 來幫忙完成，i2cset 的使用方式如下

Usage: i2cset [-f] [-y] [-m MASK] I2CBUS CHIP-ADDRESS DATA-ADDRESS [VALUE] ... [MODE]
  I2CBUS is an integer or an I2C bus name
  ADDRESS is an integer (0x03 - 0x77)
  MODE is one of:
    c (byte, no value)
    b (byte data, default)
    w (word data)
    i (I2C block data)
    s (SMBus block data)
    Append p for SMBus PEC
假如我們想要修改位於 i2c-1 上 0x50 的 0x12 暫存器，並將其數值修改為 5，我們命令就可以這樣下

i2cset -f -y 1 0x50 0x12 5
再一次使用 i2cdump，你會發現不再是清一色的 0xFF 了

		root@raspberrypi:/home/pi#
		 i2cdump -y 1 0x50
		No size specified (using byte-data access)
		     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f    0123456789abcdef
		00: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		10: ff ff 05 ff ff ff ff ff ff ff ff ff ff ff ff ff    ..?.............
		20: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		30: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		40: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		50: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		60: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		70: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		80: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		90: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		a0: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		b0: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		c0: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		d0: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		e0: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................
		f0: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff    ................

使用 i2cget 來取得暫存器的數值

有些時候我們只想要看某個暫存器位址，這時候使用 i2cget 是最快的選擇， i2cget 命令格式如下

		Usage: i2cget [-f] [-y] I2CBUS CHIP-ADDRESS [DATA-ADDRESS [MODE]]
		I2CBUS is an integer or an I2C bus name
		ADDRESS is an integer (0x03 - 0x77)
		MODE is one of:
		  b (read byte data, default)
		  w (read word data)
		  c (write byte/read byte)
		  Append p for SMBus PEC

因此，若我們要察看剛剛所設定的 0x12 暫存器，則可以用以下方式得到該暫存器的數值

root@raspberrypi:/home/pi#
 i2cget  -y 1 0x50 0x12
0x05
