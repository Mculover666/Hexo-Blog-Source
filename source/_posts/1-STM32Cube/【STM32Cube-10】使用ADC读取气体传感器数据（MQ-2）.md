---
title: 【STM32Cube_10】使用ADC读取气体传感器数据（MQ-2）
tags: STM32CubeMX 气体传感器MQ-2
categories: STM32CubeMX
abbrlink: 1249993360
date: 2019-07-31 16:48:56
---

本篇详细的记录了如何使用STM32CubeMX配置STM32L431RCT6的ADC外设，读取MQ-2气体传感器的数据并通过串口发送。
<!--more-->

# 1. 准备工作
## 硬件准备
- 开发板
首先需要准备一个开发板，这里我准备的是STM32L4的开发板（BearPi）：
![mark](http://mculover666.cn/image/20190806/9uiPTi5odYSj.png?imageslim)
- MQ-2模块
MQ-2气体传感器一般用于家庭和工厂的气体泄漏监测装置，适用于液化气、丁烷、丙烷、甲烷、酒精、氢气、烟雾等的探测，如图：
![mark](http://mculover666.cn/image/20190806/DYWw34Cxn6Jl.png?imageslim)

MQ-2的原理图如下：
![mark](http://mculover666.cn/image/20190806/39fP6PTeK7OD.png?imageslim)

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


## 配置ADC

>知识小卡片 —— ADC

ADC全称 Analog-to-Digital Converter，即模拟-数字转换器，可以将连续变化的模拟信号转换为离散的数字信号，进而使用数字电路进行处理，称之为数字信号处理。

STM32L431xx 系列有 1 个 ADC，ADC 分辨率高达 12 位，每个 ADC 具有多达 20 个的采集
通道，这些通道的 A/D 转换可以单次、连续、扫描或间断模式执行。 ADC 的结果可以左对齐
或右对齐方式存储在 16 位数据寄存器中。

STM32L431 的 ADC 最大的转换速率为 5.33Mhz，也就是转换时间为 0.188us（12 位分辨率
时），ADC 的转换时间与 AHB 总线时钟频率无关。

>知识小卡片结束啦~对ADC有没有了解呢？

### 确定ADC通道
查看小熊派E53接口的原理图：
![mark](http://mculover666.cn/image/20190806/whMjICDhBLkz.png?imageslim)

### 配置ADC（单次转换模式）
首先选择`ADC1`，开启通道3：
![mark](http://mculover666.cn/image/20190806/4W3UD4FP2vfB.png?imageslim)

接下来是对ADC的设置，这里我们保持默认即可：
![mark](http://mculover666.cn/image/20190806/y1bnlxAf4IOd.png?imageslim)

最后设置ADC的转换规则：
![mark](http://mculover666.cn/image/20190806/TnzK4JjLXoeD.png?imageslim)

其余的一些设置保持默认即可。

## 配置时钟树
STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：
![mark](http://mculover666.cn/image/20190806/1TQg7frjRpVr.png?imageslim)

![mark](http://mculover666.cn/image/20190808/EVKCwrQNEWcl.png?imageslim)

![mark](http://mculover666.cn/image/20190806/Dje8nuTMdpQY.png?imageslim)


## 生成工程设置
![mark](http://mculover666.cn/image/20190806/0siSY6f3zQJV.png?imageslim)

## 代码生成设置
最后设置生成独立的初始化文件：
![mark](http://mculover666.cn/image/20190806/T6WvSK6Dfpts.png?imageslim)

## 生成代码
点击`GENERATE CODE`即可生成MDK-V5工程：
![mark](http://mculover666.cn/image/20190806/s0jGhLBWW6Cm.png?imageslim)

# 3. 在MDK中编写、编译、下载用户代码
## 重定向printf( )函数
参考：[【STM32Cube】（八）基于串口发送函数实现printf()](https://blog.csdn.net/Mculover666/article/details/95975461)。


## 编写读取数据的测试代码
修改`main`函数如下：
```c
int main(void)
{
	uint16_t smoke_value = 0;

    HAL_Init();

    SystemClock_Config();

    MX_GPIO_Init();
    MX_ADC1_Init();
    MX_USART1_UART_Init();

    while (1)
    {
        HAL_ADC_Start(&hadc1);	                //启动ADC单次转换
        HAL_ADC_PollForConversion(&hadc1, 50);	//等待ADC转换完成
        smoke_value = HAL_ADC_GetValue(&hadc1); //读取ADC转换数据
        printf("smoke_value = %d\n", smoke_value);
        HAL_Delay(500);
    }
}
```
![mark](http://mculover666.cn/image/20190806/6jtuYrH0Mdot.png?imageslim)

至此，我们已经学会**如何使用ADC读取MQ-2传感器的值**，下一节将讲述如何使用通用定时器闪烁LED。

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![mark](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)

