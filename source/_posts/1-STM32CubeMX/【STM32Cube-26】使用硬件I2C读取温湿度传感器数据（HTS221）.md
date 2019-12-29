---
title: STM32CubeMX-26 | 使用硬件I2C读取温湿度传感器数据（HTS221）
keywords: STM32CubeMX IIC HTS221
tags: STM32CubeMX 温湿度传感器 HTS221
categories: STM32CubeMX
abbrlink: 4097081462
date: 2019-12-29 10:29:03
---

本篇详细的记录了如何使用STM32CubeMX配置STM32L431RCT6的硬件I2C外设，读取HTS221温湿度传感器的数据并通过串口发送。
<!--more-->

# 1. 准备工作
## 硬件准备
- 开发板
首先需要准备一个开发板，这里我准备的是STM32L0的开发板（ST Nucleo-64），主控芯片是STM32L073RZ：

![ST Nucleo开发板](http://mculover666.cn/blog/20191223/PutQl2dff5JU.png?imageslim)

- HTS221温湿度传感器
HTS221温湿度传感器是ST公司生产的一款超小型温湿度传感器，提供 16-bit 的温度和湿度输出数据，并且数据输出提供了IIC 和 SPI两种通信接口，具有 2 x 2 x 0.9 mm 的极小封装：

![HTS221实物图](http://mculover666.cn/blog/20191229/pYva2s2VnULB.png?imageslim)

HTS221的原理图如下：

![HTS221原理图](http://mculover666.cn/blog/20191229/XwYSdGHnBS3i.png?imageslim)

## 软件准备
- 需要安装好Keil - MDK及芯片对应的包，以便编译和下载生成的代码；
- 准备一个串口调试助手，这里我使用的是`Serial Port Utility`；

>Keil MDK和串口助手Serial Port Utility 的安装包都可以**在文末关注公众号获取**，回复关键字获取相应的安装包：

![mark](http://mculover666.cn/image/20190814/gubaOwmETp1w.png?imageslim)

# 2.生成MDK工程
## 选择芯片型号
打开STM32CubeMX，打开MCU选择器：

![芯片选择器](http://mculover666.cn/image/20190806/gBP6glmUSH80.png?imageslim)

搜索并选中芯片`STM32L431RCT6`:

![选中实验芯片](http://mculover666.cn/blog/20191229/kEmUz7AmX6fA.png?imageslim)

## 配置时钟源

该开发板上没有板载外部晶振，所以**使用内部时钟（HSI）**，RCC 设置保持默认：

![时钟源配置](http://mculover666.cn/blog/20191223/zlFwDhdqz1ba.png?imageslim)

## 配置串口

ST-Nucleo 开发板板载ST-Link并且虚拟了一个串口，该串口与STM32芯片的USART2相连。

接下来开始配置`USART2`：

![串口配置](http://mculover666.cn/blog/20191223/dV2ABytb6s9U.png?imageslim)

## 配置I2C接口

查看ST-Nucleo扩展接口的原理图：

![扩展接口原理图](http://mculover666.cn/blog/20191223/GLqiXFliNfTS.png?imageslim)

接下来开始配置`I2C1`接口：

![配置I2C1接口](http://mculover666.cn/blog/20191223/2VcRVDu4kGOW.png?imageslim)

注意，I2C1接口默认的引脚是`PA9`和`PA10`，与实际板子连接**不相符**，所以手动切换到`PA8`和`PB9`:

![切换I2C1引脚](http://mculover666.cn/blog/20191223/tXoBIVULFMlX.png?imageslim)

## 配置时钟树

STM32L0的最高主频到32M，所以配置PLL，最后使`HCLK = 32Mhz`即可：

![时钟树配置](http://mculover666.cn/blog/20191223/03l2rjMcWkIA.png?imageslim)

## 生成工程设置

![生成工程设置](http://mculover666.cn/blog/20191223/Ga0a2tExr0H7.png?imageslim)

## 代码生成设置

最后设置生成独立的初始化文件：

![代码生成设置](http://mculover666.cn/blog/20191223/TlGTUVYMQvSs.png?imageslim)

## 生成代码

点击`GENERATE CODE`即可生成MDK-V5工程：

![生成代码](http://mculover666.cn/blog/20191223/G0F33OQfLgWc.png?imageslim)

# 3. 在MDK中编写、编译、下载用户代码
## 3.1. Printf重定向

在本实验中，温湿度传感器数据需要通过串口打印，所以需要配置printf重定向：

- 参考教程：[STM32CubeMX_09 | 重定向printf函数到串口输出的多种方法](http://www.mculover666.cn/posts/2251182441/)。

>注意：教程里将printf重定向到USAR1，本实验中ST-Link的虚拟串口与USART2相连，所以需要重定向到USART2！

![重定向到USART2](http://mculover666.cn/blog/20191223/piIQj0ufW0NB.png?imageslim)

## 3.2. 编写HTS221驱动

参考[HTS221数据手册.pdf]()进行编程。

HTS221的驱动我已上传到[Github](https://github.com/Mculover666/HAL_Driver_Lib/tree/master/HTS221)，包含两个文件：

- `HTS221.h`：器件地址宏定义、寄存器地址宏定义；
- `HTS221.c`：获取温度函数实现，获取湿度函数实现；

# 4. 测试驱动程序

将驱动程序添加到你的工程中后，在main.c中测试驱动程序是否正常：

首先在main.c 开头包含头文件：

```c
/* USER CODE BEGIN Includes */
#include <stdio.h>
#include "HTS221.h"
/* USER CODE END Includes */
```

然后在main函数中编写测试程序：

```c
int main(void)
{
  /* USER CODE BEGIN 1 */
	int16_t temperature;
	int16_t humidity;

  /* USER CODE END 1 */
  
	HAL_Init();
	SystemClock_Config();

	/* Initialize all configured peripherals */
	MX_GPIO_Init();
	MX_USART2_UART_Init();
	MX_I2C1_Init();

	/* USER CODE BEGIN 2 */
	printf("HTS221 Test...\r\n");
	HTS221_Init();
	/* USER CODE END 2 */

	/* Infinite loop */
	/* USER CODE BEGIN WHILE */
	while (1)
	{
	/* USER CODE END WHILE */

	/* USER CODE BEGIN 3 */
		HTS221_Get_Temperature(&temperature);
		printf("temperature:%2.1f", temperature / 10.0);
		HTS221_Get_Humidity(&humidity);
		printf("  humidity:%2.1f\n", humidity / 10.0);
		
		HAL_Delay(500);
		
	}
	/* USER CODE END 3 */
}
```

编译下载运行，测试结果如下：

![测试结果](http://mculover666.cn/blog/20191229/BQixPqWigw6r.png?imageslim)

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![公众号](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)
