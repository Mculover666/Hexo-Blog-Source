---
title: 桌面mini网络时钟教程 | 03-获取NTP时间
keywords: RT-Thread
tags: 制作过程
categories: 制作过程
abbrlink: 2731501981
summary: 使用RT-Thread Studio打造
date: 2020-02-20 18:02:56
---

# 1. 项目进度
桌面Mini时钟项目用来演示如何使用RT-Thread Stduio开发项目，整个项目的架构如下：
![](https://img-blog.csdnimg.cn/2020020311120887.png#pic_center) 

在前两篇博文中简单的介绍了RT-Thread Studio一站式工具，基于STM32L431RCT6这个芯片创建工程，并修改时钟为使用外部时钟，以及添加SHT3x软件包获取温湿度传感器数据。

> - [使用RT-Thread Studio DIY 迷你桌面时钟（一）| 基于STM32芯片创建工程](https://blog.csdn.net/Mculover666/article/details/104146623)
>- [使用RT-Thread Studio DIY 迷你桌面时钟（二）| 获取温湿度传感器数据（I2C设备驱动+SHT3x软件包）](https://mculover666.blog.csdn.net/article/details/104153715)

接下来添加at_device设备ESP8266用于连接网络，添加netutils软件包用于获取NTP时间。

# 2. 添加ESP8266设备驱动
## 2.1. 使能libc组件
使用at_device软件包之前，需要先开启libc组件：
![](https://img-blog.csdnimg.cn/20200220210608546.png)

## 2.2. 添加at_device软件包
本项目中使用的是ESP8266设备，其基于AT框架的驱动示例代码在at_device软件包中提供。

点击添加软件包按钮，搜索at_device，添加该软件包：
![](https://img-blog.csdnimg.cn/20200220210714491.png)
添加到工程设置之后，右键单击进入该软件包配置页面：
![](https://img-blog.csdnimg.cn/20200220210851880.png)
配置实际信息：
![](https://img-blog.csdnimg.cn/20200220211138925.png)
保存设置，软件会自动添加at_device软件包到工程中。

## 2.3. 开启lpuart1串口设备

在上一小节配置软件包的时候，设置的串口设备为lpuart1，但是目前系统中并没有设备lpuart1，需要手动在`board.h`文件中开启，开启方法在该文件的注释中已经说明，如图：

![](https://img-blog.csdnimg.cn/20200220211611684.png)

按照图中注释的说明，首先添加使用lpuart1的宏定义，开启串口：
![](https://img-blog.csdnimg.cn/20200220211805851.png)
接下来修改具体lpuart1串口的引脚配置：
![](https://img-blog.csdnimg.cn/20200220212424417.png)
接下来为了后续方便操作控制台，将sht3x的测试线程先注释掉：
![](https://img-blog.csdnimg.cn/20200220212746461.png)
然后编译整个项目，下载，查看串口输出：
![](https://img-blog.csdnimg.cn/20200220212928877.png)

可以看到有一个警告，提示 RT_SERIAL_RB_BUFSZ 的值不够用了，解决方案就是在设置中加大该缓冲区的值：
![](https://img-blog.csdnimg.cn/20200220213149595.png)

重新编译，下载，在串口控制台中查看网络是否成功配置：
![](https://img-blog.csdnimg.cn/20200220213443837.png)
>两条红色的日志打印暂且没有影响，是因为使用lpuart的原因，如果换为使用普通uart，在完全相同的配置下，没有这两条信息。

## 2.4. 测试网络
再使用下面的命令进行进一步的测试，确保网络正常能访问外网：
```bash
ifconfig		//使用该命令查看当前网卡配置
ping www.baidu.com	//使用该命令测试外网是否可以ping通
```
测试结果如下：
![](https://img-blog.csdnimg.cn/20200220213716935.png)

# 3. 添加NTP对时功能
## 3.1. 添加netutils工具软件包
netutils软件包中汇集了 RT-Thread 可用的全部网络小工具集合，包括NTP工具。

NTP 是网络时间协议(Network Time Protocol)，它是用来同步网络中各个计算机时间的协议，RT-Thread 上的 NTP 客户端连接上网络后，可以获取当前 UTC 时间，并更新至 RTC 中。

打开配置文件，添加软件包，搜索NTP之后添加：
![](https://img-blog.csdnimg.cn/20200220214338407.png)
右击软件包，修改该软件包的配置：
![](https://img-blog.csdnimg.cn/20200220214452604.png)
开启NTP服务器配置即可：
![](https://img-blog.csdnimg.cn/20200220214703716.png)
## 3.2. 开启软件模拟RTC
因为NTP工具在获取到网络时间后，需要同步到本地RTC，所以需要开启本地模拟RTC功能：
![](https://img-blog.csdnimg.cn/20200220214952595.png)
开启之后保存配置，重新编译工程，下载，在串口控制台查看是否可以正常工作。

## 3.3. 测试NTP工作是否正常
使用NTP工具包自带的命令进行测试：
```bash
ntp_sync	//获取NTP时间并同步到本地
```
![](https://img-blog.csdnimg.cn/20200220215539556.png)

# 3.4. 编写上电自动同步时间代码
在本项目中，需要上电连接网络之后，自动获取NTP时间同步到本地，供后续显示使用，这段代码放在main函数中执行。

- ① 检测当前网络是否正常

netdev（network interface device），即网络接口设备，又称网卡。每一个用于网络连接的设备都可以注册成网卡，为了适配更多的种类的网卡，避免系统中对单一网卡的依赖，RT-Thread 系统提供了 netdev 组件用于网卡管理和控制。

使用 netdev 网卡功能相关操作函数，需要包含如下头文件：
```c
#include <arpa/inet.h>         /* 包含 ip_addr_t 等地址相关的头文件 */
#include <netdev.h>            /* 包含全部的 netdev 相关操作接口函数 */
```
首先获取要操作的网卡对象，每个网卡中有唯一的网卡名称，可以通过网卡名称获取网卡对象：：
```c
struct netdev *netdev_get_by_name(const char *name);
```
netdev 网卡提供了一个宏定义用于判断网卡是否为 internet_up 状态，如下：
```c
#define netdev_is_internet_up(netdev)
```
通过这个宏定义即可判断当前网络状态是否正常，在main函数中添加如下代码：
```c
//获取网卡对象
struct netdev* net = netdev_get_by_name("esp0");

//阻塞判断当前网络是否正常连接
while(netdev_is_internet_up(net) != 1)
{
   rt_thread_mdelay(200);
}
//提示当前网络已就绪
rt_kprintf("network is ok!\n");
```
编译，下载，效果如下：
![](https://img-blog.csdnimg.cn/20200221090336676.png)

- ② 网络正常后，获取NTP时间并同步到本地RTC

要使用NTP工具提供的API，首先包含头文件：
```c
#include <ntp.h>
```
接着调用获取时间并同步的API，原型如下：
```c
time_t ntp_sync_to_rtc(void);
```
在上面的代码之后继续添加NTP对时的代码：
```c
//NTP自动对时
time_t cur_time;
cur_time = ntp_sync_to_rtc(NULL);
if (cur_time)
{
    rt_kprintf("Cur Time: %s", ctime((const time_t*) &cur_time));
}
else
{
    rt_kprintf("NTP sync fail.\n");
}
```
编译，下载，效果如下：
![](https://img-blog.csdnimg.cn/20200221091157972.png)

<font color="red">**接收更多精彩文章及资源推送，欢迎订阅我的微信公众号：『mculover666』。**</font>
![](https://img-blog.csdnimg.cn/20200202092055136.png)









