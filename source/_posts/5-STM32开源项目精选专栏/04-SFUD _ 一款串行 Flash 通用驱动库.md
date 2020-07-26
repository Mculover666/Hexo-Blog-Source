---
title: SFUD | 一款串行 Flash 通用驱动库
keywords: SFUD
tags: 嵌入式开源项目精选专栏
categories: OpenSource
abbrlink: 3066735794
summary: 嵌入式开源项目精选专栏
date: 2020-04-23 08:00:56
---


![](https://img-blog.csdnimg.cn/20200321151234798.png)
# 嵌入式开源项目精选专栏
本专栏由Mculover666创建，主要内容为寻找嵌入式领域内的优质开源项目，一是帮助开发者使用开源项目实现更多的功能，二是通过这些开源项目，学习大佬的代码及背后的实现思想，提升自己的代码水平，和其它专栏相比，本专栏的优势在于：

**不会单纯的介绍分享项目，还会包含作者亲自实践的过程分享，甚至还会有对它背后的设计思想解读**。

目前本专栏包含的开源项目有：

- [cJSON | 一个轻量级C语言JSON解析器](https://blog.csdn.net/Mculover666/article/details/103796256)
- [paho | 支持10种语言编写mqtt客户端，总有一款适合你！](https://blog.csdn.net/Mculover666/article/details/103935428)
- [MultiButton | 一个小巧简单易用的事件驱动型按键驱动模块](https://blog.csdn.net/Mculover666/article/details/104992661)
- [letter-shell | 一个功能强大的嵌入式shell](https://blog.csdn.net/Mculover666/article/details/105141286)
- [EasyLogger | 一款轻量级且高性能的日志库](https://blog.csdn.net/Mculover666/article/details/105371993)

如果您自己编写或者发现的开源项目不错，欢迎留言或者私信投稿到本专栏，分享获得双倍的快乐！

# 1. SFUD
本期给大家带来的开源项目是 SFUD，**一款串行 Flash 通用驱动库**，作者armink，目前收获 407 个 star，遵循 MIT 开源许可协议。

SFUD全称Serial Flash Universal Driver，是一款开源的串行 SPI Flash 通用驱动库，由于现有市面的串行 Flash 种类居多，各个 Flash 的规格及命令存在差异， SFUD 就是**为了解决这些 Flash 的差异现状而设计**。

SFUD的特点在于：

- 支持 SPI/QSPI 接口
- 面向对象设计（同时支持多个 Flash 对象）
- 可灵活裁剪、扩展性强
- 支持 4 字节地址

>项目地址：[https://github.com/armink/SFUD](https://github.com/armink/SFUD)

# 2. 移植SFUD
## 2.1. 移植思路
在移植过程中主要参考两个资料：项目的readme文档和demo工程。

对于这些开源项目，其实移植起来也就两步：

- ① 添加源码到裸机工程中；
- ② 实现需要的接口即可；

## 2.2. 准备裸机工程
本文中我使用的是小熊派IoT开发套件，主控芯片为STM32L431RCT6：
![](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2Jsb2cvMjAxOTA5MTkvekh4NXhNWVVtU1cxLnBuZw?x-oss-process=image/format,png#pic_center)
板载Flash型号为`W25Q64JV`，大小64Mbit，与STM32的QSPI接口相连：
![](https://img-blog.csdnimg.cn/20200416112021185.png)

移植之前需要准备一份裸机工程，我使用STM32CubeMX生成，需要初始化以下配置：

- 配置SPI Flash通信接口（SPI或QSPI）
- 配置一个串口用于打印信息
- printf重定向

具体过程请参考：

- [STM32CubeMX_06 | 使用USART发送和接收数据（查询模式）](http://www.mculover666.cn/posts/2064921339/)
- [STM32CubeMX_09 | 重定向printf函数到串口输出的多种方法](http://www.mculover666.cn/posts/2251182441/)
- [STM32CubeMX_18 | 使用硬件QSPI读写SPI Flash（W25Q64）](http://www.mculover666.cn/posts/1294047065/)

>使用CubeMX配置好SPI或QSPI通信即可，不用编写W25Q64驱动。

## 2.3. 添加SFUD到工程中
① 复制源码到工程中：
![](https://img-blog.csdnimg.cn/20200414175259627.png)
② 在keil中添加 SFUD 组件的源码文件：

- `src\sfud.c`：SFUD核心功能源码；
- `src\sfud_sfdp.c`：读取并分析SFDP功能源码；
- `port\sfud_port.c`：SFUD移植接口；

![](https://img-blog.csdnimg.cn/20200414184428469.png)
③ 将`sfud/inc`头文件路径添加到keil中：
![](https://img-blog.csdnimg.cn/20200414184715437.png)
## 2.4. 实现SFUD移植接口
SFUD的移植接口都已经写好了，在`sfud_port.c`文件中，只需要在函数体中添加代码即可。

① 底层SPI/QSPI读写接口：
```c
/**
 * SPI write data then read data
 */
static sfud_err spi_write_read(const sfud_spi *spi, const uint8_t *write_buf, size_t write_size, uint8_t *read_buf, size_t read_size);
```

② 如果使用的是QSPI通信方式，还需要实现快速读取数据的接口：
```c
/**
 * QSPI fast read data
 */
static sfud_err qspi_read(const struct __sfud_spi *spi, uint32_t addr, sfud_qspi_read_cmd_format *qspi_read_cmd_format, uint8_t *read_buf, size_t read_size);
```
③ SFUD底层使用的SPI/QSPI接口和SPI设备对象初始化接口：
```c
sfud_err sfud_spi_port_init(sfud_flash *flash);
```

关于SFUD底层所抽象出来的SPI设备对象，在接下来的设计思想解读章节中会详细讲述。

本文中所使用的裸机工程是基于HAL库的，在SFUD源码的Demo中也有一份HAL库的工程，因为**基于HAL库的移植接口实现都是一样的**，所以我直接将Demo中的`sfud_port.c`文件复制过来替换：
![](https://img-blog.csdnimg.cn/20200416114215451.png)
复制过来之后，如果使用的不是STM32L4系列的芯片，则需要修改`sfud_port.c`中包含的头文件：
![](https://img-blog.csdnimg.cn/20200416114449692.png)
## 2.5. 配置SFUD
SFUD的核心功能配置文件在`sfud_cfg.h`，修改说明如下：
![](https://img-blog.csdnimg.cn/20200416115206184.png)
修改完了之后，还需要去修改刚刚复制替换的`sfud_port.c`文件，与刚刚填写的配置信息相对应：
![](https://img-blog.csdnimg.cn/20200416115539942.png)
至此，SFUD移植、配置完成，接下来就可以愉快的使用了！

# 3. 使用SFUD
使用时包含头文件：
```c
#include <sfud.h>
```
## 3.1. 初始化SFUD
初始化SFUD的API如下，该函数会初始化 Flash 设备表中的全部设备：
```c
sfud_err sfud_init(void);
```
在QSPI模式下，SFUD 对于 QSPI 模式的支持**仅限于快速读命令**，通过该函数可以配置 Flash 所使用的 QSPI 总线的实际支持的数据线最大宽度，例如：1 线（默认值，即传统的 SPI 模式）、2 线、4 线：
```c
sfud_err sfud_qspi_fast_read_enable(sfud_flash *flash, uint8_t data_line_width);
```
所以，在main函数中编写如下初始化函数：
```c
/* USER CODE BEGIN 2 */

/* SFUD初始化 */
if(sfud_init() != SFUD_SUCCESS)
{
	printf("SFUD init fail.\r\n");
}
/* 使能QSPI快读 */
sfud_qspi_fast_read_enable(sfud_get_device(SFUD_W25Q64_DEVICE_INDEX), 1);

/* USER CODE END 2 */
```
编译、下载之后，可以在串口终端中看到SFUD打印的日志：
![](https://img-blog.csdnimg.cn/20200416184246151.png)
SFUD初始化Flash设备成功后进行接下来的读写测试。
## 3.2. Flash擦除/读写操作
① 读取Flash数据：
```c
sfud_err sfud_read(const sfud_flash *flash, uint32_t addr, size_t size, uint8_t *data);
```
② 擦除 Flash 数据：
```c
sfud_err sfud_erase(const sfud_flash *flash, uint32_t addr, size_t size);
```
③ 往Flash写数据：
```c
sfud_err sfud_write(const sfud_flash *flash, uint32_t addr, size_t size, const uint8_t *data);
```

接下来使用作者编写的demo测试。

首先在`main.c`开头编写代码，开辟一块缓冲区用于存放测试数据：
```c
/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */

/* SFUD读写Flash数据测试的缓冲区 */
#define SFUD_DEMO_TEST_BUFFER_SIZE                     1024
static uint8_t sfud_demo_test_buf[SFUD_DEMO_TEST_BUFFER_SIZE];

/* SFUD读写Flash数据测试函数 */
void sfud_demo(uint32_t addr, size_t size, uint8_t *data);

/* USER CODE END 0 */
```
然后在`main.c`最后添加测试函数：
```c
/* USER CODE BEGIN 4 */
/**
 * SFUD demo for the first flash device test.
 *
 * @param addr flash start address
 * @param size test flash size
 * @param size test flash data buffer
 */
void sfud_demo(uint32_t addr, size_t size, uint8_t *data)
{
    sfud_err result = SFUD_SUCCESS;
    extern sfud_flash *sfud_dev;
    const sfud_flash *flash = sfud_get_device(SFUD_W25Q64_DEVICE_INDEX);
    size_t i;
    /* prepare write data */
    for (i = 0; i < size; i++)
    {
        data[i] = i;
    }
    /* erase test */
    result = sfud_erase(flash, addr, size);
    if (result == SFUD_SUCCESS)
    {
        printf("Erase the %s flash data finish. Start from 0x%08X, size is %zu.\r\n", flash->name, addr, size);
    }
    else
    {
        printf("Erase the %s flash data failed.\r\n", flash->name);
        return;
    }
    /* write test */
    result = sfud_write(flash, addr, size, data);
    if (result == SFUD_SUCCESS)
    {
        printf("Write the %s flash data finish. Start from 0x%08X, size is %zu.\r\n", flash->name, addr, size);
    }
    else
    {
        printf("Write the %s flash data failed.\r\n", flash->name);
        return;
    }
    /* read test */
    result = sfud_read(flash, addr, size, data);
    if (result == SFUD_SUCCESS)
    {
        printf("Read the %s flash data success. Start from 0x%08X, size is %zu. The data is:\r\n", flash->name, addr, size);
        printf("Offset (h) 00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F\r\n");
        for (i = 0; i < size; i++)
        {
            if (i % 16 == 0)
            {
                printf("[%08X] ", addr + i);
            }
            printf("%02X ", data[i]);
            if (((i + 1) % 16 == 0) || i == size - 1)
            {
                printf("\r\n");
            }
        }
        printf("\r\n");
    }
    else
    {
        printf("Read the %s flash data failed.\r\n", flash->name);
    }
    /* data check */
    for (i = 0; i < size; i++)
    {
				
        if (data[i] != i % 256)
        {
            printf("Read and check write data has an error. Write the %s flash data failed.\r\n", flash->name);
            break;
        }
    }
    if (i == size)
    {
        printf("The %s flash test is success.\r\n", flash->name);
    }
}

/* USER CODE END 4 */
```
在**main函数**中，SFUD初始化代码之后，调用该函数进行Flash测试：
```c
/* 测试Flash读写 */
sfud_demo(0, sizeof(sfud_demo_test_buf), sfud_demo_test_buf);
```
编译、下载，在串口终端中查看结果：
![](https://img-blog.csdnimg.cn/20200416185854314.png)
## 3.3. 移植前后内存占用情况
![](https://img-blog.csdnimg.cn/20200416190554551.png)
SFUD中获取Flash信息有两种方式：

- 使用SFDP 参数方式：开关宏`SFUD_USING_SFDP`；
- 使用库自带的 Flash 参数信息表：开关宏`SFUD_USING_FLASH_INFO_TABLE`；

本文中两种方式都开启，所以移植之后较大，实际使用中可以视情况关闭这两个功能。

SFDP功能关闭后，只会查询该库在 `/sfud/inc/sfud_flash_def.h` 中提供的 Flash 信息表，代码量会降低，但是软件适配性也随之降低。

查表功能关闭后，该库只驱动支持 SFDP 规范的 Flash，也会适当的降低部分代码量。

一般情况下上述二者必须要选择一个，在实际使用时视情况而定，但是也可以两者都不开启，直接指定好具体的某款 Flash 参数。

# 4. SFUD设计思想解读
## 4.1. Flash设备对象
SFUD中最重要的就是Flash设备对象，**一切操作都是对这个Flash设备对象进行的**，每个Flash设备对象独立，所以SFUD也支持系统中存在多个Flash设备对象。

Flash设备对象管理着Flash存储器的所有信息，原型在`sfud_def.h`中，定义如下：
```c
/**
 * serial flash device
 */
typedef struct {
    char *name;                                  /**< serial flash name */
    size_t index;                                /**< index of flash device information table  @see flash_table */
    sfud_flash_chip chip;                        /**< flash chip information */
    sfud_spi spi;                                /**< SPI device */
    bool init_ok;                                /**< initialize OK flag */
    bool addr_in_4_byte;                         /**< flash is in 4-Byte addressing */
    struct {
        void (*delay)(void);                     /**< every retry's delay */
        size_t times;                            /**< default times for error retry */
    } retry;
    void *user_data;                             /**< some user data */

#ifdef SFUD_USING_QSPI
    sfud_qspi_read_cmd_format read_cmd_format;   /**< fast read cmd format */
#endif

#ifdef SFUD_USING_SFDP
    sfud_sfdp sfdp;                              /**< serial flash discoverable parameters by JEDEC standard */
#endif

} sfud_flash, *sfud_flash_t;
```
其中Flash设备的通信接口信息由 sfud_spi 对象管理，包括SPI读写数据函数，加锁解锁函数定义如下：
```c
/**
 * SPI device
 */
typedef struct __sfud_spi {
    /* SPI device name */
    char *name;
    /* SPI bus write read data function */
    sfud_err (*wr)(const struct __sfud_spi *spi, const uint8_t *write_buf, size_t write_size, uint8_t *read_buf,
                   size_t read_size);
#ifdef SFUD_USING_QSPI
    /* QSPI fast read function */
    sfud_err (*qspi_read)(const struct __sfud_spi *spi, uint32_t addr, sfud_qspi_read_cmd_format *qspi_read_cmd_format,
                          uint8_t *read_buf, size_t read_size);
#endif
    /* lock SPI bus */
    void (*lock)(const struct __sfud_spi *spi);
    /* unlock SPI bus */
    void (*unlock)(const struct __sfud_spi *spi);
    /* some user data */
    void *user_data;
} sfud_spi, *sfud_spi_t;
```
## 4.2. JESD216 SFDP标准
SFDP全称 Serial Flash Discoverable Parameter，它是 JEDEC （固态技术协会）制定的串行 Flash 功能的参数表标准。

该标准规定了，每个 Flash 中会存在一个参数表，该表中会存放 Flash 容量、写粒度、擦除命令、地址模式等 Flash 规格参数。目前，除了部分厂家旧款 Flash 型号会不支持该标准，其他绝大多数新出厂的 Flash 均已支持 SFDP 标准。

所以 SFUD 在初始化时会优先读取 SFDP 表参数，以达到**SFUD在支持SFDP标准的Flash上全部适用的效果**，更加通用。

那么SFDP标准的内容是什么呢？SFDP标准强制规范必须要有：

- SFDP标题头
- 1st参数头
- JEDEC Flash基本参数表格

SFDP标题头一般为“S”“F”“U”“D”，如果能读取出这四个字符，则认为该款Flash支持SFDP标准，比如在`sfud_sfdp`源码中校验代码如下：
```c
/* check SFDP header */
if (!(header[0] == 'S' &&
      header[1] == 'F' &&
      header[2] == 'D' &&
      header[3] == 'P')) {
    SFUD_DEBUG("Error: Check SFDP signature error. It's must be 50444653h('S' 'F' 'D' 'P').");
    return false;
}
```
接下来是一些预留空内容，属于厂商可选内容，Flash厂商可以在这些空白内容中添加自己的厂商ID识别号、SFDP版本号、参数长度以及存放参数表格的地址指针，比如读取W25Q64的结果中显示：
![](https://img-blog.csdnimg.cn/20200418103340819.png)
接下来的 JEDEC Flash基本参数表格里面规范和定义了该器件的一些最基本的读取方式、指令内容、扇区大小和芯片容量等信息：
![](https://img-blog.csdnimg.cn/20200418103418179.png)

## 4.3. 添加库目前不支持的 Flash
如果你使用的Flash型号比较老或者不支持SFDP，SFUD库当然考虑到了这一点，所以提供了Flash设备参数表，在`sfdu_flash_def.h`文件的 SFUD_FLASH_CHIP_TABLE 就能看到当前所有支持的 Flash：
![](https://img-blog.csdnimg.cn/20200418103947720.png)
如果你使用的Flash型号既不支持SFDP，也不在此Flash设备参数表中，那么就需要手动添加到该设备参数表中才可以正常使用。

具体的添加方式请参考SFUD项目的README文档中2.5节，讲述的非常详细。

# 5. 项目工程源码获取和问题交流
目前我将SFUD源码、我移植到小熊派STM32L431RCT6开发板的工程源码上传到了QQ群里（包含好几份HAL库，QQ相对速度快点），**可以在QQ群里下载，有问题也可以在群里交流，当然也欢迎大家分享出来自己移植的工程到QQ群里**：
![](https://img-blog.csdnimg.cn/20200418104909667.png)
放上QQ群二维码：
![](https://img-blog.csdnimg.cn/20200321144543508.png)

<font color="red">**接收更多精彩文章及资源推送，欢迎订阅我的微信公众号：『mculover666』。**</font>
![](https://img-blog.csdnimg.cn/20200202092055136.png)
