---
title: 【STM32Cube_08】使用USART发送和接收数据（DMA模式）
tags: STM32CubeMX
categories: STM32CubeMX
abbrlink: 1606619423
date: 2019-07-29 09:00:00
---

本篇文章主要介绍如何使用STM32CubeMX初始化STM32L431RCT6的USART，并使用**DMA模式**发送数据和接收数据。
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

## USART DMA配置

>知识小卡片 —— DMA

DMA 全称 `Direct Memory Access`(直接存储器访问)， 是STM32的一个外设，它的特点在于：

在**不占用CPU的情况下**将数据从存储器直接搬运到外设，或者从外设直接搬运到存储器，当然也可以从存储器直接搬运到存储器。

比如在需要串口发送大量数据的时候，CPU只需要**发起DMA传输请求**，然后就可以去做别的事情了，DMA会将数据传输到串口发送，**DMA传输完之后会触发中断**，CPU如果有需要，可以对该中断进行处理，这样一来CPU的效率是不是大大提高了？

在STM32L431RCT6中有 2 个 DMA 外设：DMA1 和 DMA2，每个DMA外设有 7 个通道，每个通道都是独立的，配置DMA的时候有几个关键点：

- 数据从哪里来？
- 数据到哪里去？
- 有多少数据？

>知识小卡片结束啦~对STM32的DMA外设有没有了解呢？

接下来我们配置DMA，将存储器（SRAM）中的数据直接搬运到串口外设去发送：

![mark](http://mculover666.cn/image/20190816/mJTMlVPgHSCJ.png?imageslim)

## 配置时钟树
STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：

![mark](http://mculover666.cn/image/20190806/1TQg7frjRpVr.png?imageslim)

## 生成工程设置

![mark](http://mculover666.cn/image/20190816/RUWz76PbSunq.png?imageslim)

## 代码生成设置

最后设置生成独立的初始化文件：

![mark](http://mculover666.cn/image/20190812/PwTCS6QzHiyG.png?imageslim)

## 生成代码

点击`GENERATE CODE`即可生成MDK-V5工程：

![mark](http://mculover666.cn/image/20190806/s0jGhLBWW6Cm.png?imageslim)

# 3. 在MDK中编写、编译、下载用户代码
## 定义发送数据区域
```c
/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */
uint8_t dat[] = "Hello, I am Mculover666.\n";
/* USER CODE END 0 */
```
## 在main函数中发起DMA传输
```c
int main(void)
{
  HAL_Init();

  SystemClock_Config();

  MX_GPIO_Init();
  MX_USART1_UART_Init();

  /* USER CODE BEGIN 2 */
  HAL_UART_Transmit_DMA(&huart1, (uint8_t*)dat, sizeof(dat));
  /* USER CODE END 2 */

  while (1)
  {
  }
}
```
## 实验现象
编译下载运行后，实验现象如下：

![mark](http://mculover666.cn/image/20190816/pNmCADptVjqr.png?imageslim)


# 4. 使用DMA接收串口数据
## 说明
- 使用HAL库的时候不能同时使用DMA发送和接收数据，会出错。
- 所有的步骤和发送时一样，这里我只给出需要修改的部分。

## 修改串口DMA配置

![mark](http://mculover666.cn/image/20190818/CqyMnF113eWT.png?imageslim)

## 添加串口接收缓冲区
```c
/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */
uint8_t dat[] = "Hello, I am Mculover666.\n";
uint8_t recv_buf[13] = {0};		//串口接收缓冲区
/* USER CODE END 0 */
```
## 修改main函数
```c
int main(void)
{
  HAL_Init();

  SystemClock_Config();

  MX_GPIO_Init();
  MX_DMA_Init();
  MX_USART1_UART_Init();

  /* USER CODE BEGIN 2 */
  HAL_UART_Transmit(&huart1, (uint8_t*)dat, sizeof(dat), 0xFFFF);
  HAL_UART_Receive_DMA(&huart1, recv_buf, 13);  //使能DMA接收
  /* USER CODE END 2 */

  while (1)
  {
  }
}
```
## 添加串口接收中断回调函数
```c
/* USER CODE BEGIN 4 */
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart) 
{ 
	//将接收到的数据再发送
	HAL_UART_Transmit(&huart1,recv_buf,13, 0xFFFF);
}
/* USER CODE END 4 */
```
## 实验现象

![mark](http://mculover666.cn/image/20190818/lgWCp78edRWz.png?imageslim)

至此，我们已经学会了**如何配置USART使用DMA模式发送数据和接收数据**，下一节将讨论实现printf()函数的多种方法。

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![mark](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)