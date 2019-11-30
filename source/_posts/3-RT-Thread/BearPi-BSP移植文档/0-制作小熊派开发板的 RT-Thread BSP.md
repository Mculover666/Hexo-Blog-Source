---
title: 制作小熊派开发板的 RT-Thread BSP
tags: STM32CubeMX RT-Thread
categories: RT-Thread
abbrlink: 1205500533
date: 2019-08-11 16:48:56
---
本篇文章详细的讲述了如何制作小熊派开发板（STM32L431RCT6）的RT-Thread BSP。

<!--more-->
# 0. 准备和说明
## 硬件准备
- 小熊派开发板一个

![BearPi-Board](http://mculover666.cn/image/20190822/f02AzcFHINml.png?imageslim)

## 软件准备
- Keil MDK
- STM32CubeMX
- 串口终端（Putty/XShell/MobaXterm）

## RT-Thread相关准备
- Git工具
- RT-Thread代码
- RT-Thread ENV 工具

RT-Thread相关资源都可以在[RT-Thread官方下载链接](https://www.rt-thread.org/page/download.html)下载到。


# 1. 复制L4通用模板
制作新 BSP 的第一步是复制一份同系列的 BSP 模板作为基础，通过对 BSP 模板的修改来获得新 BSP。

目前提供的 BSP 模板系列如下表所示：

| 工程模板 | 说明 |
| ------- | ---- |
| libraries/templates/stm32f0xx | F0 系列 BSP 模板 |
| libraries/templates/stm32f10x | F1 系列 BSP 模板 |
| libraries/templates/stm32f4xx | F4 系列 BSP 模板 |
| libraries/templates/stm32f7xx | F7 系列 BSP 模板 |
| libraries/templates/stm32l4xx | L4 系列 BSP 模板 |

本次示例所用的 L4 系列 BSP 模板文件夹结构如下所示：

![L4 系列 BSP 模板文件夹内容](http://mculover666.cn/image/20190820/X4M4RlzVR5Fc.png?imageslim)

拷贝模板文件夹下的 `stm32l4xx` 文件夹，并将该文件夹的名称改为 `stm32l431-iotclub-bearpi` ，如下图所示：

![复制通用模板](http://mculover666.cn/image/20190820/mPaSu4gcsn6n.png?imageslim)

在接下来的 BSP 的制作过程中，将会修改 board 文件夹内的配置文件，将 L4 系列的 BSP 模板变成一个适用于IoT Club `stm32l431-iotclub-bearpi` 开发板的 BSP ，下表总结了 board 文件夹中需要修改的内容：

| 项目 | 需要修改的内容说明 |
|-------------|-------------------------------------------------------|
| CubeMX_Config （文件夹）| CubeMX 工程 |
| linker_scripts （文件夹）| BSP 特定的链接脚本 |
|board.c/h | 系统时钟、GPIO 初始化函数、芯片存储器大小 |
| Kconfig | 芯片型号、系列、外设资源 |
| SConscript | 芯片启动文件、目标芯片型号 |

# 2 使用 CubeMX 配置工程

>RT-Thread提供的官方模板中使用的是 STM32CubeMX 5.0.0 和 STM32CubeFW_L4 V1.13.0 版本，为了统一，请确保环境一致。

在制作 BSP 的第二步，需要创建一个基于目标芯片的 CubeMX 工程。

默认的 CubeMX 工程在 **CubeMX_Config** 文件夹中，双击打开 `CubeMX_Config.ioc` 工程，如下图所示：

![open_cubemx](http://mculover666.cn/image/20190820/2rgijVnNqEyf.png?imageslim)

在 CubeMX 工程中将芯片型号为修改芯片型号为 STM32L431RCTx 。

## 2.1 生成 CubeMX 工程

配置系统时钟、外设引脚等，步骤如下图所示：

1. 打开外部时钟

![打开外部时钟](http://mculover666.cn/image/20190820/HqOaVtIhT4Jn.png?imageslim)

2. 设置下载方式、打开串口外设（注意只需要选择串口外设引脚即可，无需配置其他参数）：

![设置下载方式](http://mculover666.cn/image/20190820/GJkDLUVTanHd.png?imageslim)

3. 打开串口外设

![打开串口外设](http://mculover666.cn/image/20190820/qYkRfUa0giYe.png?imageslim)

4. 配置系统时钟

![配置系统时钟](http://mculover666.cn/image/20190820/EmYQ1Mg7e4DR.png?imageslim)

5. 设置项目名称，并在指定地址重新生成 CubeMX 工程：

![生成对应的配置代码](http://mculover666.cn/image/20190820/j9IFhavKhAjq.png?imageslim)

最终 CubeMX 生成的工程目录结构如下图所示：

![CubeMX](http://mculover666.cn/image/20190820/DzMwbyQx76NA.png?imageslim)

## 2.2 拷贝初始化函数

在 **board.c** 文件中存放了函数 `SystemClock_Config()` ，该函数负责初始化系统时钟。当使用 CubeMX 工具对系统时钟重新配置的时候，需要更新这个函数。

该函数由 CubeMX 工具生成，默认存放在`board/CubeMX_Config/Src/main.c` 文件中。但是该文件并没有被包含到我们的工程中，因此需要将这个函数从 main.c 中拷贝到 board.c 文件中。在整个 BSP 的制作过程中，这个函数是唯一要要拷贝的函数。

在 **board.h** 文件中配置了 FLASH 和 RAM 的相关参数，这个文件中需要修改的是 `STM32_FLASH_SIZE` 和 `STM32_SRAM_SIZE` 这两个宏控制的参数。本次制作的 BSP 所用的 STM32L431RCTx 芯片的 flash 大小为 256k，ram 的大小为 64k，因此对该文件作出如下的修改：

![修改 board.h](http://mculover666.cn/image/20190820/4cRabPU5v6S6.png?imageslim)

# 3 修改 Kconfig 选项

在本小节中修改 `board/Kconfig` 文件的内容有如下两点：

- 芯片型号和系列
- BSP 上的外设支持选项

芯片型号和系列的修改如下表所示：

| 宏定义             | 意义     | 格式               |
| ------------------ | -------- | ------------------ |
| SOC_STM32F103RB    | 芯片型号 | SOC_STM32xxx       |
| SOC_SERIES_STM32F1 | 芯片系列 | SOC_SERIES_STM32xx |

关于 BSP 上的外设支持选项，一个初次提交的 BSP 仅仅需要支持 GPIO 驱动和串口驱动即可，因此在配置选项中只需保留这两个驱动配置项，如下图所示：

![修改 Kconfig](http://mculover666.cn/image/20190820/Q6MeERYGbW4W.png?imageslim)

![修改 Kconfig](http://mculover666.cn/image/20190820/dkwoifqR2cFy.png?imageslim)

# 4. 修改工程构建相关文件

接下来需要修改用于构建工程相关的文件。

## 4.1 修改链接脚本
**linker_scripts** 链接文件如下图所示：

![需要修改的链接脚本](http://mculover666.cn/image/20190820/nB8a8nD82eMk.png?imageslim)

下面以 MDK 使用的链接脚本 link.sct 为例，演示如何修改链接脚本：

![linkscripts_change](http://mculover666.cn/image/20190820/NoTupzavroS6.png?imageslim)

本次制作 BSP 使用的芯片为 STM32L431RC，FLASH 为 256k，因此修改 LR_IROM1 和 ER_IROM1 的参数为 0x00040000。RAM 的大小为64k， 因此修改 RW_IRAM1 的参数为 0x00010000。这样的修改方式在一般的应用下就够用了，后续如果有特殊要求，则需要按照链接脚本的语法来根据需求修改。

其他两个链接脚本的文件分别为 iar 使用的 link.icf 和 gcc 编译器使用的 link.lds，修改的方式也是类似的，如下图所示：

- link.icf 修改内容

    ![link_icf](http://mculover666.cn/image/20190820/DUvXfA465sSL.png?imageslim)

- link.lds 修改内容

    ![link_lds](http://mculover666.cn/image/20190820/bkbJ606lyJIx.png?imageslim)

##  4.2 修改构建脚本

**SConscript** 脚本决定 MDK/IAR 工程的生成以及编译过程中要添加文件。

在这一步中需要修改芯片型号以及芯片启动文件的地址，修改内容如下图所示：

![修改启动文件和芯片型号](http://mculover666.cn/image/20190820/r1VaCmXarlJO.png?imageslim)

注意：如果在文件夹中找不到相应系列的 .s 文件，可能是多个系列的芯片重用了相同的启动文件，此时可以在 CubeMX 中生成目标芯片的工程，查看使用了哪个启动文件，然后再修改启动文件名。

## 4.3 修改工程模板

**template** 文件是生成 MDK/IAR 工程的模板文件，通过修改该文件可以设置工程中使用的芯片型号以及下载方式。MDK4/MDK5/IAR 的工程模板文件，如下图所示：

![MDK/IAR 工程模板](http://mculover666.cn/image/20190820/pxyT89INq8mf.png?imageslim)

下面以 MDK5 模板的修改为例，介绍如何修改模板配置：

![选择芯片型号](http://mculover666.cn/image/20190820/tkztlpsn6HcC.png?imageslim)

修改程序下载方式：

![配置下载方式](http://mculover666.cn/image/20190820/t4hQx7SfaWKd.png?imageslim)

# 5 重新生成工程 

重新生成工程需要使用 env 工具。

## 5.1 重新生成 rtconfig.h 文件

在 env 界面输入命令 menuconfig 对工程进行配置，并生成新的 rtconfig.h 文件。如下图所示：

![输入menuconfig进入配置界面](http://mculover666.cn/image/20190820/gv3qM59HGDMS.png?imageslim)

![选择要打开的外设](http://mculover666.cn/image/20190820/KQhwvyUq9lRb.png?imageslim)

## 5.2 重新 MDK/IAR 工程
下面以重新生成 MDK 工程为例，介绍如何重新生成 BSP 工程。

使用 env 工具输入命令 `scons --target=mdk5` 重新生成工程，如下图所示：

![重新生成 BSP 工程](http://mculover666.cn/image/20190820/c5JB38StRkp7.png?imageslim)

重新生成工程成功：

![重新生成 BSP 工程](http://mculover666.cn/image/20190820/x9JN4N9n1tTl.png?imageslim)

到这一步为止，新的 BSP 就可以使用了。

接下来我们可以分别使用命令 `scons --target=mdk4` 和 `scons --target=iar`，来更新 mdk4 和 iar 的工程，使得该 BSP 变成一个完整的，可以提交到 GitHub 的 BSP。

# 6. BSP测试
打开刚刚生成的MDK5工程，在 main.c 中修改LED所在引脚：

![修改LED引脚](http://mculover666.cn/image/20190820/zco5UJa7TKLv.png?imageslim)

编译下载，在串口终端中看到如下结果：

![串口终端结果](http://mculover666.cn/image/20190820/Nunqw4Q4vHXu.png?imageslim)

同时可以看到开发板上蓝色的LED闪烁。

# 7. BSP规范
