---
title: 【STM32Cube-19】使用SDMMC接口读写SD卡数据
tags: STM32CubeMX SD Card SDMMC接口
categories: STM32CubeMX
abbrlink: 3022954032
date: 2019-08-09 09:48:56
---

本篇详细的记录了如何使用STM32CubeMX配置STM32L431RCT6的硬件SDMMC外设读取SD卡数据。

![mark](http://mculover666.cn/image/20190905/vyYEcOIniSlS.jpg?imageslim)

<!--more-->

# 1. 准备工作
## 硬件准备
- 开发板
首先需要准备一个开发板，这里我准备的是STM32L4的开发板（BearPi）：

![mark](http://mculover666.cn/image/20190806/9uiPTi5odYSj.png?imageslim)

- Micro SD卡
小熊派开发板板载 Micro SD 卡槽，最大支持 32 GB，需要提前自行准备一张 Micro SD卡，如图：

![mark](http://mculover666.cn/image/20190903/CgHfN2loFYyx.png?imageslim)

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

## 配置串口
小熊派开发板板载ST-Link并且虚拟了一个串口，原理图如下：

![mark](http://mculover666.cn/image/20190814/IwyXONVefPx9.png?imageslim)

这里我将开关拨到`AT-MCU`模式，使PC的串口与USART1之间连接。

接下来开始配置`USART1`：

![mark](http://mculover666.cn/image/20190814/nLMRMYtmzghl.png?imageslim)

## 配置 SDMMC 接口
>知识小卡片 —— SDMMC接口

SDMMC接口的全称叫`SD/SDIO MMC card host interface`，SD/SDIO MMC 卡 主机接口，通俗的来说，就是这个接口支持SD卡，支持SDIO设备，支持MMC卡。

>知识小卡片结束啦~

首先查看小熊派开发板的原理图：

![mark](http://mculover666.cn/image/20190903/5p7dMFxj3rSE.png?imageslim)

然后根据原理图配置 SDMMC 接口：

![mark](http://mculover666.cn/image/20190902/uMcmVYpIsIcy.png?imageslim)

## 配置时钟树
STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：

![mark](http://mculover666.cn/image/20190902/Bp7T9w424Prq.png?imageslim)

## 生成工程设置

![mark](http://mculover666.cn/image/20190902/9kbez0U85pn0.png?imageslim)

## 代码生成设置
最后设置生成独立的初始化文件：

![mark](http://mculover666.cn/image/20190806/T6WvSK6Dfpts.png?imageslim)

## 生成代码
点击`GENERATE CODE`即可生成MDK-V5工程：

![mark](http://mculover666.cn/image/20190806/s0jGhLBWW6Cm.png?imageslim)

# 3. 在MDK中编写、编译、下载用户代码

## 重定向printf( )函数

参考：[【STM32Cube_09】重定向printf函数到串口输出的多种方法](https://www.mculover666.cn/2019/07/30/STM32Cube/【STM32Cube-09】重定向printf函数到串口输出的多种方法/)。

## 读取SD卡信息并打印

SD 卡系统(包括主机和 SD 卡)定义了两种操作模式：

- 卡识别模式
- 数据传输模式

在**系统复位后，主机处于卡识别模式**，寻找总线上可用的 SD卡设备；同时，SD 卡也处于卡
识别模式，直到被主机识别到。

使用STM32CubeMX初始化的工程中会自动生成 SDMMC 初始化函数，向 SD 卡发送命令，**当 SD 卡接收到命令后， SD 卡就会进入数据传输模式，而主机在总线上所有卡被识别后也进入数据传输模式**。

所以在操作之前，需要先检查 SD 卡是否处于数据传输模式并且处于数据传输状态：

在`main`函数中首先定义一个变量用于存储 SD 卡状态：
```c
int sdcard_status = 0;
HAL_SD_CardCIDTypeDef sdcard_cid;
```
然后在`while(1)`之前编写如下读取信息代码：
```c
/* USER CODE BEGIN 2 */
printf("Micro SD Card Test...\r\n");

/* 检测SD卡是否正常（处于数据传输模式的传输状态） */
sdcard_status = HAL_SD_GetCardState(&hsd1);
if(sdcard_status == HAL_SD_CARD_TRANSFER)
{
    printf("SD card init ok!\r\n\r\n");
    
    //打印SD卡基本信息
    printf("SD card information!\r\n");
    printf("CardCapacity: %llu\r\n",((unsigned long long)hsd1.SdCard.BlockSize*hsd1.SdCard.BlockNbr));
    printf("CardBlockSize: %d \r\n",hsd1.SdCard.BlockSize);
    printf("RCA: %d \r\n",hsd1.SdCard.RelCardAdd);
    printf("CardType: %d \r\n",hsd1.SdCard.CardType);
    
    //读取并打印SD卡的CID信息
    HAL_SD_GetCardCID(&hsd1,&sdcard_cid);
    printf("ManufacturerID: %d \r\n",sdcard_cid.ManufacturerID);
}
else
{ 
    printf("SD card init fail!\r\n" );
    return 0;
}
/* USER CODE END 2 */
```
编译下载后串口助手输出结果如下：

![mark](http://mculover666.cn/image/20190903/Dyuawd1r1QmG.png?imageslim)

## 擦除SD卡块数据
为了验证实验的正确性或，先擦除数据：
```c
/* 擦除SD卡块 */
printf("------------------- Block Erase -------------------------------\r\n");
sdcard_status = HAL_SD_Erase(&hsd1, 0, 512);
if (sdcard_status == 0)
{
    printf("Erase block ok\r\n");
}
else
{
    printf("Erase block fail\r\n");
}
```
## 读取SD卡块数据
首先开辟一个**全局**缓冲区，用于存放从SD卡读出的数据：
```c
/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */
uint8_t read_buf[512];
/* USER CODE END 0 */
```
然后在之前读取信息的代码之后添加读取数据的代码：
```c
/* 读取未操作之前的数据 */
printf("------------------- Read SD card block data Test ------------------\r\n");
sdcard_status = HAL_SD_ReadBlocks(&hsd1,(uint8_t *)read_buf,0,1,0xffff);
if(sdcard_status == 0)
{ 
    printf("Read block data ok \r\n" );
    for(i = 0; i < 512; i++)
    {
        printf("0x%02x ", read_buf[i]);
        if((i+1)%16 == 0)
        {
            printf("\r\n");
        }
    }
}
else
{
    printf("Read block data fail!\r\n " );
}
```
## 向SD卡块写入数据
同样的，开辟一个**全局**缓冲区，用于存放即将要写入SD卡的数据：
```c
uint8_t write_buf[512];
```
然后在之前读取数据的代码之后添加的代码，将缓冲区的数据赋初值：
```c
/* 填充缓冲区数据 */
for(i = 0; i < 512; i++)
{
    write_buf[i] = i % 256;
}
```
然后继续添加代码，将该缓冲区数据写入SD卡：
```c
/* 向SD卡块写入数据 */
printf("------------------- Write SD card block data Test ------------------\r\n");
sdcard_status = HAL_SD_WriteBlocks(&hsd1,(uint8_t *)write_buf,0,1,0xffff);
if(sdcard_status == 0)
{ 
    printf("Write block data ok \r\n" );
}
else
{
    printf("Write block data fail!\r\n " );
}
```
添加完之后，为了检查数据是否正常写入，再将数据读出：
```
/* 读取操作之后的数据 */
printf("------------------- Read SD card block data after Write ------------------\r\n");
sdcard_status = HAL_SD_ReadBlocks(&hsd1,(uint8_t *)read_buf,0,1,0xffff);
if(sdcard_status == 0)
{ 
    printf("Read block data ok \r\n" );
    for(i = 0; i < 512; i++)
    {
        printf("0x%02x ", read_buf[i]);
        if((i+1)%16 == 0)
        {
            printf("\r\n");
        }
    }
}
```
将程序编译下载，最终的实验结果如下：

![mark](http://mculover666.cn/image/20190903/tzrRr67CIRLk.png?imageslim)

![mark](http://mculover666.cn/image/20190903/5XIe3u2Mh4mh.png?imageslim)

![mark](http://mculover666.cn/image/20190903/hvGkRtezU8OC.png?imageslim)

![mark](http://mculover666.cn/image/20190903/jVzz5QivHgBy.png?imageslim)

![mark](http://mculover666.cn/image/20190903/4raBihOAOhvb.png?imageslim)

至此，我们已经学会**如何使用硬件SDMMC接口读取SD数据**，STM32CubeMX系列教程完结。

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![mark](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)

