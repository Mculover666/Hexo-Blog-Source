---
title: 【STM32Cube_14】使用硬件I2C读写环境光强度传感器（BH1750）
tags: STM32CubeMX BH1750
categories: STM32CubeMX
abbrlink: 1561092257
date: 2019-08-04 10:04:56
---

本篇详细的记录了如何使用STM32CubeMX配置STM32L431RCT6的硬件I2C外设读取环境光强度传感器数据（BH1750）。
<!--more-->

# 1. 准备工作
## 硬件准备
- 开发板
首先需要准备一个开发板，这里我准备的是STM32L4的开发板（BearPi）：

![mark](http://mculover666.cn/image/20190806/9uiPTi5odYSj.png?imageslim)

- BH1750模块
BH1750FV1是两线式串行总线接口（IIC）的16位数字输出型环境光强度传感器，利用它的高分辨率可以探测较大范围内的光照强度变化（1lx - 65535lx）。

![mark](http://mculover666.cn/image/20190827/gDUPbgs5suTo.png?imageslim)

BH1750的原理图如下：

![mark](http://mculover666.cn/image/20190827/cX3i5JC2BU5x.png?imageslim)

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

## 配置串口
小熊派开发板板载ST-Link并且虚拟了一个串口，原理图如下：

![mark](http://mculover666.cn/image/20190814/IwyXONVefPx9.png?imageslim)

这里我将开关拨到`AT-MCU`模式，使PC的串口与USART1之间连接。

接下来开始配置`USART1`：

![mark](http://mculover666.cn/image/20190814/nLMRMYtmzghl.png?imageslim)

## 配置硬件I2C

首先查看小熊派开发板的原理图，确定EEPROM接在哪个I2C接口上，如图：

![mark](http://mculover666.cn/image/20190826/fxx651qDYC6O.png?imageslim)

接下来开始配置I2C接口1：

![mark](http://mculover666.cn/image/20190826/obFsy5wJHDXz.png?imageslim)

## 配置时钟树
STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：

![mark](http://mculover666.cn/image/20190806/1TQg7frjRpVr.png?imageslim)

## 生成工程设置

![mark](http://mculover666.cn/image/20190827/ytOSpCuIpUwV.png?imageslim)

## 代码生成设置
最后设置生成独立的初始化文件：

![mark](http://mculover666.cn/image/20190806/T6WvSK6Dfpts.png?imageslim)

## 生成代码
点击`GENERATE CODE`即可生成MDK-V5工程：

![mark](http://mculover666.cn/image/20190806/s0jGhLBWW6Cm.png?imageslim)

# 3. 在MDK中编写、编译、下载用户代码
## 修改I2C初始化代码的小BUG

![mark](http://mculover666.cn/image/20190826/jQbpnRxnsBF0.png?imageslim)

## 重定向printf( )函数

参考：[【STM32Cube_09】重定向printf函数到串口输出的多种方法](https://www.mculover666.cn/2019/07/30/STM32Cube/【STM32Cube-09】重定向printf函数到串口输出的多种方法/)。

# 4. 编写BH1750驱动程序

参考[bh1750FVI中文数据手册.pdf](https://download.csdn.net/download/mculover666/11368379)进行编程。

## 宏定义BH1750器件地址

BH1750的器件地址由`ADDR`端口的高低电平决定：

![mark](http://mculover666.cn/image/20190827/rwX5250rtUKp.png?imageslim)

结合原理图，在 `bh1750_i2c_drv.h` 头文件中可以定义如下：
```c
#define	BH1750_ADDR_WRITE	0x46	//01000110
#define	BH1750_ADDR_READ	0x47	//01000111
```
## 枚举BH1750工作模式
参考数据手册在 `bh1750_i2c_drv.h` 头文件中进行如下枚举定义：

```c
typedef enum
{
	POWER_OFF_CMD	=	0x00,	//断电：无激活状态
	POWER_ON_CMD	=	0x01,	//通电：等待测量指令
	RESET_REGISTER	=	0x07,	//重置数字寄存器（在断电状态下不起作用）
	CONT_H_MODE		=	0x10,	//连续H分辨率模式：在11x分辨率下开始测量，测量时间120ms
	CONT_H_MODE2	=	0x11,	//连续H分辨率模式2：在0.51x分辨率下开始测量，测量时间120ms
	CONT_L_MODE		=	0x13,	//连续L分辨率模式：在411分辨率下开始测量，测量时间16ms
	ONCE_H_MODE		=	0x20,	//一次高分辨率模式：在11x分辨率下开始测量，测量时间120ms，测量后自动设置为断电模式
	ONCE_H_MODE2	=	0x21,	//一次高分辨率模式2：在0.51x分辨率下开始测量，测量时间120ms，测量后自动设置为断电模式
	ONCE_L_MODE		=	0x23	//一次低分辨率模式：在411x分辨率下开始测量，测量时间16ms，测量后自动设置为断电模式
} BH1750_MODE;
```
## 发送命令和读取数据

接下来编写`bh1750_i2c_drv.c`驱动文件，参考数据手册中的这部分：

![mark](http://mculover666.cn/image/20190827/3C9xx1HgHlHj.png?imageslim)

本驱动程序底层使用 HAL 库的 IIC 初始化文件，所以包含如下头文件：
```c
#include "bh1750_i2c_drv.h"
#include "i2c.h"
```
根据上图，发送命令的函数如下：
```c
/**
 * @brief	向BH1750发送一条指令
 * @param	cmd —— BH1750工作模式指令（在BH1750_MODE中枚举定义）
 * @retval	成功返回HAL_OK
*/
uint8_t	BH1750_Send_Cmd(BH1750_MODE cmd)
{
	return HAL_I2C_Master_Transmit(&hi2c1, BH1750_ADDR_WRITE, (uint8_t*)&cmd, 1, 0xFFFF);
}
```
接收光照强度数据的函数如下：
```c
/**
 * @brief	从BH1750接收一次光强数据
 * @param	dat —— 存储光照强度的地址（两个字节数组）
 * @retval	成功 —— 返回HAL_OK
*/
uint8_t BH1750_Read_Dat(uint8_t* dat)
{
	return HAL_I2C_Master_Receive(&hi2c1, BH1750_ADDR_READ, dat, 2, 0xFFFF);
}
```
## 数据转换函数
根据数据手册中给出的公式，编写将从BH1750读出的两个字节数据转换为对应强度值的函数：

```c
/**
 * @brief	将BH1750的两个字节数据转换为光照强度值（0-65535）
 * @param	dat  —— 存储光照强度的地址（两个字节数组）
 * @retval	成功 —— 返回光照强度值
*/
uint16_t BH1750_Dat_To_Lux(uint8_t* dat)
{
	uint16_t lux = 0;
	lux = dat[0];
	lux <<= 8;
	lux += dat[1];
	lux = (int)(lux / 1.2);
	
	return lux;
}
```

# 5. 测试驱动程序

在main.c中测试驱动程序是否正常：
```c
int main(void)
{
    uint8_t dat[2] = {0};		//dat[0]是高字节，dat[1]是低字节

    HAL_Init();
    SystemClock_Config();
    MX_GPIO_Init();
    MX_I2C1_Init();
    MX_USART1_UART_Init();

    while (1)
    {
        if(HAL_OK == BH1750_Send_Cmd(ONCE_H_MODE))
        {
            //printf("send ok\n");
        }
        else
        {
            //printf("send fail\n");
        }

        HAL_Delay(200);
        if(HAL_OK == BH1750_Read_Dat(dat))
        {
            //printf("recv ok\n");
            printf("current: %5d lux\n", BH1750_Dat_To_Lux(dat));
            
        }
        else
        {
            //printf("recv fail");
        }

        HAL_Delay(1000);
    }
}
```
编译下载运行，测试结果如下：

![mark](http://mculover666.cn/image/20190827/sdaBrbW78OyI.png?imageslim)

至此，我们已经学会**如何使用硬件IIC接口读取环境光强度传感器数据（BH1750）**，下一节将讲述如何使用硬件IIC接口读取温湿度传感器数据并使用软件CRC校验（SHT30）。

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![mark](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)
