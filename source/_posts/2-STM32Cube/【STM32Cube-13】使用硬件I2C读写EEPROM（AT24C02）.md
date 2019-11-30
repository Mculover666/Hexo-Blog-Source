---
title: 【STM32Cube_13】使用硬件I2C读写EEPROM（AT24C02）
tags: STM32CubeMX EEPROM
categories: STM32CubeMX
abbrlink: 3523891062
date: 2019-08-03 10:04:56
---

本篇详细的记录了如何使用STM32CubeMX配置STM32L431RCT6的硬件I2C外设读取EEPROM数据（以AT24C02为例）。
<!--more-->

# 1. 准备工作
## 硬件准备
- 开发板

首先需要准备一个开发板，这里我准备的是STM32L4的开发板（BearPi）：

![mark](http://mculover666.cn/image/20190806/9uiPTi5odYSj.png?imageslim)

- EEPROM

小熊派开发板左边的接口是E53接口，用来连接E53接口的扩展板，每个扩展板都板载了一块EEPROM用来保存信息，如图：

![mark](http://mculover666.cn/image/20190826/KU3hAdAprJlG.png?imageslim)

AT24C02的原理图如下（**该原理图中有bug，A0的上拉电阻无效，实际A0为低电平**）：

![mark](http://mculover666.cn/image/20190826/LySHcKjfKLSz.png?imageslim)

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

![mark](http://mculover666.cn/image/20190826/xVjdv02ngKcq.png?imageslim)

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

## 编写EEPROM驱动程序

EEPROM的驱动编写篇幅过多，单独分出来一节讲述。

# 4. AT24C02驱动的编写

## 确定IIC器件地址

根据AT24C02的 Datasheet 可知AT24C02有**2K bit，即256B，分为32页,每页8个字节**，结合数据手册和原理图可以得知，板载AT24C02的读地址为`0xA2`，写地址为`0xA3`：

![mark](http://mculover666.cn/image/20190826/ApkRI7A59RGs.png?imageslim)

首先在`at24c02_i2c_drv.h`中编写AT24C02相关的宏定义：
```c
#define	AT24C02_ADDR_WRITE	0xA0
#define	AT24C02_ADDR_READ	0xA1
```
然后在`at24c02_i2c_drv.c`中引入`i2c.h`，基于HAL提供的硬件IIC操作函数，编写AT24C02的一些底层函数，如下。

## 任意地址写一个字节

根据AT24C02的数据手册可知，AT24C02写一个字节的格式如下：

![mark](http://mculover666.cn/image/20190826/Kne5Ozz8NWO8.png?imageslim)

编写的函数如下：
```c
/**
 * @brief		AT24C02任意地址写一个字节数据
 * @param		addr —— 写数据的地址（0-255）
 * @param		dat  —— 存放写入数据的地址
 * @retval		成功 —— HAL_OK
*/
uint8_t At24c02_Write_Byte(uint16_t addr, uint8_t* dat)
{
	return HAL_I2C_Mem_Write(&hi2c1, AT24C02_ADDR_WRITE, addr, I2C_MEMADD_SIZE_8BIT, dat, 1, 0xFFFF);
}
```
## 任意地址读一个字节

根据AT24C02的数据手册可知，AT24C02读一个字节的格式如下：

![mark](http://mculover666.cn/image/20190826/mKfAcbHNoD24.png?imageslim)

编写的函数如下：
```c
/**
 * @brief		AT24C02任意地址读一个字节数据
 * @param		addr —— 读数据的地址（0-255）
 * @param		read_buf —— 存放读取数据的地址
 * @retval		成功 —— HAL_OK
*/
uint8_t At24c02_Read_Byte(uint16_t addr, uint8_t* read_buf)
{
	return HAL_I2C_Mem_Read(&hi2c1, AT24C02_ADDR_READ, addr, I2C_MEMADD_SIZE_8BIT, read_buf, 1, 0xFFFF);
}
```
## 测试字节读写函数
在`main.c`中测试：
```c
int main(void)
{
	uint8_t write_dat = 0xa5;
	uint8_t recv_buf = 0;
  
  	HAL_Init();
  	SystemClock_Config();
 	MX_GPIO_Init();
 	MX_I2C1_Init();
 	MX_USART1_UART_Init();

	if(HAL_OK == At24c02_Write_Byte(10,&write_dat))
	{
		printf("Write ok\n");
	}
	else
	{
		printf("Write fail\n");
	}
	
	HAL_Delay(50);		//写一次和读一次之间需要短暂的延时
	
	if(HAL_OK == At24c02_Read_Byte(10,&recv_buf))
	{
		printf("Read ok, recv_buf = 0x%02X\n", recv_buf);
	}
	else
	{
		printf("Read fail\n");
	}

	while(1);
```
测试结果如下：

![mark](http://mculover666.cn/image/20190826/3OH13P3htS3Q.png?imageslim)

## 任意地址连续写多个字节

AT24C02连续写字节的时候需要注意，**不能使用写单个字节函数连续的写入**，因为AT24C02分为了32页，每页是8个字节，如果连续的单字节写入8个字节后，会重复的继续往该页写数据，所以要使用如下的写一页的格式：

![mark](http://mculover666.cn/image/20190826/JPc5Vippp1jN.png?imageslim)

```c
/**
 * @brief		AT24C02任意地址连续写多个字节数据
 * @param		addr —— 写数据的地址（0-255）
 * @param		dat  —— 存放写入数据的地址
 * @retval		成功 —— HAL_OK
*/
uint8_t At24c02_Write_Amount_Byte(uint16_t addr, uint8_t* dat, uint16_t size)
{
    uint8_t i = 0;
    uint16_t cnt = 0;		//写入字节计数
    
    /* 对于起始地址，有两种情况，分别判断 */
    if(0 == addr % 8 )
    {
        /* 起始地址刚好是页开始地址 */
        
        /* 对于写入的字节数，有两种情况，分别判断 */
        if(size <= 8)
        {
            //写入的字节数不大于一页，直接写入
            return HAL_I2C_Mem_Write(&hi2c1, AT24C02_ADDR_WRITE, addr, I2C_MEMADD_SIZE_8BIT, dat, size, 0xFFFF);
        }
        else
        {
            //写入的字节数大于一页，先将整页循环写入
            for(i = 0;i < size/8; i++)
            {
                HAL_I2C_Mem_Write(&hi2c1, AT24C02_ADDR_WRITE, addr, I2C_MEMADD_SIZE_8BIT, &dat[cnt], 8, 0xFFFF);
                addr += 8;
                cnt += 8;
            }
            //将剩余的字节写入
            return HAL_I2C_Mem_Write(&hi2c1, AT24C02_ADDR_WRITE, addr, I2C_MEMADD_SIZE_8BIT, &dat[cnt], size - cnt, 0xFFFF);
        }
    }
    else
    {
        /* 起始地址偏离页开始地址 */
        /* 对于写入的字节数，有两种情况，分别判断 */
        if(size <= (8 - addr%8))
        {
            /* 在该页可以写完 */
            return HAL_I2C_Mem_Write(&hi2c1, AT24C02_ADDR_WRITE, addr, I2C_MEMADD_SIZE_8BIT, dat, size, 0xFFFF);
        }
        else
        {
            /* 该页写不完 */
            //先将该页写完
            cnt += 8 - addr%8;
            HAL_I2C_Mem_Write(&hi2c1, AT24C02_ADDR_WRITE, addr, I2C_MEMADD_SIZE_8BIT, dat, cnt, 0xFFFF);
            addr += cnt;
            
            //循环写整页数据
            for(i = 0;i < (size - cnt)/8; i++)
            {
                HAL_I2C_Mem_Write(&hi2c1, AT24C02_ADDR_WRITE, addr, I2C_MEMADD_SIZE_8BIT, &dat[cnt], 8, 0xFFFF);
                addr += 8;
                cnt += 8;
            }
            
            //将剩下的字节写入
            return HAL_I2C_Mem_Write(&hi2c1, AT24C02_ADDR_WRITE, addr, I2C_MEMADD_SIZE_8BIT, &dat[cnt], size - cnt, 0xFFFF);
        }			
    }
}
```
## 任意地址连续读多个字节
AT24C02连续读多个字节没有限制，直接读取即可，代码如下：
```c
/**
 * @brief		AT24C02任意地址连续读多个字节数据
 * @param		addr —— 读数据的地址（0-255）
 * @param		dat  —— 存放读出数据的地址
 * @retval		成功 —— HAL_OK
*/
uint8_t At24c02_Read_Amount_Byte(uint16_t addr, uint8_t* recv_buf, uint16_t size)
{
	return HAL_I2C_Mem_Read(&hi2c1, AT24C02_ADDR_READ, addr, I2C_MEMADD_SIZE_8BIT, recv_buf, size, 0xFFFF);
}
```
## 测试任意地址连续读写多个字节
在`main.c`中测试：
```c
int main(void)
{
	uint8_t write_dat[22] = {0};
	uint8_t recv_buf[22] = {0};
  
  	HAL_Init();
  	SystemClock_Config();
 	MX_GPIO_Init();
 	MX_I2C1_Init();
 	MX_USART1_UART_Init();

	for(i = 0;i < 22; i++)
	{
		write_dat[i] = i;
		printf("%02X ", write_dat[i]);
		if((i+1) % 16 == 0)
		{
			printf("\n");
		}
	}
	if(HAL_OK == At24c02_Write_Amount_Byte(0, write_dat, 22))
	{
		printf("write ok\n");
	}
	else
	{
		printf("write fail\n");
	}
	HAL_Delay(50);
	if(HAL_OK == HAL_I2C_Mem_Read(&hi2c1, AT24C02_ADDR_READ, 0, I2C_MEMADD_SIZE_8BIT, recv_buf, 22, 0xFFFF))
	{
		printf("read ok\n");
		for(i = 0; i < 22; i++)
		{
			printf("0x%02X ", recv_buf[i]);
			if((i+1) % 8 == 0)
			{
				printf("\n");
			}
		}
	}
	else
	{
		printf("read fail\n");
	}

	while(1);
```
测试结果：

![mark](http://mculover666.cn/image/20190826/MjzEoRshe1it.png?imageslim)

将上面的读写地址由0改为5，再次测试：
```c
if(HAL_OK == At24c02_Write_Amount_Byte(5, write_dat, 22))
```
测试结果：

![mark](http://mculover666.cn/image/20190826/EbiIkU1mSSg2.png?imageslim)

至此，我们已经学会**如何使用硬件IIC接口读写EEPROM**，下一节将讲述如何使用硬件IIC接口读取环境光强度传感器数据（BH1750）。

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![mark](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)
