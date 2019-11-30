---
title: 【STM32Cube_03】使用GPIO点亮一个LED
tags: STM32CubeMX
categories: STM32CubeMX
abbrlink: 2046075734
date: 2019-07-24 10:00:56
---

本篇文章主要介绍如何使用STM32CubeMX初始化STM32L431RCT6的GPIO，并点亮一个LED。
<!--more-->
# 1. 准备工作
## 硬件准备
- 开发板
首先需要准备一个开发板，这里我准备的是STM32L4的开发板（BearPi）：

![mark](http://mculover666.cn/image/20190806/9uiPTi5odYSj.png?imageslim)

## 软件准备
- 需要安装好Keil - MDK及芯片对应的包，以便编译和下载生成的代码；

>Keil MDK和串口助手Serial Port Utility 的安装包都可以**在文末关注公众号获取**，回复关键字获取相应的安装包：

![mark](http://mculover666.cn/image/20190814/gubaOwmETp1w.png?imageslim)

# 2.生成MDK工程
## 选择芯片型号
打开STM32CubeMX，打开MCU选择器：

![mark](http://mculover666.cn/image/20190806/gBP6glmUSH80.png?imageslim)

搜索并选中芯片`STM32L431RCT6`:

![mark](http://mculover666.cn/image/20190806/gnyHwdl53uVD.png?imageslim)

## 配置时钟源

- 如果选择使用外部高速时钟（HSE），则需要在System Core中配置RCC；
- 如果使用默认内部时钟（HSI），这一步可以略过；

这里我都使用外部时钟：

![mark](http://mculover666.cn/image/20190806/k593lGGb5tlW.png?imageslim)

## 配置GPIO引脚

查看小熊派开发板的原理图，如下：

![mark](http://mculover666.cn/image/20190812/5iCtQUfKbgzA.png?imageslim)

所以接下来我们选择配置`PC13`引脚：

![mark](http://mculover666.cn/image/20190812/Ad3UrGCsgjXr.png?imageslim)

## 配置时钟树
STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：

![mark](http://mculover666.cn/image/20190806/1TQg7frjRpVr.png?imageslim)

## 生成工程设置

![mark](http://mculover666.cn/image/20190812/JRur8DQQ4saC.png?imageslim)

## 代码生成设置

最后设置生成独立的初始化文件：

![mark](http://mculover666.cn/image/20190812/PwTCS6QzHiyG.png?imageslim)

## 生成代码

点击`GENERATE CODE`即可生成MDK-V5工程：

![mark](http://mculover666.cn/image/20190806/s0jGhLBWW6Cm.png?imageslim)

## 生成成功

![mark](http://mculover666.cn/image/20190812/Rut0y7ovQGsf.png?imageslim)

# 3. 在MDK中编写、编译、下载用户代码
## 编写用户代码
STM32CubeMX生成的代码目录如下：

![mark](http://mculover666.cn/image/20190812/yEi9PbChnxjU.png?imageslim)

进入`MDK-ARM`目录，打开工程：

![mark](http://mculover666.cn/image/20190812/wsGU4swrhD4b.png?imageslim)

在`main.c`中的main函数中编写简单的用户代码：
```c
  while (1)
  {
    /* USER CODE END WHILE */
    /* USER CODE BEGIN 3 */
    HAL_Delay(200);
	HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_13);
  }
```
## 编译代码

编译整个工程：

![mark](http://mculover666.cn/image/20190812/Rh3CJlu7Hx8o.png?imageslim)

## 设置下载器

![mark](http://mculover666.cn/image/20190812/PHve6DYPkO9M.png?imageslim)

![mark](http://mculover666.cn/image/20190812/djSNbMCj6Hh6.png?imageslim)

## 下载运行

![mark](http://mculover666.cn/image/20190812/lLANsFr5iOb7.png?imageslim)

## 实验现象

![mark](http://mculover666.cn/image/20190812/YCAOK10iYrQN.png?imageslim)

至此，我们已经学会了**如何使用STM32CubeMX快速生成MDK的工程**，点亮一个LED，接下来一节讲述如何使用 STM32CubeMX初始化GPIO进行按键检测。

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![mark](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)