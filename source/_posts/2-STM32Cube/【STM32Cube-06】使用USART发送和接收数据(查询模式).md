---
title: 【STM32Cube_06】使用USART发送和接收数据（查询模式）
tags: STM32CubeMX
categories: STM32CubeMX
abbrlink: 2064921339
date: 2019-07-27 09:00:00
---

本篇文章主要介绍如何使用STM32CubeMX初始化STM32L431RCT6的USART，并使用查询模式发送数据，使用查询模式接收数据。
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


## 配置时钟树
STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：

![mark](http://mculover666.cn/image/20190806/1TQg7frjRpVr.png?imageslim)

![mark](http://mculover666.cn/image/20190814/AITGSflAXS45.png?imageslim)

## 生成工程设置

![mark](http://mculover666.cn/image/20190814/4Bb6RzhdGGnH.png?imageslim)

## 代码生成设置

最后设置生成独立的初始化文件：

![mark](http://mculover666.cn/image/20190812/PwTCS6QzHiyG.png?imageslim)

## 生成代码

点击`GENERATE CODE`即可生成MDK-V5工程：

![mark](http://mculover666.cn/image/20190806/s0jGhLBWW6Cm.png?imageslim)

# 3. 在MDK中编写、编译、下载用户代码
## 编写查询模式发送和接收代码

编写 `main` 函数如下：
```c
int main(void)
{
  /* USER CODE BEGIN 1 */
	char str[12] = "Hello World\n";
	char recv_buf[12] = {0};
  /* USER CODE END 1 */

  HAL_Init();

  SystemClock_Config();

  MX_GPIO_Init();
  MX_USART1_UART_Init();

  /* USER CODE BEGIN 2 */
	HAL_UART_Transmit(&huart1, (uint8_t*)str, 12, 0xFFFF);
  /* USER CODE END 2 */

  while (1)
  {
    /* USER CODE END WHILE */
    /* USER CODE BEGIN 3 */
    //接收12个字节的数据，不超时
    if(HAL_OK == HAL_UART_Receive(&huart1, (uint8_t*)recv_buf, 12, 0xFFFF))
    {
      //将接收到的数据发送
      HAL_UART_Transmit(&huart1, (uint8_t*)recv_buf, 12, 0xFFFF);
    }
  }
  /* USER CODE END 3 */
}
```

## 编译代码

编译整个工程：

![mark](http://mculover666.cn/image/20190814/cQclXS2zTHaV.png?imageslim)

## 设置下载器

![mark](http://mculover666.cn/image/20190812/PHve6DYPkO9M.png?imageslim)

![mark](http://mculover666.cn/image/20190812/djSNbMCj6Hh6.png?imageslim)

## 实验现象
下载运行后，实验现象如下：

![mark](http://mculover666.cn/image/20190814/kQYbI27K2aQP.png?imageslim)

至此，我们已经学会了**如何配置USART使用查询模式发送和接收数据**，下一节将讲述如何配置USART使用中断模式接收数据。

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![mark](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)