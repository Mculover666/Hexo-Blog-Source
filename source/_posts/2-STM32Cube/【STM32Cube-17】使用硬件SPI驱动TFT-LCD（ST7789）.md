---
title: 【STM32Cube_17】使用硬件SPI驱动TFT-LCD（ST7789）
tags: STM32CubeMX TFT-LCD SPI总线
categories: STM32CubeMX
abbrlink: 4251315252
date: 2019-08-07 09:48:56
---
本篇详细的记录了如何使用STM32CubeMX配置STM32L431RCT6的硬件SPI外设与ST7789通信，驱动16bit TFT-LCD 屏幕。
<!--more-->

# 0. 前言
我的一些个人观点：

>学习 SPI 外设驱动LCD屏幕没有必要手写驱动，学习这部分代码的目的是为了了解TFT-LCD的工作原理，每个像素点是如何显示的，**不要花过多的精力在弄明白每个命令的意思**，建议基于本驱动，学习一下打点，画线算法，画圆算法，画多边形算法等等，还可以学习显示英文字符，中文字符，最后还可以移植STemwin显示界面等等好玩的东西~

![mark](http://mculover666.cn/image/20190830/uNWWafiL7nsd.png?imageslim)

![mark](http://mculover666.cn/image/20190830/AE6zUCi3kdTR.png?imageslim)

![mark](http://mculover666.cn/image/20190830/H63SNPAj4yeq.png?imageslim)

# 1. 准备工作
## 硬件准备
- 开发板
首先需要准备一个开发板，这里我准备的是STM32L4的开发板（BearPi）：

![mark](http://mculover666.cn/image/20190806/9uiPTi5odYSj.png?imageslim)

- LCD屏幕
小熊派开发板板载LCD屏幕**大小1.3寸，分辨率240*240，色彩深度16bit，使用ST7789V2液晶控制器**。

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

![mark](http://mculover666.cn/image/20190829/dBG4s5JFoXGM.png?imageslim)

## 配置LCD控制GPIO

![mark](http://mculover666.cn/image/20190829/4Y0g6NTXlmsc.png?imageslim)

## 配置SPI2接口

查看小熊派LCD接口的原理图：

![mark](http://mculover666.cn/image/20190829/J5lfuBmdy0q2.png?imageslim)

![mark](http://mculover666.cn/image/20190829/rIsAkWbeqWjH.png?imageslim)

引脚对应表如下：

|LCD引脚|MCU引脚|
|:---:|:---:|
|SPI2_MOSI|PC3|
|SPI2_CLK|PB13|
|LCD_WR_RS|PC6|
|LCD_RESET|PC7|
|LCD_POWER|PB15|

MCU只需要通过SPI向LCD控制器发送命令/数据即可，所以硬件上接 SPI2 的 SCK 和 MOSI 引脚，软件上将SPI2配置为发送主机模式，接下来开始配置SPI2接口：

参数设置如下：

![mark](http://mculover666.cn/image/20190829/1DHh2ytSwgcp.png?imageslim)

SPI2默认SCK引脚是PB10，和开发板不对应，所以重新修改引脚为PB13：

![mark](http://mculover666.cn/image/20190829/J0O0uJFSEap1.png?imageslim)

## 配置时钟树
STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：
![mark](http://mculover666.cn/image/20190806/1TQg7frjRpVr.png?imageslim)

## 生成工程设置

![mark](http://mculover666.cn/image/20190829/MQ6wAeYd1mMw.png?imageslim)

## 代码生成设置
最后设置生成独立的初始化文件：
![mark](http://mculover666.cn/image/20190806/T6WvSK6Dfpts.png?imageslim)

## 生成代码
点击`GENERATE CODE`即可生成MDK-V5工程：
![mark](http://mculover666.cn/image/20190806/s0jGhLBWW6Cm.png?imageslim)

# 3. 编写LCD驱动（ST7789）—— 封装宏和底层函数

## 3.1. 封装控制LCD控制引脚高低电平的宏

控制引脚宏定义已经包含在 `main.h` 中，如图：

![mark](http://mculover666.cn/image/20190829/X2Pj10IvIdq0.png?imageslim)

在编写驱动的过程中需要不断的控制这些控制引脚的电平，所以首先在 `lcd_spi2_drv.h` 头文件中编写控制这些引脚的宏：
```c
#include "main.h"

#define	LCD_PWR(n)		(n?\
						HAL_GPIO_WritePin(LCD_PWR_GPIO_Port,LCD_PWR_Pin,GPIO_PIN_SET):\
						HAL_GPIO_WritePin(LCD_PWR_GPIO_Port,LCD_PWR_Pin,GPIO_PIN_RESET))
#define	LCD_WR_RS(n)	(n?\
						HAL_GPIO_WritePin(LCD_WR_RS_GPIO_Port,LCD_WR_RS_Pin,GPIO_PIN_SET):\
						HAL_GPIO_WritePin(LCD_WR_RS_GPIO_Port,LCD_WR_RS_Pin,GPIO_PIN_RESET))
#define	LCD_RST(n)		(n?\
						HAL_GPIO_WritePin(LCD_RST_GPIO_Port,LCD_RST_Pin,GPIO_PIN_SET):\
						HAL_GPIO_WritePin(LCD_RST_GPIO_Port,LCD_RST_Pin,GPIO_PIN_RESET))
```

## 3.2. 宏定义屏幕分辨率和颜色值

```c
//LCD屏幕分辨率定义
#define LCD_Width   240
#define LCD_Height  240
//颜色定义
#define WHITE   0xFFFF	//白色
#define YELLOW  0xFFE0	//黄色
#define BRRED   0XFC07  //棕红色
#define PINK    0XF81F	//粉色
#define RED     0xF800	//红色
#define BROWN   0XBC40  //棕色
#define GRAY    0X8430  //灰色
#define GBLUE   0X07FF	//兰色
#define GREEN   0x07E0	//绿色
#define BLUE    0x001F  //蓝色
#define BLACK   0x0000	//黑色
```

接下来开始在 `lcd_spi2_drv.c` 编写驱动程序~

## 3.3. 封装LCD控制引脚初始化函数

首先包含必要的头文件：
```c
#include "lcd_spi2_drv.h"
#include "gpio.h"
#include "spi.h"
```

这个函数只能在本文件内由LCD初始化函数调用，所以使用static修饰为静态的：

```c
/**
 *@brief    LCD控制引脚和通信接口初始化
 *@param    none
 *@retval   none
*/
static void LCD_GPIO_Init(void)
{
    /* 初始化引脚 */
	MX_GPIO_Init();
		
	/* 复位LCD */
    LCD_PWR(0);
    LCD_RST(0);
    HAL_Delay(100);
    LCD_RST(1);

	/* 初始化SPI2接口 */
    MX_SPI2_Init();
}
```

## 3.4. 封装LCD发送数据和发送命令函数
数据都是由 SPI2 的MOSI发送，由 LCD_WR_RS 引脚指明该数据是命令还是数据。

首先在 `spi.c` 的最后调用HAL库封装一个函数，供驱动程序调用：
```c
/* USER CODE BEGIN 1 */
/**
 * @brief    SPI 发送字节函数
 * @param    TxData	要发送的数据
 * @param    size	发送数据的字节大小
 * @return  0:写入成功,其他:写入失败
 */
uint8_t SPI_WriteByte(uint8_t *TxData,uint16_t size)
{
	return HAL_SPI_Transmit(&hspi2,TxData,size,1000);
}
/* USER CODE END 1 */
```
>不要忘了在spi.h中声明该函数！

然后基于spi发送字节函数，在**驱动文件中**继续封装一个向LCD发送数据的函数，一个向LCD发送命令的函数：

```c
/**
 * @brief   写命令到LCD
 * @param   cmd —— 需要发送的命令
 * @return  none
 */
static void LCD_Write_Cmd(uint8_t cmd)
{
    LCD_WR_RS(0);
    SPI_WriteByte(&cmd, 1);
}

/**
 * @brief   写数据到LCD
 * @param   dat —— 需要发送的数据
 * @return  none
 */
static void LCD_Write_Data(uint8_t dat)
{
    LCD_WR_RS(1);
    SPI_WriteByte(&dat, 1);
}
```

# 4. 编写LCD驱动（ST7789）—— 对照datasheet编程

## 4.1. 打开/关闭背光函数

这两个函数比较简单，直接调用控制LCD背光的引脚控制宏即可：

```c
/**
 * @breif   打开LCD显示背光
 * @param   none
 * @return  none
 */
void LCD_DisplayOn(void)
{
    LCD_PWR(1);
}
/**
 * @brief   关闭LCD显示背光
 * @param   none
 * @return  none
 */
void LCD_DisplayOff(void)
{
    LCD_PWR(0);
}
```

## 4.2. 指定显示RAM操作地址
根据数据手册，当要改变某个区域像素点的颜色时，首先应该确定X方向起始地址和X方向结束地址：

![mark](http://mculover666.cn/image/20190829/SfifEyv85boN.png?imageslim)

然后确定Y方向起始地址和Y方向结束地址：

![mark](http://mculover666.cn/image/20190829/zMRTyBS1Vv9j.png?imageslim)

最后再确定该区域内每个像素点的值(16bit)：

![mark](http://mculover666.cn/image/20190829/0CfYcB8Xj56X.png?imageslim)

综上，我们每次操作的时候都需要指定操作区域，所以编写该函数：

```c
/**
 * @brief   设置数据写入LCD显存区域
 * @param   x1,y1	—— 起点坐标
 * @param   x2,y2	—— 终点坐标
 * @return  none
 */
void LCD_Address_Set(uint16_t x1, uint16_t y1, uint16_t x2, uint16_t y2)
{
    /* 指定X方向操作区域 */
    LCD_Write_Cmd(0x2a);
    LCD_Write_Data(x1 >> 8);
    LCD_Write_Data(x1);
    LCD_Write_Data(x2 >> 8);
    LCD_Write_Data(x2);

    /* 指定Y方向操作区域 */
    LCD_Write_Cmd(0x2b);
    LCD_Write_Data(y1 >> 8);
    LCD_Write_Data(y1);
    LCD_Write_Data(y2 >> 8);
    LCD_Write_Data(y2);

    /* 发送该命令，LCD开始等待接收显存数据 */
    LCD_Write_Cmd(0x2C);
}
```
## 4.3. 清屏函数

编写完指定显存操作区域后，趁热打铁，编写清屏函数就很简单啦，直接调用上面编写的函数，指定操作地址为全屏幕，然后循环发送颜色值即可：

```c
#define LCD_TOTAL_BUF_SIZE	(240*240*2)
#define LCD_Buf_Size 1152
static uint8_t lcd_buf[LCD_Buf_Size];
/**
 * @brief   以一种颜色清空LCD屏
 * @param   color —— 清屏颜色(16bit)
 * @return  none
 */
void LCD_Clear(uint16_t color)
{
    uint16_t i, j;
    uint8_t data[2] = {0};  //color是16bit的，每个像素点需要两个字节的显存

    /* 将16bit的color值分开为两个单独的字节 */
    data[0] = color >> 8;
    data[1] = color;
    
    /* 显存的值需要逐字节写入 */
    for(j = 0; j < LCD_Buf_Size / 2; j++)
    {
        lcd_buf[j * 2] =  data[0];
        lcd_buf[j * 2 + 1] =  data[1];
    }
    /* 指定显存操作地址为全屏幕 */
    LCD_Address_Set(0, 0, LCD_Width - 1, LCD_Height - 1);
    /* 指定接下来的数据为数据 */
    LCD_WR_RS(1);
    /* 将显存缓冲区的数据全部写入缓冲区 */
    for(i = 0; i < (LCD_TOTAL_BUF_SIZE / LCD_Buf_Size); i++)
    {
        SPI_WriteByte(lcd_buf, (uint16_t)LCD_Buf_Size);
    }
}
```

## 4.4. LCD初始化函数
至此，LCD的一些操作函数全部编写完成，最后编写初始化LCD模式的函数：
```c
/**
 * @brief   LCD初始化
 * @param   none
 * @return  none
 */
void LCD_Init(void)
{
    /* 初始化和LCD通信的引脚 */
    LCD_GPIO_Init();
    HAL_Delay(120);
	
    /* 关闭睡眠模式 */
    LCD_Write_Cmd(0x11);
    HAL_Delay(120);

    /* 开始设置显存扫描模式，数据格式等 */
    LCD_Write_Cmd(0x36);
    LCD_Write_Data(0x00);
    /* RGB 5-6-5-bit格式  */
    LCD_Write_Cmd(0x3A);
    LCD_Write_Data(0x65);
    /* porch 设置 */
    LCD_Write_Cmd(0xB2);
    LCD_Write_Data(0x0C);
    LCD_Write_Data(0x0C);
    LCD_Write_Data(0x00);
    LCD_Write_Data(0x33);
    LCD_Write_Data(0x33);
    /* VGH设置 */
    LCD_Write_Cmd(0xB7);
    LCD_Write_Data(0x72);
    /* VCOM 设置 */
    LCD_Write_Cmd(0xBB);
    LCD_Write_Data(0x3D);
    /* LCM 设置 */
    LCD_Write_Cmd(0xC0);
    LCD_Write_Data(0x2C);
    /* VDV and VRH 设置 */
    LCD_Write_Cmd(0xC2);
    LCD_Write_Data(0x01);
    /* VRH 设置 */
    LCD_Write_Cmd(0xC3);
    LCD_Write_Data(0x19);
    /* VDV 设置 */
    LCD_Write_Cmd(0xC4);
    LCD_Write_Data(0x20);
    /* 普通模式下显存速率设置 60Mhz */
    LCD_Write_Cmd(0xC6);
    LCD_Write_Data(0x0F);
    /* 电源控制 */
    LCD_Write_Cmd(0xD0);
    LCD_Write_Data(0xA4);
    LCD_Write_Data(0xA1);
    /* 电压设置 */
    LCD_Write_Cmd(0xE0);
    LCD_Write_Data(0xD0);
    LCD_Write_Data(0x04);
    LCD_Write_Data(0x0D);
    LCD_Write_Data(0x11);
    LCD_Write_Data(0x13);
    LCD_Write_Data(0x2B);
    LCD_Write_Data(0x3F);
    LCD_Write_Data(0x54);
    LCD_Write_Data(0x4C);
    LCD_Write_Data(0x18);
    LCD_Write_Data(0x0D);
    LCD_Write_Data(0x0B);
    LCD_Write_Data(0x1F);
    LCD_Write_Data(0x23);
    /* 电压设置 */
    LCD_Write_Cmd(0xE1);
    LCD_Write_Data(0xD0);
    LCD_Write_Data(0x04);
    LCD_Write_Data(0x0C);
    LCD_Write_Data(0x11);
    LCD_Write_Data(0x13);
    LCD_Write_Data(0x2C);
    LCD_Write_Data(0x3F);
    LCD_Write_Data(0x44);
    LCD_Write_Data(0x51);
    LCD_Write_Data(0x2F);
    LCD_Write_Data(0x1F);
    LCD_Write_Data(0x1F);
    LCD_Write_Data(0x20);
    LCD_Write_Data(0x23);
    /* 显示开 */
    LCD_Write_Cmd(0x21);
    LCD_Write_Cmd(0x29);
    
    /* 清屏为白色 */
    LCD_Clear(WHITE);

    /*打开显示*/
    LCD_PWR(1);
}
```


至此，驱动编写完成。

# 5. 测试驱动程序

在 `main函数` 中编写驱动测试代码，在 `while(1)` 之前添加如下代码：

```c
  /* USER CODE BEGIN 2 */
  LCD_Init();
  LCD_Clear(GREEN);
  /* USER CODE END 2 */
```
测试结果如图：

![mark](http://mculover666.cn/image/20190829/2rf6XcYSMCAE.png?imageslim)

绿绿的，是不是很好看哈哈(斜眼笑.jpg)~


至此，我们已经学会**如何使用硬件SPI驱动LCD屏幕（ST7789）**，下一节将讲述如何使用硬件QSPI接口读写SPI Flash的数据。

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![mark](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)