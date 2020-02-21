---
title: 桌面mini网络时钟教程 | 02-获取温湿度传感器数据
keywords: RT-Thread
tags: 制作过程
categories: 制作过程
abbrlink: 2192950495
summary: 使用RT-Thread Studio打造
date: 2020-02-20 18:01:56
---

# 1. 项目进度
桌面Mini时钟项目用来演示如何使用RT-Thread Stduio开发项目，整个项目的架构如下：

![](https://img-blog.csdnimg.cn/2020020311120887.png#pic_center) 

在上一篇博文中简单的介绍了RT-Thread Studio一站式工具，基于STM32L431RCT6这个芯片创建工程，并修改时钟为使用外部时钟。

>[使用RT-Thread Studio DIY 迷你桌面时钟（一）| 基于STM32芯片创建工程](https://blog.csdn.net/Mculover666/article/details/104146623)

接下里我们开始添加I2C设备，添加SHT3x软件包，获取SHT3x温湿度传感器数据。

# 2. 添加I2C设备
## 2.1. 打开I2C设备驱动框架
双击左侧 `RT-Thread Setting` 文件，即可打开RT-Thread图形化配置工具，软件模拟I2C这一项是灰色的，表示没有打开，单击一下即可打开软件 I2C 的驱动框架，图标变为彩色表示打开：
![](https://img-blog.csdnimg.cn/20200203113745577.png)
右击该选项可以打开更多配置，比如查看该驱动设备的依赖、查看该驱动设备的详细配置，查看该驱动设备的API文档，查看在线文档等操作：
![](https://img-blog.csdnimg.cn/20200203113918385.png)
按`Ctrl+S`保存，配置生效，软件会自动添加I2C设备驱动框架到工程中：
![](https://img-blog.csdnimg.cn/20200203114145426.png)
## 2.2. 添加软件 I2C 源码
打开了软件 I2C 的驱动框架之后，还要添加软件I2C的驱动底层实现，具体芯片的软件 I2C 驱动源码不同，本例中下载添加 STM32 系列的软件 I2C 驱动：[Gitee 下载地址](https://gitee.com/tyustli/tyustli/tree/master/stm32/soft-i2c)。
```
git clone https://gitee.com/tyustli/tyustli.git
```
下载之后源码只有两个文件：
![](https://img-blog.csdnimg.cn/20200203114518477.png)
将这两个文件添加到项目中的`drivers`文件夹中：
![](https://img-blog.csdnimg.cn/2020020311463736.png)
回到RT-Thread Studio IDE，在项目名称上右击，选择刷新，即可在目录中看到添加的文件：
![](https://img-blog.csdnimg.cn/20200203114732443.png)
## 2.3. 注册 I2C 设备
软件 I2C 添加到工程中之后就可以调用软件 I2C 注册函数 `rt_hw_i2c_init` 来注册软件 I2C 设备了，该函数的原型如下：
```c
int rt_hw_i2c_init(char *name, rt_uint8_t scl, rt_uint8_t sda)
```
- name：设备名称
- scl：软件模拟I2C的SCL引脚
- sda：软件模拟I2C的SDA引脚

在小熊派IoT开发板上，温湿度传感器SHT30连接在PB6（SCL）和PB7（SDA） ，所以在`main.c`文件中先添加头文件：
```c
#include <drv_soft_i2c.h>
```
然后在文件**最后**添加如下注册软件 I2C到系统中的代码：
```c
int register_i2c(void)
{
    rt_hw_i2c_init("i2c1", GET_PIN(B,6), GET_PIN(B,7));

    return RT_EOK;
}
//注册到系统中，自动初始化设备
INIT_BOARD_EXPORT(register_i2c);
```
添加完成之后点击编译，下载到开发板中运行，即可在串口终端中看到日志信息（绿色），提示I2C总线设备已注册成功：
![](https://img-blog.csdnimg.cn/20200220205525306.png)

因为main线程中循环打印对使用控制台有影响，所以将打印函数注释：
![](https://img-blog.csdnimg.cn/20200203115933671.png)

重新编译下载，在串口终端中输入命令`list_device`查看系统中注册的设备吗，再次确认I2C总线设备注册成功：
![](https://img-blog.csdnimg.cn/20200220205625815.png)
# 3. 添加SHT3x软件包
## 3.1. 搜索添加软件包
RT-Thread Studio有在线软件包中心，里面有非常丰富的软件包供用户使用，点击立即添加进入：
![](https://img-blog.csdnimg.cn/20200203125004277.png)

搜索`SHT`，使用的传感器型号为SHT30，所以添加SHT3x软件包：
![](https://img-blog.csdnimg.cn/20200203125057439.png)
添加之后，在项目设置中即可看到该软件包，右击可以查看软件包在线文档：
![](https://img-blog.csdnimg.cn/2020020312523714.png)

软件会自动打开该软件包的在线说明文档，在使用软件包之前，该文档必须要看：
![](https://img-blog.csdnimg.cn/20200203125300994.png)
接下来按`Ctrl+S`保存项目，软件会自动添加软件包代码到工程中，其中`README.md`是软件包详细使用文档，使用前必须要看：
![](https://img-blog.csdnimg.cn/20200203125559261.png)

接下来编译、下载项目到开发板中，查看串口终端输出。
## 3.2. 使用命令测试软件包
大部分软件包都提供测试命令，SHT3x软件包也是一样，提供了如下命令供测试：
```c
- sht3x probe <i2c_dev_name> <pu/pd>  --挂载SHT3x设备，需要指定i2c设备名称和上下拉方式，默认下拉
- sht3x read --阅读SHT3x温湿度
- sht3x status --读取查看状态寄存器值
- sht3x reset --软件复位SHT3x
- sht3x heater <on/off> --开/关heater
```
首先挂载SHT30温湿度传感器到之前注册的I2C总线设备上：
![](https://img-blog.csdnimg.cn/20200220205752729.png)

挂载之后可以进行查看寄存器，读取温湿度，复位，开/关heater操作，比如读取一次温度和湿度：
![](https://img-blog.csdnimg.cn/20200220205821175.png)
# 4. 获取温湿度传感器数据
在使用软件包提供的命令测试设备成功之后，创建一个线程，使用SHT3x软件包提供的API获取SHT3x数据。

在Applicaition分组之下，新建一个文件`sht30_ccollect.c`用于存放SHT3x获取数据的相关代码，编辑以下内容：

```c
/*
 * Copyright (c) 2006-2019, RT-Thread Development Team
 *
 * SPDX-License-Identifier: Apache-2.0
 *
 * Change Logs:
 * Date           Author       Notes
 * 2020-02-20     Mculover666  the first version
 */

#include <rtthread.h>
#include <board.h>
#include <sht3x.h>

#define THREAD_PRIORITY         25
#define THREAD_STACK_SIZE       512
#define THREAD_TIMESLICE        5

static rt_thread_t tid1 = RT_NULL;

/* 入口函数 */
static void sht30_collect_thread_entry(void *parameter)
{
    sht3x_device_t  sht3x_device;

    sht3x_device = sht3x_init("i2c1", 0x44);

    while (1)
    {
        if(RT_EOK == sht3x_read_singleshot(sht3x_device))
        {
            rt_kprintf("sht30 humidity   : %d.%d  ", (int)sht3x_device->humidity, (int)(sht3x_device->humidity * 10) % 10);
            rt_kprintf("temperature: %d.%d\n", (int)sht3x_device->temperature, (int)(sht3x_device->temperature * 10) % 10);
        }
        else
        {
            rt_kprintf("read sht3x fail.\r\n");
            break;
        }
        rt_thread_mdelay(2000);
    }
}

/* 创建线程 */
int sht30_collect(void)
{
    /* 创建线程*/
    tid1 = rt_thread_create("sht30_collect_thread",
            sht30_collect_thread_entry, RT_NULL,
                            THREAD_STACK_SIZE,
                            THREAD_PRIORITY, THREAD_TIMESLICE);

    /* 如果获得线程控制块，启动这个线程 */
    if (tid1 != RT_NULL)
        rt_thread_startup(tid1);

    return 0;
}
```

在main.c中测试该程序：
![](https://img-blog.csdnimg.cn/20200203131526137.png)
编译下载程序，在串口终端查看结果：

![](https://img-blog.csdnimg.cn/20200220205943216.png)

<font color="red">**接收更多精彩文章及资源推送，欢迎订阅我的微信公众号：『mculover666』。**</font>
![](https://img-blog.csdnimg.cn/20200202092055136.png)

