---
title: 【STM32Cube_09】重定向printf函数到串口输出的多种方法
tags: STM32CubeMX
categories: STM32CubeMX
abbrlink: 2251182441
date: 2019-07-30 09:00:00
---
本文详细的介绍了如何重定向printf输出到串口输出的多种方法，包括调用MDK微库（MicroLib）的方法，调用标准库的方法，以及适用于 `GNUC` 系列编译器的方法。

<!--more-->
# 1.printf与fputc

对于 printf 函数相信大家都不陌生，第一个C语言程序就是使用 printf 函数在屏幕上的控制台打印出`Hello World`，之后使用 printf 函数输出各种类型的数据，使用格式控制输出各种长度的字符，甚至输出各种各样的图案。

除此之外，在程序出错的时候，懒得调试，直接简单粗暴的加个 printf 找bug，有时候也不失为一种有效的方法。

对于已经习惯的 printf 函数，你了解多少呢？

printf 定义在 `<stdio.h>` 头文件中，如下：
```c
int printf(const char *format, ...);
```
printf 函数根据 `format` 字符串给出的格式打印输出到 `stdout`（标准输出）中，当然，**printf 函数是不会一个字符一个字符去输出，它会调用更底层的 I/O 函数：`fputc`去逐个字符打印**。

fputc 也定义于头文件 `<stdio.h>`中，如下：
```c
int fputc(int ch, FILE *stream);
```
fputc 函数写入字符 ch 到给定输出流 stream，printf函数在调用该函数时，会向stream参数传入`stdout`从而打印数据到标准输出。

那么，要实现printf打印到串口就变得非常简单了，**只需要重新定义fputc函数，在fputc的函数中将数据通过串口发送，称之为：fputc重定向或者printf重定向。**

# 2.在MDK中使用MicroLib重定向printf
## 勾选Use MicroLib
MicroLib是对标准C库进行了高度优化之后的库，供MDK默认使用，相比之下，MicroLIB的代码更少，资源占用更少：

![mark](http://mculover666.cn/image/20190819/oFhTzahaD7Kk.png?imageslim)

## 重定义fputc到串口
重新实现fputc函数，编写代码将这个字符通过串口发送，因为发送每个字符时都会调用该函数，所以**为了效率**，不再调用库函数 `HAL_UART_Transmit` 发送，而是直接操作寄存器发送。

- 检测串口当前状态

STM32L431的USART串口外设有一个 `ISR` 寄存器，全名 `Interrupt and status register`， 用来指示当前串口的状态，如图：

![mark](http://mculover666.cn/image/20190819/2hNLPDdErguD.png?imageslim)

其中 `BIT6 TC`用来指示当前串口是否发送完成，如图：

![mark](http://mculover666.cn/image/20190819/HqS3iDUO5Ce7.png?imageslim)

可以通过判断该位来判断串口当前是否处于发送状态，代码如下：
```c
while((USART1->ISR & 0X40) == 0);
```

- 串口发送字符ch

同样，为了提高发送效率，直接使用寄存器来操作：
```c
USART1->TDR = (uint8_t) ch;
```
最后实现fputc函数就变的非常简单了，这里我放在`usart.c`文件的末尾：
```c
/* USER CODE BEGIN 1 */
#if 1
#include <stdio.h>

int fputc(int ch, FILE *stream)
{
    /* 堵塞判断串口是否发送完成 */
    while((USART1->ISR & 0X40) == 0);

    /* 串口发送完成，将该字符发送 */
    USART1->TDR = (uint8_t) ch;

    return ch;
}
#endif

/* USER CODE END 1 */
```
## 测试printf
在main函数中测试一下printf函数是否可以正常使用：
```c
    /* USER CODE BEGIN 2 */
    printf("Hello, i am %s\n", "mculover666");
    printf("Test int: i = %d", 100);
    printf("Test float: i = %f", 1.234);
    printf("Test hex: i = 0x%2x",100);
    /* USER CODE END 2 */
```
结果如下：

![mark](http://mculover666.cn/image/20190819/l3kwHp8d4DPW.png?imageslim)

# 3.在MDK中使用标准库重定向printf

printf 函数使用了半主机模式，所以直接使用标准库会导致程序无法运行，因此必须提前告知编译器不使用半主机模式：

- 不使用半主机模式
```c
/* 告知连接器不从C库链接使用半主机的函数 */
#pragma import(__use_no_semihosting)

/* 定义 _sys_exit() 以避免使用半主机模式 */
void _sys_exit(int x)
{
    x = x;
}
```
所以，重定向fputc()函数完整的代码如下：
```c
#if 1
#include <stdio.h>

/* 告知连接器不从C库链接使用半主机的函数 */
#pragma import(__use_no_semihosting)

/* 定义 _sys_exit() 以避免使用半主机模式 */
void _sys_exit(int x)
{
    x = x;
}

/* 标准库需要的支持类型 */
struct __FILE
{
    int handle;
};

FILE __stdout;

/*  */
int fputc(int ch, FILE *stream)
{
    /* 堵塞判断串口是否发送完成 */
    while((USART1->ISR & 0X40) == 0);

    /* 串口发送完成，将该字符发送 */
    USART1->TDR = (uint8_t) ch;

    return ch;
}

#endif
```

## 测试printf
测试printf函数的代码不变，在MDK设置中取消勾选`USE MICROLIB`，然后重新编译，下载代码后试验现象如下：

![mark](http://mculover666.cn/image/20190819/8zhddFdIleFV.png?imageslim)

# 4.在GCC中使用标准库重定向printf

**不同的编译器对于C库的底层实现机制是不同的，所以上面两种在MDK中的实现方法，在使用Gcc编译器的时候是不可行的。**

在Gcc中重定向printf函数时注意两个关键点：

- 与重定义fputs()函数一样，在使用Gcc编译器的时候，需要重新定义`_write`函数；
- Gcc中没有MicroLib，只能使用标准库；

所以重定向printf函数的代码如下：
```c
/* USER CODE BEGIN 1 */
#if 1
#include <stdio.h>

int _write(int fd, char *ptr, int len)  
{  
  HAL_UART_Transmit(&huart1, (uint8_t*)ptr, len, 0xFFFF);
  return len;
}
#endif
/* USER CODE END 1 */
```
使用STM32CubeMX生成makefile，然后使用arm-none-eabi-gcc编译没有问题，再使用STM32 ST-LINK utility 下载后实验现象如下：

![mark](http://mculover666.cn/image/20190819/PDr0G8VvtqN9.png?imageslim)

至此，我们已经学会**实现printf()函数的多种方法**，下一节将讲述如何使用ADC读取MQ-2气体传感器的值。

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![mark](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)
