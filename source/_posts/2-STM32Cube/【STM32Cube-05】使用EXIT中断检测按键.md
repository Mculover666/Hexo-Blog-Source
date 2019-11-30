---
title: 【STM32Cube_05】使用EXIT中断检测按键
tags: STM32CubeMX
categories: STM32CubeMX
abbrlink: 2504113390
date: 2019-07-26 10:00:56
---

本篇文章主要介绍如何使用STM32CubeMX初始化STM32L431RCT6的EXIT检测按键，讲述了一些NVIC的小知识，并一步一步探索了HAL库的中断处理机制。
<!--more-->
# 1. 准备工作
## 硬件准备
- 开发板
首先需要准备一个开发板，这里我准备的是STM32L4的开发板（BearPi）：

![mark](http://mculover666.cn/image/20190806/9uiPTi5odYSj.png?imageslim)

## 软件准备
- 需要安装好Keil - MDK及芯片对应的包，以便编译和下载生成的代码；

>Keil MDK和串口助手Serial Port Utility 的安装包都可以**在文末关注公众号获取**，回复关键字获取相应的安装包：

![mark](http://mculover666.cn/image/20190814/gubaOwmETp1w.png?imageslim)

# 2.生成MDK工程
## 选择芯片型号
打开STM32CubeMX，打开MCU选择器：

![mark](http://mculover666.cn/image/20190806/gBP6glmUSH80.png?imageslim)

搜索并选中芯片`STM32L431RCT6`:

![mark](http://mculover666.cn/image/20190806/gnyHwdl53uVD.png?imageslim)

## 配置时钟源

- 如果选择使用外部高速时钟（HSE），则需要在System Core中配置RCC；
- 如果使用默认内部时钟（HSI），这一步可以略过；

这里我都使用外部时钟：

![mark](http://mculover666.cn/image/20190806/k593lGGb5tlW.png?imageslim)

## 配置LED的GPIO引脚

查看小熊派开发板的原理图，如下：

![mark](http://mculover666.cn/image/20190812/5iCtQUfKbgzA.png?imageslim)

所以接下来我们选择配置`PC13`引脚：

![mark](http://mculover666.cn/image/20190812/Ad3UrGCsgjXr.png?imageslim)

设置用户标签为LED：

![mark](http://mculover666.cn/image/20190813/ClKDFmVJYceI.png?imageslim)

## 配置GPIO引脚为外部中断引脚
查看小熊派开发板的原理图，如下：

![mark](http://mculover666.cn/image/20190813/QNbG5i6QKGk3.png?imageslim)

所以接下来我们选择配置`PB2`引脚和`PB3`引脚为外部中断引脚：

![mark](http://mculover666.cn/image/20190814/uqRDYJldwiwk.png?imageslim)

因为没有设置硬件上拉，所以我们配置开启上拉电阻，并设置用户标签为`KEY1`和`KEY2`，接下来是最重要的一步：

- 开启下降沿触发中断：即在**按下按键时**电平由高变为低时触发
- 开启上升沿触发中断：即在**按下按键后松开时**电平由低变为高时触发
- 开启下降沿上升沿都触发中断：即在**按下时触发，松开时再次触发**

这里我选择开启下降沿触发中断：

![mark](http://mculover666.cn/image/20190814/Kt7hrzxmbjVq.png?imageslim)

## 配置NVIC设置中断优先级

>知识小卡片 —— NVIC

NVIC全称`Nested vectored interrupt controller`，即嵌套向量中断控制器，用来决定**中断的优先级**。

NVIC在 ARM Conrtex-M 内核中，用一个 8 位的寄存器来配置，总共可以配置$2^8=256$级中断，但是 ST 公司在生产 STM32 的时候，发现一个小小的单片机根本用不了这么多，纯属浪费，所以将该寄存器的`低 4 位` 全部置0，只使用`高 4 位`来配置，这样一来 STM32 就只有$2^4=16$级中断啦。

简化为16级中断后，ST发现 STM32 内部这么丰富的外设，还是不方便配置，干脆**人工给这4位来个分组**，划分出了5个分组：

|优先级分组|抢占优先级占的位数|子优先级占的位数|
|:-:|:-:|:-:|
|NVIC_PriorityGroup_0|0 bit|4 bit|
|NVIC_PriorityGroup_1|1 bit|3 bit|
|NVIC_PriorityGroup_2|2 bit|2 bit|
|NVIC_PriorityGroup_3|3 bit|1 bit|
|NVIC_PriorityGroup_4|4 bit|0 bit|

再次强调一下，这5种中断分组规则是人为的，用哪种规则，之后设置具体的优先级时对应就行，STM32默认使用的规则是 NVIC_PriorityGroup_0 。

STM32 的CPU判断优先级的方法如下：

- 先判断抢占优先级，数字越小，优先级越高；
- 若抢占优先级相同，判断子优先级，同样，数字越小，优先级越高；

>知识小卡片结束啦~ 对NVIC有没有了解呢？

接下来在STM32CubeMX中配置中断优先级：

### 配置优先级分组

这里我配置使用中断优先级分组规则 NVIC_PriorityGroup_2：

![mark](http://mculover666.cn/image/20190814/bfVkymV1nHBy.png?imageslim)

### 配置具体的优先级大小
根据中断优先级分组规则 NVIC_PriorityGroup_2来设置具体的优先级大小：

![mark](http://mculover666.cn/image/20190814/SvOnnceuPG9n.png?imageslim)

## 配置时钟树
STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：

![mark](http://mculover666.cn/image/20190806/1TQg7frjRpVr.png?imageslim)

## 生成工程设置

![mark](http://mculover666.cn/image/20190814/H2VnDFkiF0JQ.png?imageslim)

## 代码生成设置

最后设置生成独立的初始化文件：

![mark](http://mculover666.cn/image/20190812/PwTCS6QzHiyG.png?imageslim)

## 生成代码

点击`GENERATE CODE`即可生成MDK-V5工程：

![mark](http://mculover666.cn/image/20190806/s0jGhLBWW6Cm.png?imageslim)

# 3. 在MDK中编写、编译、下载用户代码

## STM32 HAL库中断处理机制
先打开`stm32l4xx_it.c`文件：

![mark](http://mculover666.cn/image/20190814/U0mhU0pBakNc.png?imageslim)

可以看到其中处理EXIT2和EXIT3中断都调用了同一个函数，但是EXIT2和EXIT3向该函数传入的参数不同：
```c
HAL_GPIO_EXTI_IRQHandler();
```
那么，HAL库对于中断是如何处理的呢？我们打开 `stm32l4xx_hal_gpio.c` 文件，看一下该函数的原型，一探究竟：
```c
/**
  * @brief  Handle EXTI interrupt request.
  * @param  GPIO_Pin Specifies the port pin connected to corresponding EXTI line.
  * @retval None
  */
void HAL_GPIO_EXTI_IRQHandler(uint16_t GPIO_Pin)
{
  /* EXTI line interrupt detected */
  if(__HAL_GPIO_EXTI_GET_IT(GPIO_Pin) != 0x00u)
  {
    __HAL_GPIO_EXTI_CLEAR_IT(GPIO_Pin);
    HAL_GPIO_EXTI_Callback(GPIO_Pin);
  }
}
```
可以看到，在该函数中首先读取了一下中断寄存器，确认该中断是否发生，确认之后又调用了一个函数，并将接收到的参数 `GPIO_Pin` 继续传给该函数：
```c
HAL_GPIO_EXTI_Callback(GPIO_Pin);
```
**该函数称为EXIT中断的回调函数，用来处理所有发生的EXIT中断事件。**

那么，这个函数又干了什么呢？接着探索哈哈哈~

同样在`stm32l4xx_hal_gpio.c`文件中找到该函数的原型：
```c
/**
  * @brief  EXTI line detection callback.
  * @param  GPIO_Pin: Specifies the port pin connected to corresponding EXTI line.
  * @retval None
  */
__weak void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
  /* Prevent unused argument(s) compilation warning */
  UNUSED(GPIO_Pin);

  /* NOTE: This function should not be modified, when the callback is needed,
           the HAL_GPIO_EXTI_Callback could be implemented in the user file
   */
}
```
哈哈哈，这下是不是非常清楚了~

该回调函数使用`__weak`进行了弱定义，所以**用户可以再次定义该函数**，并且这个`note`写的非常清楚：

>这个函数不应该被改变，如果需要使用回调函数，请重新在用户文件中实现该函数。

## 自己实现EXIT中断处理回调函数
这个函数放在哪都行，为了方便，我们放在`gpio.c`的最后。

实现的基本思想是：

- 因为所有的EXIT中断都会调用该函数，所以首先判断具体的中断事件；
- 对该中断事件进行处理

实现代码如下：
```c
/* USER CODE BEGIN 2 */
/**
 * @brief	EXIT中断回调函数
 * @param GPIO_Pin —— 触发中断的引脚
 * @retval	none
*/
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
	/* 判断哪个引脚触发了中断 */
	switch(GPIO_Pin)
	{
		case GPIO_PIN_2:
			/* 处理GPIO2发生的中断 */
			//点亮LED
			HAL_GPIO_WritePin(LED_GPIO_Port,LED_Pin,GPIO_PIN_SET);
			break;
		case GPIO_PIN_3:
			/* 处理GPIO3发生的中断 */
			//熄灭LED
			HAL_GPIO_WritePin(LED_GPIO_Port,LED_Pin,GPIO_PIN_RESET);
			break;
		default:
			break;
	}
}
/* USER CODE END 2 */
```

## 编译代码

编译整个工程：

![mark](http://mculover666.cn/image/20190814/cQclXS2zTHaV.png?imageslim)

## 设置下载器

![mark](http://mculover666.cn/image/20190812/PHve6DYPkO9M.png?imageslim)

![mark](http://mculover666.cn/image/20190812/djSNbMCj6Hh6.png?imageslim)

## 实验现象
下载运行后，实验现象如下：

- 上电复位时LED处于熄灭状态；
- 按下KEY1，LED点亮；
- 按下KEY2，LED熄灭；

![mark](http://mculover666.cn/image/20190814/M5YtUckjugDP.png?imageslim)

至此，我们已经学会了**如何配置NVIC使用外部中断检测按键**，并了解了NVIC和HAL库中断处理机制的一些基本知识，下一节讲述如何配置USART以及实现`printf`函数。

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![mark](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)