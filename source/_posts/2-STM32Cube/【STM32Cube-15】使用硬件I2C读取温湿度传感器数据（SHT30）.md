---
title: 【STM32Cube_15】使用硬件I2C读取温湿度传感器数据（SHT30）
tags: STM32CubeMX 温湿度传感器 SHT30
categories: STM32CubeMX
abbrlink: 2508748577
date: 2019-08-05 08:48:56
---

本篇详细的记录了如何使用STM32CubeMX配置STM32L431RCT6的硬件I2C外设，读取SHT30温湿度传感器的数据并通过串口发送。
<!--more-->

# 1. 准备工作
## 硬件准备
- 开发板
首先需要准备一个开发板，这里我准备的是STM32L4的开发板（BearPi）：

![mark](http://mculover666.cn/image/20190806/9uiPTi5odYSj.png?imageslim)

- SHT30温湿度传感器
SHT30温湿度传感器是一个完全校准的、现行的、带有温度补偿的**数字输出型**传感器，具有 2.4V-5.5V 的宽电压支持，使用IIC接口进行通信，最高速率可达1M并且有两个用户可选地址，除此之外，它还具有8个引脚的DFN超小封装，如图：

![mark](http://mculover666.cn/image/20190808/XthdnB8dTXFW.png?imageslim)

SHT30的原理图如下：
![mark](http://mculover666.cn/image/20190808/BSqAexcE2Pir.png?imageslim)

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

## 配置I2C接口
查看小熊派E53接口的原理图：
![mark](http://mculover666.cn/image/20190808/gHWofMib3ISQ.png?imageslim)

接下来开始配置I2C接口1：
![mark](http://mculover666.cn/image/20190808/GuKoTgin8iDJ.png?imageslim)

## 配置时钟树
STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：
![mark](http://mculover666.cn/image/20190806/1TQg7frjRpVr.png?imageslim)

## 生成工程设置
![mark](http://mculover666.cn/image/20190808/vJ8NCpGew9fg.png?imageslim)

## 代码生成设置
最后设置生成独立的初始化文件：
![mark](http://mculover666.cn/image/20190806/T6WvSK6Dfpts.png?imageslim)

## 生成代码
点击`GENERATE CODE`即可生成MDK-V5工程：
![mark](http://mculover666.cn/image/20190806/s0jGhLBWW6Cm.png?imageslim)

# 3. 在MDK中编写、编译、下载用户代码
## 重定向printf( )函数

参考：[【STM32Cube_09】重定向printf函数到串口输出的多种方法](https://www.mculover666.cn/2019/07/30/STM32Cube/【STM32Cube-09】重定向printf函数到串口输出的多种方法/)。


## 修改I2C初始化代码的小BUG
![mark](http://mculover666.cn/image/20190808/zGOS6y9hjXhu.png?imageslim)

# 4. 编写SHT30驱动程序

参考[SHT30数据手册.pdf](https://download.csdn.net/download/mculover666/11368379)进行编程。

## 宏定义SHT30器件地址

先来编写`sht30_i2c_drv.h`头文件，SHT30的器件地址由`ADDR`端口的高低电平决定：

![mark](http://mculover666.cn/image/20190808/mYPfGe7jdXS3.png?imageslim)

注意数据手册中给出了8位数据，只有低7位用作地址，结合原理图，可以定义如下：
```c
/* ADDR Pin Conect to VSS */

#define	SHT30_ADDR_WRITE	0x44<<1         //10001000
#define	SHT30_ADDR_READ		(0x44<<1)+1	    //10001011
```
## 枚举SHT30命令列表
参考数据手册，在`sht30_i2c_drv.h`头文件中给出如下枚举定义：
```c
typedef enum
{
    /* 软件复位命令 */

    SOFT_RESET_CMD = 0x30A2,	
    /*
    单次测量模式
    命名格式：Repeatability_CS_CMD
    CS： Clock stretching
    */
    HIGH_ENABLED_CMD    = 0x2C06,
    MEDIUM_ENABLED_CMD  = 0x2C0D,
    LOW_ENABLED_CMD     = 0x2C10,
    HIGH_DISABLED_CMD   = 0x2400,
    MEDIUM_DISABLED_CMD = 0x240B,
    LOW_DISABLED_CMD    = 0x2416,

    /*
    周期测量模式
    命名格式：Repeatability_MPS_CMD
    MPS：measurement per second
    */
    HIGH_0_5_CMD   = 0x2032,
    MEDIUM_0_5_CMD = 0x2024,
    LOW_0_5_CMD    = 0x202F,
    HIGH_1_CMD     = 0x2130,
    MEDIUM_1_CMD   = 0x2126,
    LOW_1_CMD      = 0x212D,
    HIGH_2_CMD     = 0x2236,
    MEDIUM_2_CMD   = 0x2220,
    LOW_2_CMD      = 0x222B,
    HIGH_4_CMD     = 0x2334,
    MEDIUM_4_CMD   = 0x2322,
    LOW_4_CMD      = 0x2329,
    HIGH_10_CMD    = 0x2737,
    MEDIUM_10_CMD  = 0x2721,
    LOW_10_CMD     = 0x272A,
	/* 周期测量模式读取数据命令 */
	READOUT_FOR_PERIODIC_MODE = 0xE000,
} SHT30_CMD;
```
## 发送命令函数
```c
/**
 * @brief	向SHT30发送一条指令(16bit)
 * @param	cmd —— SHT30指令（在SHT30_MODE中枚举定义）
 * @retval	成功返回HAL_OK
*/
static uint8_t	SHT30_Send_Cmd(SHT30_CMD cmd)
{
    uint8_t cmd_buffer[2];
    cmd_buffer[0] = cmd >> 8;
    cmd_buffer[1] = cmd;
    return HAL_I2C_Master_Transmit(&hi2c1, SHT30_ADDR_WRITE, (uint8_t* cmd_buffer, 2, 0xFFFF);
}
```
## 复位函数
```c
/**
 * @brief	复位SHT30
 * @param	none
 * @retval	none
*/
void SHT30_reset(void)
{
    SHT30_Send_Cmd(SOFT_RESET_CMD);
    HAL_Delay(20);
}
```
## SHT30工作模式初始化函数（周期测量模式）
```c
/**
 * @brief	初始化SHT30
 * @param	none
 * @retval	成功返回HAL_OK
 * @note	周期测量模式
*/
uint8_t SHT30_Init(void)
{
    return SHT30_Send_Cmd(MEDIUM_2_CMD);
}
```
## 从SHTY30读取一次数据（周期测量模式下）
从SHT30数据手册中可以得到在周期测量模式下读取一次数据的时序，如图：

![mark](http://mculover666.cn/image/20190810/et530Xi83sku.png?imageslim)

根据该时序可以看出，首先要发送读数据的命令，然后接收6个字节的数据，编写程序如下：
```c
/**
 * @brief	从SHT30读取一次数据
 * @param	dat —— 存储读取数据的地址（6个字节数组）
 * @retval	成功 —— 返回HAL_OK
*/
uint8_t SHT30_Read_Dat(uint8_t* dat)
{
	SHT30_Send_Cmd(READOUT_FOR_PERIODIC_MODE);
	return HAL_I2C_Master_Receive(&hi2c1, SHT30_ADDR_READ, dat, 6, 0xFFFF);
}
```
## 从接收数据中校验并解析温度值和湿度值
在数据手册中可知，SHT30分别在温度数据和湿度数据之后发送了8-CRC校验码，确保了数据可靠性。

关于CRC校验请参考我的另一篇博客：[如何通俗的理解CRC校验并用C语言实现](https://www.mculover666.cn/2019/08/09/%E5%A6%82%E4%BD%95%E9%80%9A%E4%BF%97%E7%9A%84%E7%90%86%E8%A7%A3CRC%E6%A0%A1%E9%AA%8C%E5%B9%B6%E7%94%A8C%E8%AF%AD%E8%A8%80%E5%AE%9E%E7%8E%B0/)。

CRC-8校验程序如下：
```c
#define CRC8_POLYNOMIAL 0x31

uint8_t CheckCrc8(uint8_t* const message, uint8_t initial_value)
{
    uint8_t  remainder;	    //余数
    uint8_t  i = 0, j = 0;  //循环变量

    /* 初始化 */
    remainder = initial_value;

    for(j = 0; j < 2;j++)
    {
        remainder ^= message[j];

        /* 从最高位开始依次计算  */
        for (i = 0; i < 8; i++)
        {
            if (remainder & 0x80)
            {
                remainder = (remainder << 1)^CRC8_POLYNOMIAL;
            }
            else
            {
                remainder = (remainder << 1);
            }
        }
    }

    /* 返回计算的CRC码 */
    return remainder;
}
```
计算温度值和湿度值的公式在数据手册中已给出，如图：

![mark](http://mculover666.cn/image/20190810/akji0RTvX77c.png?imageslim)

接下来编写解析数据的函数：
```c
/**
 * @brief	将SHT30接收的6个字节数据进行CRC校验，并转换为温度值和湿度值
 * @param	dat  —— 存储接收数据的地址（6个字节数组）
 * @retval	校验成功  —— 返回0
 * 			校验失败  —— 返回1，并设置温度值和湿度值为0
*/
uint8_t SHT30_Dat_To_Float(uint8_t* const dat, float* temperature, float* humidity)
{
	uint16_t recv_temperature = 0;
	uint16_t recv_humidity = 0;
	
	/* 校验温度数据和湿度数据是否接收正确 */
	if(CheckCrc8(dat, 0xFF) != dat[2] || CheckCrc8(&dat[3], 0xFF) != dat[5])
		return 1;
	
	/* 转换温度数据 */
	recv_temperature = ((uint16_t)dat[0]<<8)|dat[1];
	*temperature = -45 + 175*((float)recv_temperature/65535);
	
	/* 转换湿度数据 */
	recv_humidity = ((uint16_t)dat[3]<<8)|dat[4];
	*humidity = 100 * ((float)recv_humidity / 65535);
	
	return 0;
}
```

# 5. 测试SHT30驱动程序
在main函数中对该驱动进行测试，在`main.c`中添加如下代码：
```c
#include <stdio.h>
#include "sht30_i2c_drv.h"

int main(void)
{
    /* USER CODE BEGIN 1 */
    uint8_t recv_dat[6] = {0};
    float temperature = 0.0;
    float humidity = 0.0;
    /* USER CODE END 1 */

    HAL_Init();

    SystemClock_Config();

    MX_GPIO_Init();
    MX_I2C1_Init();
    MX_USART1_UART_Init();

    /* USER CODE BEGIN 2 */
    SHT30_Reset();
    if(SHT30_Init() == HAL_OK)
        printf("sht30 init ok.\n");
    else
        printf("sht30 init fail.\n");
  /* USER CODE END 2 */

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* USER CODE END WHILE */
		
    /* USER CODE BEGIN 3 */
		HAL_Delay(1000);
		if(SHT30_Read_Dat(recv_dat) == HAL_OK)
		{
			if(SHT30_Dat_To_Float(recv_dat, &temperature, &humidity)==0)
			{
				printf("temperature = %f, humidity = %f\n", temperature, humidity);
			}
			else
			{
				printf("crc check fail.\n");
			}
		}
		else
		{
			printf("read data from sht30 fail.\n");
		}
	}
  /* USER CODE END 3 */
}

```
测试结果如图：

![mark](http://mculover666.cn/image/20190810/vlDKzTOJ8cBE.png?imageslim)

至此，我们已经学会**如何使用硬件IIC接口读取温湿度传感器数据并使用软件CRC校验（SHT30）**，下一节将讲述如何使用硬件CRC校验SHT30的数据。

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![mark](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)