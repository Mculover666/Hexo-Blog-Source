---
title: STM32CubeMX | 28-片内Flash的使用
keywords: STM32CubeMX 片内Flash
tags: STM32CubeMX Flash
categories: STM32CubeMX
abbrlink: 2342727574
date: 2020-06-10 12:29:03
---
本篇文章主要介绍如何使用STM32中的片内FLash。

<!--more-->

# 1. 准备工作
## 硬件准备
- 开发板
首先需要准备一个开发板，这里我准备的是STM32L4的开发板（BearPi）：

![](http://mculover666.cn/image/20190806/9uiPTi5odYSj.png?imageslim)

## 软件准备
- 需要安装好Keil - MDK及芯片对应的包，以便编译和下载生成的代码；

# 2.生成MDK工程

>如果使用的是STM32F1系列，请先看这篇文章！！！（[STM32CubeMX生成F1的工程中造成 下载器无法下载 问题的解决方案](https://blog.csdn.net/Mculover666/article/details/104802410)）

## 选择芯片型号
打开STM32CubeMX，打开MCU选择器：

![](http://mculover666.cn/image/20190806/gBP6glmUSH80.png?imageslim)

搜索并选中芯片`STM32L431RCT6`:

![](http://mculover666.cn/image/20190806/gnyHwdl53uVD.png?imageslim)

## 配置时钟源

- 如果选择使用外部高速时钟（HSE），则需要在System Core中配置RCC；
- 如果使用默认内部时钟（HSI），这一步可以略过；

这里我都使用外部时钟：

![](http://mculover666.cn/blog/20200320/xjfBJCiQiIqs.png?imageslim)


## 配置串口
小熊派开发板板载ST-Link并且虚拟了一个串口，原理图如下：

![mark](http://mculover666.cn/image/20190814/IwyXONVefPx9.png?imageslim)

这里我将开关拨到`AT-MCU`模式，使PC的串口与USART1之间连接。

接下来开始配置`USART1`：

![mark](http://mculover666.cn/image/20190814/nLMRMYtmzghl.png?imageslim)


## 配置时钟树

STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：

![](http://mculover666.cn/image/20190806/1TQg7frjRpVr.png?imageslim)

## 生成工程设置

![](http://mculover666.cn/blog/20200610/zENXSh7HqWOq.png?imageslim)

## 代码生成设置

最后设置生成独立的初始化文件：

![](http://mculover666.cn/image/20190812/PwTCS6QzHiyG.png?imageslim)

## 生成代码

点击`GENERATE CODE`即可生成MDK-V5工程：

![](http://mculover666.cn/image/20190806/s0jGhLBWW6Cm.png?imageslim)

# 3. 在MDK中编写、编译、下载用户代码
## 重定向printf

- [STM32CubeMX_09 | 重定向printf函数到串口输出的多种方法](http://www.mculover666.cn/posts/2251182441/)

## STM32内部Flash及HAL库API

查看所使用芯片的信息，Flash起始地址为0x08000000，大小为0x00040000（256KB）：

![](http://mculover666.cn/blog/20200610/lI9tcVKRXJ6g.png?imageslim)

STM32L4x1芯片内部的Flash存储器内存分布如下：

![](http://mculover666.cn/blog/20200610/ENQ89nE2Nb81.png?imageslim)

**STM32L431RCT6的Flash容量是256KB,所以只有Bank1，有128页，每页2KB。**
### 解锁/上锁Flash操作

擦除或者写入内部Flash的时候，**需要先解锁再操作**，操作完毕之后上锁：

```c
HAL_StatusTypeDef  HAL_FLASH_Unlock(void);
HAL_StatusTypeDef  HAL_FLASH_Lock(void);
```

### 擦除操作

HAL库中定义了一个Flash初始化结构体，如下：
```c
/**
  * @brief  FLASH Erase structure definition
  */
typedef struct
{
  uint32_t TypeErase;   /*!< Mass erase or page erase.
                             This parameter can be a value of @ref FLASH_Type_Erase */
  uint32_t Banks;       /*!< Select bank to erase.
                             This parameter must be a value of @ref FLASH_Banks
                             (FLASH_BANK_BOTH should be used only for mass erase) */
  uint32_t Page;        /*!< Initial Flash page to erase when page erase is disabled
                             This parameter must be a value between 0 and (max number of pages in the bank - 1)
                             (eg : 255 for 1MB dual bank) */
  uint32_t NbPages;     /*!< Number of pages to be erased.
                             This parameter must be a value between 1 and (max number of pages in the bank - value of initial page)*/
} FLASH_EraseInitTypeDef;
```
第一个参数TypeErase是参数类型，分为页擦除和块擦除：
```c
/** @defgroup FLASH_Type_Erase FLASH Erase Type
  * @{
  */
#define FLASH_TYPEERASE_PAGES     ((uint32_t)0x00)  /*!<Pages erase only*/
#define FLASH_TYPEERASE_MASSERASE ((uint32_t)0x01)  /*!<Flash mass erase activation*/
/**
  * @}
  */
```

第二个参数Banks是选择需要擦除哪一块：
```c
/** @defgroup FLASH_Banks FLASH Banks
  * @{
  */
#define FLASH_BANK_1              ((uint32_t)0x01)                          /*!< Bank 1   */
#if defined (STM32L471xx) || defined (STM32L475xx) || defined (STM32L476xx) || defined (STM32L485xx) || defined (STM32L486xx) || \
    defined (STM32L496xx) || defined (STM32L4A6xx) || defined (STM32L4R5xx) || \
    defined (STM32L4R7xx) || defined (STM32L4R9xx) || defined (STM32L4S5xx) || defined (STM32L4S7xx) || defined (STM32L4S9xx)
#define FLASH_BANK_2              ((uint32_t)0x02)                          /*!< Bank 2   */
#define FLASH_BANK_BOTH           ((uint32_t)(FLASH_BANK_1 | FLASH_BANK_2)) /*!< Bank1 and Bank2  */
#else
#define FLASH_BANK_BOTH           ((uint32_t)(FLASH_BANK_1))                /*!< Bank 1   */
#endif
/**
  * @}
  */
```

由参数中可以看到，STM32L431RCT6中只有Bank1可选。

第三个参数Page是初始化擦除页，在STM32L431RCT6中，该值范围是0-127。

第四个参数NbPages是要擦除的页数，在STM32L431RCT6中，在1-（127-初始化参数页的编号）。


擦除的时候，调用的API如下：
```c
HAL_StatusTypeDef HAL_FLASHEx_Erase(FLASH_EraseInitTypeDef *pEraseInit, uint32_t *PageError);
```

### 写入操作
```c
HAL_StatusTypeDef  HAL_FLASH_Program(uint32_t TypeProgram, uint32_t Address, uint64_t Data);
```
第一个参数是写入类型：
```c
/** @defgroup FLASH_Type_Program FLASH Program Type
  * @{
  */
#define FLASH_TYPEPROGRAM_DOUBLEWORD    ((uint32_t)0x00)  /*!<Program a double-word (64-bit) at a specified address.*/
#define FLASH_TYPEPROGRAM_FAST          ((uint32_t)0x01)  /*!<Fast program a 32 row double-word (64-bit) at a specified address.
                                                                 And another 32 row double-word (64-bit) will be programmed */
#define FLASH_TYPEPROGRAM_FAST_AND_LAST ((uint32_t)0x02)  /*!<Fast program a 32 row double-word (64-bit) at a specified address.
                                                                 And this is the last 32 row double-word (64-bit) programmed */
/**
  * @}
  */
```

### 读取操作

这个就不用API啦~CPU可以直接访问到地址的，读取就好。

## 实验内容

首先包含进来头文件：
```c
/* Private includes ----------------------------------------------------------*/
/* USER CODE BEGIN Includes */
#include <stdio.h>
#include <string.h> //使用到了memcpy
/* USER CODE END Includes */
```

然后定义一个测试数据长度：
```c
/* Private define ------------------------------------------------------------*/
/* USER CODE BEGIN PD */
#define LEN						10
/* USER CODE END PD */
```

编写测试函数：
```c
/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */
void Onchip_Flash_Test(void)
{
	int i;
	uint32_t PageError = 0;
	FLASH_EraseInitTypeDef FlashSet;
	HAL_StatusTypeDef status;
	
	uint32_t addr = 0x0803F800;
	uint32_t data_buf[LEN];
	
	/* 读取Flash内容 */
	memcpy(data_buf, (uint32_t*)addr, sizeof(uint32_t)*LEN);
	printf("read before erase:\r\n\t");
	for(i = 0;i < LEN;i++)
	{
		printf("0x%08x ", data_buf[i]);
	}
	printf("\r\n");
	
	/* 写入新的数据 */
	//擦除最后一页
	FlashSet.TypeErase = FLASH_TYPEERASE_PAGES;
	FlashSet.Banks = FLASH_BANK_1;
	FlashSet.Page = 127;
	FlashSet.NbPages = 1;
	//解锁Flash操作
	HAL_FLASH_Unlock();
	status = HAL_FLASHEx_Erase(&FlashSet, &PageError);
	HAL_FLASH_Lock();
	if(status != HAL_OK)
	{
		printf("erase fail, PageError = %d\r\n", PageError);
	}
	printf("erase success\r\n");
	
	
	/* 读取Flash内容 */
	memcpy(data_buf, (uint32_t*)addr, sizeof(uint32_t)*LEN);
	printf("read after erase:\r\n\t");
	for(i = 0;i < LEN;i++)
	{
		printf("0x%08x ", data_buf[i]);
	}
	printf("\r\n");

	//写入Flash内容
	HAL_FLASH_Unlock();
	for (i = 0; i < LEN * sizeof(uint32_t); i+=8)
	{
			//一个字是32位，一次写入两个字，64位，8个字节
			status = HAL_FLASH_Program(FLASH_TYPEPROGRAM_DOUBLEWORD, addr + i, (uint64_t)i);
			if(status != HAL_OK)
			{
				break;
			}
	}
	HAL_FLASH_Lock();
	if(i < LEN)
	{
		printf("write fail\r\n");
	}
	else
	{
		printf("write success\r\n");
	}

	/* 读取Flash内容 */
	addr = 0x0803F800;
	memcpy(data_buf, (uint32_t*)addr, sizeof(uint32_t)*LEN);
	printf("read after write:\r\n\t");
	for(i = 0;i < LEN;i++)
	{
		printf("0x%08x ", data_buf[i]);
	}
	printf("\r\n");
	
}
/* USER CODE END 0 */
```

最后在main.c 调用：
```c
/* USER CODE BEGIN 2 */
printf("stm32l4 onchip flash test...\r\n");
Onchip_Flash_Test();

/* USER CODE END 2 */
```

## 实验现象

编译、下载、实验现象如下：

![](http://mculover666.cn/blog/20200610/rF2tYnKICvuC.png?imageslim)


**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)