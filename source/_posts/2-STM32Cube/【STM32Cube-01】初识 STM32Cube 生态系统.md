---
title: 【STM32Cube_01】初识 STM32Cube 生态系统
tags: STM32CubeMX
categories: STM32CubeMX
abbrlink: 1350058916
date: 2019-07-22 16:48:56
---

本篇文章主要介绍STM32Cube生态系统。
<!--more-->

# STM32Cube Ecosystem
[STM32Cube](https://www.st.com/content/st_com/en/stm32cube-ecosystem.html)是ST公司开发的一套生态系统，致力于使STM32的开发变的更简单，并且100%开源免费。

在开始介绍之前，先放上两段ST官方的视频，作以欣赏了解：

- [STM32Cube生态系统宣传片](https://www.bilibili.com/video/av52092742)
- [STM32Cube产品概览 - 使STM32开发更简单](https://www.bilibili.com/video/av58474599/)

STM32Cube生态系统包括两大部分：

- PC软件工具：STM32CubeMX、STM32CubeIDE、STM32CubeProgrammer、STM32CubeMnitor等
- 软件库：STM32 Embedded Software bricks 

![mark](http://mculover666.cn/image/20190810/U1EMmSu41E2Y.png?imageslim)

## STM32Cube PC Tools

- [STM32Cube MX](https://www.st.com/en/development-tools/stm32cubemx.html)：**适用于任何STM32设备的配置工具**
该工具用Java编写，所以可以在Windows、Linux、Mac上运行，它可以使用户通过图形用户界面对微控制器进行配置，然后为Cortex-M内核生成初始化C代码，或者为Cortex-A内核生成Linux设备树源（下面两张图对STM32CubeMX的作用作以诠释）：

![mark](http://mculover666.cn/image/20190810/VnUNz6EJlc6F.png?imageslim)

![mark](http://mculover666.cn/image/20190810/GsSG2OKAfAPt.png?imageslim)

- [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html)：**集成开发环境**
该工具是基于Eclipse+GNU C/C++工具链的，除了基本的编辑和编译功能，还包括代码编译报告功能和高级调试功能，另外，该IDE还集成了CubeMX。

![mark](http://mculover666.cn/image/20190810/DO71YWc0Rpio.png?imageslim)

- [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html)：**编程工具（给编程指给单片机烧录程序）**
该工具通过各种可用的通信方式（比如`JTAG，SWD，UART，USB DFU，I2C，SPI，CAN`等），提供了易于使用且高效的环境，用于读取，写入和验证存储器。

![mark](http://mculover666.cn/image/20190810/SnlnTMBgOkpg.png?imageslim)

- `STM32CubeMnitor`：**强大的监控工具**
帮助开发人员实时调试和监控应用程序的行为和性能。


这四个工具伴随着整个STM32的开发流程：

![mark](http://mculover666.cn/image/20190810/90UpnSazgV9e.png?imageslim)

## STM32 Embedded Software
 STM32 Embedded Softwares是STM32Cube提供的软件包，包括两大部分：

- STM32Cube MCU Packages
- STM32Cube Expansion
 
### STM32Cube MCU Packages
STM32Cube MCU Packages是STM32Cube提供的对于每个MCU产品的软件包，其中包括：

 - 底层库代码
 - 中间件代码
 - 用户代码

#### 底层库代码
STM32Cube提供的HAL库或者LL库，**覆盖STM32全系列**，包括：

![mark](http://mculover666.cn/image/20190810/zIeIjybfYev2.png?imageslim)

#### 中间件代码
STM32Cube提供的中间件代码非常丰富，包括：

![mark](http://mculover666.cn/image/20190810/Obq52NRryhXo.png?imageslim)

#### 用户代码
STM32Cube提供初步写好的用户代码，开发者可以在此基础上开发各种应用：

![mark](http://mculover666.cn/image/20190810/TnMsuJf7jvu0.png?imageslim)

截止2019年2月，STM32Cube软件包对STM32全系列产品的支持情况如下表：

![mark](http://mculover666.cn/image/20190810/G4zgELzVrWTW.png?imageslim)

### STM32Cube Expansion
[STM32Cube扩展包](https://www.st.com/en/embedded-software/stm32cube-expansion-packages.html#overview)**补充了STM32Cube MCU Packages的功能**，目前已有的软件扩展包有：

- 用于云连接的即用型扩展包（Amazon AWS，Microsoft Azure，IBW Watson等）
- LoRa
- 蜂窝连接
- NFC
- 工业通信协议
- 加密库
- 传感器驱动程序
- 电机控制算法
- 安全自测库
- ……


至此，对STM32Cube生态系统的介绍完毕，下一节讲述如何获取STM32Cube生态系统中的PC tools和Embeded Software。

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![mark](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)