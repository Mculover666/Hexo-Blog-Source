---
title: SFUD | 一款开源的串行 SPI Flash 通用驱动库
tags: SFUD
categories: 开源项目
img: 'http://mculover666.cn/blog/20190929/IMAKhD4KwvAo.png?imageslim'
summary: mculover666 带你一起手把手在裸机移植 SFUD
cover: true
coverImg: 'http://mculover666.cn/blog/20190929/IMAKhD4KwvAo.png?imageslim'
abbrlink: 1095322032
date: 2019-09-01 11:06:52
---

![ ](http://mculover666.cn/blog/20190929/IMAKhD4KwvAo.png?imageslim)

你是否因为搞不定 SPI Flash 而掉了好多头发？

你是否因为手撸 SPI Flash 驱动而浪费了大量开发时间？

你是否因为突然之间更换 SPI Flash 型号而去找产品打架？

如果没有，可以关闭这篇文章啦，有这时间去刷抖音开心开心不好吗~

如果有的话，你很幸运哈哈，在对的时间遇到对的库，接下来 mculover666 带你一起**手把手在裸机移植 SFUD**。


废话少说，接下来有请主角 SFUD 登场~

# 1. SFUD

>[SFUD](https://github.com/armink/SFUD) 全称 `Serial Flash Universal Driver`，是一款开源的串行 SPI Flash 通用驱动库。

SFUD主要特点有：

- 支持 SPI/QSPI 接口
- 面向对象思想编写（同时支持多个 Flash 对象）
- 可灵活裁剪、扩展性强

SFUD的资源占用情况非常小：

-  标准占用：RAM:0.2KB ROM:5.5KB
- 最小占用：RAM:0.1KB ROM:3.6KB

SFUD 开源项目由 armink 大神发起，遵循 MIT 开源协议，代码在 Github 上的托管仓库如下：

https://github.com/armink/SFUD

# 2. 移植 SFUD 之前的准备

## 带有 SPI Flash的开发板准备

这里我准备的是小熊派开发板，主控芯片 `STM32L431RCT6`，板载 SPI Flash 型号为 `W25Q64JV`，板载 ST-Link下载器：

![ ](http://mculover666.cn/blog/20190919/zHx5xMYUmSW1.png?imageslim)

## 工具准备

- STM32CubeMX：用于配置QSPI外设和串口外设，并生成 MDK 工程；
- Keil MDK：用于编译和下载工程；
- 串口助手：用于查看开发板串口调试信息输出；

## 裸机工程准备

在移植之前，首先需要准备好一份裸机工程，主要完成以下配置：

- 时钟配置
- 根据实际 SPI Flash 硬件连接情况配置通信接口（SPI接口或者QSPI接口）
- 根据实际情况配置一个串口；

具体的配置过程可以参考我的 STM32CubeMX 系列教程：

- 

## SFUD源码准备

使用 Git 工具拉取 SFUD 源码到本地：
```bash
git clone https://github.com/armink/SFUD.git
```

SFUD 源码的几个文件夹作用如下：

![ ](http://mculover666.cn/blog/20190924/9anNoAdhumvt.png?imageslim)

# 3. 复制 SFUD 文件到裸机工程

## 3.1. 复制裸机工程到 demo 文件夹
复制之前使用STM32CubeMX生成的裸机工程到 `demo` 文件夹中：

![ ](http://mculover666.cn/blog/20190924/m1ibhvFtql0a.png?imageslim)

## 3.2. 创建 components 文件夹

在工程中创建 components 文件夹，用于**存放 SFUD 适配本工程相关的一些代码**，包括针对本工程的 printf 实现代码和 SFUD 配置代码。

### 建立 components/others 文件夹

>该文件夹中主要存放两个文件：`bsp.c`和`types.h`。

`types.h`是 SFUD 用到的一些数据类型相关宏定义，无论什么型号芯片，都是通用的，可以直接从别的demo工程中复制过来。

`bsp.c`是printf重定向到串口的实现，SFUD 中的调试信息都是使用 printf 打印的，所以该文件非常重要，这里我使用的是STM32 HAL库，所以可以直接从STM32的demo工程中复制过来。

![ ](http://mculover666.cn/blog/20190924/OxIpfsUIs4G5.png?imageslim)

### 建立 components/sfud 文件夹

>该文件夹主要存放 SFUD 适配本工程的文件， 包括 SFUD 底层移植文件`sfud_port.c`和 SFUD 配置文件`sfud_cfg.h`。

这两个文件是需要自己适配的，这里我底层使用的是STM32 HAL库，所以我直接从其他 STM32 Demo 工程中拷贝过来：

![ ](http://mculover666.cn/blog/20190924/eevxLNaUH61U.png?imageslim)


# 4. 添加文件并测试printf实现

打开工程中的 Keil-MDK 工程，开始向工程中添加printf实现文件 `bsp.c`，如图：

![ ](http://mculover666.cn/blog/20190924/joS1xMsY0Uzd.png?imageslim)

添加之后在`main.c`中测试printf是否可以正常打印：

在 `main.c` 开始添加头文件：

![ ](http://mculover666.cn/blog/20190924/cTqeC62FBmRF.png?imageslim)

然后在 `main函数` 中开始测试：

![ ](http://mculover666.cn/blog/20190924/8pDlbaufLzlU.png?imageslim)

编译下载运行，在串口助手可以看到 printf 可以正常打印：

![ ](http://mculover666.cn/blog/20190924/BDlMm2q3UP88.png?imageslim)

如果打印失败的话，就要停下来检查错误了，不能再进行下面的工作。

# 5. 添加和使用 SFUD 库

## 添加 SFUD 相关文件

向 MDK 工程中添加 SFUD 相关的文件：

新建 SFUD 组：

![ ](http://mculover666.cn/blog/20190924/RM7zXS4WYbgc.png?imageslim)

添加 SFUD 实现文件：

![ ](http://mculover666.cn/blog/20190924/yEuLB6Ai01Pn.png?imageslim)

添加 SFUD 移植适配文件：

![ ](http://mculover666.cn/blog/20190924/wbs75ASlE2Yo.png?imageslim)

## 添加 SFUD 相关头文件路径

![ ](http://mculover666.cn/blog/20190924/LdWlQfg8rVun.png?imageslim)

## 修改 SFUD 配置文件

首先在`sfud_cfg.h`中配置SFUD，具体的说明可以查看 SFUD 源码中的`readme.md`文件：

![ ](http://mculover666.cn/blog/20190924/8MR9YFY2aAX6.png?imageslim)

## 编写demo使用函数

首先包含 SFUD 头文件：

![ ](http://mculover666.cn/blog/20190924/PNA0Qvut30DY.png?imageslim)

然后开辟一个测试缓冲区：

```c
/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */
void sfud_demo(uint32_t addr, size_t size, uint8_t *data);

#define SFUD_DEMO_TEST_BUFFER_SIZE                     1024
static uint8_t sfud_demo_test_buf[SFUD_DEMO_TEST_BUFFER_SIZE];

/* USER CODE END 0 */
```

编写声明的 `sfud_demo`函数测试 SFUD 使用：
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
    const sfud_flash *flash = sfud_get_device(SFUD_W25_DEVICE_INDEX);
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

最后在 `main函数` 中调用：
```c
/* USER CODE BEGIN 2 */

printf("SFUD port by mculover666.\r\n");
/* SFUD initialize */
if (sfud_init() == SFUD_SUCCESS)
{
        /* enable qspi fast read mode, set one data lines width */
        sfud_qspi_fast_read_enable(sfud_get_device(SFUD_W25_DEVICE_INDEX), 1);
        sfud_demo(0, sizeof(sfud_demo_test_buf), sfud_demo_test_buf);
}
/* USER CODE END 2 */
```
## 测试结果
编译下载之后，在串口终端可以看到测试结果：

![ ](http://mculover666.cn/blog/20190924/Db0el6CaDB0e.png?imageslim)

![ ](http://mculover666.cn/blog/20190924/PmQybl7ON6d2.png?imageslim)

![ ](http://mculover666.cn/blog/20190924/Tpdhr5wCEr5F.png?imageslim)

至此，SFUD移植测试完毕，大家有问题欢迎留言~

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![mark](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)


