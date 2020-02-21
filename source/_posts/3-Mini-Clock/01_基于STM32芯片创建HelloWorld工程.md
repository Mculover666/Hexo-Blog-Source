---
title: 桌面mini网络时钟教程 | 01-基于STM32芯片创建HelloWorld工程
keywords: RT-Thread
tags: 制作过程
categories: 制作过程
abbrlink: 2303339885
summary: 使用RT-Thread Studio打造
date: 2020-02-20 18:00:56
---

# 1. RT-Thread Studio
RT-Thread Studio 是**一站式的 RT-Thread 开发工具**，通过简单易用的图形化配置系统以及丰富的软件包和组件资源，让物联网开发变得简单和高效。
![](https://img-blog.csdnimg.cn/20200202170325301.png#pic_center)
RT-Thread主要包括工程创建和管理，代码编辑，SDK管理，RT-Thread配置，构建配置，调试配置，程序下载和调试等功能，结合图形化配置系统以及软件包和组件资源，减少重复工作，提高开发效率。

![](https://img-blog.csdnimg.cn/20200202170406175.png)

RT-Thread STudio了可以从[RT-Thread官网](https://www.rt-thread.org/page/download.html#studio)下载，下载之后一路next安装即可，注意安装路径不要有中文或者空格。
![](https://img-blog.csdnimg.cn/20200203110427759.png)

# 2. 桌面mini时钟项目
迷你桌面时钟项目基于小熊派IoT开发板，使用RT-Thread物联网操作系统，使用RT-Thread Studio一站式开发工具，在极短的时间内开发完成一个桌面mini时钟。

整个项目的架构如下：
![](https://img-blog.csdnimg.cn/2020020217203582.png)
项目所用的芯片型号如下：

- 主控芯片：STM32L431RCT6
- 温湿度传感器：	SHT30
- 通信模组：ESP8266（WIFI）
- 显示模组：0.96'OLED（SSD1306）

其中，SHT30传感器挂载到STM32的I2C1引脚上，OLED挂载到STM32的I2C3引脚上，两个设备均使用模拟I2C总线通信，ESP8266与STM32之间采用串口发送AT指令通信。

# 3. 基于芯片创建项目
① 打开RT-Thread Studio，点击左上角新建项目，新建一个RT-Thread项目：
![](https://img-blog.csdnimg.cn/20200202172950506.png)

② 填写项目信息，基于芯片创建项目：

>**RT-Thead支持全系列STM32芯片，所以可以使用任何STM32开发板！**

![](https://img-blog.csdnimg.cn/20200202173448981.png)
③ 点击“小锤子”编译整个工程：
![](https://img-blog.csdnimg.cn/20200202173803445.png)
所有编译信息都会在控制台输出：
![](https://img-blog.csdnimg.cn/20200202173912355.png)

④ 点击按钮，在Putty创建一个串口终端：
![](https://img-blog.csdnimg.cn/20200202174335624.png)
填写串口信息，打开串口：
![](https://img-blog.csdnimg.cn/20200220204939424.png)
打开之后的串口终端如图所示：
![](https://img-blog.csdnimg.cn/20200220205025561.png)

⑤ 点击按钮下载工程：
![](https://img-blog.csdnimg.cn/20200202174444835.png)
所有下载信息都在控制台输出：
![](https://img-blog.csdnimg.cn/20200202174525706.png)
⑥ 下载完毕自动复位运行，在串口终端查看输出：
![](https://img-blog.csdnimg.cn/20200220205213254.png)

<font color="red">**一行代码都不用写，RT-Thread就在板子上跑起来了，爽不爽！**</font>

# 4. 让LED闪烁起来
RT-Thread Studio默认生成的main./c中已经包含了LED闪烁的程序，我们只需要修改引脚即可。

小熊派IoT开发板板载一个LED，默认连接在PC13引脚上，修改这行代码设置引脚：
![](https://img-blog.csdnimg.cn/20200202190419592.png)
再次编译，下载，可以看到板载蓝色LED开始闪烁。

# 5. 使用外部晶振时钟
使用RT-Thread Studio**基于芯片创建的工程全部使用内部时钟HSI**，要使用外部晶振时钟HSE，在`board.c`中修改`SystemClock_Config`函数，这里我修改如下：
![](https://img-blog.csdnimg.cn/20200202201426822.png)

再次编译下载，可以看到现象和之前使用HSI时候的现象一致。

<font color="red">**接收更多精彩文章及资源推送，欢迎订阅我的微信公众号：『mculover666』。**</font>
![](https://img-blog.csdnimg.cn/20200202092055136.png)









