---
title: EasyLogger | 一款轻量级且高性能的日志库
keywords: EasyLogger
tags: 嵌入式开源项目精选专栏
categories: OpenSource
abbrlink: 524269974
summary: 嵌入式开源项目精选专栏
date: 2020-04-10 08:00:56
---

![](https://img-blog.csdnimg.cn/20200321151234798.png)
# 嵌入式开源项目精选专栏
本专栏由Mculover666创建，主要内容为寻找嵌入式领域内的优质开源项目，一是帮助开发者使用开源项目实现更多的功能，二是通过这些开源项目，学习大佬的代码及背后的实现思想，提升自己的代码水平，和其它专栏相比，本专栏的优势在于：

**不会单纯的介绍分享项目，还会包含作者亲自实践的过程分享，甚至还会有对它背后的设计思想解读**。

目前本专栏包含的开源项目有：

- [SFUD | 一个简洁实用的开源项目，帮你轻松搞定SPI Flash](https://blog.csdn.net/Mculover666/article/details/101351255)
- [cJSON | 一个轻量级C语言JSON解析器](https://blog.csdn.net/Mculover666/article/details/103796256)
- [paho | 支持10种语言编写mqtt客户端，总有一款适合你！](https://blog.csdn.net/Mculover666/article/details/103935428)
- [MultiButton | 一个小巧简单易用的事件驱动型按键驱动模块](https://blog.csdn.net/Mculover666/article/details/104992661)
- [letter-shell | 一个功能强大的嵌入式shell](https://blog.csdn.net/Mculover666/article/details/105141286)

如果您自己编写或者发现的开源项目不错，欢迎留言或者私信投稿到本专栏，分享获得双倍的快乐！

# 1. EasyLogger
本期给大家带来的开源项目是 EasyLogger，**一款轻量级且高性能的日志库**，作者armink，目前收获 1.1K 个 star，遵循 MIT 开源许可协议。

EasyLogger 是一款超轻量级、高性能的 C/C++ 日志库，非常适合对资源敏感的软件项目，相比之下， EasyLogger 的功能更加简单，提供给用户的接口更少，上手会更快，更多实用功能支持以插件形式进行动态扩展。

目前EasyLogger支持以下功能：

- 日志输出方式支持串口、Flash、文件等；
- 日志内容可包含级别、时间戳、线程信息、进程信息等；
- 支持多种操作系统，支持裸机；
- 各级别日志支持不同颜色显示；

![](https://img-blog.csdnimg.cn/20200407195013428.png)

>项目地址：[https://github.com/armink/EasyLogger](https://github.com/armink/EasyLogger)

# 2. 移植EasyLogger
## 2.1. 移植思路
在移植过程中主要参考两个资料：项目的readme文档和demo工程。

对于这些开源项目，其实移植起来也就两步：

- ① 添加源码到裸机工程中；
- ② 实现需要的接口即可；

## 2.2. 准备裸机工程
本文中我使用的是小熊派IoT开发套件，主控芯片为STM32L431RCT6：
![](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2Jsb2cvMjAxOTA5MTkvekh4NXhNWVVtU1cxLnBuZw?x-oss-process=image/format,png#pic_center)
移植之前需要准备一份裸机工程，我使用STM32CubeMX生成，**使用USART1的查询方式发送数据，并将printf重定向到USART1**，具体过程请参考：

- [STM32CubeMX_06 | 使用USART发送和接收数据（查询模式）](http://www.mculover666.cn/posts/2064921339/)
- [STM32CubeMX_09 | 重定向printf函数到串口输出的多种方法](http://www.mculover666.cn/posts/2251182441/)

串口USART1配置如下：
![](https://img-blog.csdnimg.cn/20200409110435421.png)
生成工程后printf重定向代码如下：
```c
#include <stdio.h>

int fputc(int ch, FILE *stream)
{
    /* 堵塞判断串口是否发送完成 */
    while((USART1->ISR & 0X40) == 0);

    /* 串口发送完成，将该字符发送 */
    USART1->TDR = (uint8_t) ch;

    return ch;
}
```

裸机工程准备好之后开始移植easylogger。


## 2.3. 添加elog到工程中

① 复制源码到工程中：
![](https://img-blog.csdnimg.cn/20200409104650187.png)
② 在keil中添加easylogge组件的源码文件：

- `port/elog_port.c`：elog移植接口文件；
- `src/elog.c`：elog核心功能源码；
- `src/elog_utils.c`：elog所用到的一些c库工具函数实现；
- `src/elog_buf.c`（可选添加）：elog缓冲输出模式源码；
- `src/elog_async.c`（可选添加）：elog异步输出模式源码；

![](https://img-blog.csdnimg.cn/20200409105846494.png)

③ 将`easylogger/inc`头文件路径添加到keil中：
![](https://img-blog.csdnimg.cn/2020040910593925.png)
## 2.4. 实现elog移植接口
elog的移植接口都已经写好了，在`elog_port.c`文件中，只需要在函数体中添加代码即可。

① elog初始化接口
```c
ElogErrCode elog_port_init(void);
```
如果涉及到后续elog使用资源的初始化，比如动态申请分配缓冲区内存，可以放在此接口中，本文中保持默认。

② elog日志输出接口（重点）
```c
//开头添加
#include <stdio.h>

……

//接口实现
void elog_port_output(const char *log, size_t size) {
	//日志使用printf输出，printf已经重定向到串口USART1
	printf("%.*s", size, log);
}
```
这儿有个小知识点，`%s`表示字符串输出，`.<十进制数>`是精度控制格式符，输出字符时表示输出字符的位数，在精度控制时，**小数点后的十进制数**可以使用`*`来占位，在后面提供一个变量作为精度控制的具体值。

③ 日志输出上锁/解锁接口

该接口可以对日志输出接口进行上锁/解锁，以**保证日志在并发输出时的正确性**，本文中使用的是裸机程序，所以在此使用关闭全局中断来加锁，打开全局中断来解锁：
```c
//开头添加
#include <stm32l4xx_hal.h>

……

//接口实现
void elog_port_output_lock(void) {
    
    //关闭全局中断
	__set_PRIMASK(1);
  
}
void elog_port_output_unlock(void) {
    
    //开启全局中断
	__set_PRIMASK(0);
    
}
```
STM32开关全局中断的方式很多，本文中直接操作 PRIMASK 寄存器来快速的屏蔽/打开全局中断，参考文章：

>https://blog.csdn.net/working24hours/article/details/88323241

④ 系统信息获取接口

elog提供了三个接口用来获取当前时间、获取进程号、获取线程号，因为本文中移植到裸机工程中，并且没有提供时间支持，所以这三个接口都返回空字符串，如下：
```c
const char *elog_port_get_time(void) {
    
	return "";
    
}
const char *elog_port_get_p_info(void) {

	return "";
    
}
const char *elog_port_get_t_info(void) {

	return "";
    
}
```
## 2.5. 配置elog
elog的核心功能开启宏定义和核心参数宏定义都在配置文件`elog_cfg.h`中，在本文中只讲述其中重要的宏定义。

日志输出总开关：
```c
/* enable log output. */
#define ELOG_OUTPUT_ENABLE
```
换行符宏定义修改如下：
```c
/* output newline sign */
#define ELOG_NEWLINE_SIGN                        "\r\n"
```
带有颜色的日志输出开关：
```c
/* enable log color */
#define ELOG_COLOR_ENABLE
```


移植时并没有添加异步输出和缓冲区输出的源码，所以将这两个功能关掉：
![](https://img-blog.csdnimg.cn/20200409155130390.png)
至此，移植配置完成，接下来可以开始愉快的使用啦！

# 3. 使用easylogger
## 3.1. 初始化elog
elog使用之前需要初始化，过程有三步：
① 初始化elog
```c
ElogErrCode elog_init(void);
```
② 设置日志输出格式
```c
void elog_set_fmt(uint8_t level, size_t set);
```
其中第一个参数表示设置哪个日志输出级别对应的输出格式，从以下宏定义中选择一个：
```c
/* output log's level */
#define ELOG_LVL_ASSERT                      0
#define ELOG_LVL_ERROR                       1
#define ELOG_LVL_WARN                        2
#define ELOG_LVL_INFO                        3
#define ELOG_LVL_DEBUG                       4
#define ELOG_LVL_VERBOSE                     5
```
其二个参数是日志输出格式，枚举给出，可以自由组合搭配：
```c
/* all formats index */
typedef enum {
    ELOG_FMT_LVL    = 1 << 0, /**< level */
    ELOG_FMT_TAG    = 1 << 1, /**< tag */
    ELOG_FMT_TIME   = 1 << 2, /**< current time */
    ELOG_FMT_P_INFO = 1 << 3, /**< process info */
    ELOG_FMT_T_INFO = 1 << 4, /**< thread info */
    ELOG_FMT_DIR    = 1 << 5, /**< file directory and name */
    ELOG_FMT_FUNC   = 1 << 6, /**< function name */
    ELOG_FMT_LINE   = 1 << 7, /**< line number */
} ElogFmtIndex;

/* macro definition for all formats */
#define ELOG_FMT_ALL    (ELOG_FMT_LVL|ELOG_FMT_TAG|ELOG_FMT_TIME|ELOG_FMT_P_INFO|ELOG_FMT_T_INFO| ELOG_FMT_DIR|ELOG_FMT_FUNC|ELOG_FMT_LINE)
```

③ 启动elog
```c
void elog_start(void);
```

接下来在main函数中的usart1初始化函数之后，while(1)之前编写elog初始化代码：
```c
/* USER CODE BEGIN 2 */
/* 初始化elog */
elog_init();

/* 设置每个级别的日志输出格式 */
//输出所有内容
elog_set_fmt(ELOG_LVL_ASSERT, ELOG_FMT_ALL);
//输出日志级别信息和日志TAG
elog_set_fmt(ELOG_LVL_ERROR, ELOG_FMT_LVL | ELOG_FMT_TAG);
elog_set_fmt(ELOG_LVL_WARN, ELOG_FMT_LVL | ELOG_FMT_TAG);
elog_set_fmt(ELOG_LVL_INFO, ELOG_FMT_LVL | ELOG_FMT_TAG);
//除了时间、进程信息、线程信息之外，其余全部输出
elog_set_fmt(ELOG_LVL_DEBUG, ELOG_FMT_ALL & ~(ELOG_FMT_TIME | ELOG_FMT_P_INFO | ELOG_FMT_T_INFO));
//输出所有内容
elog_set_fmt(ELOG_LVL_VERBOSE, ELOG_FMT_ALL);

/* 启动elog */
elog_start();

/* USER CODE END 2 */
```
## 3.2. elog日志输出
elog中每种级别都有一种完整方式，两种简化方式，使用时自行选择：
```c
#define elog_assert(tag, ...) 
#define elog_a(tag, ...) //简化方式1，每次需填写 LOG_TAG
#define log_a(...)       //简化方式2，LOG_TAG 在文件顶部定义，使用前无需填写 LOG_TAG

#define elog_error(tag, ...)
#define elog_e(tag, ...)
#define log_e(...)

#define elog_warn(tag, ...)
#define elog_w(tag, ...)
#define log_w(...)

#define elog_info(tag, ...)
#define elog_i(tag, ...)
#define log_i(...)

#define elog_debug(tag, ...)
#define elog_d(tag, ...)
#define log_d(...)

#define elog_verbose(tag, ...)
#define elog_v(tag, ...)
#define log_v(...)
```
前两种在使用的时候只需要包含`<elog.h>`头文件即可，第三种方式除了包含头文件之外，还需要在文件开始定义TAG宏定义，使用起来和printf相同，所以这里我使用第三种方法演示。

首先在main.c文件开始定义TAG宏，包含头文件：
```c
/* USER CODE BEGIN Includes */
#define LOG_TAG    "main"

#include <elog.h>

/* USER CODE END Includes */
```
然后在main函数中编写的elog初始化代码之后，继续添加代码，测试elog的使用：
```c
log_a("Hello EasyLogger!");
log_e("Hello EasyLogger!");
log_w("Hello EasyLogger!");
log_i("Hello EasyLogger!");
log_d("Hello EasyLogger!");
log_v("Hello EasyLogger!");
```
编译，烧写，使用串口终端（Mobaxterm）查看串口输出：
![](https://img-blog.csdnimg.cn/20200410101431630.png)
## 3.3. 五彩缤纷的输出
要想五彩缤纷的日志，仅在`elog_cfg.h`中使能颜色输出还不够，还需要使用API开启输出：
```c
void elog_set_text_color_enabled(bool enabled);
```
在初始化elog的时候使能文字颜色输出：
![](https://img-blog.csdnimg.cn/20200410102118570.png)
再次编译、下载、查看输出：
![](https://img-blog.csdnimg.cn/20200410102216449.png)
每个级别日志的前景色、背景色、字体都可以在`elog_cfg.h`中修改宏定义，宏定义的值在`elog.c`中给出，可自行查看，比如这里我将ERROR级别的日志修改为闪烁字体：
![](https://img-blog.csdnimg.cn/20200410110004206.png)
编译、下载、查看输出：
![](https://img-blog.csdnimg.cn/20200410110119238.gif)
## 3.4. 移植前后内存占用情况
移植前的裸机工程只具有usart1收发功能，移植easylogger之后两者内存对比如下：
![](https://img-blog.csdnimg.cn/2020041014115440.png)
## 3.5. elog的高级功能
elog除了基本的日志功能之外，还提供了一些高级功能，比如：

- 日志输出过滤功能：可以按级别、TAG、关键词过滤日志；
- 缓冲输出模式；
- 异步输出模式；

这些功能如何使用，在项目的readme文档中讲述的很详细，本文限于篇幅，这些高级功能不详细讲述，如有兴趣深入，可以自行研究。

# 4. 设计思想解读
## 4.1. 数据加工
使用日志打印组件与使用printf最基本的区别在于：输出了更多有利于调试的信息，可以理解为对输出数据进行了一次加工。

打印语句所在文件、函数名、行号这些信息是利用了**编译器内置宏**的功能：

- `__FILE__`：文件名
- `__FUNCTION__`：函数名
- `__LINE__`：行号

而在终端中输出有颜色的字符则是利用了ANSI escape code，即Escape 序列屏幕控制码，关于这两个知识点详细的解释和示例请阅读：

- [编译器宏详解](https://mculover666.blog.csdn.net/article/details/104151957)
- [ANSI escape code详解](https://mculover666.blog.csdn.net/article/details/105433609)

在elog中对输出内容进行加工处理的函数为：
```c
/**
 * output the log
 *
 * @param level level
 * @param tag tag
 * @param file file name
 * @param func function name
 * @param line line number
 * @param format output format
 * @param ... args
 *
 */
void elog_output(uint8_t level, const char *tag, const char *file, const char *func,const long line, const char *format, ...) ;
```

## 4.2. 日志输出模式
俗话说，师傅领进门，修行在个人，本文所讲述的只是日志打印组件的基本功能，使用printf直接实现日志输出接口，所以在日志输出模式上和使用printf输出没有区别，只不过多了些信息。

elog支持**异步输出模式**，开启异步输出模式后，将会提升用户应用程序的执行效率。应用程序在进行日志输出时，无需等待日志彻底输出完成，即可**直接返回**。

elog也支持**缓冲输出模式**，开启缓冲输出模式后，如果缓冲区不满，用户线程在进行日志输出时，无需等待日志彻底输出完成，即可**直接返回**。但当日志缓冲区满以后，将会占用用户线程，自动将缓冲区中的日志全部输出干净。

这两种无需等待，直接返回的日志输出模式，在打印大量日志信息的时候非常重要，打印日志的代码对正常应用程序的影响越小越好，本文不再讲述，还请读者自行研究。

# 5. 项目工程源码获取和问题交流
目前我将Easylogger源码、我移植到小熊派STM32L431RCT6开发板的工程源码上传到了QQ群里（包含好几份HAL库，QQ相对速度快点），**可以在QQ群里下载，有问题也可以在群里交流，当然也欢迎大家分享出来自己移植的工程到QQ群里**：
![](https://img-blog.csdnimg.cn/20200411211314637.png)
放上QQ群二维码：
![](https://img-blog.csdnimg.cn/20200321144543508.png)

<font color="red">**接收更多精彩文章及资源推送，欢迎订阅我的微信公众号：『mculover666』。**</font>
![](https://img-blog.csdnimg.cn/20200202092055136.png)
