---
title: STM32CubeMX | 29-使用硬件I2C读取甲醛传感器SGP30
keywords: STM32CubeMX SGP30
tags: STM32CubeMX SGP30
categories: STM32CubeMX
abbrlink: 3295727006
date: 2020-07-26 12:29:03
---

本篇详细的记录了如何使用STM32CubeMX配置STM32L431RCT6的硬件I2C外设读取甲醛传感器SGP30。
<!--more-->

# 1. 准备工作
## 开发板
首先需要准备一个开发板，这里我准备的是STM32L4的开发板（BearPi）。

## SGP30传感器模块

![](https://img-blog.csdnimg.cn/2020072314374872.png#pic_center)

SGP30是一款单一芯片上具有多个传感元件的金属氧化物**室内气体传感器**，内集成4个气体传感元件，具有完全校准的空气质量输出信号，主要是对空气质量进行检测。TVOC（Total Volatile Organic Compounds，总挥发性有机物）是一项重要指标，一般我们可以用它来反映甲醛的浓度，所以**SGP主要用于甲醛的检测**，另外还可以用于监测CO2浓度。

SGP引脚的定义如下：
![](https://img-blog.csdnimg.cn/20200723144637443.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01jdWxvdmVyNjY2,size_16,color_FFFFFF,t_70)
典型应用电路如下：
![](https://img-blog.csdnimg.cn/20200723144659652.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01jdWxvdmVyNjY2,size_16,color_FFFFFF,t_70)
电气特性如下：
![](https://img-blog.csdnimg.cn/20200723145220517.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01jdWxvdmVyNjY2,size_16,color_FFFFFF,t_70)

二氧化碳浓度含量会影响人类的生活作息，整理出二氧化碳浓度含量与人体生理反应如下：

350～450ppm：同一般室外环境
350～1000ppm：空气清新，呼吸顺畅。
`>1000ppm`：感觉空气浑浊，并开始觉得昏昏欲睡。


# 2.生成MDK工程
## 选择芯片型号
打开STM32CubeMX，打开MCU选择器：
![mark](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwODA2L2dCUDZnbG1VU0g4MC5wbmc?x-oss-process=image/format,png)

搜索并选中芯片`STM32L431RCT6`:
![mark](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwODA2L2dueUh3ZGw1M3VWRC5wbmc?x-oss-process=image/format,png)

## 配置时钟源
- 如果选择使用外部高速时钟（HSE），则需要在System Core中配置RCC；
- 如果使用默认内部时钟（HSI），这一步可以略过；

这里我都使用外部时钟：
![mark](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwODA2L2s1OTNsR0diNXRsVy5wbmc?x-oss-process=image/format,png)

## 配置串口
小熊派开发板板载ST-Link并且虚拟了一个串口，原理图如下：

![mark](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwODE0L0l3eVhPTlZlZlB4OS5wbmc?x-oss-process=image/format,png)

这里我将开关拨到`AT-MCU`模式，使PC的串口与USART1之间连接。

接下来开始配置`USART1`：

![mark](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwODE0L25MTVJNWXRtemdobC5wbmc?x-oss-process=image/format,png)

## 配置硬件I2C

首先选择将SGP30传感器接在哪个I2C接口上，如图：

![mark](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwODI2L2Z4eDY1MXFEWUM2Ty5wbmc?x-oss-process=image/format,png)

接下来开始配置I2C接口1：

![mark](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwODI2L29iRnN5NXdKSERYei5wbmc?x-oss-process=image/format,png)

## 配置时钟树
STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：

![mark](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwODA2LzFUUWc3ZnJqUnBWci5wbmc?x-oss-process=image/format,png)

## 生成工程设置

![mark](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwODI3L3l0T1NwQ3VJcFV3Vi5wbmc?x-oss-process=image/format,png)

## 代码生成设置
最后设置生成独立的初始化文件：

![mark](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwODA2L1Q2V3ZTSzZEZnB0cy5wbmc?x-oss-process=image/format,png)

## 生成代码
点击`GENERATE CODE`即可生成MDK-V5工程：

![mark](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwODA2L3MwakdoTEJXVzZDbS5wbmc?x-oss-process=image/format,png)

# 3. 在MDK中编写、编译、下载用户代码
## 重定向printf函数

参考：[【STM32Cube_09】重定向printf函数到串口输出的多种方法](http://www.mculover666.cn/posts/2251182441//)。

# 4. 编写SGP30驱动程序

>驱动源码：[https://github.com/Mculover666/HAL_Driver_Lib](https://github.com/Mculover666/HAL_Driver_Lib)

参考[Sensirion_Gas_Sensors_SGP30_Datasheet_EN.PDF.pdf](https://www.mouser.com/pdfdocs/Sensirion_Gas_Sensors_SGP30_Datasheet_EN-1148053.pdf)进行编程。

## 宏定义SGP30器件地址

先来编写`sgp30.h`头文件，SGP30的器件地址在数据手册中已给出：
![](https://img-blog.csdnimg.cn/20200723151610151.png)

注意**数据手册中给出了8位数据，只有低7位用作地址**，结合原理图，可以定义如下：
```c
#define SGP30_ADDR          0x58
#define	SGP30_ADDR_WRITE	SGP30_ADDR<<1       //0xb0
#define	SGP30_ADDR_READ		(SGP30_ADDR<<1)+1   //0xb1
```
## 传感器数据封装
```c
typedef struct sgp30_data_st {
    uint16_t co2;
    uint16_t tvoc;
}sgp30_data_t;
```
## 枚举SHT30命令列表
参考数据手册，在`sgp30.h`头文件中给出如下枚举定义：
```c
typedef enum sgp30_cmd_en {
    /* 初始化空气质量测量 */
    INIT_AIR_QUALITY = 0x2003,
    
    /* 开始空气质量测量 */
    MEASURE_AIR_QUALITY = 0x2008
    
} sgp30_cmd_t;
```
## 发送命令函数
```c
/**
 * @brief	向SGP30发送一条指令(16bit)
 * @param	cmd SGP30指令
 * @retval	成功返回HAL_OK
*/
static uint8_t sgp30_send_cmd(sgp30_cmd_t cmd)
{
    uint8_t cmd_buffer[2];
    cmd_buffer[0] = cmd >> 8;
    cmd_buffer[1] = cmd;
    return HAL_I2C_Master_Transmit(&hi2c1, SGP30_ADDR_WRITE, (uint8_t*) cmd_buffer, 2, 0xFFFF);
}
```
## 复位函数
```c
/**
 * @brief	软复位SGP30
 * @param	none
 * @retval	成功返回HAL_OK
*/
static int sgp30_soft_reset(void)
{
    uint8_t cmd = 0x06;
    return HAL_I2C_Master_Transmit(&hi2c1, 0x00, &cmd, 1, 0xFFFF);
}
```
## SGP30初始化函数
```c
/**
 * @brief	初始化SGP30空气质量测量模式
 * @param	none
 * @retval	成功返回0，失败返回-1
*/
int sgp30_init(void)
{
    int status;
    
    status = sgp30_soft_reset();
    if (status != HAL_OK) {
        return -1;
    }
    
    HAL_Delay(100);
    
    status = sgp30_send_cmd(INIT_AIR_QUALITY);
    
    HAL_Delay(100);
    
    return status == 0 ? 0 : -1;
}
```
## 发送一次测量开始命令
```c
/**
 * @brief	初始化SGP30空气质量测量模式
 * @param	none
 * @retval	成功返回HAL_OK
*/
static int sgp30_start(void)
{
    return sgp30_send_cmd(MEASURE_AIR_QUALITY);
}
```
## 从SGP30读取一次数据并校验解析
在数据手册中可知，SGP30分别在co2浓度之后和TVOC浓度数据之后发送了8-CRC校验码，确保了数据可靠性。

关于CRC校验请参考我的另一篇博客：[如何通俗的理解CRC校验并用C语言实现](https://www.mculover666.cn/2019/08/06/如何通俗的理解CRC校验并用C语言实现/)。

SGP30校验的参数已经在数据手册中给出：
![](https://img-blog.csdnimg.cn/20200723170052234.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01jdWxvdmVyNjY2,size_16,color_FFFFFF,t_70)

编写CRC-8校验函数如下：
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

接下来编写读取并校验数据的函数：
```c
/**
 * @brief	读取一次空气质量数据
 * @param	none
 * @retval	成功返回0，失败返回-1
*/
int spg30_read(void)
{
    int status;
    uint8_t recv_buffer[6]={0};
    
    /* 启动测量 */
    status = sgp30_start();
    if (status != 0) {
        printf("sgp30 start fail\r\n");
        return -1;
    }
    
    HAL_Delay(100);
    
    /* 读取测量数据 */
    status = HAL_I2C_Master_Receive(&hi2c1, SGP30_ADDR_READ, (uint8_t*)recv_buffer, 6, 0xFFFF);
    if (status != HAL_OK) {
        printf("I2C Master Receive fail\r\n");
        return -1;
    }
    
    /* 校验接收的测量数据 */
    if (CheckCrc8(&recv_buffer[0], 0xFF) != recv_buffer[2]) {
        printf("co2 recv data crc check fail\r\n");
        return -1;
    }
    if (CheckCrc8(&recv_buffer[3], 0xFF) != recv_buffer[5]) {
        printf("tvoc recv data crc check fail\r\n");
        return -1;
    }
    
    
    /* 转换测量数据 */
    sgp30_data.co2  = recv_buffer[0] << 8 | recv_buffer[1];
    sgp30_data.tvoc = recv_buffer[3] << 8 | recv_buffer[4];
    
    return 0;
}
```

# 5. 测试SHT30驱动程序
在main.c中包含头文件：
```c
#include <stdio.h>
#include "sgp30.h"
```
在main函数中对该驱动进行测试，修改main函数：
```c
int main(void)
{
	 HAL_Init();
	 SystemClock_Config();
	 
	 /* Initialize all configured peripherals */
	 MX_GPIO_Init();
	 MX_I2C1_Init();
	 MX_USART1_UART_Init();
	 
	 /* USER CODE BEGIN 2 */
    printf("SGP30 Test By mculover666\r\n");
  
    if (-1 == sgp30_init()) {
        printf("sgp30 init fail\r\n");
        
        /* 因为是裸机，所以直接进入死机 */
        while(1);
    }
    printf("sgp30 init success\r\n");
 	/* USER CODE END 2 */

  while (1)
  {
	 /* USER CODE END WHILE */
	
		/* USER CODE BEGIN 3 */
		 if( -1 == spg30_read()) {
		     printf("sgp30 read fail\r\n");
		 }
		 else {
		     printf("sgp30 read success, co2:%4d ppm, tvoc:%4d ppd\r\n", sgp30_data.co2, sgp30_data.tvoc);
		 }
		 
		 HAL_Delay(2000);
	}
	/* USER CODE END 3 */
}

```
测试结果如图：
![](https://img-blog.csdnimg.cn/20200723171659186.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01jdWxvdmVyNjY2,size_16,color_FFFFFF,t_70)


**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![mark](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwODE0L05RcXQxZVJ4cmwxSy5wbmc?x-oss-process=image/format,png)
