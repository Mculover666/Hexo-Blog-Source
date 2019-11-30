---
title: 【STM32Cube_16】使用硬件CRC校验数据（以SHT30为例）
tags: STM32CubeMX 温湿度传感器 SHT30 CRC校验
categories: STM32CubeMX
abbrlink: 842429667
date: 2019-08-06 09:48:56
---

本篇详细的记录了如何使用STM32CubeMX配置STM32L431RCT6的硬件CRC外设校验数据，并用SHT30温湿度传感器为例检查是否可以正确校验。
<!--more-->

# 1. 准备工作
## 硬件准备
- 开发板
首先需要准备一个开发板，这里我准备的是STM32L4的开发板（BearPi）：

![mark](http://mculover666.cn/image/20190806/9uiPTi5odYSj.png?imageslim)

## 软件准备
- 需要安装好Keil - MDK及芯片对应的包，以便编译和下载生成的代码；
- 准备一个串口调试助手，这里我使用的是`Serial Port Utility`；

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

## 配置串口
小熊派开发板板载ST-Link并且虚拟了一个串口，原理图如下：

![mark](http://mculover666.cn/image/20190814/IwyXONVefPx9.png?imageslim)

这里我将开关拨到`AT-MCU`模式，使PC的串口与USART1之间连接。

接下来开始配置`USART1`：

![mark](http://mculover666.cn/image/20190814/nLMRMYtmzghl.png?imageslim)

## 配置CRC外设
首先激活CRC：
![mark](http://mculover666.cn/image/20190811/AdBDzKeEu9tX.png?imageslim)

然后配置CRC校验的初始值：

这里我们以SHT30为例，其数据手册中已给出，如图：

![mark](http://mculover666.cn/image/20190809/wtLIFxbSLyon.png?imageslim)

据此，CRC外设的配置如下：
![mark](http://mculover666.cn/image/20190811/GHPasXPYsCiX.png?imageslim)

## 配置时钟树
STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：
![mark](http://mculover666.cn/image/20190806/1TQg7frjRpVr.png?imageslim)

## 生成工程设置
![mark](http://mculover666.cn/image/20190811/8pAljFetp84X.png?imageslim)

## 代码生成设置
最后设置生成独立的初始化文件：
![mark](http://mculover666.cn/image/20190806/T6WvSK6Dfpts.png?imageslim)

## 生成代码
点击`GENERATE CODE`即可生成MDK-V5工程：
![mark](http://mculover666.cn/image/20190806/s0jGhLBWW6Cm.png?imageslim)

# 3. 在MDK中编写、编译、下载用户代码
## 重定向printf( )函数

参考：[【STM32Cube_09】重定向printf函数到串口输出的多种方法](https://www.mculover666.cn/2019/07/30/STM32Cube/【STM32Cube-09】重定向printf函数到串口输出的多种方法/)。


## 测试CRC校验
在`main.c`文件中添加如下代码:
```c
/* USER CODE BEGIN Includes */
#include <stdio.h>
/* USER CODE END Includes */
```
然后修改`main`函数：
```c
int main(void)
{
    /* USER CODE BEGIN 1 */
	uint8_t dat[2] = {0xBE, 0xEF};
	uint8_t crc = 0;

    /* USER CODE END 1 */
    HAL_Init();

    SystemClock_Config();

    MX_GPIO_Init();
    MX_CRC_Init();
    MX_USART1_UART_Init();
  
    /* USER CODE BEGIN 2 */
	printf("Test CRC check:\n");
	crc = HAL_CRC_Accumulate(&hcrc, (uint32_t*)dat, 2);
	printf("crc = %#x\n", crc);
    /* USER CODE END 2 */

    while (1)
    {
    }
}
```
## 测试结果
测试结果如下：

![mark](http://mculover666.cn/image/20190811/cmbdUIcWEkcQ.png?imageslim)

至此，我们已经学会**如何使用硬件CRC校验SHT30的数据**，下一节将讲述如何使用硬件SPI驱动LCD屏幕（ST7789）。

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![mark](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)