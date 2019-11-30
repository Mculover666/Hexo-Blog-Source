---
title: TencentOS-tiny上手体验
tags: TencentOS-tiny RTOS
categories: TencentOS-tiny
img: 'http://mculover666.cn/blog/20190919/putrWsdu2xu1.png?imageslim'
summary: TencentOS-tiny正式开源，赶快上车体验吧~
cover: true
coverImg: 'http://mculover666.cn/blog/20190919/putrWsdu2xu1.png?imageslim'
abbrlink: 1219336465
date: 2019-09-02 11:06:52
---

![](http://mculover666.cn/blog/20190919/putrWsdu2xu1.png?imageslim)

<!--more-->

# 1. TencentOS-tiny 正式开源

国产 RTOS 如雨后春笋般诞生的今天，腾讯于昨日正式开源发布了自己的物联网操作系统：`TencentOS-tiny`，来看看官方怎么说：

>[TencentOS tiny](https://cloud.tencent.com/product/tos-tiny)是腾讯面向物联网领域开发的实时操作系统，具有低功耗，低资源占用，模块化，安全可靠等特点，可有效提升物联网终端产品开发效率。TencentOS tiny 提供精简的 RTOS 内核，内核组件可裁剪可配置，可快速移植到多种主流 MCU (如STM32全系列)及模组芯片上。而且，基于RTOS内核提供了丰富的物联网组件，内部集成主流物联网协议栈（如 CoAP/MQTT/TLS/DTLS/LoRaWAN/NB-IoT 等），可助力物联网终端设备及业务快速接入腾讯云物联网平台。

作为一个码农，我要这堆balabala的文字有何用？？？

Talk is cheap, Show me the code. 

![](http://mculover666.cn/blog/20190919/TdChnAuiyO5A.png?imageslim)

放上Github，用代码说话，开干！

- [TencentOS-tiny官网](https://cloud.tencent.com/product/tos-tiny)
- [TencentOS-tiny源码仓库](https://github.com/Tencent/TencentOS-tiny)

# 2. 文件目录架构概览

`TencentOS-tiny`的整个文件目录如图，嗯，是我熟悉的风格：

![](http://mculover666.cn/blog/20190919/9EEgIry86rDq.png?imageslim)

这个文件目录组织架构，普普通通，给个中肯的评价吧。

不过其中有几个特点倒是值得一提：

## board文件夹

这个文件夹是 TencentOS-tiny 适配的开发板集合，这点做的非常好，**开发者在移植完之后可以提交PR合并上去，避免后续开发者再进行重复的移植工作，到手就可以用**，目前的情况还是和可观的，如图：

![](http://mculover666.cn/blog/20190919/l3WAJQqRLuGK.png?imageslim)

## device文件夹

在这个文件夹中，可以看到 TencentOS-tiny 支持的通信模组设备还是很丰富的，**覆盖了常用的 NB-iot 模组、WIFI模组、2G/4G模组、lora模组**：

![](http://mculover666.cn/blog/20190919/rll05cEGGYkP.png?imageslim)

## TencentCloud_SDK 上云组件

在 `components/connectivity` 这个文件夹中，可以看到 TencentCloud_SDK 上云组件，毫无疑问，腾讯自家的OS，肯定对自家的云平台支持性最好：

![](http://mculover666.cn/blog/20190919/VuNFowp0B0TF.png?imageslim)

# 3. 眼见为虚，上手为实

这里我使用的是小熊派开发板，和官网EVK板是亲兄弟（斜眼笑.jpg），主控是STM32L431RCT6，刚好TencentOS-tiny/board中有移植好的，直接拿来用哈哈哈，先体验一下这个操作系统：

![](http://mculover666.cn/blog/20190919/zHx5xMYUmSW1.png?imageslim)

进入`board/TencentOS_tiny_EVB_MX`:

![](http://mculover666.cn/blog/20190919/Q9rydxUNACTa.png?imageslim)

## OS配置文件

进入后即可看到 `TencnetOS-tiny` 系统的配置文件 `tos_config.h`，同样，该OS使用宏定义开关来配置需要的模块，比如内核中的信号量、事件集、队列等等：

![](http://mculover666.cn/blog/20190919/YrTDtt3Wsb1H.png?imageslim)

对于系统中一些重要的参数，也在宏定义中配置：

![](http://mculover666.cn/blog/20190919/nLEfcmsMQ9C9.png?imageslim)

## BSP板级支持包

对于开发板上的硬件，**TencentOS-tiny并没有提供设备驱动框架，所以直接使用STM32CubeMX + HAL 库来操作板上硬件**：

![](http://mculover666.cn/blog/20190919/MulYdEj2B5Wr.png?imageslim)

## Keil工程 —— HelloWorld

这里我使用的是 `Keil MDK 5.28`，所以进入 Keil 文件夹，**进入后 TencentOS-tiny 提供了针对该开发板的很多示例工程，这点是我没有想到的，点个赞**：

![](http://mculover666.cn/blog/20190919/WA4CAxIuVJS8.png?imageslim)

刚接触到 TencentOS-tiny，先来个 HelloWorld，直接上云的话，步子有点大，容易扯到...进入 `hello_world`文件夹打开工程。

# 4. 初探 TencentOS-tiny
在上一步打开工程后，开始探索一下这个新系统~

## 工程架构

在MDK中整个工程架构还是很清晰的，如图：

![](http://mculover666.cn/blog/20190919/KXqEKDV6K6fq.png?imageslim)

## TencentOS-tiny内核启动流程

这一点要值得赞赏，内核启动非常简洁，就是一个main函数：

![](http://mculover666.cn/blog/20190919/PWYRuVFCvfQ4.png?imageslim)

## 板级初始化

先来看启动流程第一步：板级初始化。

该函数在`mcu_init.c`文件中，因为我使用的板子是小熊派开发板，没有DHT11和OLED，只想串口打印HelloWorld，所以将需要的代码都屏蔽了：

![](http://mculover666.cn/blog/20190919/qRnRCaBjDiwv.png?imageslim)

## printf重定向到串口

在 main 函数中，OS启动时有这样一句代码：
```c
printf("Welcome to TencentOS tiny\r\n");
```
那么，TencentOS-tiny是如何重定向printf函数呢？

答案还是在`mcu_init.c`：

![](http://mculover666.cn/blog/20190919/Ov5lnAikqDJ6.png?imageslim)

如果想知道为什么实现这三个函数就可以将printf重定向到串口，可以参考我的这篇博客：[【STM32Cube-09】重定向printf函数到串口输出的多种方法](https://www.mculover666.cn/2019/07/30/2-STM32Cube/【STM32Cube-09】重定向printf函数到串口输出的多种方法/)。

在代码中可以看到，printf函数被重定向到了**串口2**，因为这是其他开发板的支持包，不能更改实现代码，所以**只能再找一个USB转串口，将小熊派的UART2连接到电脑上**，如图：

![](http://mculover666.cn/blog/20190919/M7kyqMmO7xmF.png?imageslim)

## HelloWorld示例

探索完OS是如何启动的之后，再来探索一下HelloWorld线程是如何创建的，打开`hello_world.c`文件可以看到创建线程的代码非常简洁：

首先进行线程相关宏定义：

![](http://mculover666.cn/blog/20190919/0QctK9AeeG6b.png?imageslim)

然后是线程主体函数：

![](http://mculover666.cn/blog/20190919/yYobR5and3et.png?imageslim)

最后是创建线程的函数：

![](http://mculover666.cn/blog/20190919/qhwIHfphX6ug.png?imageslim)

那么，创建线程的函数如何执行呢？

别急，答案就在 `main.c` 中：

![](http://mculover666.cn/blog/20190919/Tph5ENaglP2v.png?imageslim)


所以，**当在`hello_world.c`中定义了函数`application_entry`时，main.c中弱定义的函数失效，系统就会执行用户定义的函数**。

## 编译代码

接下来编译代码，编译之后的信息如下：

![](http://mculover666.cn/blog/20190919/C7eJvwuG9OFQ.png?imageslim)

在map文件中看看：

![](http://mculover666.cn/blog/20190919/edX6Mw18jIJB.png?imageslim)

我的天~这代码写的也太简洁了吧~好评！！！

## 下载运行

下载代码到开发板上，可以在串口助手中看到系统正常运行并打印结果：

![](http://mculover666.cn/blog/20190919/xMknpAa4EMCc.png?imageslim)

至此，对 TencentOS-tiny 这个新的操作系统有所了解了吧哈哈，后续再来带大家玩玩上云，不然都对不起生它的腾讯爸爸~

# 5. 体验心得

总的来说，TencentOS-tiny给我的感觉还是很好的：

- 和云厂家的物联网操作系统相比（比如菊厂的LiteOS）
**TOS显然做的很有优势**，在board中适配了主流的一些开发板，极大的方便了开发者，并且在代码风格方面和裸机开发没太大的区别，代码量上也做的非常小。

- 和RT-Thread相比：
这个，，，没法比，拿轿车和卡车比，不在一个级别上。

最后给点建议，对于还没有深入学习RTOS的小伙伴，建议学习RT-Thread，生态非常完善，如果已经学习过RTOS，那么TencentOS-tiny非常值得一试，相对于LiteOS，AliOS，非常简洁，玩的很爽，而且是腾讯家的东西，后期要开发微信小程序也是非常方便的，都是自己的东西。

最后，给TencentOS-tiny点个赞~如果有想要加入TencentOS tiny技术交流群的小伙伴，可以加官方人员微信：

![](http://mculover666.cn/blog/20190919/jpQMiKlprwFp.png?imageslim)

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)

