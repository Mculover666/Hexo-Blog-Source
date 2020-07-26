---
title: STM32CubeMX | 30-使用硬件SPI读写Flash(W25Q64)
keywords: STM32CubeMX W25Q64
tags: STM32CubeMX W25Q64
categories: STM32CubeMX
abbrlink: 858512530
date: 2020-07-26 14:29:03
---

本篇详细的记录了如何使用STM32CubeMX配置 STM32G070RBT6 的硬件SPI外设与 SPI Flash 通信（W25Q64）。

<!--more-->

# 1. 准备工作
## 硬件准备
- 开发板
首先需要准备一个开发板，这里我准备的是STM32G070RB的开发板

- SPI Flash
开发板板载一片SPI Flash，型号为 `W25Q64JV`，大小为 8 MB。

## 软件准备
- 需要安装好Keil - MDK及芯片对应的包，以便编译和下载生成的代码；
- 准备一个串口调试助手，这里我使用的是`Serial Port Utility`；

>Keil MDK和串口助手Serial Port Utility 的安装包都可以**在文末关注公众号获取**，回复关键字获取相应的安装包：

![mark](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwODE0L2d1YmFPd21FVHAxdy5wbmc?x-oss-process=image/format,png)

# 2.生成MDK工程
## 选择芯片型号
打开STM32CubeMX，打开MCU选择器：
![](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwODA2L2dCUDZnbG1VU0g4MC5wbmc?x-oss-process=image/format,png)

