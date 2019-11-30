---
title: 【STM32Cube-18】使用硬件QSPI读写SPI Flash（W25Q64）
tags: STM32CubeMX SPI Flash QSPI接口
categories: STM32CubeMX
abbrlink: 1294047065
date: 2019-08-08 09:48:56
---
本篇详细的记录了如何使用STM32CubeMX配置STM32L431RCT6的硬件QSPI外设与 SPI Flash 通信（W25Q64）。
<!--more-->

# 1. 准备工作
## 硬件准备
- 开发板
首先需要准备一个开发板，这里我准备的是STM32L4的开发板（BearPi）：

![mark](http://mculover666.cn/image/20190806/9uiPTi5odYSj.png?imageslim)

- SPI Flash
小熊派开发板板载一片SPI Flash，型号为 `W25Q64`，大小为 8 MB，最大支持 80 Mhz的操作频率。

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

## 配置QSPI接口

首先查看小熊派开发板上 SPI Flash 的原理图：

![mark](http://mculover666.cn/image/20190903/IBQ1uUzXevoT.png?imageslim)

其引脚连接情况如下：

|SPI Flash连接引脚|对应引脚|
|:---:|:---:|
|QUADSPI_BK1_NCS|PB11|
|QUADSPI_BK1_CLK|PB10|
|QUADSPI_BK1_IO0|PB1|
|QUADSPI_BK1_IO1|PB0|

接下来配置 QSPI 接口：

![mark](http://mculover666.cn/image/20190903/0VBw2fFuGswz.png?imageslim)


## 配置时钟树
STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：
![mark](http://mculover666.cn/image/20190806/1TQg7frjRpVr.png?imageslim)

## 生成工程设置

![mark](http://mculover666.cn/image/20190903/38BkgTe2YX03.png?imageslim)

## 代码生成设置
最后设置生成独立的初始化文件：

![mark](http://mculover666.cn/image/20190806/T6WvSK6Dfpts.png?imageslim)

## 生成代码
点击`GENERATE CODE`即可生成MDK-V5工程：

![mark](http://mculover666.cn/image/20190806/s0jGhLBWW6Cm.png?imageslim)

# 3. 在MDK中编写、编译、下载用户代码

## 重定向printf( )函数

参考：[【STM32Cube_09】重定向printf函数到串口输出的多种方法](https://www.mculover666.cn/2019/07/30/STM32Cube/【STM32Cube-09】重定向printf函数到串口输出的多种方法/)。

# 4. 封装 SPI Flash（W25Q64）的命令和底层函数

MCU 通过向 SPI Flash **发送各种命令** 来读写 SPI Flash内部的寄存器，所以这种裸机驱动，首先要先宏定义出需要使用的命令，然后利用 HAL 库提供的库函数，封装出三个底层函数，**便于移植**：

- 向 SPI Flash 发送命令的函数
- 向 SPI Flash 发送数据的函数
- 从 SPI Flash 接收数据的函数

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
## 封装发送命令的函数（重点）

```c
/**
 * @brief		向SPI Flash发送指令
 * @param		instruction —— 要发送的指令
 * @param		address     —— 要发送的地址
 * @param		dummyCycles	—— 空指令周期数
 * @param		instructionMode —— 指令发送模式
 * @param		addressMode —— 地址发送模式
 * @param		addressSize	—— 地址大小
 * @param		dataMode    —— 数据发送模式
 * @retval	    成功返回HAL_OK
*/
HAL_StatusTypeDef QSPI_Send_Command(uint32_t instruction, 
									uint32_t address, 
									uint32_t dummyCycles, 
									uint32_t instructionMode, 
									uint32_t addressMode, 
									uint32_t addressSize, 
									uint32_t dataMode)
{
    QSPI_CommandTypeDef cmd;

    cmd.Instruction = instruction;                 	//指令
    cmd.Address = address;                          //地址
    cmd.DummyCycles = dummyCycles;                  //设置空指令周期数
    cmd.InstructionMode = instructionMode;			//指令模式
    cmd.AddressMode = addressMode;   				//地址模式
    cmd.AddressSize = addressSize;   				//地址长度
    cmd.DataMode = dataMode;             			//数据模式
    cmd.SIOOMode = QSPI_SIOO_INST_EVERY_CMD;       	//每次都发送指令
    cmd.AlternateByteMode = QSPI_ALTERNATE_BYTES_NONE; //无交替字节
    cmd.DdrMode = QSPI_DDR_MODE_DISABLE;           	//关闭DDR模式
    cmd.DdrHoldHalfCycle = QSPI_DDR_HHC_ANALOG_DELAY;
    
	return HAL_QSPI_Command(&hqspi, &cmd, 5000);
}
```
## 封装发送数据的函数
```c
/**
* @brief    QSPI发送指定长度的数据
* @param    buf  —— 发送数据缓冲区首地址
* @param    size —— 要发送数据的字节数
 * @retval	成功返回HAL_OK
 */
HAL_StatusTypeDef QSPI_Transmit(uint8_t* send_buf, uint32_t size)
{
    hqspi.Instance->DLR = size - 1;                         //配置数据长度
    return HAL_QSPI_Transmit(&hqspi, send_buf, 5000);	    //接收数据
}
```
## 封装接收数据的函数
```c
/**
 * @brief	  QSPI接收指定长度的数据
 * @param   buf  —— 接收数据缓冲区首地址
 * @param   size —— 要接收数据的字节数
 * @retval	成功返回HAL_OK
 */
HAL_StatusTypeDef QSPI_Receive(uint8_t* recv_buf, uint32_t size)
{
    hqspi.Instance->DLR = size - 1;                       //配置数据长度
    return HAL_QSPI_Receive(&hqspi, recv_buf, 5000);			//接收数据
}
```

# 5. 编写W25Q64的驱动程序

接下来开始利用上一节封装的宏定义和底层函数，编写W25Q64的驱动程序：

## 读取Manufacture ID和Device ID
读取 Flash 内部这两个ID有两个作用：

- 检测SPI Flash是否存在
- 可以根据ID判断Flash具体型号

数据手册上给出的操作时序如图：

![mark](http://mculover666.cn/image/20190903/GkfvfgRgRA53.png?imageslim)

根据该时序，编写代码如下：
```c
/**
 * @brief   读取Flash内部的ID
 * @param   none
 * @retval	成功返回device_id
 */
uint16_t W25QXX_ReadID(void)
{
	uint8_t recv_buf[2] = {0};	//recv_buf[0]存放Manufacture ID, recv_buf[1]存放Device ID
	uint16_t device_id = 0;
	if(HAL_OK == QSPI_Send_Command(ManufactDeviceID_CMD, 0, 0, QSPI_INSTRUCTION_1_LINE, QSPI_ADDRESS_1_LINE, QSPI_ADDRESS_24_BITS, QSPI_DATA_1_LINE))
	{
        //读取ID
        if(HAL_OK == QSPI_Receive(recv_buf, 2))
        {
            device_id = (recv_buf[0] << 8) | recv_buf[1];
            return device_id;
        }
        else
        {
            return 0;
        }
	}
	else
	{
		return 0;
	}
}
```

## 读取数据

SPI Flash读取数据可以任意地址（地址长度32bit）读任意长度数据（最大 65535 Byte），没有任何限制，数据手册给出的时序如下：

![mark](http://mculover666.cn/image/20190903/UMtHmQEcDSj5.png?imageslim)

根据该时序图编写代码如下：
```c
/**
 * @brief	读取SPI FLASH数据
 * @param   dat_buffer —— 数据存储区
 * @param   start_read_addr —— 开始读取的地址(最大32bit)
 * @param   byte_to_read —— 要读取的字节数(最大65535)
 * @retval  none
 */
void W25QXX_Read(uint8_t* dat_buffer, uint32_t start_read_addr, uint16_t byte_to_read)
{
	QSPI_Send_Command(READ_DATA_CMD, start_read_addr, 0, QSPI_INSTRUCTION_1_LINE, QSPI_ADDRESS_1_LINE, QSPI_ADDRESS_24_BITS, QSPI_DATA_1_LINE);
    QSPI_Receive(dat_buffer, byte_to_read);
}
```
## 读取状态寄存器数据并判断Flash是否忙碌

上文中提到，SPI Flash的所有操作都是靠发送命令完成的，但是 Flash 接收到命令后，需要一段时间去执行该操作，这段时间内 Flash 处于“忙”状态，MCU 发送的命令无效，不能执行，在 Flash 内部有2-3个状态寄存器，指示出 Flash 当前的状态，有趣的一点是：

当 Flash 内部在执行命令时，不能再执行 MCU 发来的命令，但是 MCU 可以一直读取状态寄存器，这下就很好办了，**MCU可以一直读取，然后判断Flash是否忙完**：

![mark](http://mculover666.cn/image/20190903/NaLt1TbTLkEQ.png?imageslim)

首先读取状态寄存器的代码如下：
```c
/**
 * @brief	读取W25QXX的状态寄存器，W25Q64一共有2个状态寄存器
 * @param 	reg  —— 状态寄存器编号(1~2)
 * @retval	状态寄存器的值
 */
uint8_t W25QXX_ReadSR(uint8_t reg)
{
	uint8_t cmd = 0, result = 0;	
	switch(reg)
	{
		case 1:
			/* 读取状态寄存器1的值 */
			cmd = READ_STATU_REGISTER_1;
		case 2:
			cmd = READ_STATU_REGISTER_2;
		case 0:
		default:
			cmd = READ_STATU_REGISTER_1;
	}
	QSPI_Send_Command(cmd, 0, 0, QSPI_INSTRUCTION_1_LINE, QSPI_ADDRESS_NONE, QSPI_ADDRESS_24_BITS, QSPI_DATA_1_LINE);
	QSPI_Receive(&result, 1);
	
	return result;
}
```
然后编写**阻塞判断**Flash是否忙碌的函数：
```c
/**
 * @brief	阻塞等待Flash处于空闲状态
 * @param   none
 * @retval  none
 */
void W25QXX_Wait_Busy(void)
{
    while((W25QXX_ReadSR(1) & 0x01) == 0x01); // 等待BUSY位清空
}
```
## 写使能/禁止

Flash 芯片默认禁止写数据，所以**在向 Flash 写数据之前，必须发送命令开启写使能**，数据手册中给出的时序如下：

![mark](http://mculover666.cn/image/20190903/3AEI5vo1PAt6.png?imageslim)

![mark](http://mculover666.cn/image/20190903/XFJeqVP2Synu.png?imageslim)

编写函数如下：
```c
/**
 * @brief	W25QXX写使能,将S1寄存器的WEL置位
 * @param	none
 * @retval
 */
void W25QXX_Write_Enable(void)
{
    QSPI_Send_Command(WRITE_ENABLE_CMD, 0, 0, QSPI_INSTRUCTION_1_LINE, QSPI_ADDRESS_NONE, QSPI_ADDRESS_8_BITS, QSPI_DATA_NONE);
		W25QXX_Wait_Busy();
}

/**
 * @brief	W25QXX写禁止,将WEL清零
 * @param	none
 * @retval	none
 */
void W25QXX_Write_Disable(void)
{
    QSPI_Send_Command(WRITE_DISABLE_CMD, 0, 0, QSPI_INSTRUCTION_1_LINE, QSPI_ADDRESS_NONE, QSPI_ADDRESS_8_BITS, QSPI_DATA_NONE);
		W25QXX_Wait_Busy();
}
```
## 擦除扇区

SPI Flash有个特性：

**数据位可以由1变为0，但是不能由0变为1。**

所以在向 Flash 写数据之前，必须要先进行擦除操作，并且 Flash **最小只能擦除一个扇区**，擦除之后该扇区所有的数据变为 `0xFF`（即全为1），数据手册中给出的时序如下：

![mark](http://mculover666.cn/image/20190903/PSxaMby1uMur.png?imageslim)

根据此时序编写函数如下：

```c
/**
 * @brief	W25QXX擦除一个扇区
 * @param   sector_addr	—— 扇区地址 根据实际容量设置
 * @retval  none
 * @note	阻塞操作
 */
void W25QXX_Erase_Sector(uint32_t sector_addr)
{
    sector_addr *= 4096;	//每个块有16个扇区，每个扇区的大小是4KB，需要换算为实际地址
    W25QXX_Write_Enable();  //擦除操作即写入0xFF，需要开启写使能
    W25QXX_Wait_Busy();		//等待写使能完成
    QSPI_Send_Command(SECTOR_ERASE_CMD, sector_addr, 0, QSPI_INSTRUCTION_1_LINE, QSPI_ADDRESS_1_LINE, QSPI_ADDRESS_24_BITS, QSPI_DATA_NONE);
    W25QXX_Wait_Busy();   	//等待扇区擦除完成
}
```
## 页写入操作

向 Flash 芯片写数据的时候，因为 Flash 内部的构造，可以按页写入：

![mark](http://mculover666.cn/image/20190903/djkcs1yIqr4P.png?imageslim)

页写入的时序如图：

![mark](http://mculover666.cn/image/20190903/N0cmYLXQXItA.png?imageslim)

编写代码如下：
```c
/**
 * @brief	页写入操作
 * @param	dat —— 要写入的数据缓冲区首地址
 * @param	WriteAddr —— 要写入的地址
 * @param   byte_to_write —— 要写入的字节数（0-256）
 * @retval	none
 */
void W25QXX_Page_Program(uint8_t* dat, uint32_t WriteAddr, uint16_t byte_to_write)
{
	W25QXX_Write_Enable();
	QSPI_Send_Command(PAGE_PROGRAM_CMD, WriteAddr, 0, QSPI_INSTRUCTION_1_LINE, QSPI_ADDRESS_1_LINE, QSPI_ADDRESS_24_BITS, QSPI_DATA_1_LINE);
	QSPI_Transmit(dat, byte_to_write);
    W25QXX_Wait_Busy();
}
```

# 6. 测试驱动

在 `main.c` 函数中编写代码，测试驱动：

首先定义两个缓存：
```c
/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */
uint8_t dat[11] = "mculover666";
uint8_t read_buf[11] = {0};
/* USER CODE END 0 */
```
然后在 main 函数中编写代码：
```c
/* USER CODE BEGIN 2 */
printf("Test W25QXX...\r\n");
device_id = W25QXX_ReadID();
printf("device_id = 0x%04X\r\n\r\n", device_id);

/* 为了验证，首先读取要写入地址处的数据 */
printf("-------- read data before write -----------\r\n");
W25QXX_Read(read_buf, 5, 11);
printf("read date is %s\r\n", (char*)read_buf);

/* 擦除该扇区 */
printf("-------- erase sector 0 -----------\r\n");
W25QXX_Erase_Sector(0);

/* 写数据 */
printf("-------- write data -----------\r\n");
W25QXX_Page_Program(dat, 5, 11);

/* 再次读数据 */
printf("-------- read data after write -----------\r\n");
W25QXX_Read(read_buf, 5, 11);
printf("read date is %s\r\n", (char*)read_buf);
/* USER CODE END 2 */
```

测试结果如下：

![mark](http://mculover666.cn/image/20190903/lw04M0RJ7gYp.png?imageslim)

至此，我们已经学会**如何使用硬件QSPI接口读写SPI Flash的数据**，下一节将讲述如何使用硬件SDMMC接口读取SD卡数据。

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![mark](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)




