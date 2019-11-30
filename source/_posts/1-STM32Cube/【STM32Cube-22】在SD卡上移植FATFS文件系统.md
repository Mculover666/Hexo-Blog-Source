---
title: 【STM32Cube-22】在SD卡上移植FATFS文件系统
tags: STM32CubeMX SD Card SDMMC接口 FATFS文件系统
categories: STM32CubeMX
abbrlink: 2214138023
date: 2019-08-12 09:48:56
---

本篇详细的记录了如何使用STM32CubeMX移植FATFS文件系统到SD卡上。

<!--more-->

# 1. 准备工作
## 硬件准备
- 开发板
首先需要准备一个开发板，这里我准备的是STM32L4的开发板（BearPi）：

![](http://mculover666.cn/image/20190806/9uiPTi5odYSj.png?imageslim)

- Micro SD卡
小熊派开发板板载 Micro SD 卡槽，需要提前自行准备一张 Micro SD卡，如图：

![](http://mculover666.cn/image/20190903/CgHfN2loFYyx.png?imageslim)

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

![](http://mculover666.cn/image/20190829/dBG4s5JFoXGM.png?imageslim)

## 配置串口
小熊派开发板板载ST-Link并且虚拟了一个串口，原理图如下：

![](http://mculover666.cn/image/20190814/IwyXONVefPx9.png?imageslim)

这里我将开关拨到`AT-MCU`模式，使PC的串口与USART1之间连接。

接下来开始配置`USART1`：

![](http://mculover666.cn/image/20190814/nLMRMYtmzghl.png?imageslim)

## 配置 SDMMC 接口
>知识小卡片 —— SDMMC接口

SDMMC接口的全称叫`SD/SDIO MMC card host interface`，SD/SDIO MMC 卡 主机接口，通俗的来说，就是这个接口支持SD卡，支持SDIO设备，支持MMC卡。

>知识小卡片结束啦~

首先查看小熊派开发板的原理图：

![](http://mculover666.cn/image/20190903/5p7dMFxj3rSE.png?imageslim)

然后根据原理图配置 SDMMC 接口：

![](http://mculover666.cn/image/20190902/uMcmVYpIsIcy.png?imageslim)

## 配置FATFS文件系统

使用STM32CubeMX配置FATFS文件系统非常方便，只需要在软件中开启即可，软件会自动帮我们移植好。

这里需要修改两个配置：

- 开启文件名支持简体中文；
- 开启长文件名支持，并将长文件名动态缓存在栈中（普通文件名最多8个字节，开启长文件名支持后可达255个字节）

![](http://mculover666.cn/blog/20191022/P5AzTDIfJnIS.png?imageslim)

## 配置时钟树
STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：

![](http://mculover666.cn/image/20190902/Bp7T9w424Prq.png?imageslim)

## 生成工程设置

因为之前开启FATFS选择了长文件名动态缓存在栈中，所以我们要将栈空间修改大一点：

![](http://mculover666.cn/blog/20191022/IqNwGwcesMXv.png?imageslim)

## 代码生成设置
最后设置生成独立的初始化文件：

![](http://mculover666.cn/image/20190806/T6WvSK6Dfpts.png?imageslim)

## 生成代码
点击`GENERATE CODE`即可生成MDK-V5工程：

![](http://mculover666.cn/image/20190806/s0jGhLBWW6Cm.png?imageslim)

# 3. 在MDK中编写、编译、下载用户代码

## 重定向printf( )函数

参考：[【STM32Cube_09】重定向printf函数到串口输出的多种方法](https://www.mculover666.cn/2019/07/30/STM32Cube/【STM32Cube-09】重定向printf函数到串口输出的多种方法/)。

## SD卡分区并格式化为FAT文件系统

>正常SD卡不需要该步骤！

如果已经使用SD卡进行了裸机读写SD卡的实验（[【STM32Cube-19】使用SDMMC接口读写SD卡数据](https://www.mculover666.cn/posts/3022954032/)），那么需要注意：该实验中读写的是0扇区，实验之后**已经破坏了SD卡的分区表和FAT文件系统信息**！

重新建立SD卡的分区表和FAT文件系统有两种方法：

- 使用FATFS提供的API
- 在PC上直接格式化
- 在PC上使用`DiskGenius`软件重新分区和格式化

这里我使用第二种方法，比较简单方便，如果对FATFS提供的API感兴趣，请前去FATFS官网查看：

首先使用读卡器将SD卡插到电脑上，会显示如下：

![](http://mculover666.cn/blog/20191022/M3apb1xdgjSJ.png?imageslim)

然后直接右键选择格式化：

![](http://mculover666.cn/blog/20191022/tMwv1QRhKWkN.png?imageslim)

如果第二种方法没用的话，可以使用第三种方法，来打开 `DiskGenius` 软件查看SD卡：

![](http://mculover666.cn/blog/20191022/10vaq858v4oe.png?imageslim)

重新建立分区表并格式化：

![](http://mculover666.cn/blog/20191022/adQLdhB2DX44.png?imageslim)

![](http://mculover666.cn/blog/20191022/KmTfhw3FGdUT.png?imageslim)

![](http://mculover666.cn/blog/20191022/zsb1s4h0dtE8.png?imageslim)

之后可以看到SD卡恢复正常，可以进行FATFS实验啦：

![](http://mculover666.cn/blog/20191022/4h4Nhsweutwt.png?imageslim)

## 使用FATFS挂载SD卡

>注意：在挂载之前必须要保证SD卡正常拥有FAT文件系统。

挂载文件系统使用`f_mount` API，该API将文件系统对象注册/注销到FatFs模块，API原型如下：
```c
FRESULT f_mount (
  FATFS*       fs,    /* [IN] Filesystem object */
  const TCHAR* path,  /* [IN] Logical drive number */
  BYTE         opt    /* [IN] Initialization option */
);
```

在main.c文件中添加如下代码，先定义FATFS所使用的一些全局变量：

```c
/* Private variables ---------------------------------------------------------*/

/* USER CODE BEGIN PV */
FATFS   fs;			/* FATFS 文件系统对象 */
FRESULT fr; 		/* FATFS API 返回值 */
/* USER CODE END PV */
```
然后在 main 函数中,while(1)之前添加如下代码：

```c
/* USER CODE BEGIN 2 */
printf("FATFS test...\r\n");
/* 挂载SD卡 */
fr = f_mount(&fs, "", 0);
if(fr == FR_OK)
{
    printf("SD card mount ok!\r\n");
}
else
{
    printf("SD card mount error, error code:%d.\r\n",fr);
}
/* USER CODE END 2 */
```
编译下载，运行结果如下：

![](http://mculover666.cn/blog/20191022/sVUKbrsxp6Hc.png?imageslim)

## 创建文件并向文件中写入内容

要想操作文件，需要先创建文件对象：
```c
/* USER CODE BEGIN PV */
FATFS   fs;			/* FATFS 文件系统对象 */
FRESULT fr; 		/* FATFS API 返回值 */
FIL     fd;         /* FATFS 文件对象    */
/* USER CODE END 2 */
```

在main函数中的开始定义要写入文件的内容：
```c
/* USER CODE BEGIN 1 */
//要操作的文件名
char filename[] = "test.txt";
//文件写入内容
uint8_t write_dat[] = "Hello,FATFS!\n";
//用于接收API返回写入成功的字节数
uint16_t write_num = 0;
/* USER CODE END 1 */
```
然后**在挂载操作成功之后**进行`打开->写入->关闭`一个完整的操作：
```c
/* 打开文件（若文件不存在则创建） */
fr = f_open(&fd, filename, FA_CREATE_ALWAYS | FA_WRITE);
if(fr == FR_OK)
{
    printf("open file \"%s\" ok! \r\n", filename);
}
else
{
printf("open file \"%s\" error : %d\r\n", filename, fr);
}

/* 向打开的文件中写入内容 */
fr = f_write(&fd, write_dat, sizeof(write_dat), (void *)&write_num);
if(fr == FR_OK)
{
    printf("write %d dat to file \"%s\" ok,dat is \"%s\".\r\n", write_num, filename, write_dat);
}
else
{
    printf("write dat to file \"%s\" error,error code is:%d\r\n", filename, fr);
}

/* 操作完成，关闭文件 */
fr = f_close(&fd);
if(fr == FR_OK)
{
    printf("close file \"%s\" ok!\r\n", filename);
}
else
{
    printf("close file \"%s\" error, error code is:%d.\r\n", filename, fr);
}
```

实验结果如下：

![](http://mculover666.cn/blog/20191022/4fevYJlQoYwc.png?imageslim)

再将SD卡插到电脑，可以看到文件及其内容：

![](http://mculover666.cn/blog/20191022/peWQKMqHHJv8.png?imageslim)

## 读取SD卡中的文件内容

同样的，先在main函数开始开辟一块缓冲区，用于存放读取的数据：

```c
/* USER CODE BEGIN 1 */
//要操作的文件名
char filename[] = "test.txt";
//文件写入内容
uint8_t write_dat[] = "Hello,FATFS!";
//用于接收API返回写入成功的字节数
uint16_t write_num = 0;

//用于存放从文件中读取出的内容
uint8_t read_dat[20];
//用于接收API返回成功读取的字节数
uint16_t read_num = 0;
/* USER CODE END 1 */
```
然后进行`打开->读取->关闭`一个完整的操作：
```c
/* 打开文件用于读取 */
fr = f_open(&fd, filename, FA_READ);
if(fr == FR_OK)
{
    printf("open file \"%s\" ok! \r\n", filename);
}
else
{
printf("open file \"%s\" error : %d\r\n", filename, fr);
}

/* 从打开的文件中读取内容 */
fr = f_read(&fd, read_dat, sizeof(read_dat), (void *)&read_num);
if(fr == FR_OK)
{
    printf("read %d dat to file \"%s\" ok,dat is \"%s\".\r\n", read_num, filename, read_dat);
}
else
{
    printf("read dat to file \"%s\" error,error code is:%d\r\n", filename, fr);
}

/* 操作完成，关闭文件 */
fr = f_close(&fd);
if(fr == FR_OK)
{
    printf("close file \"%s\" ok!\r\n", filename);
}
else
{
    printf("close file \"%s\" error, error code is:%d.\r\n", filename, fr);
}
```
实验现象如下：

![](http://mculover666.cn/blog/20191022/AJ4liJi5FLkW.png?imageslim)

## FATFS API 错误码的使用

不知道大家有没有注意到，在本文中所有使用FATFS API的时候，都是如下的格式：

- 使用`FRESULT`类型的变量fr接收API返回值
- API执行之后进行判断，错误的话输出错误码

那么，API 所返回的错误码，有什么用呢？下面用一个实例来给大家演示一下~

假如本文中的实验现象如下：

![](http://mculover666.cn/blog/20191022/myqAnyjE4EFG.png?imageslim)

可以看到，FATFS创建文件时，返回的错误码是13，那么如何定位该问题呢？`13`代表什么？

打开FATFS的`ff.h`文件即可看到所有错误码所表示的含义：

![](http://mculover666.cn/blog/20191022/ii3BbPouSbzf.png?imageslim)

这样问题就定位到了，**我们使用的SD卡是之前用于裸机实验的卡，SD卡分区被破坏，SD卡文件系统被破坏，所以FATFS创建文件时才会提示`FR_NO_FILESYSTEM`问题**。


至此，我们已经学会如何在SD卡上移植​FATFS文件系统。

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)

