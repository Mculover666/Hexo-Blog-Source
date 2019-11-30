---
title: 【STM32Cube_07】使用USART发送和接收数据（中断模式）
tags: STM32CubeMX
categories: STM32CubeMX
abbrlink: 1803605667
date: 2019-07-28 09:00:00
---

本篇文章主要介绍如何使用STM32CubeMX初始化STM32L431RCT6的USART，并使用**中断模式**发送和接收数据。
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

## NVIC配置
在NVIC中配置USART中断优先级：

![mark](http://mculover666.cn/image/20190816/Uw6jxzmblvJW.png?imageslim)

## 配置时钟树
STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：

![mark](http://mculover666.cn/image/20190806/1TQg7frjRpVr.png?imageslim)

![mark](http://mculover666.cn/image/20190814/AITGSflAXS45.png?imageslim)

## 生成工程设置

![mark](http://mculover666.cn/image/20190816/RUWz76PbSunq.png?imageslim)

## 代码生成设置

最后设置生成独立的初始化文件：

![mark](http://mculover666.cn/image/20190812/PwTCS6QzHiyG.png?imageslim)

## 生成代码

点击`GENERATE CODE`即可生成MDK-V5工程：

![mark](http://mculover666.cn/image/20190806/s0jGhLBWW6Cm.png?imageslim)

# 3. 在MDK中编写、编译、下载用户代码
## 定义发送和接收缓冲区
```c
/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */
uint8_t hello[] = "USART1 is ready...\n";
uint8_t recv_buf[13] = {0};
/* USER CODE END 0 */
```
## 重新实现中断回调函数
在NVIC一讲中我们探索了HAL库的中断处理机制，HAL中弱定义了一个中断回调函数 `HAL_UART_RxCpltCallback`， 我们需要在用户文件中重新定义该函数，放在哪都可以，这里我放在 `main.c` 中：
```c
/* USER CODE BEGIN 4 */
/* 中断回调函数 */
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
	/* 判断是哪个串口触发的中断 */
	if(huart ->Instance == USART1)
	{
		//将接收到的数据发送
		HAL_UART_Transmit_IT(huart, (uint8_t*)recv_buf, 13);
		//重新使能串口接收中断
		HAL_UART_Receive_IT(huart, (uint8_t*)recv_buf, 13);
	}
}
/* USER CODE END 4 */
```
## 修改main函数
在main函数中首先开启串口中断接收，然后发送提示信息：
```c
int main(void)
{
  HAL_Init();

  SystemClock_Config();

  MX_GPIO_Init();
  MX_USART1_UART_Init();

  /* USER CODE BEGIN 2 */
  //使能串口中断接收
  HAL_UART_Receive_IT(&huart1, (uint8_t*)recv_buf, 13);
  //发送提示信息
  HAL_UART_Transmit_IT(&huart1, (uint8_t*)hello, sizeof(hello));
  /* USER CODE END 2 */

  while (1)
  {
  }
}
```

## 编译代码

编译整个工程：

![mark](http://mculover666.cn/image/20190816/u6VALN7Rhqr0.png?imageslim)

## 设置下载器

![mark](http://mculover666.cn/image/20190812/PHve6DYPkO9M.png?imageslim)

![mark](http://mculover666.cn/image/20190812/djSNbMCj6Hh6.png?imageslim)

## 实验现象
下载运行后，实验现象如下：

![mark](http://mculover666.cn/image/20190816/RJHLbTakyimI.png?imageslim)

至此，我们已经学会了**如何配置USART使用中断模式发送和接收数据**，下一节将讨论实现printf()函数的多种方法。

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![mark](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)