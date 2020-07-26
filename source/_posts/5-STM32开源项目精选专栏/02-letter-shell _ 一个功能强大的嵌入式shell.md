---
title: letter-shell | 一个功能强大的嵌入式shell
keywords: lettershell
tags: 嵌入式开源项目精选专栏
categories: OpenSource
abbrlink: 131757493
summary: 嵌入式开源项目精选专栏
date: 2020-04-09 08:00:56
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

如果您自己编写或者发现的开源项目不错，欢迎留言或者私信投稿到本专栏，分享获得双倍的快乐！

# 1. letter-shell
本期给大家带来的开源项目是 letter-shell，**一个功能强大的嵌入式shell**，作者NevermindZZT，目前收获 155 个star，遵循 MIT 开源许可协议。

letter shell 3.0是一个C语言编写的，可以嵌入在程序中的嵌入式shell，通俗一点说就是一个串口命令行，可以通过命令行调用、运行程序中的函数。

![](https://img-blog.csdnimg.cn/20200327145222418.png#pic_center)
目前 letter-shell 3.0版本支持的功能有：

- 命令自动补全
- 快捷键功能定义
- 命令权限管理
- 用户管理
- 变量支持

>项目地址：https://github.com/NevermindZZT/letter-shell

# 2. 移植letter-shell
## 2.1. 移植思路
① 看项目readme文件中的移植说明，一般都比较完善；
② 看项目中的demo，举一反三；
③ 看别人移植好的博客；
## 2.2. 移植过程
letter-shell的移植非常简单，**自己实现串口读写一个字符的接口，自己编写一个初始化函数**，完成。

本文中我使用的是小熊派IoT开发套件，主控芯片为STM32L431RCT6：
![](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2Jsb2cvMjAxOTA5MTkvekh4NXhNWVVtU1cxLnBuZw?x-oss-process=image/format,png#pic_center)
移植之前需要准备一份裸机工程，我使用STM32CubeMX生成，使用USART1的查询方式发送数据、使用USART1的中断方式接收数据，参考教程：

- [STM32CubeMX_06 | 使用USART发送和接收数据（查询模式）](http://www.mculover666.cn/posts/2064921339/)
- [STM32CubeMX_07 | 使用USART发送和接收数据（中断模式）](http://www.mculover666.cn/posts/1803605667/)

① 复制源码到工程中：
![](https://img-blog.csdnimg.cn/20200327152927521.png)
② 新建存放实现移植接口的文件：
![](https://img-blog.csdnimg.cn/20200327153402563.png)
③ 在keil中添加文件，添加头文件路径：
![](https://img-blog.csdnimg.cn/20200327154614874.png)
目录下还有头文件，添加到头文件路径中：
![](https://img-blog.csdnimg.cn/20200327154705670.png)
④ 编辑shell_port.c文件，**实现向串口写入一个字符的接口**，并编写shell的初始化函数：
```c
/**
 * @brief	shell移植到STM32L431时的接口实现
 * @author	mculover666
 * @date	2020/03/27 
*/

#include "shell.h"
#include <stm32l4xx_hal.h>
#include "usart.h"
#include "shell_port.h"

/* 1. 创建shell对象，开辟shell缓冲区 */
Shell shell;
char shell_buffer[512];


/* 2. 自己实现shell写函数 */

//shell写函数原型：typedef void (*shellWrite)(const char);
void User_Shell_Write(const char ch)
{
	//调用STM32 HAL库 API 使用查询方式发送
	HAL_UART_Transmit(&huart1, (uint8_t*)&ch, 1, 0xFFFF);
}

/* 3. 编写初始化函数 */
void User_Shell_Init(void)
{
	//注册自己实现的写函数
    shell.write = User_Shell_Write;
	
	//调用shell初始化函数
    shellInit(&shell, shell_buffer, 512);
}
```
⑤ 编辑shell_port.h文件，声明自己编写的初始化函数：
```c
#ifndef _SHELL_PORT_H_
#define	_SHELL_PORT_H_

#include "shell.h"

/* 将shell定义为外部变量，在串口中断回调函数中还要使用 */
extern Shell shell;

/* 声明自己编写的初始化函数 */
void User_Shell_Init(void);

#endif /* _SHELL_PORT_H_ */
```
⑥ 在`main.c`文件的末尾编写串口中断回调函数，在接收到一个字符之后调用 shellHandler 函数进行处理，首先把头文件包含进来：
```c
/* USER CODE BEGIN Includes */
#include "shell_port.h"
/* USER CODE END Includes */
```

接着开辟一个串口接收缓冲区（一个字符）：
```c
/* USER CODE BEGIN PV */
uint8_t recv_buf = 0;
/* USER CODE END PV */
```
然后编写回调函数
```c
/* USER CODE BEGIN 4 */
/* 中断回调函数 */
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
    /* 判断是哪个串口触发的中断 */
    if(huart ->Instance == USART1)
    {
        //调用shell处理数据的接口
			  shellHandler(&shell, recv_buf);
        //使能串口中断接收
			  HAL_UART_Receive_IT(&huart1, (uint8_t*)&recv_buf, 1);
    }
}
/* USER CODE END 4 */
```
⑥ 在`main函数`中使能中断接收，调用自己编写的shell初始化函数：
```c
/* USER CODE BEGIN 2 */
//使能串口中断接收
HAL_UART_Receive_IT(&huart1, (uint8_t*)&recv_buf, 1);
User_Shell_Init();
/* USER CODE END 2 */
```
至此，移植完成，编译、下载之后使用串口终端软件Mobaxterm即可看到效果：
![](https://img-blog.csdnimg.cn/20200327170352712.png)
## 2.3. 宏定义配置
letter-shell具备很多功能，可以通过宏定义来开启或者关闭，在`shell_cfg.h`文件中根据需要进行配置：
![](https://img-blog.csdnimg.cn/20200327171321707.png)

在本次移植过程中，我将shell默认用户改为了mculover666，其它宏保持默认：
```c
#define     SHELL_DEFAULT_USER          "mculover666"
```
# 3. 使用letter-shell
## 3.1. 应用场景
在嵌入式项目中，做出一个项目之后还需要用户在串口终端中进行操作，这样的情况很少，串口的作用基本都是用来在调试阶段打印信息。

但是在调试的模组比较复杂时，比如SPI Flash、LCD屏幕这些，我希望可以在串口直接调用某几个功能函数开始执行，当移植了shell之后，**在代码中只需要添加一行宏定义，就可以在串口中调用此函数开始执行**，多爽！

当然也可以在main函数中调用，然后打断点进入调试，根据个人喜好选择就行。

## 3.2. 可执行命令定义宏
这个宏可以实现将函数添加到shell的可执行命令列表中，使用时需要确保shell_cfg.h中的宏定义要开启：
```c
#define     SHELL_USING_CMD_EXPORT      1
```
该宏定义的定义如下：
```c
/**
 * @brief shell 命令定义
 *
 * @param _attr 命令属性
 * @param _name 命令名
 * @param _func 命令函数
 * @param _desc 命令描述
 */
#define SHELL_EXPORT_CMD(_attr, _name, _func, _desc) \
        const char shellCmd##_name[] = #_name; \
        const char shellDesc##_name[] = #_desc; \
        const ShellCommand \
        shellCommand##_name SECTION("shellCommand") =  \
        { \
            .attr.value = _attr, \
            .data.cmd.name = shellCmd##_name, \
            .data.cmd.function = (int (*)())_func, \
            .data.cmd.desc = shellDesc##_name \
        }
```
第一个参数attr表示该命令的属性，包括命令权限和命令类型等，但是对于目前这种应用场合下多用户没什么用，所以设置为下面的值就ok：
```c
SHELL_CMD_PERMISSION(0)|SHELL_CMD_TYPE(SHELL_TYPE_CMD_FUNC)
```
最后需要注意一点，这个宏目前测试**在MDK中可以正常使用**，在IAR和GCC中请移步项目地址阅读文档。
## 3.3. 调用函数示例
在main.c文件中随便定义一个函数，然后使用宏定义导出到命令列表进行测试：
```c
/* USER CODE BEGIN 0 */
int test(int i, char ch, char *str)
{
    printf("input int: %d, char: %c, string: %s\r\n", i, ch, str);
	
	return 0;
}

//导出到命令列表里
SHELL_EXPORT_CMD(SHELL_CMD_PERMISSION(0)|SHELL_CMD_TYPE(SHELL_TYPE_CMD_FUNC), test, test, test);

/* USER CODE END 0 */
```
![](https://img-blog.csdnimg.cn/20200327175605777.png#pic_center)
有两点需要注意：

① **目前支持的参数类型为整型、字符型、字符串型，不支持浮点型参数**。

② **函数最大传入参数个数由shell_cfg.h中的宏定义配置**：
```c

#define     SHELL_PARAMETER_MAX_NUMBER  8
```
所以在使用时导出到命令列表中的参数最大是 `8 - 1 = 7`个，测试如下：
```c
int test2(int i1, int i2, int i3, int i4, int i5, int i6, int i7, int i8)
{
	printf("input int: %d, %d, %d, %d, %d, %d, %d, %d\r\n", i1, i2, i3, i4, i5, i6, i7, i8);
	return 0;
}
//导出到命令列表里
SHELL_EXPORT_CMD(SHELL_CMD_PERMISSION(0)|SHELL_CMD_TYPE(SHELL_TYPE_CMD_FUNC), test2, test2, test2);
```
执行之后失败：
![](https://img-blog.csdnimg.cn/20200327180426899.png#pic_center)
# 4. letter-shell设计思想解读
## 4.1. 终端软件交互原理
当我们使用诸如 mobaxterm 这类软件作为串口终端交互时，首先需要注意三点：

① 当在终端软件中输入一个字符时，这个字符会被直接通过串口发送，屏幕上不会显示任何东西；
② 如果单片机做了回显，终端软件中收到后才会显示出来；
③ 当在终端软件中按下回车键、退格键、删除键、四个方向键时，会直接发送其**键值**，单片机中靠这个键值来解析是否按下了按键。

在lettershell中输入`keys`命令即可查看当前支持的按键键值解析：
![](https://img-blog.csdnimg.cn/20200403093922523.png)
解析这些键值的源码在`shell.c`中，分别对应以下的函数：
```c
void shellUp(Shell *shell);
……
void shellTab(Shell *shell);
void shellBackspace(Shell *shell);
void shellEnter(Shell *shell);
……
```
除此之外，lettershell还支持**设置快捷键**来快速调用函数执行，比如这里我设置在终端中按下`Ctrl+A`之后调用hello函数，过程如下：

① 首先在串口中断中添加一行代码，查看按下`Ctrl+A`快捷键之后串口发送的键值：
![](https://img-blog.csdnimg.cn/20200403095335802.png)
编译下载，在终端软件中按下`Ctrl+A`，得到的键值如下：
![](https://img-blog.csdnimg.cn/20200403095452396.png)
此处需要注意，终端发送的字符串序列以**大端模式**表示，所以单片机解析的键值应该为`0x01000000`。

② 一行代码搞定键值解析，在main文件中添加测试函数hello，并使用宏定义向lettershell中添加按键解析值：
```c
int hello()
{
	printf("Hello World\r\n");
	return 0;
}
SHELL_EXPORT_KEY(SHELL_CMD_PERMISSION(0), 0x01000000, hello, hello);
```
将之前串口接收中断函数中添加的printt注释，编译，下载：
![](https://img-blog.csdnimg.cn/20200403100027600.png)
## 4.2. 如何解析命令
在letter shell中，当它接收到一个字符后，首先判断是不是特殊的按键，否则直接扔进命令解析缓冲区，因为它对左右方向键、退格键处理的比较好，相当于不停的在维护命令解析缓冲区的内容，非常整齐，解析时就变得相对容易了。

当用户按下回车键时，首先去命令列表中匹配：
```c
/**
 * @brief shell匹配命令
 * 
 * @param shell shell对象
 * @param cmd 命令
 * @param base 匹配命令表基址
 * @param compareLength 匹配字符串长度
 * @return ShellCommand* 匹配到的命令
 */
ShellCommand* shellSeekCommand(Shell *shell,
                               const char *cmd,
                               ShellCommand *base,
                               unsigned short compareLength)
```
如果匹配到之后则开始执行这条命令：
```c
/**
 * @brief shell运行命令
 * 
 * @param shell shell对象
 * @param command 命令
 */
static void shellRunCommand(Shell *shell, ShellCommand *command)
```

限于文章篇幅，具体解析的过程和方法不再深入讲述，如有兴趣可以自行研究学习，另外，=除了本文所讲述的自定义函数调用，自定义快捷键设置，letter shell还有非常多的功能，比如自定义变量查看，用户登录，用户权限，自动锁定shell等高级内容，也欢迎有兴趣的读者探索，研究！

# 5. 项目工程源码获取和问题交流
目前我将LetterShell源码、我移植到小熊派STM32L431RCT6开发板的工程、移植到STM32Nucleo-STM32G071RB开发板的工程源码上传到了QQ群里（包含好几份HAL库，QQ相对速度快点），**可以在QQ群里下载，有问题也可以在群里交流，当然也欢迎大家分享出来自己移植的工程到QQ群里**：
![](https://img-blog.csdnimg.cn/20200321144423738.png)
放上QQ群二维码：
![](https://img-blog.csdnimg.cn/20200321144543508.png)

<font color="red">**接收更多精彩文章及资源推送，欢迎订阅我的微信公众号：『mculover666』。**</font>
![](https://img-blog.csdnimg.cn/20200202092055136.png)
