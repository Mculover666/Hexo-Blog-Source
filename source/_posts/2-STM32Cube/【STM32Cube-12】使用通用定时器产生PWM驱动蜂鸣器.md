---
title: 【STM32Cube_12】使用通用定时器产生PWM驱动蜂鸣器
tags: STM32CubeMX
categories: STM32CubeMX
abbrlink: 650884631
date: 2019-08-02 15:04:56
---

本篇详细的记录了如何使用STM32CubeMX配置STM32L431RCT6的通用定时器外设，产生PWM驱动无源蜂鸣器。
<!--more-->

# 1. 准备工作
## 硬件准备
- 开发板
首先需要准备一个开发板，这里我准备的是STM32L4的开发板（BearPi）：

![mark](http://mculover666.cn/image/20190806/9uiPTi5odYSj.png?imageslim)

- 蜂鸣器

这里我直接使用扩展板上的蜂鸣器，如图：

![mark](http://mculover666.cn/image/20190807/egsTj4DhhwM8.png?imageslim)

蜂鸣器的原理图如下：

![mark](http://mculover666.cn/image/20190807/kjUo6ctmGGXS.png?imageslim)

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

## 配置通用定时器TIM16

>知识小卡片——STM32L431的定时器

STM32L431xx 系列有 1 个高级定时器（TIM1）, 3 个通用定时器（TIM2、TIM15、TIM16），两个基本定时器（TIM6、TIM7），还有两个低功耗定时器（LPTIM1、LPTIM2）。

STM32L431 的通用 TIMx (TIM2、TIM15、TIM16)定时器功能包括：

- 16 位(TIM15,TIM16)/32 位(TIM2)向上、向下、向上/向下自动装载计数器，注意：
TIM15、TIM16 只支持向上（递增）计数方式；
- 16 位可编程(可以实时修改)预分频器，计数器时钟频率的分频系数为 1～65535 之间的任
意数值。
- 4 个独立通道（TIMx_CH1~4， 其中 TIM15 最多 2 个通道， TIM16 最多 1 个
通道），这些通道可以用来作为：
  - 输入捕获
  - 输出比较
  - PWM 生成(边缘或中间对齐模式)
  - 单脉冲模式输出

- 可使用外部信号控制定时器和定时器互连的同步电路。
- 如下事件发生时产生中断/DMA：
  - 更新：计数器向上溢出/向下溢出，计数器初始化(通过软件或者内部/外部触发)
  - 触发事件(计数器启动、停止、初始化或者由内部/外部触发计数)
  - 输入捕获
  - 输出比较

>知识小卡片结束啦~

接下来开始配置TIM16定时器的PWM功能：

首先选择`TIM`，选择通道1的功能，默认的CH1是`PA6`引脚，但是开发板上是与 PB8 连接的，所以在右边将PB8配置为`TIM16_CH1`：
![mark](http://mculover666.cn/image/20190807/3Ru6wXY95H7s.png?imageslim)


接下来是对TIM16的参数设置，参照数据手册中的RCC时钟树，TIM16内部时钟来源是`PCLK2 = 80Mhz`，我们的目的是产生`1khz`的PWM，所以预分频系数设置为`80-1`，自动重载值为`1000-1`，得到的计时器更新中断频率即为`80000000/80/1000 = 1000 Hz = 1K Hz`：
![mark](http://mculover666.cn/image/20190807/FsrFLXtFVJBc.png?imageslim)

其余的一些设置保持默认即可，最后配置PWM占空比：
![mark](http://mculover666.cn/image/20190807/kVj0GfgtTFw9.png?imageslim)

## 配置时钟树
STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：
![mark](http://mculover666.cn/image/20190806/1TQg7frjRpVr.png?imageslim)

## 生成工程设置
![mark](http://mculover666.cn/image/20190807/UibRvxrbe4JC.png?imageslim)

## 代码生成设置
最后设置生成独立的初始化文件：
![mark](http://mculover666.cn/image/20190806/T6WvSK6Dfpts.png?imageslim)

## 生成代码
点击`GENERATE CODE`即可生成MDK-V5工程：
![mark](http://mculover666.cn/image/20190806/s0jGhLBWW6Cm.png?imageslim)

# 3. 在MDK中编写、编译、下载用户代码
## 启动定时器并产生PWM
最后在`main`函数中开启TIM2并使能其中断（TIM2初始化代码之后）：
```c
while (1)
{
  HAL_TIM_PWM_Start(&htim16,TIM_CHANNEL_1);
  HAL_Delay(1000);
  HAL_TIM_PWM_Stop(&htim16,TIM_CHANNEL_1);
  HAL_Delay(1000);
}
```

## 测试结果
编译下载后即可听到无源蜂鸣器开始工作。

至此，我们已经学会**如何使用通用定时器产生PWM驱动蜂鸣器**，下一节将讲述如何使用硬件IIC接口读写EEPROM。

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![mark](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)