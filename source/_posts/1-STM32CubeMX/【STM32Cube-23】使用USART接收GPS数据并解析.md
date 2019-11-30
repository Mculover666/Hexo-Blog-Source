---
title: STM32CubeMX-23 | 使用USART接收GPS数据并解析(L80-R)
keywords: STM32CubeMX GPS GPS数据
tags: STM32CubeMX
categories: STM32CubeMX
abbrlink: 3463670498
date: 2019-08-13 09:48:56
---

本篇详细的记录了如何使用STM32CubeMX配置STM32L431RCT6的 USART 外设，接收 GPS 模块的数据并解析。
<!--more-->

# 1. 准备工作
## 硬件准备
- 开发板
首先需要准备一个开发板，这里我准备的是STM32L4的开发板（BearPi）：

![](http://mculover666.cn/image/20190806/9uiPTi5odYSj.png?imageslim)

- GPS模块（L80-R）
Quectel L80-R 是一款集成了贴片天线的紧凑型GPS模块，非常适合在物联网设备中使用，尤其适合在车载、个人跟踪、工业PDA及各种手持式设备中使用：

![](http://mculover666.cn/blog/20191031/STXCYEk4YMpO.png?imageslim)

GPS模块的原理图如下：

![](http://mculover666.cn/blog/20191031/XiH6ImpbeBCA.png?imageslim)

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

## 配置GPS使能引脚

小熊派开发板设置了一个使能引脚，用于控制GPS模块的电源：

![](http://mculover666.cn/blog/20191031/Lsu2Fdp80u0U.png?imageslim)

所以要配置这个使能引脚（PC9）：

![](http://mculover666.cn/blog/20191031/HOdzkKcfGbgs.png?imageslim)

## 配置调试串口
小熊派开发板板载ST-Link并且虚拟了一个串口，原理图如下：

![](http://mculover666.cn/image/20190814/IwyXONVefPx9.png?imageslim)

这里我将开关拨到`AT-MCU`模式，使PC的串口与USART1之间连接。

接下来开始配置`USART1`：

![](http://mculover666.cn/image/20190814/nLMRMYtmzghl.png?imageslim)


## 配置GPS模块通信串口

GPS模块与USART3串口相连接，接下来开始配置USART3，波特率9600：

![](http://mculover666.cn/blog/20191031/1b5EMIvH7vLK.png?imageslim)

## NVIC配置

配置 USART3 的中断优先级，首先选择一个中断优先级分组：

![](http://mculover666.cn/blog/20191031/8x88G2Sa6bLo.png?imageslim)

然后设置优先级：

![](http://mculover666.cn/blog/20191031/M8cHst9XdVHJ.png?imageslim)

## 配置时钟树
STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：
![](http://mculover666.cn/image/20190806/1TQg7frjRpVr.png?imageslim)

![](http://mculover666.cn/image/20190808/EVKCwrQNEWcl.png?imageslim)

![](http://mculover666.cn/image/20190806/Dje8nuTMdpQY.png?imageslim)

## 生成工程设置

![](http://mculover666.cn/blog/20191031/f6F3UpBjP0kM.png?imageslim)

## 代码生成设置
最后设置生成独立的初始化文件：

![](http://mculover666.cn/image/20190806/T6WvSK6Dfpts.png?imageslim)

## 生成代码
点击`GENERATE CODE`即可生成MDK-V5工程：

![](http://mculover666.cn/image/20190806/s0jGhLBWW6Cm.png?imageslim)

# 3. 在MDK中编写、编译、下载用户代码
## 重定向printf( )函数

参考：[【STM32Cube_09】重定向printf函数到串口输出的多种方法](http://www.mculover666.cn/posts/2251182441/)。

## 转发GPS模块的数据

GPS 使能后不断的接收信号定位，并输出数据，但是 GPS 模块与 USART3 连接，无法直接查看输出的数据，何谈解析，所以先将 USART3 接收到的数据使用 USART1 发送，在电脑上使用串口助手查看，如果对于USART的中断接收方式还不明白，可以查看这篇文章：[【STM32Cube_07】使用USART发送和接收数据（中断模式）](http://www.mculover666.cn/posts/1803605667/)。

首先在 `main.c` 中实现 USART3 接收中断的回调函数：
```c
/* USER CODE BEGIN 4 */
/* 中断回调函数 */
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
	static uint8_t ch;
	/* 判断是哪个串口触发的中断 */
	if(huart ->Instance == USART3)
	{
		//将接收到的数据发送
		HAL_UART_Transmit(&huart1, &ch, 1, 0Xffff);
		//重新使能串口接收中断
		HAL_UART_Receive_IT(&huart3, &ch, 1);
	}
}
/* USER CODE END 4 */
```

然后修改`main`函数如下：
```c
int main(void)
{
    HAL_Init();

    SystemClock_Config();

    /* Initialize all configured peripherals */
    MX_GPIO_Init();
    MX_USART1_UART_Init();
    MX_USART3_UART_Init();
    /* USER CODE BEGIN 2 */
	
    /* 使能USART3接收中断 */
    HAL_UART_Receive_IT(&huart3, (uint8_t*)gps_uart, 1);
	
	/* 使能GPS模块 */
    printf("L80-R GPS Module test...\r\n");
	HAL_GPIO_WritePin(GPS_EN_GPIO_Port,GPS_EN_Pin, GPIO_PIN_RESET);
	printf("GPE Enable ok.\r\n");
	
  /* USER CODE END 2 */

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
  }
  /* USER CODE END 3 */
}
```

编译下载，可以在串口助手上看到输出的数据：

刚上电时蓝色的LED灯保持常亮状态，表示未定位成功，数据如下：

![](http://mculover666.cn/blog/20191031/4amfHyTh318C.png?imageslim)

定位成功后蓝色的LED开始闪烁，数据如下：

![](http://mculover666.cn/blog/20191031/1TkQ75Is66J0.png?imageslim)

## 解析GPS的数据

将GPS数据转发取消，删除USART3添加的中断函数：

![](http://mculover666.cn/blog/20191101/vh0Dfo3ks0il.png?imageslim)

接下来开辟一块缓冲区，来存放接收的GPS数据，并且定义GPS数据结构体，用来存放解析GPS数据得到的经纬度信息：

```c
/* Private variables ---------------------------------------------------------*/
/* USER CODE BEGIN PV */
/***************************************************\
*GPS NMEA-0183协议重要参数结构体定义
*卫星信息
\***************************************************/
__packed typedef struct
{
	uint32_t latitude_bd;					//纬度   分扩大100000倍，实际要除以100000
	uint8_t nshemi_bd;						//北纬/南纬,N:北纬;S:南纬	
	uint32_t longitude_bd;			  //经度 分扩大100000倍,实际要除以100000
	uint8_t ewhemi_bd;						//东经/西经,E:东经;W:西经
}gps_msg;

/* E53_ST1传感器数据类型定义 ------------------------------------------------------------*/
typedef struct
{
		float    Longitude;				//经度
		float    Latitude;        //纬度
} E53_ST1_Data_TypeDef;

gps_msg              gpsmsg;
static unsigned char gps_uart[1000];
E53_ST1_Data_TypeDef E53_ST1_Data;

/* USER CODE END PV */

```

然后编写解析GPS数据的函数，先要在开头加上头文件支持：
```c
/* Private includes ----------------------------------------------------------*/
/* USER CODE BEGIN Includes */
#include <stdio.h>
#include <string.h>

/* USER CODE END Includes */
```
开始编写解析GPS数据所使用的函数：

```c
/* USER CODE BEGIN 4 */

/***************************************************\
* 函数名称: NMEA_Comma_Pos
*	函数功能：从buf里面得到第cx个逗号所在的位置
*	输入值：
*	返回值：0~0xFE，代表逗号所在位置的便宜
*				 	0xFF，代表不存在第cx个逗号
\***************************************************/

uint8_t NMEA_Comma_Pos(uint8_t *buf,uint8_t cx)
{
	uint8_t *p = buf;
	while(cx)
	{
		if(*buf=='*'||*buf<' '||*buf>'z')return 0xFF;
		if(*buf==',')cx--;
		buf++;
	}
	return buf-p;
}
/***************************************************\
* 函数名称: NMEA_Pow
*	函数功能：返回m的n次方值
*	输入值：底数m和指数n
*	返回值：m^n
\***************************************************/
uint32_t NMEA_Pow(uint8_t m,uint8_t n)
{
	uint32_t result = 1;
	while(n--)result *= m;
	return result;
}
/***************************************************\
* 函数名称: NMEA_Str2num
*	函数功能：str数字转换为（int）数字，以','或者'*'结束
*	输入值：buf，数字存储区
*				 	dx，小数点位数，返回给调用函数
*	返回值：转换后的数值
\***************************************************/
int NMEA_Str2num(uint8_t *buf,uint8_t*dx)
{
	uint8_t *p = buf;
	uint32_t ires = 0,fres = 0;
	uint8_t ilen = 0,flen = 0,i;
	uint8_t mask = 0;
	int res;
	while(1)
	{
		if(*p=='-'){mask |= 0x02;p++;}//说明有负数
		if(*p==','||*p=='*')break;//遇到结束符
		if(*p=='.'){mask |= 0x01;p++;}//遇到小数点
		else if(*p>'9'||(*p<'0'))//数字不在0和9之内，说明有非法字符
		{
			ilen = 0;
			flen = 0;
			break;
		}
		if(mask&0x01)flen++;//小数点的位数
		else ilen++;//str长度加一
		p++;//下一个字符
	}
	if(mask&0x02)buf++;//移到下一位，除去负号
	for(i=0;i<ilen;i++)//得到整数部分数据
	{
		ires += NMEA_Pow(10,ilen-1-i)*(buf[i]-'0');
	}
	if(flen>5)flen=5;//最多取五位小数
	*dx = flen;
	for(i=0;i<flen;i++)//得到小数部分数据
	{
		fres +=NMEA_Pow(10,flen-1-i)*(buf[ilen+1+i]-'0');
	}
	res = ires*NMEA_Pow(10,flen)+fres;
	if(mask&0x02)res = -res;
	return res;
}
/***************************************************\
* 函数名称: NMEA_BDS_GPRMC_Analysis
*	函数功能：解析GPRMC信息
*	输入值：gpsx,NMEA信息结构体
*				 buf：接收到的GPS数据缓冲区首地址
\***************************************************/
void NMEA_BDS_GPRMC_Analysis(gps_msg *gpsmsg,uint8_t *buf)
{
	uint8_t *p4,dx;			 
	uint8_t posx;     
	uint32_t temp;	   
	float rs;  
	p4=(uint8_t*)strstr((const char *)buf,"$GPRMC");//"$GPRMC",经常有&和GPRMC分开的情况,故只判断GPRMC.
	posx=NMEA_Comma_Pos(p4,3);								//得到纬度
	if(posx!=0XFF)
	{
		temp=NMEA_Str2num(p4+posx,&dx);		 	 
		gpsmsg->latitude_bd=temp/NMEA_Pow(10,dx+2);	//得到°
		rs=temp%NMEA_Pow(10,dx+2);				//得到'		 
		gpsmsg->latitude_bd=gpsmsg->latitude_bd*NMEA_Pow(10,5)+(rs*NMEA_Pow(10,5-dx))/60;//转换为° 
	}
	posx=NMEA_Comma_Pos(p4,4);								//南纬还是北纬 
	if(posx!=0XFF)gpsmsg->nshemi_bd=*(p4+posx);					 
 	posx=NMEA_Comma_Pos(p4,5);								//得到经度
	if(posx!=0XFF)
	{												  
		temp=NMEA_Str2num(p4+posx,&dx);		 	 
		gpsmsg->longitude_bd=temp/NMEA_Pow(10,dx+2);	//得到°
		rs=temp%NMEA_Pow(10,dx+2);				//得到'		 
		gpsmsg->longitude_bd=gpsmsg->longitude_bd*NMEA_Pow(10,5)+(rs*NMEA_Pow(10,5-dx))/60;//转换为° 
	}
	posx=NMEA_Comma_Pos(p4,6);								//东经还是西经
	if(posx!=0XFF)gpsmsg->ewhemi_bd=*(p4+posx);		  
}
/* USER CODE END 4 */
```

最后编写使用中断方式接收数据到缓冲区，然后调用GPS数据解析函数的函数：
```c
void E53_ST1_Read_Data(void)
{	
  /* 使用中断方式接收一次数据 */
	HAL_UART_Receive_IT(&huart3,gps_uart,1000);

  /* 分析缓冲区的字符串，解析GPS数据 */
	NMEA_BDS_GPRMC_Analysis(&gpsmsg,(uint8_t*)gps_uart);

  /* 将解析到的经纬度数据存放到结构体中，便于其他函数使用 */	
	E53_ST1_Data.Longitude=(float)((float)gpsmsg.longitude_bd/100000);	
	E53_ST1_Data.Latitude=(float)((float)gpsmsg.latitude_bd/100000);    
}
```
当然，别忘了在 `main` 函数之前声明这些函数：
```c
/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */
uint8_t NMEA_Comma_Pos(uint8_t *buf,uint8_t cx);
uint32_t NMEA_Pow(uint8_t m,uint8_t n);
int NMEA_Str2num(uint8_t *buf,uint8_t*dx);
void NMEA_BDS_GPRMC_Analysis(gps_msg *gpsmsg,uint8_t *buf);
void E53_ST1_Read_Data(void);
/* USER CODE END 0 */
```

大功告成，在main函数中调用：
```c
int main(void)
{
  HAL_Init();

  SystemClock_Config();

  /* Initialize all configured peripherals */
  MX_GPIO_Init();
  MX_USART1_UART_Init();
  MX_USART3_UART_Init();

  /* USER CODE BEGIN 2 */

  HAL_UART_Receive_IT(&huart3, (uint8_t*)gps_uart, 1);
  printf("L80-R GPS Module test...\r\n");

  /* 使能GPS模块 */
  HAL_GPIO_WritePin(GPS_EN_GPIO_Port,GPS_EN_Pin, GPIO_PIN_RESET);
  printf("GPE Enable ok.\r\n");

  /* USER CODE END 2 */

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
    E53_ST1_Read_Data();
    printf("Longitude: %f, Latitude: %f.\r\n", E53_ST1_Data.Longitude, E53_ST1_Data.Latitude);
    HAL_Delay(2000);
  }
  /* USER CODE END 3 */
}
```

## 实验现象

编译下载后，**尽量将小熊派开发板放在窗户边**，等待定位成功，串口输出数据如下：

![](http://mculover666.cn/blog/20191101/ere3mnVix0io.png?imageslim)

定位成功后，定位数据如下：

![](http://mculover666.cn/blog/20191101/PdwjPdmndKB8.png?imageslim)

## 查看定位

有了GPS定位数据之后，可以在地图上查看具体的位置，也可以上传到华为云IoT平台查看，这里我使用 GPS经纬度查询工具进行查看：

>工具链接：http://www.gpsspg.com/maps.htm

![](http://mculover666.cn/blog/20191101/ut7W41Mz90sV.png?imageslim)

至此，我们已经学会**如何接收GPS数据并解析出经纬度**。

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)

