---
title: MultiTimer | 一款可无限扩展的软件定时器
keywords: MultiTimer
tags: 嵌入式开源项目精选专栏
categories: OpenSource
abbrlink: 876450753
summary: 嵌入式开源项目精选专栏
date: 2020-05-05 18:00:56
---

![](https://img-blog.csdnimg.cn/20200321151234798.png)

# 1. MultiTimer
本期给大家带来的开源项目是 MultiTimer，**一款可无限扩展的软件定时器**，作者0x1abin，目前收获 95 个 star，遵循 MIT 开源许可协议。

MultiTimer 是一个软件定时器扩展模块，可无限扩展你所需的定时器任务，取代传统的标志位判断方式， 更优雅更便捷地管理程序的时间触发时序。

>项目地址：[https://github.com/0x1abin/MultiTimer](https://github.com/0x1abin/MultiTimer)

# 2. 移植MultiTimer
## 2.1. 移植思路
开源项目在移植过程中主要参考项目的readme文档，一般只需两步：

- ① 添加源码到裸机工程中；
- ② 实现需要的接口；

## 2.2. 准备裸机工程
本文中我使用的是小熊派IoT开发套件，主控芯片为STM32L431RCT6：
![](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2Jsb2cvMjAxOTA5MTkvekh4NXhNWVVtU1cxLnBuZw?x-oss-process=image/format,png#pic_center)

移植之前需要准备一份裸机工程，我使用STM32CubeMX生成，需要初始化以下配置：

- 配置一个串口用于打印信息
- printf重定向

# 2.3. 添加MultiTimer到工程中
① 复制MultiTimer源码到工程中：
![](https://img-blog.csdnimg.cn/20200430110841828.png)

② 在keil中添加 MultiTimer的源码文件：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200430111007848.png)

③ 将MultiTimer头文件路径添加到keil中：
![](https://img-blog.csdnimg.cn/20200430111059907.png)
# 3. 使用MultiTimer
使用时包含头文件：
```c
#include "multi_timer.h"
```
>如果遇到multi_timer.c文件中NULL宏定义报错，则在multi_timer.h中添加<stddef.h>头文件即可。
## 3.1. 创建Timer对象
```c
/* USER CODE BEGIN PV */
struct Timer timer1;
struct Timer timer2;

/* USER CODE END PV */
```

## 3.2. Timer回调函数
```c
/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */
void timer1_callback()
{
    printf("timer1 timeout!\r\n");
}

void timer2_callback()
{
    printf("timer2 timeout!\r\n");
}
/* USER CODE END 0 */
```

## 3.3. 初始化并启动Timer
始化定时器对象，注册定时器回调处理函数，设置定时时间（ms），循环定时触发时间：
```c
/* USER CODE BEGIN 2 */
printf("multi timer test...\r\n");

//重复计时，周期为1000次，即1000ms=1s
timer_init(&timer1, timer1_callback, 1000, 1000);
timer_start(&timer1);

//单次计时，周期为50次，即50ms
timer_init(&timer2, timer2_callback, 50, 0);
timer_start(&timer2);

/* USER CODE END 2 */
```

## 3.4. Timer对象处理
在循环中调用Timer对象处理函数，处理函数会判断链表上的每个定时器是否超时，如果超过，则拉起注册的回调函数：
```c
/* Infinite loop */
/* USER CODE BEGIN WHILE */
while (1)
{
  /* USER CODE END WHILE */

  /* USER CODE BEGIN 3 */
  timer_loop();
}
 /* USER CODE END 3 */
```

## 3.5. 提供Timer时基信号
MultiTimer中所有的定时器都是通过一个32位的计数值`_timer_ticks`来判断的，所以需要一个硬件定时器提供时基信号，递增该值。

本文中使用的是STM32HAL库，所以通过Systick来提供，无需设置额外的定时器。

在`main.c`文件的最后编写Systick回调函数：
```c
/* USER CODE BEGIN 4 */
void HAL_SYSTICK_Callback(void)
{
	//给multi timer提供时基信号
    timer_ticks(); //1ms ticks
}

/* USER CODE END 4 */
```
然后在`stm32l4xx_it.c`中调用该回调函数：
```c
/**
  * @brief This function handles System tick timer.
  */
void SysTick_Handler(void)
{
  /* USER CODE BEGIN SysTick_IRQn 0 */
  HAL_SYSTICK_IRQHandler();

  /* USER CODE END SysTick_IRQn 0 */
  HAL_IncTick();
  /* USER CODE BEGIN SysTick_IRQn 1 */

  /* USER CODE END SysTick_IRQn 1 */
}
```
接下来编译下载，看在串口助手中看到打印的日志：
![](https://img-blog.csdnimg.cn/2020043017040247.png)

# 4. MultiTimer设计思想解读
## 4.1. 软件定时器设计思想
MultiTimer的设计比较简洁。

设置一个计数值`_timer_ticks`不断递增，由定时器提供的中断驱动，只计次数，不计时间，有了很大的自由度，一般时基信号设置为1ms一次：
```c
/**
  * @brief  background ticks, timer repeat invoking interval 1ms.
  * @param  None.
  * @retval None.
  */
void timer_ticks()
{
	_timer_ticks++;
}
```

在程序运行时循环比较定时器设置的超时值是否大于当前_timer_ticks的计数值，如果是则再次判断是否重复计数值是否为0，是则停止定时器，完成单次计时效果，否则修改计数值，最后拉起注册到该定时器的回调函数执行：
```c
/**
  * @brief  main loop.
  * @param  None.
  * @retval None
  */
void timer_loop()
{
	struct Timer* target;
	for(target=head_handle; target; target=target->next) {
		if(_timer_ticks >= target->timeout) {
			if(target->repeat == 0) {
				timer_stop(target);
			} else {
				target->timeout = _timer_ticks + target->repeat;
			}
			target->timeout_cb();
		}
	}
}
```
## 4.2. 单链表操作
MultiTimer的代码少，非常适合拿来学习单链表的操作，学习数据结构的过程是乏味的，不如直接来个实例看看是如何操作的。

① 链表的**节点设计为一个软件定时器**，所以理论上支持的定时器数量只受内存限制。

```c
typedef struct Timer {
    uint32_t timeout;
    uint32_t repeat;
    void (*timeout_cb)(void);
    struct Timer* next;
}Timer;
```
定时器初始化函数`timer_init`就是初始化一个链表节点：
```c
void timer_init(struct Timer* handle, void(*timeout_cb)(), uint32_t timeout, uint32_t repeat)
{
	// memset(handle, sizeof(struct Timer), 0);
	handle->timeout_cb = timeout_cb;
	handle->timeout = _timer_ticks + timeout;
	handle->repeat = repeat;
}
```
② 设置**链表头指针**，只需知道头指针就能完成对整个单链表的操作：
```c
//timer handle list head.
static struct Timer* head_handle = NULL;
```
③ 向单链表**增加一个节点**

向单链表增加一个节点有三种方式：

- 在单链表尾部增加一个节点
- 在单链表头部增加一个节点
- 在单链表中间增加一个节点

MultiTimer中所有的结点都是定时器，每个定时器之间相互独立，不存在先后次序关系，所以无论加到中间，还是加到尾部，还是加到头部，最后的功能都是一样的，但是在插入算法上有优劣性能之分。


先来看看再单链表尾部增加一个节点的算法：
![](https://img-blog.csdnimg.cn/20200430211524417.gif)
```c
int timer_start(struct Timer* handle)
{
	/** 
	 * 算法1 —— 向单链表尾部添加节点
	 * 时间复杂度O(n)
	 * Mculover666
	 */
	struct Timer* target = head_handle;
	if(head_handle == NULL)
	{
		/* 链表为空 */
		head_handle = handle;
		handle->next = NULL;
	}
	else
	{
		/* 链表中存在节点，遍历找最后一个节点 */
		while(target->next != NULL)
		{
			if(target == handle)
				return -1;
			target = target->next;
		}
		target->next = handle;
		handle->next = NULL;
	}
	
	return 0;
}
```
这种算法理解简单，实现简单，但是算法**时间复杂度秒变为O(n)**，当n很大时，插入一个节点的时间就会非常久。

再来看看在链表头部插入一个新节点的情况：
![](https://img-blog.csdnimg.cn/20200430215442603.gif)
```c
int timer_start(struct Timer* handle)
{
	/** 
	 * 算法2 —— 向单链表头部添加节点
	 * 时间复杂度O(n)，如果去掉判断重复，则时间复杂度O(1)
	 * 0x1abin
	 */
	 struct Timer *target = head_handle;
	 
	 //判断是否有重复的定时器
	 while(target)
	 {
		if(target == handle)
		{
			return -1;
		}
		target = target->next;
	 }
	 handle->next = head_handle;
	 head_handle = handle;
	 return 0;
}
```
这里第二种头部插入节点的算法时间复杂度依然是O(n)，emmm?

其实，这里因为单链表节点是定时器，**在插入的时候需要对整个链表进行判断，避免重复添加同样的定时器节点**，所以无论任何一种算法，都需要对单链表进行遍历。

如果在不需要判断重复的情况下，尾部插入算法仍然需要遍历，但是**头部插入算法只需要插入就可以，时间复杂度为O(1)，算法更优**。

④ 单链表**删除其中一个节点**

删除单链表的节点时，因为节点自身只保存有下一个节点的指针，并没有指向上一个节点的指针，所以不能直接入手删除节点，那么如何删除单链表的节点呢？

方法是：设置二级指针（指向Timer类型指针的指针），通过遍历链表的方式来寻找**节点中next指针指向删除节点的**那个节点，代码如下。

```c
void timer_stop(struct Timer* handle)
{
	struct Timer** curr;
	for(curr = &head_handle; *curr; ) {
		struct Timer* entry = *curr;
		if (entry == handle) {
			*curr = entry->next;
//			free(entry);
		} else
			curr = &entry->next;
	}
}
```

# 5. 项目工程源码获取和问题交流
目前我将MultiTimer源码、我移植到小熊派STM32L431RCT6开发板的工程源码上传到了QQ群里（包含好几份HAL库，QQ相对速度快点），**可以在QQ群里下载，有问题也可以在群里交流，当然也欢迎大家分享出来自己移植的工程到QQ群里**：
![](https://img-blog.csdnimg.cn/20200501174938684.png)

放上QQ群二维码：
![](https://img-blog.csdnimg.cn/20200321144543508.png)

<font color="red">**接收更多精彩文章及资源推送，欢迎订阅我的微信公众号：『mculover666』。**</font>
![](https://img-blog.csdnimg.cn/20200202092055136.png)
