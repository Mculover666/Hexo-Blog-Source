---
title: MultiButton | 一个小巧简单易用的事件驱动型按键驱动模块
keywords: MultiButton
tags: 嵌入式开源项目精选专栏
categories: OpenSource
abbrlink: 2795548875
summary: 嵌入式开源项目精选专栏
date: 2020-04-08 08:00:56
---

![](https://img-blog.csdnimg.cn/20200321151234798.png)

# 嵌入式开源项目精选专栏
本专栏由Mculover666创建，主要内容为寻找嵌入式领域内的优质开源项目，一是帮助开发者使用开源项目实现更多的功能，二是通过这些开源项目，学习大佬的代码及背后的实现思想，提升自己的代码水平，和其它专栏相比，本专栏的优势在于：

**不会单纯的介绍分享项目，还会包含作者亲自实践的过程分享，甚至还会有对它背后的设计思想解读**。

目前本专栏包含的开源项目有：

- [SFUD | 一个简洁实用的开源项目，帮你轻松搞定SPI Flash](https://blog.csdn.net/Mculover666/article/details/101351255)
- [cJSON | 一个轻量级C语言JSON解析器](https://blog.csdn.net/Mculover666/article/details/103796256)
- [paho | 支持10种语言编写mqtt客户端，总有一款适合你！](https://blog.csdn.net/Mculover666/article/details/103935428)

如果您自己编写或者发现的开源项目不错，欢迎留言或者私信投稿到本专栏，分享获得双倍的快乐！

# 1. MultiButton
本期给大家带来的开源项目是 MultiButton，**一个小巧简单易用的事件驱动型按键驱动模块**，作者 0x1abin，目前收获 222 个star，遵循 MIT 开源许可。

这个项目非常精简，只有两个文件，可无限量扩展按键，按键事件的回调异步处理方式可以简化程序结构，去除冗余的按键处理硬编码，让你的按键业务逻辑更清晰。
![](https://img-blog.csdnimg.cn/20200320163601795.png)
MuliButton 支持如下的按钮事件：
|事件|说明|
|:---:|:---:|
|PRESS_DOWN	|按键按下，每次按下都触发|
|PRESS_UP	|按键弹起，每次松开都触发|
|PRESS_REPEAT|	重复按下触发，变量repeat计数连击次数|
|SINGLE_CLICK	|单击按键事件|
|DOUBLE_CLICK|	双击按键事件|
|LONG_RRESS_START|	达到长按时间阈值时触发一次|
|LONG_PRESS_HOLD|	长按期间一直触发|

>GIthub地址：[https://github.com/0x1abin/MultiButton](https://github.com/0x1abin/MultiButton)

# 2. 使用MultiButton
## 2.1. 准备一份裸机工程
需要掌握使用HAL库读取GPIO输入的函数、串口的使用、printf重定向、以及systick的使用：

- [STM32CubeMX | 04-使用GPIO进行按键检测](https://blog.csdn.net/Mculover666/article/details/95907760)
- [STM32CubeMX | 06-使用USART发送和接收数据（查询模式）](https://blog.csdn.net/Mculover666/article/details/95941795)
- [STM32CubeMX | 09-重定向printf函数到串口输出的多种方法](https://blog.csdn.net/Mculover666/article/details/99842909)
- []()

本文中我使用小熊派IoT开发板，主控为STM32L431RCT6：
![](https://img-blog.csdnimg.cn/20200320201839463.png)
配置外部时钟：
![](https://img-blog.csdnimg.cn/20200320202248791.png)
按键GPIO配置：
![](https://img-blog.csdnimg.cn/20200320202336887.png)
打印串口配置：
![](https://img-blog.csdnimg.cn/20200320202436671.png)
时钟配置：
![](https://img-blog.csdnimg.cn/20200320202521420.png)
配置工程，生成代码，重定向printf，**printf可以正常打印后进行下面的步骤**。
## 2.2. 移植MultiButton
① 复制MultiButton源码到裸机工程中：
![](https://img-blog.csdnimg.cn/20200320202819445.png)
② 添加MultiButton源码到项目中：
![](https://img-blog.csdnimg.cn/20200320203315805.png)
![](https://img-blog.csdnimg.cn/20200320203343127.png)
此时编译没有问题。

## 2.3. 编写MultiButton应用代码
在main.c文件中编写以下代码。

① **包含头文件**
```c
/* USER CODE BEGIN Includes */

#include <stdio.h>		//要使用printf
#include "multi_button.h"

/* USER CODE END Includes */
```
② **定义一个按键结构（按键对象）**
```c
/* USER CODE BEGIN PV */

//申请一个按键结构
struct Button button1;

/* USER CODE END PV */
```
③ **初始化按键对象**

初始化按键对象使用的API为：
![](https://img-blog.csdnimg.cn/20200320204526906.png)
- 第一个参数为刚刚创建的按键对象的指针；
- 第二个参数为绑定按键的GPIO电平读取接口；
- 第三个参数为设置有效触发电平；

首先在main函数之前实现一个GPIO电平读取接口：
```c
/* USER CODE BEGIN 0 */

//按键状态读取接口
uint8_t read_button1_GPIO() 
{
	return HAL_GPIO_ReadPin(KEY1_GPIO_Port, KEY1_Pin);
}

/* USER CODE END 0 */
```

初始化按键对象的代码在main函数中，while(1)之前编写，如下：
```c
/* USER CODE BEGIN 2 */
printf("MultiButton Test...\r\n");

//初始化按键对象
button_init(&button1, read_button1_GPIO, 0);

/* USER CODE END 2 */
```

④ 注册按键事件

注册按钮事件的API如下：
![](https://img-blog.csdnimg.cn/20200320205411492.png)
- 第一个参数为按钮对象指针；
- 第二个参数为MultiButton支持的按钮事件；
- 第三个参数为要注册的该事件回调函数；

MultiButton支持的按钮事件枚举如下：
![](https://img-blog.csdnimg.cn/20200320205957544.png)
首先**在main函数之前**定义这两个事件的回调函数，回调函数有两种写法。

第一种**适合于按键事件较少的情况**：
```c
//按键1按下事件回调函数
void btn1_press_down_Handler(void* btn)
{
	printf("---> key1 press down! <---\r\n");
}

//按键1松开事件回调函数
void btn1_press_up_Handler(void* btn)
{
	printf("***> key1 press up! <***\r\n");
}
```
在main函数中，while(1)之前注册这两个回调函数：
```c
//注册按钮事件回调函数
button_attach(&button1, PRESS_DOWN, btn1_press_down_Handler);
button_attach(&button1, PRESS_UP, btn1_press_up_Handler);
```


第二种**适合于按键事件较多的情况**，如果每个按键都要写 7 个回调函数，那么代码量会非常的大，所以可以将这 7 个回调函数写在一起，一次性全部注册，回调函数如下：
```c
void button_callback(void *button)
{
    uint32_t btn_event_val; 
    
    btn_event_val = get_button_event((struct Button *)button); 
    
    switch(btn_event_val)
    {
	    case PRESS_DOWN:
	        printf("---> key1 press down! <---\r\n"); 
	    	break; 
	
	    case PRESS_UP: 
	        printf("***> key1 press up! <***\r\n");
	    	break; 
	
	    case PRESS_REPEAT: 
	        printf("---> key1 press repeat! <---\r\n");
	    	break; 
	
	    case SINGLE_CLICK: 
	        printf("---> key1 single click! <---\r\n");
	    	break; 
	
	    case DOUBLE_CLICK: 
	        printf("***> key1 double click! <***\r\n");
	    	break; 
	
	    case LONG_RRESS_START: 
	        printf("---> key1 long press start! <---\r\n");
	   		break; 
	
	    case LONG_PRESS_HOLD: 
	        printf("***> key1 long press hold! <***\r\n");
	    	break; 
	}
}
```
使用这种回调函数的时候需要在MultiButton的源码中添加一行代码：
![](https://img-blog.csdnimg.cn/20200321132828355.png)
注册回调函数的代码如下：
```c
//注册按钮事件回调函数

button_attach(&button1, PRESS_DOWN,       button_callback);
button_attach(&button1, PRESS_UP,         button_callback);
//button_attach(&button1, PRESS_REPEAT,     button_callback);
//button_attach(&button1, SINGLE_CLICK,     button_callback);
//button_attach(&button1, DOUBLE_CLICK,     button_callback);
//button_attach(&button1, LONG_RRESS_START, button_callback);
//button_attach(&button1, LONG_PRESS_HOLD,  button_callback);
```


⑤ 启动按键
启动按键的API如下：
![](https://img-blog.csdnimg.cn/20200320210755871.png)
接着在main函数中，while(1)之前编写代码，启动按键：
```c
//启动按键
button_start(&button1);
```

⑥ 设置一个5ms间隔的定时器循环调用后台处理函数

这里就要用到systick了，在main函数的while(1)循环中编写如下代码：
```c
  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
		//每隔5ms调用一次后台处理函数
		button_ticks();
		HAL_Delay(5);
  }
  /* USER CODE END 3 */
```
## 2.4. 实验现象
编译、下载之后，每次按下Key1时打印按下提示，松开Key1时打印松开提示：
![](https://img-blog.csdnimg.cn/20200320211941501.png)

## 2.5. 扩展实验
在注册回调函数时将这按下和松开屏蔽，将单击和双击打开进行测试：
```c
//注册按钮事件回调函数
//button_attach(&button1, PRESS_DOWN,       button_callback);
//button_attach(&button1, PRESS_UP,         button_callback);
//button_attach(&button1, PRESS_REPEAT,     button_callback);
button_attach(&button1, SINGLE_CLICK,     button_callback);
button_attach(&button1, DOUBLE_CLICK,     button_callback);
//button_attach(&button1, LONG_RRESS_START, button_callback);
//button_attach(&button1, LONG_PRESS_HOLD,  button_callback);
```
![](https://img-blog.csdnimg.cn/20200321133109749.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01jdWxvdmVyNjY2,size_16,color_FFFFFF,t_70)
再测试长按：
```c
	//注册按钮事件回调函数
	//button_attach(&button1, PRESS_DOWN,       button_callback);
	//button_attach(&button1, PRESS_UP,         button_callback);
	//button_attach(&button1, PRESS_REPEAT,     button_callback);
	//button_attach(&button1, SINGLE_CLICK,     button_callback);
	//button_attach(&button1, DOUBLE_CLICK,     button_callback);
	button_attach(&button1, LONG_RRESS_START, button_callback);
	button_attach(&button1, LONG_PRESS_HOLD,  button_callback);
```
![](https://img-blog.csdnimg.cn/20200321133329271.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01jdWxvdmVyNjY2,size_16,color_FFFFFF,t_70)


# 3. MultiButton设计思想解读
## 3.1. 面向对象思想
MultiButton中每个按键都抽象为了一个**按键对象**，每个按键对象是独立的，系统中所有的按键对象使用**单链表**串起来，结构如下：
![](https://img-blog.csdnimg.cn/20200321112311826.png)
其中在变量后面跟冒号的语法称为**位域**，使用位域的优势是**节省内存**。

比如在这个结构体中，本来 6 个uint8_t 类型的变量需要占用 6 个字节，但使用位域语法后，这6个变量**只占用两个字节**：
![](https://img-blog.csdnimg.cn/2020032111334119.png)

## 3.2. 按键对象单链表
MultiButton自己定义了一个**头指针**：
```c
//button handle list head.
static struct Button* head_handle = NULL;
```
用户插入一个按键对象的代码如下：
```c
//启动按键
button_start(&button1);
```
那么，button_start插入新的按键对象之后，单链表长啥样呢？

理解了 button_start 的源码就很好知道答案了：
![](https://img-blog.csdnimg.cn/20200321120539957.png)
第一次插入时，因为head_hanler 为 NULL，所以只需要执行while之后的代码，
![](https://img-blog.csdnimg.cn/20200321121556782.png)
按照它的插入于原理，如果再插入一个buuton2按键对象，结果是不是可以猜出来了呢？

没错，它长这样：
![](https://img-blog.csdnimg.cn/20200321122110969.png)
这样做是不是有点不符合常理？后插入Button2竟然在button1前面，凭什么？

这又不是排队抢鸡蛋，在前在后没什么关系的。只是这样的插入方法**在代码算法上会非常简洁**，两行代码完成插入。

## 3.3. 状态机处理思想
MultiButton中使用状态机来处理每个按键对象（的状态），比如在上述应用中根据Systick提供的时基信号，每隔5ms调用一次 `button_tick()`，该函数会依次调用状态机对单链表上的所有按键对象进行遍历处理：
![](https://img-blog.csdnimg.cn/20200321114739484.png)
根据上一节的单链表讲解，**系统中定义的链表头指针 head_handle 永远指向最后一个插入的按键对象，所以无需任何参数即可遍历整个单链表上的对象**，非常之牛逼。

使用 button_handler 来对按键对象的状态进行处理，该函数源码如下：

（读源码的时候只需要记住该函数每隔5ms进入一次就很好分析了）

① **读取当前引脚状态**

调用该按键对象注册的读取状态函数进行读取：
![](https://img-blog.csdnimg.cn/20200321133718186.png)
② 读取之后，判断当前状态机的状态，如果有功能正在执行（state不为0），则按键对象的tick值加1（**后续一切功能的基础**）：
![](https://img-blog.csdnimg.cn/2020032113383453.png)
③ **按键消抖**（连续读取3次，15ms，如果引脚状态一直与之前不同，则改变按键对象中的引脚状态）：
![](https://img-blog.csdnimg.cn/20200321134036739.png)
④ **状态机**（整个设计的灵魂所在）

![](https://img-blog.csdnimg.cn/20200321140602485.png)

# 4. 项目工程源码获取和问题交流
目前我将MultiButton源码、我移植到小熊派STM32L431RCT6开发板的工程、移植到STM32Nucleo-STM32G071RB开发板的工程源码上传到了QQ群里（包含好几份HAL库，QQ相对速度快点），**可以在QQ群里下载，有问题也可以在群里交流，当然也欢迎大家分享出来自己移植的工程到QQ群里**：
![](https://img-blog.csdnimg.cn/20200321144423738.png)
放上QQ群二维码：
![](https://img-blog.csdnimg.cn/20200321144543508.png)


<font color="red">**接收更多精彩文章及资源推送，欢迎订阅我的微信公众号：『mculover666』。**</font>
![](https://img-blog.csdnimg.cn/20200202092055136.png)

