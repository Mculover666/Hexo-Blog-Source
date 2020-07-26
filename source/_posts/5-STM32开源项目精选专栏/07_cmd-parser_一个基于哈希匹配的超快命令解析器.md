---
title: cmd-parser | 一个基于哈希匹配的超快命令解析器
keywords: cmd-parser
tags: 嵌入式开源项目精选专栏
categories: OpenSource
abbrlink: 987422422
summary: 嵌入式开源项目精选专栏
date: 2020-05-18 08:00:56
---


![](https://img-blog.csdnimg.cn/20200321151234798.png)

# 1. cmd-parser
本期给大家带来的开源项目是 cmd-parser，**一款非常轻量级的命令解析器**，作者jiejie，目前收获 32 个 star，遵循 Apache-2.0 开源许可协议。

cmd-parser一个非常简单好用的命令解析器，占用资源极少极少，采用哈希算法超快匹配命令！

>项目地址：[https://github.com/jiejieTop/cmd-parser](https://github.com/jiejieTop/cmd-parser)

# 2. 移植cmd-parser
## 2.1. 移植思路
开源项目在移植过程中主要参考项目的readme文档，一般只需两步：

- ① 添加源码到裸机工程中；
- ② 实现需要的接口；

## 2.2. 准备裸机工程
本文中我使用的是小熊派IoT开发套件，主控芯片为STM32L431RCT6：
![](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2Jsb2cvMjAxOTA5MTkvekh4NXhNWVVtU1cxLnBuZw?x-oss-process=image/format,png#pic_center)

移植之前需要准备一份裸机工程，我使用STM32CubeMX生成，需要初始化以下配置：

- 配置一个串口用于中断方式接收数据，发送数据；
- printf重定向

具体过程可以参考：

- [STM32CubeMX_07 | 使用USART发送和接收数据（中断模式）](http://www.mculover666.cn/posts/1803605667/)
- [STM32CubeMX_09 | 重定向printf函数到串口输出的多种方法](http://www.mculover666.cn/posts/2251182441/)

# 2.3. 添加cmd-parser到工程中
① 复制cmd-parser源码到工程中：
![](https://img-blog.csdnimg.cn/20200513171828377.png)

② 在keil中添加 cmd-parser 的源码文件：
![](https://img-blog.csdnimg.cn/2020051317200363.png)

③ 将 cmd-parser 头文件路径添加到keil中：
![](https://img-blog.csdnimg.cn/20200513172042417.png)

# 3. 使用cmd-parser解析命令
使用时包含头文件：
```c
/* USER CODE BEGIN Includes */
#include "cmd.h"
#include <stdio.h>	//用于printf打印

/* USER CODE END Includes */
```
## 3.1. 初始化cmd-parser
在main.c中添加初始化代码，首先开辟一块接收缓冲区：
```c
/* USER CODE BEGIN PV */
char recv_buf[6] = {0};

/* USER CODE END PV */
```
然后初始化cmd-parser，并使能串口接收中断：
```c
/* USER CODE BEGIN 2 */
printf("cmd parser testing...\r\n");
cmd_init();
HAL_UART_Receive_IT(&huart1, (uint8_t*)recv_buf, 5);
/* USER CODE END 2 */
```
## 3.2. 注册命令
在main.c的开始定义两个函数，作为命令回调函数，使用`REGISTER_CMD`注册：
```c
/* USER CODE BEGIN 0 */

void led_on_cmd(void)
{
    printf("led on!\n");
}
void led_of_cmd(void)
{
    printf("led off!\n");
}

REGISTER_CMD(ledon, led_on_cmd);
REGISTER_CMD(ledof, led_of_cmd);

/* USER CODE END 0 */
```
使用REGISTER_CMD宏定义的时候，需要注意，第二个参数是函数指针，第一个参数是命令内容，**此处不需要双引号，cmd-parser会进行处理**。

## 3.3. 解析命令
在main.c的末尾编写串口中断回调函数，在串口中断回调函数中从接收缓冲区解析命令：
```c
/* USER CODE BEGIN 4 */
/* 中断回调函数 */
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
    /* 判断是哪个串口触发的中断 */
    if(huart ->Instance == USART1)
    {
		//解析数据
		cmd_parsing((char*)recv_buf);
		
        //重新使能串口接收中断
        HAL_UART_Receive_IT(huart, (uint8_t*)recv_buf, 5);
    }
}
/* USER CODE END 4 */
```
## 3.4. 解析结果
编译、下载到开发板，使用串口助手进行测试：
![](https://img-blog.csdnimg.cn/20200516103552164.png)

# 4. cmd-parser设计思想解读
cmd-parser组件的意义在于优化了字符串匹配算法。

在本文中的命令应用中，串口接收缓冲区的字符串是主字符串，而我们注册的命令是模式字符串，一般情况下，在主字符串中寻找模式字符串使用的是暴力算法，即直接从主字符串的第一个字符开始，双重循环判断字符是否匹配。

这种暴力算法可以解决大多数问题，但在一些特殊情况下，比如模式字符串是`ledon`，而主字符串是`ledoledoledoledoledon`，如果依然使用暴力算法，则算法时间复杂度为O(mn)，m为主串长度，n为模式串长度，极其浪费时间。

cmd-parser组件没有使用这种暴力匹配算法，而是直接匹配主字符串和模式字符串的哈希值（hashcode），将两个字符串的匹配**转换为两个整数比较**，非常高效，这种算法的发明人Rabin Karp，所以称之为RK算法。

接下来逐步解析cmd-parser是如何使用RK算法高效匹配的。

## 4.1. 生成模式串的hashcode
此部分源码在`cmd.c`中，如下：
```c
static unsigned int _cmd_hash(const char* str)
{
    unsigned int hash = CMD_HASH;  /* 'jiejie' string hash */  
    int c = *str;
    int tmp;
	
	//mculover666添加
	printf("str=[%s]\n", str);
    
    while(*str) {
        tmp = _cmd_to_lower(c);
        hash = ((hash << 5) + (hash ^ tmp) + tmp); 
        str++;
        c = *str;
    }
	
	//mculover666添加
	printf("hashcode = %d\n", hash);
	
    return hash;
}
```
此函数输入一个字符串，输出一个整型hashcode，生成hashcode的算法非常多，各有优缺点，此处匹配时初始hash值为字符串"jiejie"的hashcode，所以暂且称之为“杰算法”，不用关心具体算法实现。

在函数前后添加两行打印代码，编译，下载，在串口即可看到字符串生成的hashcode具体值：
![](https://img-blog.csdnimg.cn/20200516111246564.png)

emmmmm?使用杰算法生成的“ledon”和“ledof”的hash值竟然一样！

不用慌，这个在hashcode生成的时候是非常常见的事情，对于hashcode相同的字符串，只能老老实实的进行暴力算法匹配，没有骚操作了，源码如下：
```c
static int _cmd_match(const char *str, const char *cmd)
{
    int c1, c2;

    do {
        c1 = _cmd_to_lower(*str++);
        c2 = _cmd_to_lower(*cmd++);
    } while((c1 == c2) && c1);

    return c1 - c2;
}
```
## 4.2.  命令解析
源码在`cmd.c`中，先生成输入字符串的hashcode，如果两个字符串的hashcode相同，则进行逐个字符匹配，如下：
```c
void cmd_parsing(char *str)
{
    cmd_t *index;
    unsigned int hash = _cmd_hash(str);
	
	//mculover666添加
	printf("recv str is [%s]\n", str);
	printf("recv str hashcode = %d\n", hash);
    
    for (index = _cmd_begin; index < _cmd_end; index = _get_next_cmd(index)) {
        if (hash == index->hash) {
            if (_cmd_match(str, index->cmd) == 0) {
                index->handler();
                break;
            }
        }
    }
}
```
添加两行打印代码后，编译下载，在串口终端中查看结果：
![](https://img-blog.csdnimg.cn/20200516111958204.png)

以上就是关于使用RK算法超快匹配字符串的算法讲解，也是cmd-parser的设计灵魂所在，但是这种算法也有缺点：当hashcode冲突值较多时，就起不到优化作用了，和直接暴力匹配没有区别。

比如本实验中“ledon"和"ledof"这两个模式串的匹配，使用暴力算法匹配和使用RK算法匹配就没有区别，所以在实际应用中，还要根据自己的协议情况，自行选择最优算法解决！

# 5. 项目工程源码获取和问题交流
目前我将cmd-parser源码、我移植到小熊派STM32L431RCT6开发板的工程源码上传到了QQ群里（包含好几份HAL库，QQ相对速度快点），**可以在QQ群里下载，有问题也可以在群里交流，当然也欢迎大家分享出来自己移植的工程到QQ群里**：
![](https://img-blog.csdnimg.cn/20200516103153959.png)

放上QQ群二维码：
![](https://img-blog.csdnimg.cn/20200321144543508.png)

<font color="red">**接收更多精彩文章及资源推送，欢迎订阅我的微信公众号：『mculover666』。**</font>
![](https://img-blog.csdnimg.cn/20200202092055136.png)