搜索并选中芯片`STM32G070RB`:
![](https://img-blog.csdnimg.cn/20200726092835777.png)

## 配置时钟源
- 如果选择使用外部高速时钟（HSE），则需要在System Core中配置RCC；
- 如果使用默认内部时钟（HSI），这一步可以略过；

这里我都使用内部时钟：
![](https://img-blog.csdnimg.cn/20200726092928549.png)


## 配置串口
开发板板载了一个CH340z换串口，连接到USART1。

接下来开始配置`USART1`：
![](https://img-blog.csdnimg.cn/20200726093117174.png)

## 配置SPI接口
开发板上SPI Flash的原理图如下：
![](https://img-blog.csdnimg.cn/20200726092623733.png#pic_center)
>原理图中虽然将CS片选接到了硬件SPI1的NSS引脚，因为硬件NSS使用比较麻烦，所以后面直接把PA4配置为普通GPIO，手动控制片选信号。


接下来配置 SPI1 接口。

配置SPI接口的时候有三个需要注意的点：

① 分频系数；
② CPOL：CLK空闲时候的电平为高电平或者低电平；
③ CPHA：在第1个时钟边缘采样，还是在第2个时钟边缘采样；

首先配置硬件SPI的模式：
![](https://img-blog.csdnimg.cn/20200726093714976.png)
接着根据W25Q64数据手册中给出的通信协议，配置SPI外设具体参数：
![](https://img-blog.csdnimg.cn/20200726093903610.png)
其实CPOL参数和CPHA参数，从W25Q64中随意找个时序图就可以看出，比如我以读取ID的时序为例：
![](https://img-blog.csdnimg.cn/20200726094226471.png)
## 配置时钟树
STM32G070RB的最高主频到64M，所以配置PLL，最后使`HCLK = 64Mhz`即可：
![](https://img-blog.csdnimg.cn/20200726094317380.png)

## 生成工程设置
![](https://img-blog.csdnimg.cn/20200726094350788.png)

## 代码生成设置
最后设置生成独立的初始化文件：
![](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwODA2L1Q2V3ZTSzZEZnB0cy5wbmc?x-oss-process=image/format,png)

## 生成代码
点击`GENERATE CODE`即可生成MDK-V5工程：

![](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwODA2L3MwakdoTEJXVzZDbS5wbmc?x-oss-process=image/format,png)

# 3. 重定向printf函数到USART1

参考：[【STM32Cube_09】重定向printf函数到串口输出的多种方法](http://www.mculover666.cn/posts/2251182441/)。

# 4. 封装 SPI Flash（W25Q64）的命令和底层函数

MCU 通过向 SPI Flash **发送各种命令** 来读写 SPI Flash内部的寄存器，所以这种裸机驱动，首先要先宏定义出需要使用的命令，然后利用 HAL 库提供的库函数，封装出三个底层函数，**便于移植**：

- 向 SPI Flash 发送数据的函数
- 从 SPI Flash 接收数据的函数
- 发送数据的同时读取数据的函数

接下来开始编写代码~

## 宏定义操作命令
```c
#define ManufactDeviceID_CMD	0x90
#define READ_STATU_REGISTER_1   0x05
#define READ_STATU_REGISTER_2   0x35
#define READ_DATA_CMD	        0x03
#define WRITE_ENABLE_CMD	    0x06
#define WRITE_DISABLE_CMD	    0x04
#define SECTOR_ERASE_CMD	    0x20
#define CHIP_ERASE_CMD	        0xc7
#define PAGE_PROGRAM_CMD        0x02
```
## 封装发送数据的函数
```c
/**
 * @brief    SPI发送指定长度的数据
 * @param    buf  —— 发送数据缓冲区首地址
 * @param    size —— 要发送数据的字节数
 * @retval   成功返回HAL_OK
 */
static HAL_StatusTypeDef SPI_Transmit(uint8_t* send_buf, uint16_t size)
{
    return HAL_SPI_Transmit(&hspi1, send_buf, size, 100);
}
```
## 封装接收数据的函数
```c
/**
 * @brief   SPI接收指定长度的数据
 * @param   buf  —— 接收数据缓冲区首地址
 * @param   size —— 要接收数据的字节数
 * @retval  成功返回HAL_OK
 */
static HAL_StatusTypeDef SPI_Receive(uint8_t* recv_buf, uint16_t size)
{
   return HAL_SPI_Receive(&hspi1, recv_buf, size, 100);
}
```
## 封装发送数据同时读取数据的函数
```c
/**
 * @brief   SPI在发送数据的同时接收指定长度的数据
 * @param   send_buf  —— 接收数据缓冲区首地址
 * @param   recv_buf  —— 接收数据缓冲区首地址
 * @param   size —— 要发送/接收数据的字节数
 * @retval  成功返回HAL_OK
 */
static HAL_StatusTypeDef SPI_TransmitReceive(uint8_t* send_buf, uint8_t* recv_buf, uint16_t size)
{
   return HAL_SPI_TransmitReceive(&hspi1, send_buf, recv_buf, size, 100);
}
```

# 5. 编写W25Q64的驱动程序

接下来开始利用上一节封装的宏定义和底层函数，编写W25Q64的驱动程序：

## 读取Manufacture ID和Device ID
读取 Flash 内部这两个ID有两个作用：

- 检测SPI Flash是否存在
- 可以根据ID判断Flash具体型号

数据手册上给出的操作时序如图：

![](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwOTAzL0drZnZmZ1JnUkE1My5wbmc?x-oss-process=image/format,png)

根据该时序，编写代码如下：
```c
/**
 * @brief   读取Flash内部的ID
 * @param   none
 * @retval  成功返回device_id
 */
uint16_t W25QXX_ReadID(void)
{
    uint8_t recv_buf[2] = {0};    //recv_buf[0]存放Manufacture ID, recv_buf[1]存放Device ID
    uint16_t device_id = 0;
    uint8_t send_data[4] = {ManufactDeviceID_CMD,0x00,0x00,0x00};   //待发送数据，命令+地址
    
    /* 使能片选 */
    HAL_GPIO_WritePin(W25Q64_CHIP_SELECT_PORT, W25Q64_CHIP_SELECT_PIN, GPIO_PIN_RESET);
    
    /* 发送并读取数据 */
    if (HAL_OK == SPI_Transmit(send_data, 4)) {
        if (HAL_OK == SPI_Receive(recv_buf, 2)) {
            device_id = (recv_buf[0] << 8) | recv_buf[1];
        }
    }
    
    /* 取消片选 */
    HAL_GPIO_WritePin(W25Q64_CHIP_SELECT_PORT, W25Q64_CHIP_SELECT_PIN, GPIO_PIN_SET);
    
    return device_id;
}
```
## 读取状态寄存器数据并判断Flash是否忙碌
上文中提到，SPI Flash的所有操作都是靠发送命令完成的，但是 Flash 接收到命令后，需要一段时间去执行该操作，这段时间内 Flash 处于“忙”状态，MCU 发送的命令无效，不能执行，在 Flash 内部有2-3个状态寄存器，指示出 Flash 当前的状态，有趣的一点是：

当 Flash 内部在执行命令时，不能再执行 MCU 发来的命令，但是 MCU 可以一直读取状态寄存器，这下就很好办了，**MCU可以一直读取，然后判断Flash是否忙完**：

![mark](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwOTAzL05hTHQxVGJUTGtFUS5wbmc?x-oss-process=image/format,png)
读取协议如下：
![](https://img-blog.csdnimg.cn/20200726095919624.png)

根据此协议实现的读取状态寄存器的代码如下：
```c
/**
 * @brief     读取W25QXX的状态寄存器，W25Q64一共有2个状态寄存器
 * @param     reg  —— 状态寄存器编号(1~2)
 * @retval    状态寄存器的值
 */
static uint8_t W25QXX_ReadSR(uint8_t reg)
{
    uint8_t result = 0; 
    uint8_t send_buf[4] = {0x00,0x00,0x00,0x00};
    switch(reg)
    {
        case 1:
            send_buf[0] = READ_STATU_REGISTER_1;
        case 2:
            send_buf[0] = READ_STATU_REGISTER_2;
        case 0:
        default:
            send_buf[0] = READ_STATU_REGISTER_1;
    }
    
     /* 使能片选 */
    HAL_GPIO_WritePin(W25Q64_CHIP_SELECT_PORT, W25Q64_CHIP_SELECT_PIN, GPIO_PIN_RESET);
    
    if (HAL_OK == SPI_Transmit(send_buf, 4)) {
        if (HAL_OK == SPI_Receive(&result, 1)) {
            HAL_GPIO_WritePin(W25Q64_CHIP_SELECT_PORT, W25Q64_CHIP_SELECT_PIN, GPIO_PIN_SET);
            
            return result;
        }
    }
    
    /* 取消片选 */
    HAL_GPIO_WritePin(W25Q64_CHIP_SELECT_PORT, W25Q64_CHIP_SELECT_PIN, GPIO_PIN_SET);

    return 0;
}
```
然后编写**阻塞判断**Flash是否忙碌的函数：
```c
/**
 * @brief	阻塞等待Flash处于空闲状态
 * @param   none
 * @retval  none
 */
static void W25QXX_Wait_Busy(void)
{
    while((W25QXX_ReadSR(1) & 0x01) == 0x01); // 等待BUSY位清空
}
```
## 读取数据

SPI Flash读取数据可以任意地址（地址长度32bit）读任意长度数据（最大 65535 Byte），没有任何限制，数据手册给出的时序如下：

![](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwOTAzL1VNdEhtUUVjRFNqNS5wbmc?x-oss-process=image/format,png)

根据该时序图编写代码如下：
```c
/**
 * @brief   读取SPI FLASH数据
 * @param   buffer      —— 数据存储区
 * @param   start_addr  —— 开始读取的地址(最大32bit)
 * @param   nbytes      —— 要读取的字节数(最大65535)
 * @retval  成功返回0，失败返回-1
 */
int W25QXX_Read(uint8_t* buffer, uint32_t start_addr, uint16_t nbytes)
{
    uint8_t cmd = READ_DATA_CMD;
    
    start_addr = start_addr << 8;
    
	W25QXX_Wait_Busy();
    
     /* 使能片选 */
    HAL_GPIO_WritePin(W25Q64_CHIP_SELECT_PORT, W25Q64_CHIP_SELECT_PIN, GPIO_PIN_RESET);
    
    SPI_Transmit(&cmd, 1);
    
    if (HAL_OK == SPI_Transmit((uint8_t*)&start_addr, 3)) {
        if (HAL_OK == SPI_Receive(buffer, nbytes)) {
            HAL_GPIO_WritePin(W25Q64_CHIP_SELECT_PORT, W25Q64_CHIP_SELECT_PIN, GPIO_PIN_SET);
            return 0;
        }
    }
    
    HAL_GPIO_WritePin(W25Q64_CHIP_SELECT_PORT, W25Q64_CHIP_SELECT_PIN, GPIO_PIN_SET);
    return -1;
}
```
## 写使能/禁止

Flash 芯片默认禁止写数据，所以**在向 Flash 写数据之前，必须发送命令开启写使能**，数据手册中给出的时序如下：

![mark](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwOTAzLzNBRUk1dm8xUEF0Ni5wbmc?x-oss-process=image/format,png)

![mark](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwOTAzL1hGSmVxVlAyU3ludS5wbmc?x-oss-process=image/format,png)

编写函数如下：
```c
/**
 * @brief    W25QXX写使能,将S1寄存器的WEL置位
 * @param    none
 * @retval
 */
void W25QXX_Write_Enable(void)
{
    uint8_t cmd= WRITE_ENABLE_CMD;
    
    HAL_GPIO_WritePin(W25Q64_CHIP_SELECT_PORT, W25Q64_CHIP_SELECT_PIN, GPIO_PIN_RESET);
    
    SPI_Transmit(&cmd, 1);
    
    HAL_GPIO_WritePin(W25Q64_CHIP_SELECT_PORT, W25Q64_CHIP_SELECT_PIN, GPIO_PIN_SET);
    
    W25QXX_Wait_Busy();

}

/**
 * @brief    W25QXX写禁止,将WEL清零
 * @param    none
 * @retval    none
 */
void W25QXX_Write_Disable(void)
{
    uint8_t cmd = WRITE_DISABLE_CMD;

    HAL_GPIO_WritePin(W25Q64_CHIP_SELECT_PORT, W25Q64_CHIP_SELECT_PIN, GPIO_PIN_RESET);
    
    SPI_Transmit(&cmd, 1);
    
    HAL_GPIO_WritePin(W25Q64_CHIP_SELECT_PORT, W25Q64_CHIP_SELECT_PIN, GPIO_PIN_SET);
    
    W25QXX_Wait_Busy();
}
```
## 擦除扇区

SPI Flash有个特性：

**数据位可以由1变为0，但是不能由0变为1。**

所以在向 Flash 写数据之前，必须要先进行擦除操作，并且 Flash **最小只能擦除一个扇区**，擦除之后该扇区所有的数据变为 `0xFF`（即全为1），数据手册中给出的时序如下：

![mark](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwOTAzL1BTeGFNYnkxdU11ci5wbmc?x-oss-process=image/format,png)

根据此时序编写函数如下：

```c
/**
 * @brief    W25QXX擦除一个扇区
 * @param   sector_addr    —— 扇区地址 根据实际容量设置
 * @retval  none
 * @note    阻塞操作
 */
void W25QXX_Erase_Sector(uint32_t sector_addr)
{
    uint8_t cmd = SECTOR_ERASE_CMD;
    
    sector_addr *= 4096;    //每个块有16个扇区，每个扇区的大小是4KB，需要换算为实际地址
    sector_addr <<= 8;
    
    W25QXX_Write_Enable();  //擦除操作即写入0xFF，需要开启写使能
    W25QXX_Wait_Busy();        //等待写使能完成
   
     /* 使能片选 */
    HAL_GPIO_WritePin(W25Q64_CHIP_SELECT_PORT, W25Q64_CHIP_SELECT_PIN, GPIO_PIN_RESET);
    
    SPI_Transmit(&cmd, 1);
    
    SPI_Transmit((uint8_t*)&sector_addr, 3);
    
    HAL_GPIO_WritePin(W25Q64_CHIP_SELECT_PORT, W25Q64_CHIP_SELECT_PIN, GPIO_PIN_SET);
    
    W25QXX_Wait_Busy();       //等待扇区擦除完成
}
```
## 页写入操作

向 Flash 芯片写数据的时候，因为 Flash 内部的构造，可以按页写入：

![mark](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwOTAzL2Rqa2NzMXlJcXI0UC5wbmc?x-oss-process=image/format,png)

页写入的时序如图：

![mark](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwOTAzL04wY21ZTFhRWEl0QS5wbmc?x-oss-process=image/format,png)

编写代码如下：
```c
/**
 * @brief    页写入操作
 * @param    dat —— 要写入的数据缓冲区首地址
 * @param    WriteAddr —— 要写入的地址
 * @param   byte_to_write —— 要写入的字节数（0-256）
 * @retval    none
 */
void W25QXX_Page_Program(uint8_t* dat, uint32_t WriteAddr, uint16_t nbytes)
{
    uint8_t cmd = PAGE_PROGRAM_CMD;
    
    WriteAddr <<= 8;
    
    W25QXX_Write_Enable();
    
    /* 使能片选 */
    HAL_GPIO_WritePin(W25Q64_CHIP_SELECT_PORT, W25Q64_CHIP_SELECT_PIN, GPIO_PIN_RESET);
    
    SPI_Transmit(&cmd, 1);

    SPI_Transmit((uint8_t*)&WriteAddr, 3);
    
    SPI_Transmit(dat, nbytes);
    
    HAL_GPIO_WritePin(W25Q64_CHIP_SELECT_PORT, W25Q64_CHIP_SELECT_PIN, GPIO_PIN_SET);
    
    W25QXX_Wait_Busy();
}
```

# 6. 测试驱动

在 `main.c` 函数中编写代码，测试驱动：

首先定义两个缓存：
```c
/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */
uint16_t device_id;
uint8_t read_buf[10] = {0};
uint8_t write_buf[10] = {0};
int i;
/* USER CODE END 0 */
```
然后在 main 函数中编写代码：
```c
	 /* USER CODE BEGIN 2 */
 
    printf("W25Q64 SPI Flash Test By Mculover666\r\n");
    device_id = W25QXX_ReadID();
    printf("W25Q64 Device ID is 0x%04x\r\n", device_id);

    /* 为了验证，首先读取要写入地址处的数据 */
    printf("-------- read data before write -----------\r\n");
    W25QXX_Read(read_buf, 0, 10);
    
    for (i = 0;i < 10;i++) {
        printf("[0x%08x]:0x%02x\r\n", i, *(read_buf+i));
    }
    
    /* 擦除该扇区 */
    printf("-------- erase sector 0 -----------\r\n");
    W25QXX_Erase_Sector(0);

    /* 再次读数据 */
    printf("-------- read data after erase -----------\r\n");
    W25QXX_Read(read_buf, 0, 10);
    for (i = 0;i < 10;i++) {
        printf("[0x%08x]:0x%02x\r\n", i, *(read_buf+i));
    }
    
    /* 写数据 */
    printf("-------- write data -----------\r\n");
    for (i = 0; i < 10;i++) {
        write_buf[i] = i;
    }
    W25QXX_Page_Program(write_buf, 0, 10);
    
    /* 再次读数据 */
    printf("-------- read data after write -----------\r\n");
    W25QXX_Read(read_buf, 0, 10);
    for (i = 0;i < 10;i++) {
        printf("[0x%08x]:0x%02x\r\n", i, *(read_buf+i));
    }

  	/* USER CODE END 2 */
```
![](https://img-blog.csdnimg.cn/20200726105809813.png)
![](https://img-blog.csdnimg.cn/20200726105839150.png)
![](https://img-blog.csdnimg.cn/2020072610591247.png)

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![mark](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2ltYWdlLzIwMTkwODE0L05RcXQxZVJ4cmwxSy5wbmc?x-oss-process=image/format,png)





