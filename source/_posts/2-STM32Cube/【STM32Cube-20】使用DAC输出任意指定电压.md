---
title: 【STM32Cube_20】使用DAC输出任意指定电压值
tags: STM32CubeMX DAC
categories: STM32CubeMX
abbrlink: 2707806886
date: 2019-08-10 09:48:56
---

本篇详细的记录了如何使用STM32CubeMX配置STM32L431RCT6的DAC外设，输出任意指定电压值。
<!--more-->

# 1. 准备工作
## 硬件准备
- 开发板
首先需要准备一个开发板，这里我准备的是STM32L4的开发板（BearPi）：
![](http://mculover666.cn/image/20190806/9uiPTi5odYSj.png?imageslim)
- 万用表

## 软件准备
- 需要安装好Keil - MDK及芯片对应的包，以便编译和下载生成的代码；
- 准备一个串口调试助手，这里我使用的是`Serial Port Utility`；

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


## 配置DAC

### 确定DAC输出通道
查看小熊派E53接口的原理图：

![](http://mculover666.cn/blog/20191016/QlodtDUnaD4B.png?imageslim)

### 配置DAC
选择`DAC1`，开启输出通道2，配置保持默认即可：

![](http://mculover666.cn/blog/20191016/hSKFOGJtERVf.png?imageslim)

## 配置时钟树
STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：
![](http://mculover666.cn/image/20190806/1TQg7frjRpVr.png?imageslim)

## 生成工程设置
![](http://mculover666.cn/blog/20191016/G55e7qyHVG6Y.png?imageslim)

## 代码生成设置
最后设置生成独立的初始化文件：
![](http://mculover666.cn/image/20190806/T6WvSK6Dfpts.png?imageslim)

## 生成代码
点击`GENERATE CODE`即可生成MDK-V5工程：
![](http://mculover666.cn/image/20190806/s0jGhLBWW6Cm.png?imageslim)

# 3. 在MDK中编写、编译、下载用户代码
## 重定向printf( )函数
参考：[【STM32Cube】（八）基于串口发送函数实现printf()](https://blog.csdn.net/Mculover666/article/details/95975461)。


## 编写测试代码
首先设置DAC输出的数据为**12位右对齐**，然后指定输出的值0-4096，实际输出的电压为
`value/4096x3.3V`，最后使能DAC转换，代码如下：
```c
int main(void)
{
    /* USER CODE BEGIN 1 */
    uint16_t i = 0;
    /* USER CODE END 1 */

    HAL_Init();
    SystemClock_Config();
    MX_GPIO_Init();
    MX_DAC1_Init();
    MX_USART1_UART_Init();

    /* USER CODE BEGIN 2 */
    printf("DAC Test...\r\n");
    HAL_DAC_Start(&hdac1, DAC_CHANNEL_2);

    /* USER CODE END 2 */

    /* Infinite loop */
    /* USER CODE BEGIN WHILE */
    while (1)
    {
    /* USER CODE END WHILE */
        for(i = 0; i < 4096; i++)
        {
            HAL_DAC_SetValue(&hdac1, DAC_CHANNEL_2, DAC_ALIGN_12B_R, i);
            HAL_Delay(2);
        }
        printf("DAC test finish, test again!\r\n");

    /* USER CODE BEGIN 3 */
    }
    /* USER CODE END 3 */
}
```
![](http://mculover666.cn/blog/20191016/xMwqiE5p5Dxt.png?imageslim)

至此，我们已经学会**如何使用DAC输出任意指定电压值**。

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)

