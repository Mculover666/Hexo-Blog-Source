---
title: STM32CubeMX_24 | 使用通用定时器产生PWM驱动舵机
keywords: STM32CubeMX 舵机 PWM
tags: STM32CubeMX
categories: STM32CubeMX
abbrlink: 933841213
date: 2019-12-05 20:04:56
---

本篇详细的记录了如何使用STM32CubeMX配置STM32L431RCT6的通用定时器外设，产生PWM驱动舵机。

<!--more-->

# 1. 准备工作
## 硬件准备
- 开发板
首先需要准备一个开发板，这里我准备的是STM32L4的开发板（BearPi）：

![小熊派IoT开发套件](http://mculover666.cn/image/20190806/9uiPTi5odYSj.png?imageslim)

- 舵机

这里我使用常见的 SG90 舵机：

![9g舵机](http://mculover666.cn/blog/20191205/FHWF7Azy542U.png?imageslim)

>知识小卡片 —— 舵机

舵机是电机的一种，又叫伺服电机，舵机的优势是**可以设定转到指定的位置**，本文中使用的SG90型号的舵机可以在0°-180°的范围内转动到指定角度，在实际项目中使用非常广泛。

在硬件上，SG90 舵机有三根线，红色的为电源线（5V），棕色的为 GND ，橙色的为控制线，用来传输 PWM 信号。

那么，应该产生怎样的PMW波形来控制舵机的转动角度呢？

SG90的舵机要求**控制舵机的 PWM 信号频率在50Hz左右**，即周期为 20ms 的 PWM 信号，**当该信号的高电平部分在0.5ms - 2.5ms之间时，对应舵机转动的角度**，具体对应情况如下表：

|高电平脉宽|舵机转动角度|
|:---:|:---:|
|0.5ms|0°|
|1.0ms|45°|
|1.5ms|90°|
|2.0ms|135°|
|2.5ms|180°|

下面结合一个动图来理解：

![图片来源八色木](http://mculover666.cn/blog/20191205/iKyLJIGWewhP.gif)

>知识小卡片结束啦！对舵机有了解了吗？

## 软件准备
- 需要安装好Keil - MDK及芯片对应的包，以便编译和下载生成的代码；

>Keil MDK和串口助手Serial Port Utility 的安装包都可以**在文末关注公众号获取**，回复关键字获取相应的安装包：

![](http://mculover666.cn/image/20190814/gubaOwmETp1w.png?imageslim)

# 2.生成MDK工程
## 选择芯片型号
打开STM32CubeMX，打开MCU选择器：
![打开MCU选择器](http://mculover666.cn/image/20190806/gBP6glmUSH80.png?imageslim)

搜索并选中芯片`STM32L431RCT6`:
![选择芯片](http://mculover666.cn/image/20190806/gnyHwdl53uVD.png?imageslim)

## 配置时钟源
- 如果选择使用外部高速时钟（HSE），则需要在System Core中配置RCC；
- 如果使用默认内部时钟（HSI），这一步可以略过；

这里我都使用外部时钟：

![打开外部时钟](http://mculover666.cn/blog/20191205/krcW6G9qAj0A.png?imageslim)

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

![打开TIM16并选择PWM输出引脚](http://mculover666.cn/image/20190807/3Ru6wXY95H7s.png?imageslim)


接下来是对TIM16的参数设置，参照数据手册中的RCC时钟树，TIM16内部时钟来源是`PCLK2 = 80Mhz`，我们的目的是产生`20Hz`的PWM，所以预分频系数设置为`80-1`，自动重载值为`20000-1`，得到的计时器更新中断频率即为`80000000/80/20000 = 50 Hz`：

![设置PWM输出频率](http://mculover666.cn/blog/20191205/bukWGxaWdSG6.png?imageslim)

其余的一些设置保持默认即可，最后配置PWM占空比：

![设置PWM占空比](http://mculover666.cn/blog/20191205/JMXATy2qaXlx.png?imageslim)

## 配置时钟树
STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：

![设置时钟树](http://mculover666.cn/image/20190806/1TQg7frjRpVr.png?imageslim)

## 生成工程设置

![工程设置](http://mculover666.cn/blog/20191205/VsuAG8EIkAog.png?imageslim)

## 代码生成设置
最后设置生成独立的初始化文件：

![代码生成设置](http://mculover666.cn/image/20190806/T6WvSK6Dfpts.png?imageslim)

## 生成代码
点击`GENERATE CODE`即可生成MDK-V5工程：

![生成代码](http://mculover666.cn/image/20190806/s0jGhLBWW6Cm.png?imageslim)

# 3. 在MDK中编写、编译、下载用户代码
## 启动定时器并产生PWM
最后在`main`函数中开启TIM2并使能其中断（TIM2初始化代码之后）：
```c
int main(void)
{
  /* USER CODE BEGIN 1 */

  /* USER CODE END 1 */
  
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();
  MX_TIM16_Init();

  /* USER CODE BEGIN 2 */

  HAL_TIM_PWM_Start(&htim16,TIM_CHANNEL_1);

  /* USER CODE END 2 */

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* USER CODE END WHILE */
    /* USER CODE BEGIN 3 */
  }
  /* USER CODE END 3 */
}
```

编译下载之后，可以看到舵机旋转到45°：

![舵机转动45°现象](http://mculover666.cn/blog/20191205/96mLb22EUSrL.png?imageslim)

## 动态改变舵机角度

上一个实验中，我们配置了PWM波的高电平时长计数个数为1000，即时长为1ms，对应旋转角度为45°，在本实验中，我们来动态改变 PWM 占空比，使舵机在0°到180之间来回旋转。

编写如下代码：

```c
int main(void)
{
  /* USER CODE BEGIN 1 */

	uint16_t pluse = 500;

  /* USER CODE END 1 */
  
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();
  MX_TIM16_Init();

  /* USER CODE BEGIN 2 */

  /* USER CODE END 2 */

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
    //产生PWM，舵机转动
    HAL_TIM_PWM_Start(&htim16,TIM_CHANNEL_1);

    //1s后改变舵机角度，增加45°
    HAL_Delay(1000);
    pluse += 500;
    if(pluse == 3000)
    {
      //如果舵机角度大于180°，回零
      pluse = 500;
    }

    //设置PWM占空比
    __HAL_TIM_SetCompare(&htim16, TIM_CHANNEL_1, (uint16_t)pluse);
  }
  /* USER CODE END 3 */
}
```

>注意：STM32F1系列会报错找不到__HAL_TIM_SetCompare函数，解决方案啊：[STM32CubeMX生成F1的工程中提示找不到 __HAL_TIM_SetCompare 问题的解决方案](https://blog.csdn.net/Mculover666/article/details/104801386)。

编译下载后可以看到舵机在0°-180°之间来回旋转：

![舵机角度动态调整效果](http://mculover666.cn/blog/20191205/ImBhKMzpWqKa.gif)

至此，我们已经学会**如何使用通用定时器产生PWM驱动舵机**。

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![公众号Mculover666](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)