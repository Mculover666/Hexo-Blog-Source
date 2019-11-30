---
title: 【STM32Cube_04】使用GPIO进行按键检测
tags: STM32CubeMX
categories: STM32CubeMX
abbrlink: 1763774108
date: 2019-07-25 10:00:56
---

本篇文章主要介绍如何使用STM32CubeMX初始化STM32L431RCT6的GPIO，并扫描检测按键。
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

## 配置LED的GPIO引脚

查看小熊派开发板的原理图，如下：

![mark](http://mculover666.cn/image/20190812/5iCtQUfKbgzA.png?imageslim)

所以接下来我们选择配置`PC13`引脚：

![mark](http://mculover666.cn/image/20190812/Ad3UrGCsgjXr.png?imageslim)

设置用户标签为LED：

![mark](http://mculover666.cn/image/20190813/ClKDFmVJYceI.png?imageslim)

## 配置按键的GPIO引脚
查看小熊派开发板的原理图，如下：

![mark](http://mculover666.cn/image/20190813/QNbG5i6QKGk3.png?imageslim)

所以接下来我们选择配置`PB2`引脚和`PB3`引脚：

![mark](http://mculover666.cn/image/20190813/3fRF4dWxJ6iw.png?imageslim)

因为没有设置硬件上拉，所以我们配置开启上拉电阻，并设置用户标签为`KEY1`和`KEY2`：

![mark](http://mculover666.cn/image/20190813/sjhFpsdWkQSB.png?imageslim)

## 配置时钟树
STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：

![mark](http://mculover666.cn/image/20190806/1TQg7frjRpVr.png?imageslim)

## 生成工程设置

![mark](http://mculover666.cn/image/20190813/4cGrWhVnjEdp.png?imageslim)

## 代码生成设置

最后设置生成独立的初始化文件：

![mark](http://mculover666.cn/image/20190812/PwTCS6QzHiyG.png?imageslim)

## 生成代码

点击`GENERATE CODE`即可生成MDK-V5工程：

![mark](http://mculover666.cn/image/20190806/s0jGhLBWW6Cm.png?imageslim)

# 3. 在MDK中编写、编译、下载用户代码
## 编写用户代码
进入`MDK-ARM`目录，打开工程，在`main.c`中的main函数中编写简单的用户代码：
```c
int main(void)
{

  HAL_Init();

  SystemClock_Config();

  MX_GPIO_Init();

  while (1)
  {
    /* USER CODE BEGIN 3 */
		if(0 == HAL_GPIO_ReadPin(KEY1_GPIO_Port, KEY1_Pin))
		{
			HAL_GPIO_WritePin(LED_GPIO_Port,LED_Pin,GPIO_PIN_SET);
		}
		if(0 == HAL_GPIO_ReadPin(KEY2_GPIO_Port, KEY2_Pin))
		{
			HAL_GPIO_WritePin(LED_GPIO_Port,LED_Pin,GPIO_PIN_RESET);
		}
  }
  /* USER CODE END 3 */
}
```
## 编译代码

编译整个工程：

![mark](http://mculover666.cn/image/20190813/T9TxAoP6pyiI.png?imageslim)

## 设置下载器

![mark](http://mculover666.cn/image/20190812/PHve6DYPkO9M.png?imageslim)

![mark](http://mculover666.cn/image/20190812/djSNbMCj6Hh6.png?imageslim)

## 实验现象
下载运行后，实验现象如下：

- 上电复位时LED处于熄灭状态；
- 按下KEY1，LED点亮；
- 按下KEY2，LED熄灭；

![mark](http://mculover666.cn/image/20190813/jrREJBcJukl3.png?imageslim)

至此，我们已经学会了**如何使用STM32CubeMX快速生成MDK的工程**，以及**如何使用 STM32CubeMX初始化GPIO进行按键检测**，下一节讲述如何配置NVIC使用外部中断检测按键。

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![mark](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)