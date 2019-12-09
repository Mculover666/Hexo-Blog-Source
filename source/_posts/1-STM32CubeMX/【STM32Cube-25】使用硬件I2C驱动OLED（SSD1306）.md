---
title: STM32CubeMX_25 | 使用硬件I2C驱动OLED(SSD1306)
keywords: STM32CubeMX OLED IIC
tags: STM32CubeMX
categories: STM32CubeMX
abbrlink: 390108705
date: 2019-12-09 10:04:56
---

本篇详细的记录了如何使用STM32CubeMX配置STM32L431RCT6的硬件I2C外设驱动0.96'OLED屏幕。
<!--more-->

# 1. 准备工作
## 硬件准备
- 开发板
首先需要准备一个开发板，这里我准备的是STM32L4的开发板（BearPi）：

![](http://mculover666.cn/image/20190806/9uiPTi5odYSj.png?imageslim)

- OLED屏幕
这里我使用的是0.96'的OLED屏幕，使用IIC接口通信，驱动芯片为SD1306：

![OLED屏幕](http://mculover666.cn/blog/20191209/PULv8RRTRqlt.png?imageslim)

## 软件准备
- 需要安装好Keil - MDK及芯片对应的包，以便编译和下载生成的代码；

>Keil MDK和串口助手Serial Port Utility 的安装包都可以**在文末关注公众号获取**，回复关键字获取相应的安装包：

![](http://mculover666.cn/image/20190814/gubaOwmETp1w.png?imageslim)

# 2.生成MDK工程
## 选择芯片型号
打开STM32CubeMX，打开MCU选择器：
![](http://mculover666.cn/image/20190806/gBP6glmUSH80.png?imageslim)

搜索并选中芯片`STM32L431RCT6`:
![](http://mculover666.cn/image/20190806/gnyHwdl53uVD.png?imageslim)

## 配置时钟源
- 如果选择使用外部高速时钟（HSE），则需要在System Core中配置RCC；
- 如果使用默认内部时钟（HSI），这一步可以略过；

这里我都使用外部时钟：
![](http://mculover666.cn/image/20190806/k593lGGb5tlW.png?imageslim)

## 配置串口
小熊派开发板板载ST-Link并且虚拟了一个串口，原理图如下：

![](http://mculover666.cn/image/20190814/IwyXONVefPx9.png?imageslim)

这里我将开关拨到`AT-MCU`模式，使PC的串口与USART1之间连接。

接下来开始配置`USART1`：

![](http://mculover666.cn/image/20190814/nLMRMYtmzghl.png?imageslim)

## 配置硬件I2C

在本实验中，我们将OLED接在小熊派开发板左边的E53扩展板接口上，与 I2C1 接口相连。

接下来开始配置I2C接口1：

![](http://mculover666.cn/image/20190826/obFsy5wJHDXz.png?imageslim)

## 配置时钟树
STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：

![](http://mculover666.cn/image/20190806/1TQg7frjRpVr.png?imageslim)

## 生成工程设置

![](http://mculover666.cn/blog/20191209/J7Atsprks4Qt.png?imageslim)

## 代码生成设置
最后设置生成独立的初始化文件：

![](http://mculover666.cn/image/20190806/T6WvSK6Dfpts.png?imageslim)

## 生成代码
点击`GENERATE CODE`即可生成MDK-V5工程：

![](http://mculover666.cn/image/20190806/s0jGhLBWW6Cm.png?imageslim)

# 3. 在MDK中编写、编译、下载用户代码
## 重定向printf()函数

参考：[STM32CubeMX-09 | 重定向printf函数到串口输出的多种方法](http://www.mculover666.cn/posts/2251182441/)

## OLED屏幕驱动程序

OLED屏幕驱动我已移植好，包含的文件较多，代码就不放在文中了，我已上传到[Github](https://github.com/Mculover666/HAL_Driver_Lib)：

- oledfont.h：OLED ASCII英文字符字库文件和中文字库文件
- bmp.h：图片库文件
- oled.h：OELD功能函数声明
- oled.c：OLED功能函数实现


# 4. 测试驱动程序

将驱动程序添加到你的工程中后，在main.c中测试驱动程序是否正常：

首先在main.c 开头包含头文件：

```c
/* USER CODE BEGIN Includes */
#include <stdio.h>
#include "oled.h"
#include "bmp.h"
/* USER CODE END Includes */
```

然后在main函数中编写测试程序：
```c
nt main(void)
{
    HAL_Init();
    SystemClock_Config();
    MX_GPIO_Init();
    MX_I2C1_Init();
    MX_USART1_UART_Init();

    /* USER CODE BEGIN 2 */
    printf("OLED 0.96' TEST...\r\n");
	OLED_Init();
    /* USER CODE END 2 */

    /* Infinite loop */
    /* USER CODE BEGIN WHILE */
    while (1)
    {
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
    OLED_Clear();
    OLED_ShowChar(0, 0, 'A', 16);
    OLED_ShowChar(0, 2, 'B', 16);
    OLED_ShowChar(0, 4, 'C', 16);
    OLED_ShowChar(0, 6, 'D', 16);

    OLED_ShowChar(15, 0, 'A', 12);
    OLED_ShowChar(15, 1, 'B', 12);
    OLED_ShowChar(15, 2, 'C', 12);
    OLED_ShowChar(15, 3, 'D', 12);
    OLED_ShowChar(15, 4, 'E', 12);
    OLED_ShowChar(15, 5, 'F', 12);
    OLED_ShowChar(15, 6, 'G', 12);
    OLED_ShowChar(15, 7, 'H', 12);

    OLED_ShowString(30, 0, "mculover666", 12);

    OLED_ShowCHinese(35, 2, 0);
    OLED_ShowCHinese(65, 2, 1);
    OLED_ShowCHinese(95, 2, 2);

    OLED_ShowString(36, 6, "IoT Board", 16);

    HAL_Delay(5000);
    OLED_DrawBMP(0, 0, 128, 8,BMP1);

    HAL_Delay(5000);
    }
    /* USER CODE END 3 */
}
```
编译下载运行，测试结果如下：

![OLED字符显示测试](http://mculover666.cn/blog/20191209/GenHBrWdhjWW.png?imageslim)

![OLED图片显示测试](http://mculover666.cn/blog/20191209/Ft0SI7xEEHuw.png?imageslim)

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![公众号](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)
