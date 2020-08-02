---
title: ringbuff | 通用FIFO环形缓冲区实现库
keywords: ringbuff
tags: 嵌入式开源项目精选专栏
categories: OpenSource
abbrlink:
summary: 嵌入式开源项目精选专栏
date: 2020-08-02 08:00:00
---

![](https://img-blog.csdnimg.cn/20200321151234798.png)

# 1. ringbuff 
本期给大家带来的开源项目是 ringbuff ，**一款通用FIFO环形缓冲区实现的开源库**，作者MaJerle，目前收获 79 个 star，遵循 MIT 开源许可协议。

目前 ringbuff 的特点有：

- 使用C99语法编写，并且没有平台相关代码；
- 没有动态内存分配；
- 使用更优的内存复制而不是循环从内存读取数据/向内存写入数据；

>项目地址：[https://github.com/MaJerle/ringbuff](https://github.com/MaJerle/ringbuff)

# 2. 移植ringbuff
## 2.1. 移植思路
在移植过程中主要参考两个资料：项目的readme文档和demo工程。

对于这些开源项目，其实移植起来也就两步：

- ① 添加源码到裸机工程中；
- ② 实现需要的接口即可；

## 2.2. 准备裸机工程
本文中我使用的是小熊派IoT开发套件，主控芯片为STM32L431RCT6：
![](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2Jsb2cvMjAxOTA5MTkvekh4NXhNWVVtU1cxLnBuZw?x-oss-process=image/format,png#pic_center)

移植之前需要准备一份裸机工程，我使用STM32CubeMX生成，需要初始化以下配置：

- 配置一个串口，中断方式接收数据，查询方式发送数据；
- printf重定向；

# 2.3. 添加ringbuff 到工程中
① 复制 ringbuff 源码到工程中：
![](https://img-blog.csdnimg.cn/20200531142100461.png)

② 在keil中添加 ringbuff 组件的源码文件：
![](https://img-blog.csdnimg.cn/20200531142140594.png)

③ 添加 ringbuff 的头文件路径：
![](https://img-blog.csdnimg.cn/20200531142257553.png)

## 2.4. 配置ringbuff
ringbuff中默认volatile关键词没有定义，需要手动配置一下，在`ringbuff.h`中：
![](https://img-blog.csdnimg.cn/20200531142708689.png)
至此，ringbuff移植修改完成，可以愉快的使用ringbuff啦~

# 3. 使用ringbuff
## 3.1. 为什么使用ringbuff
缓冲区一般用于解决设备接收数据的速度和设备处理速度不匹配的情况下，防止丢包，通俗的来说就是：收到数据先存进缓冲区，等到CPU来处理的时候一次性取出处理。

缓冲区有两种形式，一种是数组，一种就是本文所介绍的环形缓冲区ringbuff。

相较于数组，**环形缓冲区对整段内存的利用达到最大**，并且使用非常方便，如下：

- ① 写入的时候不用手动维护下标，直接写入即可（由缓冲区的实现维护）；
- ② 读取的时候不用判断从哪里读，直接读取即可（有缓冲区的实现维护）

本文设计的一个简单的不定长串口协议如下：

![](https://img-blog.csdnimg.cn/20200606100513275.png)

- 数据类型：比如0x3F表示这是通道1的数据，0x4E表示通道2的数据；
- 数据长度：表示后面跟着有效数据的长度；
- 有效数据：有效字节数；
- 校验数据：省略；

接下来演示如何用环形缓冲区做到不丢包解析。

## 3.2. 计算缓冲区大小
假定数据每200ms处理一次，而数据10ms接收一次，每次接收的数据包长度为7个字节。

要想做到不丢包，就需要将200ms内接收到的所有数据包都存进缓冲区，所以缓冲区大小至少为：200/10*7 = 140 个字节。

保险起见，可以将缓冲区适当的扩大一下，设置为150个字节。

## 3.3. 初始化缓冲区
使用时包含头文件：
```c
#include "ringbuff/ringbuff.h"
```
接着初始化缓冲区：
```c
uint8_t	ringbuff_init(RINGBUFF_VOLATILE ringbuff_t* buff, void* buffdata, size_t size);
```
该 API 用来初始化一个ringbuff句柄（指向ringbuff结构体的指针），其中传入的参数分别为：

- `buff`：ringbuff句柄；
- `buffdata`：缓冲区地址；
- `size`：缓冲区大小；

首先创建一个缓冲区句柄，开辟一块缓冲区：
```c
/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */
//用于串口接收
uint8_t recv_data = 0;

//用于存储从缓冲区读取出的数据
uint8_t read_data = 0;

//用于串口1的ringbuff句柄
ringbuff_t	usart1_ringbuff;

//开辟一块内存用于缓冲区
#define USART1_BUFFDATA_SIZE	150
uint8_t usart1_buffdata[USART1_BUFFDATA_SIZE];

/* USER CODE END 0 */
```

然后在main函数中初始化ringbuff：
```c
/* USER CODE BEGIN 2 */
printf("ringbuff Port By Mculover666\r\n");

//初始化ringbuff句柄
if(1 != ringbuff_init(&usart1_ringbuff, (uint8_t*)usart1_buffdata, USART1_BUFFDATA_SIZE))
{
	printf("usart1 ringbuff init fail.\r\n");
}

//使能串口中断接收
HAL_UART_Receive_IT(&huart1, (uint8_t*)&recv_data, 1);

/* USER CODE END 2 */
```
## 3.4. 数据接收
接收到一个字节数据后，话不多说，直接往缓冲区扔：
```c
/* USER CODE BEGIN 4 */
/* 中断回调函数 */
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
    /* 判断是哪个串口触发的中断 */
    if(huart ->Instance == USART1)
    {
		/* 将接收到的数据写入缓冲区 */
		ringbuff_write(&usart1_ringbuff, &recv_data, 1);
        //重新使能串口接收中断
        HAL_UART_Receive_IT(huart, (uint8_t*)&recv_data, 1);
    }
}
/* USER CODE END 4 */
```
## 3.5. 数据处理
数据处理在while(1)中进行，每隔200ms将缓冲区数据全部读出进行处理：
```c
 /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* USER CODE END WHILE */
	  

    /* USER CODE BEGIN 3 */
	while((len = ringbuff_read(&usart1_ringbuff, (uint8_t*)&read_data, sizeof(read_data))) > 0)
	{
		/* 捕获起始标志 */
		if(read_data == 0x3F)
		{
			//读取数据字节数，最大支持0xFF
			if((len = ringbuff_read(&usart1_ringbuff, (uint8_t*)&read_data, sizeof(read_data))) > 0)
			{
				data_len = read_data;
				printf("your data has %d byte(s):\r\n\t", data_len);
			}
			
			//提取data_len个数据
			for(i = 0; i < data_len; i++)
			{
				if((len = ringbuff_read(&usart1_ringbuff, (uint8_t*)&read_data, sizeof(read_data))) > 0)
				{
					printf("[0x%02x] ", read_data); 
				}
			}
			printf("over\r\n");
		}
	}
	HAL_Delay(200);
	  
  }
  /* USER CODE END 3 */
```
编译下载测试，实验结果如下，可以做到不丢包解析：
![](https://img-blog.csdnimg.cn/20200606104034363.png)
## 3.6. 丢包测试
经过3.2节的测试，不丢包的最小缓冲区大小是140个字节，接下里我们将缓冲区大小修改为100个字节，测试一下是否产生丢包：
```c
//开辟一块内存用于缓冲区
#define USART1_BUFFDATA_SIZE	100		//会发生丢包
//#define USART1_BUFFDATA_SIZE	150		//10ms接收7byte的协议包时不丢包
uint8_t usart1_buffdata[USART1_BUFFDATA_SIZE];
```
再次编译下载，查看串口输出：
![](https://img-blog.csdnimg.cn/20200606104706159.png)
# 4. 设计思想解读
关于环形缓冲区背后的设计实现，请阅读这篇文章，写的非常棒：

 - [STM32进阶之串口环形缓冲区实现](https://blog.csdn.net/jiejiemcu/article/details/80563422)

# 5. 项目工程源码获取和问题交流
目前我将ringbuff源码、我移植到小熊派STM32L431RCT6开发板的工程源码上传到了QQ群里（包含好几份HAL库，QQ相对速度快点），**可以在QQ群里下载，有问题也可以在群里交流，当然也欢迎大家分享出来自己移植的工程到QQ群里**：
![](https://img-blog.csdnimg.cn/20200606105554888.png)
放上QQ群二维码：
![](https://img-blog.csdnimg.cn/20200321144543508.png)

<font color="red">**接收更多精彩文章及资源推送，欢迎订阅我的微信公众号：『mculover666』。**</font>
![](https://img-blog.csdnimg.cn/20200202092055136.png)
