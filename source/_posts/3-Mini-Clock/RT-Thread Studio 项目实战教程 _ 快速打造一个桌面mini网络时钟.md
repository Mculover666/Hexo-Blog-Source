---
title: 桌面mini网络时钟教程
keywords: RT-Thread
tags: 制作过程
categories: 制作过程
abbrlink: 2151341594
top: true
summary: 使用RT-Thread Studio打造
date: 2020-02-20 18:05:56
---

<iframe src="//player.bilibili.com/player.html?aid=90635321&cid=154783725&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>

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
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020020217203582.png)
项目所用的芯片型号如下：

- 主控芯片：STM32L431RCT6
- 温湿度传感器：	SHT30
- 通信模组：ESP8266（WIFI）
- 显示模组：0.96'OLED（SSD1306）

其中，SHT30传感器挂载到STM32的I2C1引脚上，OLED挂载到STM32的I2C3引脚上，两个设备均使用模拟I2C总线通信，ESP8266与STM32之间采用串口发送AT指令通信。

项目开源地址：[https://github.com/Mculover666/Mini-Clock-RT-thread](https://github.com/Mculover666/Mini-Clock-RT-thread)。

# 3. 项目开发教程
> - [使用RT-Thread Studio DIY 迷你桌面时钟（一）| 基于STM32芯片创建工程](https://blog.csdn.net/Mculover666/article/details/104146623)
>- [使用RT-Thread Studio DIY 迷你桌面时钟（二）| 获取温湿度传感器数据（I2C设备驱动+SHT3x软件包）](https://mculover666.blog.csdn.net/article/details/104153715)
>- [使用RT-Thread Studio DIY 迷你桌面时钟（三）| 获取NTP时间（at_device软件包 + netutils软件包）](https://mculover666.blog.csdn.net/article/details/104418075)
> - [使用RT-Thread Studio DIY 迷你桌面时钟（四）| OLED显示时钟和温湿度（cpp组件 + u8g2软件包）](https://mculover666.blog.csdn.net/article/details/104422501)
> - [使用RT-Thread Studio DIY 迷你桌面时钟（五）| 使用内置 Git 插件管理项目](https://mculover666.blog.csdn.net/article/details/104427445)

# 4. 基于RT-Thread Studio的项目开发总结
这次使用 RT-Thread Studio开发这个小项目的过程，用两个字来概括就是：舒服！

- **舒服点① - 高度集成化的开发体验**

RT-Thread Studio支持STM32全系列芯片，只需要建立工程时选择型号即可，创建之后直接编译、下载一条龙服务，RT-Thread就跑起来了，才不管用的什么板子呢~

它使用的**编译器**是arm-none-eabi-gcc系列编译器，和MDK所使用的ARMCC编译器相比，效率上不敢比，水平太菜，但是从编译时间上，快很多。

它使用的**下载器**是基于STM32CubeProg的，支持J-Link和ST-Link，下载速度就一个字：快！

它还内置了**串口终端putty**，不用再去新开串口终端软件，直接在一个窗口中就可以搞定，多爽：
![](https://img-blog.csdnimg.cn/20200221152942463.png)

- **舒服点② - 编码体验**

在编码过程中，首先是主题，可以切换为各种常用IDE的暗黑系风格，比如下面这种：
![](https://img-blog.csdnimg.cn/20200221151647111.png)
其次就是自动提示功能，输入一个字母之后就会有提示，而且带有参数说明功能，在快速编码的时候也是很方便；

最后还有一个非常重要的功能——跳转到定义，在MDK中编译时不开这个功能（都懂得），导致跳转查看源码时非常不方便，而在RT-Thread Studio中，鼠标悬停就可以快速查看定义：
![](https://img-blog.csdnimg.cn/20200221151958749.png)
如果想跳转过去，直接右键查看声明即可。

**- 舒服点③ 软件包中心的支持**

如果在功能开发的时候，有非常丰富的软件包可用，使得自己的编码量大幅降低，比如本项目中使用的SHT3x软件包、AT_Device软件包、U8G2软件包，项目开发过程中很顺手，经过很多项目验证的开源代码，为什么不用呢？

![](https://img-blog.csdnimg.cn/20200221152329899.png)

- **舒服点④ - 图形化配置**

讲真，这个界面好看不？点一下就能开启/关闭组件，非常方便：
![](https://img-blog.csdnimg.cn/20200221152422241.png)
具体的配置项也是图形化的，一目了然：
![](https://img-blog.csdnimg.cn/2020022115251326.png)

- **舒服点⑤ - 内置Git插件**

对于项目开发来说，最必须的就是使用Git来管理本地库和远程库，可以进行项目版本管理，可以进行团队协作，但是比较头疼的是git需要使用命令行来操作，而RT-Thread Studio中内置了git插件，一切操作都可以可视化进行，点点鼠标即可，实为方便：

![](https://img-blog.csdnimg.cn/20200221152819780.png)
项目中常用的这些功能都有了，非常齐全，但是还有一个需要特别说明的地方。

使用RT-Thread Studio开发项目，我觉得需要建立在对RT-Thread这个操作系统有一定的了解基础之上，会使用MDK+ENV的方式进行开发，了解内核、组件、软件包的基本使用，有了这些基础，使用该软件开发项目时才会觉得**得心应手**，否则，高度集成化会导致你觉得这一切来得太容易了，项目做的云里雾里，出问题了也没法定位，这样的行为是非常不可取的。

好啦！对于本次项目开发体验就总结到这里，其实按照整套教程做出来，花费不到半个小时的时间，听我在这儿吹爆，还不如下载动手试试！

<font color="red">**接收更多精彩文章及资源推送，欢迎订阅我的微信公众号：『mculover666』。**</font>
![](https://img-blog.csdnimg.cn/20200202092055136.png)

