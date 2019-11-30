---
title: 【STM32Cube_11】使用通用定时器闪烁LED
tags: STM32CubeMX
categories: STM32CubeMX
abbrlink: 1598873035
date: 2019-08-01 10:48:56
---

本篇详细的记录了如何使用STM32CubeMX配置STM32L431RCT6的通用定时器外设，以中断的方式使LED闪烁。
<!--more-->

# 1. 准备工作
## 硬件准备
- 开发板
首先需要准备一个开发板，这里我准备的是STM32L4的开发板（BearPi）：

![mark](http://mculover666.cn/image/20190806/9uiPTi5odYSj.png?imageslim)

- 测试LED

这里我直接使用板载LED，原理图如下：
![mark](http://mculover666.cn/image/20190807/mvxaRos96773.png?imageslim)

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

## 配置LED的GPIO引脚
查看小熊派开发板的原理图，如下：

![mark](http://mculover666.cn/image/20190812/5iCtQUfKbgzA.png?imageslim)

所以接下来我们选择配置`PC13`引脚：

![mark](http://mculover666.cn/image/20190812/Ad3UrGCsgjXr.png?imageslim)

## 配置通用定时器TIM2
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

### 配置定时器TIM2
首先选择`TIM2`，时钟源选择内部时钟：
![mark](http://mculover666.cn/image/20190807/NltQViLTCKmj.png?imageslim)

接下来是对TIM2的参数设置，参照数据手册中的RCC时钟树，TIM2内部时钟来源是`PCLK1 = 80Mhz`，我们的目的是每秒钟产生2次中断，所以预分频系数设置为`40000-1`，自动重载值为`1000-1`，得到的计时器更新中断频率即为`80000000/40000/1000=2Hz`：
![mark](http://mculover666.cn/image/20190807/FsrFLXtFVJBc.png?imageslim)

其余的一些设置保持默认即可，最后开启TIM2中断：
![mark](http://mculover666.cn/image/20190807/q8syGvcE19c0.png?imageslim)

## 配置时钟树
STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：
![mark](http://mculover666.cn/image/20190806/1TQg7frjRpVr.png?imageslim)

## 生成工程设置
![mark](http://mculover666.cn/image/20190807/RmgpLb30TEyD.png?imageslim)

## 代码生成设置
最后设置生成独立的初始化文件：
![mark](http://mculover666.cn/image/20190806/T6WvSK6Dfpts.png?imageslim)

## 生成代码
点击`GENERATE CODE`即可生成MDK-V5工程：
![mark](http://mculover666.cn/image/20190806/s0jGhLBWW6Cm.png?imageslim)

# 3. 在MDK中编写、编译、下载用户代码
## 编写中断回调函数
在`stm32l4xx_it.c`中生成的中断处理函数如下，定时器TIM2所有的中断都会调用该中断服务函数`TIM2_IRQHandler`：

![mark](http://mculover666.cn/image/20190807/j4GAcl4onlAR.png?imageslim)

在中断处理函数中自动生成了`HAL_TIM_IRQHandler(&htim2)`代码，该代码会自动根据中断事件回调相应的函数，这里我们需要处理**更新中断的事件**，回调函数默认是`__weak`定义的，所以在`tim.c`中重新定义该回调函数，并且在该函数中添加功能的时候，因为该回调函数会被所有的定时器共用，所以需要先判断是哪个定时器在调用：
```c
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef* tim_baseHandle)
{
	if(tim_baseHandle->Instance == htim2.Instance)
		HAL_GPIO_TogglePin(LED_GPIO_Port, LED_Pin);
}
```
## 启动定时器并使能中断
最后在`main`函数中开启TIM2并使能其中断（TIM2初始化代码之后，while之前）：
```c
HAL_TIM_Base_Start_IT(&htim2);
```

## 测试结果
编译下载后即可看到LED以 2 Hz的频率闪烁。

至此，我们已经学会**如何使用通用定时器闪烁LED**，下一节将讲述如何使用通用定时器产生PWM驱动蜂鸣器。

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![mark](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)