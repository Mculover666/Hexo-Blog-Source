---
title: CmBacktrace | 一款 ARM Cortex-M 系列 MCU 错误追踪库
keywords: CmBacktrace
tags: 嵌入式开源项目精选专栏
categories: OpenSource
abbrlink:
summary: 嵌入式开源项目精选专栏
date: 2020-08-01 08:00:00
---

![](https://img-blog.csdnimg.cn/20200321151234798.png)

# 1. CmBacktrace
本期给大家带来的开源项目是 CmBacktrace，**一款针对 ARM Cortex-M 系列 MCU 的错误代码自动追踪、定位，错误原因自动分析的开源库**，作者armink，目前收获 611 个 star，遵循 MIT 开源许可协议。

目前 CmBacktrace支持以下功能：

- 支持断言（assert）和故障（Hard Fault）
- 故障原因自动诊断
- 输出错误现场的 函数调用栈
- 适配 Cortex-M0/M3/M4/M7 MCU；
- 支持 IAR、KEIL、GCC 编译器；

>项目地址：[https://github.com/armink/CmBacktrace](https://github.com/armink/CmBacktrace)

# 2. 移植CmBacktrace 
## 2.1. 移植思路
在移植过程中主要参考两个资料：项目的readme文档和demo工程。

对于这些开源项目，其实移植起来也就两步：

- ① 添加源码到裸机工程中；
- ② 实现需要的接口即可；

## 2.2. 准备裸机工程
本文中我使用的是小熊派IoT开发套件，主控芯片为STM32L431RCT6：

![](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2Jsb2cvMjAxOTA5MTkvekh4NXhNWVVtU1cxLnBuZw?x-oss-process=image/format,png#pic_center)

移植之前需要准备一份裸机工程，我使用STM32CubeMX生成，需要初始化以下配置：

- 配置一个串口用于打印信息
- printf重定向

# 2.3. 添加CmBacktrace到工程中
① 复制CmBacktrace源码到工程中：
![](https://img-blog.csdnimg.cn/20200527101937275.png)

② 在keil中添加 CmBacktrace 组件的源码文件：

![](https://img-blog.csdnimg.cn/20200527102229339.png)

③ 添加CmBacktrace的头文件路径：
![](https://img-blog.csdnimg.cn/20200527102329965.png)
## 2.4. 去除原有的HardFault_Handler
在`stm32l4xx_it.c`文件中注释该函数，防止冲突：
![](https://img-blog.csdnimg.cn/20200527102549811.png)
## 2.5. 配置CmBacktrace
CmBacktrace的配置文件在`cmb_cfg.h`，针对不同的平台和场景，需要自行手动配置：
![](https://img-blog.csdnimg.cn/20200529105437224.png)
本文使用的是Cortex-M4裸机平台，配置如下：
![](https://img-blog.csdnimg.cn/20200529105727957.png)

至此，CmBacktrace移植、配置完成，接下来就可以愉快的使用了！

# 3. 使用CmBacktrace
使用时包含头文件：
```c
#include "cm_backtrace.h"
```
## 3.1. 初始化CmBacktrace 
```c
void cm_backtrace_init(const char *firmware_name, const char *hardware_ver, const char *software_ver);
```
该 API 用来初始化CmBacktrace，其中传入的参数分别为：

- `firmware_name`：固件名称，需与编译器生成的固件名称对应；
- `hardware_ver`：固件对应的硬件版本号；
- `software_ver`：固件的软件版本号；

这三个参数用宏定义的方式，会比较方便：
```c
/* Private define ------------------------------------------------------------*/
/* USER CODE BEGIN PD */
#define APPNAME                        "CmBacktrace"
#define HARDWARE_VERSION               "V1.0.0"
#define SOFTWARE_VERSION               "V0.0.1"

/* USER CODE END PD */
```

在main函数中初始化CmBacktrace ：
```c
/* USER CODE BEGIN 2 */
printf("CmBacktrace Test...\r\n");
cm_backtrace_init(APPNAME, HARDWARE_VERSION, SOFTWARE_VERSION);

/* USER CODE END 2 */
```
## 3.2. 追踪故障错误信息
库本身提供了 HardFault 处理的汇编文件（`cmb_fault.S`），会在故障时自动调用 cm_backtrace_fault 方法，之前移植时已经添加，这里直接人工制作一个错误，来看看使用效果。

定义一个除0错误的函数：
```c
/* Private variables ---------------------------------------------------------*/
/* USER CODE BEGIN PV */
void fault_test_by_div0(void)
{
    volatile int * SCB_CCR = (volatile int *) 0xE000ED14; // SCB->CCR
    int x, y, z;

    *SCB_CCR |= (1 << 4); /* bit4: DIV_0_TRP. */

    x = 10;
    y = 0;
    z = x / y;
    printf("z:%d\n", z);
}
/* USER CODE END PV */
```
然后在初始化之后调用执行此函数：
```c
/* USER CODE BEGIN 2 */
printf("CmBacktrace Test...\r\n");
cm_backtrace_init(APPNAME, HARDWARE_VERSION, SOFTWARE_VERSION);
fault_test_by_div0();
/* USER CODE END 2 */
```
编译，下载运行，在串口助手中查看结果：
![](https://img-blog.csdnimg.cn/20200529115346866.png)
查看进一步的信息，需要使用addr2line工具，此工具在cmbacktrace的源码中，按照你的电脑选择32位的或者64位的：
![](https://img-blog.csdnimg.cn/20200529115958395.png)
将此工具复制到裸机工程的固件所在文件夹中：
![](https://img-blog.csdnimg.cn/20200529120122348.png)

>此处是为了演示方便，实际使用时请将此工具复制到任何一个地方，然后添加到环境变量中，这样就可以在命令行中使用了。

打开cmd命令行，进入axf所在目录，执行串口中提示的命令：
![](https://img-blog.csdnimg.cn/20200529120322660.png)
除了英文之外，还提供了中文支持，修改配置：
![](https://img-blog.csdnimg.cn/20200529212639669.png)
再次编译下载，查看效果：
![](https://img-blog.csdnimg.cn/20200529212710242.png)
# 4. 设计思想解读
## 4.1. 条件编译的使用
CmBacktrace作为一个与底层汇编指令打交道的库，适配非常完善：

- 支持 Cortex-M0/M3/M4/M7 MCU；
- 支持 IAR、KEIL、GCC 编译器；
- 支持裸机平台
- 支持UCos/RT-Thread/FreeRTOS操作系统

要做到如此广泛的支持，armink大神大量的使用了宏定义和条件编译，代码非常优雅，值得一读，比如下面这段代码用来处理不同编译器生成的可执行文件名称不同问题，如果是除MDK，IAR，GCC之外的编译工具，编译将报错：
![](https://img-blog.csdnimg.cn/20200530091710770.png)
再比如下面这端代码更是优雅，本来语言配置是留给用户配置的，如果用户忘了配置，此段代码将设置一个默认值，编译依然正常：
![](https://img-blog.csdnimg.cn/20200530092230242.png)
以上两种条件编译的用法在CmBacktrace中大量出现，在我们写代码的时候值得模仿和学习。

## 4.2. 如何追踪错误
其实要做到自动追踪错误，就是在系统进入故障的时候将CPU环境打印出来，便于分析定位错误。

CmBacktrace库对于CPU环境的抽象是cmb_hard_fault_regs结构体，源码在`cmb_def.h`：
```c
/**
 * Cortex-M fault registers
 */
struct cmb_hard_fault_regs{
  struct {
	//……
  } saved;

  union {
	//……
  } syshndctrl;                          // System Handler Control and State Register (0xE000ED24)

  union {
   //……
  } mfsr;                                // Memory Management Fault Status Register (0xE000ED28)
  unsigned int mmar;                     // Memory Management Fault Address Register (0xE000ED34)

  union {
   //……
  } bfsr;                                // Bus Fault Status Register (0xE000ED29)
  unsigned int bfar;                     // Bus Fault Manage Address Register (0xE000ED38)

  union {
    //……
  } ufsr;                                // Usage Fault Status Register (0xE000ED2A)

  union {
    //……
  } hfsr;                                // Hard Fault Status Register (0xE000ED2C)

  union {
   //……
  } dfsr;                                // Debug Fault Status Register (0xE000ED30)

  unsigned int afsr;                     // Auxiliary Fault Status Register (0xE000ED3C), Vendor controlled (optional)
};
```
① saved结构体：
发生错误时必须要保存R0-R12、LR、PC这些CPU中的寄存器组，本节讲述的重点是PSR寄存器，全称`Program status register`，程序状态寄存器，包括三个，如图：

- Application Program Status Register (APSR)
- Interrupt Program Status Register (IPSR)
- Execution Program Status Register (EPSR)

![](https://img-blog.csdnimg.cn/20200530095429150.png)
因为CPU中的寄存器都是32位的，避免浪费，这三个寄存器**合并在一个PSR寄存器中**，如图：
![](https://img-blog.csdnimg.cn/20200530095803791.png)
```c
  struct {
    unsigned int r0;                     // Register R0
    unsigned int r1;                     // Register R1
    unsigned int r2;                     // Register R2
    unsigned int r3;                     // Register R3
    unsigned int r12;                    // Register R12
    unsigned int lr;                     // Link register
    unsigned int pc;                     // Program counter
    union {
      unsigned int value;
      struct {
        unsigned int IPSR : 8;           // Interrupt Program Status register (IPSR)
        unsigned int EPSR : 19;          // Execution Program Status register (EPSR)
        unsigned int APSR : 5;           // Application Program Status register (APSR)
      } bits;
    } psr;                               // Program status register.
  } saved;
```
② syshndctrl共用体
此共用体用来存储SCB_SHCSR寄存器的值，全称`System handler control and state register`，系统处理程序控制和状态寄存器，该寄存器指示出了系统处理程序是否启用，通俗点说就是异常处理是否使能，内容如图：
![](https://img-blog.csdnimg.cn/20200530101007991.png)
```c
  union {
    unsigned int value;
    struct {
      unsigned int MEMFAULTACT    : 1;   // Read as 1 if memory management fault is active
      unsigned int BUSFAULTACT    : 1;   // Read as 1 if bus fault exception is active
      unsigned int UnusedBits1    : 1;
      unsigned int USGFAULTACT    : 1;   // Read as 1 if usage fault exception is active
      unsigned int UnusedBits2    : 3;
      unsigned int SVCALLACT      : 1;   // Read as 1 if SVC exception is active
      unsigned int MONITORACT     : 1;   // Read as 1 if debug monitor exception is active
      unsigned int UnusedBits3    : 1;
      unsigned int PENDSVACT      : 1;   // Read as 1 if PendSV exception is active
      unsigned int SYSTICKACT     : 1;   // Read as 1 if SYSTICK exception is active
      unsigned int USGFAULTPENDED : 1;   // Usage fault pended; usage fault started but was replaced by a higher-priority exception
      unsigned int MEMFAULTPENDED : 1;   // Memory management fault pended; memory management fault started but was replaced by a higher-priority exception
      unsigned int BUSFAULTPENDED : 1;   // Bus fault pended; bus fault handler was started but was replaced by a higher-priority exception
      unsigned int SVCALLPENDED   : 1;   // SVC pended; SVC was started but was replaced by a higher-priority exception
      unsigned int MEMFAULTENA    : 1;   // Memory management fault handler enable
      unsigned int BUSFAULTENA    : 1;   // Bus fault handler enable
      unsigned int USGFAULTENA    : 1;   // Usage fault handler enable
    } bits;
  } syshndctrl;                          // System Handler Control and State Register (0xE000ED24)
```
③ mfsr共用体、bfsr共用体、ufsr共用体

**这个是故障追踪得以实现的原理**，它用来存储寄存器SCB_CFSR的值，全称`Configurable fault status register`，可配置的故障状态寄存器，顾名思义，它指示出当前系统发生的是什么故障，现在，CmBacktrace没有那么神秘了吧~

CFSR寄存器是一个32位的寄存器，但是它可以按字节访问，所以就分为MMFSR、
BFSR、UFSR三个，如图：
![](https://img-blog.csdnimg.cn/20200530101756186.png)
其中具体每一位代表什么故障类型，如图：
![](https://img-blog.csdnimg.cn/20200530101948308.png)
我用红框圈出的就是本文中演示的故障错误——除0错误，如果你对其它错误有兴趣，请阅读《STM32F10xxx Cortex-M3 programming manual》（编程手册）第141页。

源码如下，可以看到其中每一位都和图中对应：
```c
  union {
    unsigned char value;
    struct {
      unsigned char IACCVIOL    : 1;     // Instruction access violation
      unsigned char DACCVIOL    : 1;     // Data access violation
      unsigned char UnusedBits  : 1;
      unsigned char MUNSTKERR   : 1;     // Unstacking error
      unsigned char MSTKERR     : 1;     // Stacking error
      unsigned char MLSPERR     : 1;     // Floating-point lazy state preservation (M4/M7)
      unsigned char UnusedBits2 : 1;
      unsigned char MMARVALID   : 1;     // Indicates the MMAR is valid
    } bits;
  } mfsr;                                // Memory Management Fault Status Register (0xE000ED28)
  unsigned int mmar;                     // Memory Management Fault Address Register (0xE000ED34)

  union {
    unsigned char value;
    struct {
      unsigned char IBUSERR    : 1;      // Instruction access violation
      unsigned char PRECISERR  : 1;      // Precise data access violation
      unsigned char IMPREISERR : 1;      // Imprecise data access violation
      unsigned char UNSTKERR   : 1;      // Unstacking error
      unsigned char STKERR     : 1;      // Stacking error
      unsigned char LSPERR     : 1;      // Floating-point lazy state preservation (M4/M7)
      unsigned char UnusedBits : 1;
      unsigned char BFARVALID  : 1;      // Indicates BFAR is valid
    } bits;
  } bfsr;                                // Bus Fault Status Register (0xE000ED29)
  unsigned int bfar;                     // Bus Fault Manage Address Register (0xE000ED38)

  union {
    unsigned short value;
    struct {
      unsigned short UNDEFINSTR : 1;     // Attempts to execute an undefined instruction
      unsigned short INVSTATE   : 1;     // Attempts to switch to an invalid state (e.g., ARM)
      unsigned short INVPC      : 1;     // Attempts to do an exception with a bad value in the EXC_RETURN number
      unsigned short NOCP       : 1;     // Attempts to execute a coprocessor instruction
      unsigned short UnusedBits : 4;
      unsigned short UNALIGNED  : 1;     // Indicates that an unaligned access fault has taken place
      unsigned short DIVBYZERO0 : 1;     // Indicates a divide by zero has taken place (can be set only if DIV_0_TRP is set)
    } bits;
  } ufsr;
```
剩余的hfsr寄存器、dfsr寄存器、afsr寄存器知道了也没多大意义，省略不讲。

# 5. 项目工程源码获取和问题交流
目前我将CmBacktrace 源码、我移植到小熊派STM32L431RCT6开发板的工程源码上传到了QQ群里（包含好几份HAL库，QQ相对速度快点），**可以在QQ群里下载，有问题也可以在群里交流，当然也欢迎大家分享出来自己移植的工程到QQ群里**：
![](https://img-blog.csdnimg.cn/20200529213645919.png)

放上QQ群二维码：
![](https://img-blog.csdnimg.cn/20200321144543508.png)

<font color="red">**接收更多精彩文章及资源推送，欢迎订阅我的微信公众号：『mculover666』。**</font>
![](https://img-blog.csdnimg.cn/20200202092055136.png)
