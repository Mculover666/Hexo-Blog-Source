---
title: STM32CubeMX | 27-系统滴答定时器Systick的使用
keywords: STM32CubeMX Systick
tags: STM32CubeMX Systick
categories: STM32CubeMX
abbrlink: 4283984198
date: 2019-12-29 12:29:03
---
本篇文章主要介绍如何使用STM32中的系统滴答定时器Systick。
<!--more-->
# 1. 准备工作
## 硬件准备
- 开发板
首先需要准备一个开发板，这里我准备的是STM32L4的开发板（BearPi）：

![](http://mculover666.cn/image/20190806/9uiPTi5odYSj.png?imageslim)

## 软件准备
- 需要安装好Keil - MDK及芯片对应的包，以便编译和下载生成的代码；

>Keil MDK和串口助手Serial Port Utility 的安装包都可以**在文末关注公众号获取**，点击资源获取菜单即可。

# 2.生成MDK工程

>如果使用的是STM32F1系列，请先看这篇文章！！！（[STM32CubeMX生成F1的工程中造成 下载器无法下载 问题的解决方案](https://blog.csdn.net/Mculover666/article/details/104802410)）

## 选择芯片型号
打开STM32CubeMX，打开MCU选择器：

![](http://mculover666.cn/image/20190806/gBP6glmUSH80.png?imageslim)

搜索并选中芯片`STM32L431RCT6`:

![](http://mculover666.cn/image/20190806/gnyHwdl53uVD.png?imageslim)

## 配置时钟源

- 如果选择使用外部高速时钟（HSE），则需要在System Core中配置RCC；
- 如果使用默认内部时钟（HSI），这一步可以略过；

这里我都使用外部时钟：

![](http://mculover666.cn/blog/20200320/xjfBJCiQiIqs.png?imageslim)

## 配置GPIO引脚

查看小熊派开发板的原理图，如下：

![](http://mculover666.cn/image/20190812/5iCtQUfKbgzA.png?imageslim)

所以接下来我们选择配置`PC13`引脚：

![](http://mculover666.cn/blog/20200320/75KToL1Wb9Hi.png?imageslim)

给PC13引脚设置一个user_label:

![](http://mculover666.cn/blog/20200320/RQ2wj7LvOpYB.png?imageslim)

## 系统滴答定时器Systick

SysTick 是一个24位的**向下计数定时器**，当计到0时，将从RELOAD寄存器中自动重装载定时初值并继续计数，且同时触发中断，SysTick 的主要作用是作为系统的时基，产生一个周期性的中断信号。

STM32CubeMX使用的是HAL库，默认已经开启，也可以选择其它的定时器作为系统时基：

![](http://mculover666.cn/blog/20200320/wPr86c2AAhrV.png?imageslim)

中断默认使能，无法关闭：

![](http://mculover666.cn/blog/20200320/gpq3D6yxcV0E.png?imageslim)

## 配置时钟树

STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：

![](http://mculover666.cn/image/20190806/1TQg7frjRpVr.png?imageslim)

需要注意，其中`To Cortex System timer`这一路是Systick的时钟频率，有`/1`和`/8`两种选择，这里我们使用8分频：

![](http://mculover666.cn/blog/20200320/B3hmj9mq9XFN.png?imageslim)

## 生成工程设置

![](http://mculover666.cn/blog/20200320/AomciGE6pPn3.png?imageslim)

## 代码生成设置

最后设置生成独立的初始化文件：

![](http://mculover666.cn/image/20190812/PwTCS6QzHiyG.png?imageslim)

## 生成代码

点击`GENERATE CODE`即可生成MDK-V5工程：

![](http://mculover666.cn/image/20190806/s0jGhLBWW6Cm.png?imageslim)

# 3. 在MDK中编写、编译、下载用户代码
## 自动生成的代码
STM32CubeMX会自动生成Systick相关的代码，其中比较重要的有：

① 默认Systick频率值设定：

![](http://mculover666.cn/blog/20200320/QQ0mkE2nXt6X.png?imageslim)

频率设定有三个值，在`stm32l4xx_hal.h`文件中：

![](http://mculover666.cn/blog/20200320/PVW77Adaevs9.png?imageslim)

② Systick默认中断服务函数

![](http://mculover666.cn/blog/20200320/NeiPiq1Qp6hD.png?imageslim)

HAL_IncTick函数会把当前系统中定义的计数值变量递加，在`stm32l4xx_hal.c`文件中，实现如下：

![](http://mculover666.cn/blog/20200320/GmxKupDBS7cU.png?imageslim)

## 编写用户代码

HAL库中还定义了一个函数 HAL_GetTick()，使用此API可以获取到当前系统中的计数值，定义如下：

![](http://mculover666.cn/blog/20200320/BUl5TwumCMxN.png?imageslim)

接下来使用此API来编写LED闪烁程序。

在main.c函数中首先定义一个32位的uint型变量，用于存放计数起始值：
```c
/* USER CODE BEGIN 1 */

uint32_t tickstart;

/* USER CODE END 1 */
```
然后在while(1)循环中调用 HAL_GetTick() 来获取当前的计数值，通过比较当前计数值和起始值即可起到延时的作用：
```c
while (1)
{
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */

    //获取延时起始值
    tickstart = HAL_GetTick();
    
    //死循环比较，直到到达延时值，每次1ms，1000次为1s
    while ((HAL_GetTick() - tickstart) < 1000);
    
    //翻转LED
    HAL_GPIO_TogglePin(LED_GPIO_Port, LED_Pin);
}
```
上面的这种应用方式比较准确，但系统进入了死循环，还有一种比较灵活的使用方式：
```c
while (1)
{
/* USER CODE END WHILE */

/* USER CODE BEGIN 3 */
    
    //翻转LED
    if(HAL_GetTick() % 1000 == 0)
    {
        HAL_GPIO_TogglePin(LED_GPIO_Port, LED_Pin);
    }
}
```
这样就不会堵塞CPU了。

## 实验现象
编译下载之后，即可看到LED按1s的间隔闪烁：

![](http://mculover666.cn/image/20190812/YCAOK10iYrQN.png?imageslim)

## 补充 —— HAL_Delay的实现原理

最开始使用HAL库的时候，觉得 HAL_Delay 简直太方便了，其实 HAL_Delay 也是依靠系统的时基信号来实现的，在`stm32l4xx_hal.c`文件中，实现如下：

![](http://mculover666.cn/blog/20200320/GA45wsv1k8Wa.png?imageslim)

可以看到它采用的还是死等的方式，所以相比起来，还是第二种方式应用起来更加灵活，方便，喜欢文章的话，点个赞鼓励一下吧，笔芯~

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)