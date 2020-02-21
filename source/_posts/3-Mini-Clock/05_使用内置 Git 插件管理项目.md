---
title: 桌面mini网络时钟教程 | 05-使用内置 Git 可视化插件管理项目
keywords: RT-Thread
tags: 制作过程
categories: 制作过程
abbrlink: 210720430
summary: 使用RT-Thread Studio打造
date: 2020-02-20 18:04:56
---
# 1. 项目进度
桌面Mini时钟项目用来演示如何使用RT-Thread Stduio开发项目，整个项目的架构如下：
![](https://img-blog.csdnimg.cn/2020020311120887.png#pic_center) 

在前四篇博文中简单的介绍了RT-Thread Studio一站式工具，基于STM32L431RCT6这个芯片创建工程，并修改时钟为使用外部时钟，以及添加SHT3x软件包获取温湿度传感器数据，添加了ESP8266设备连接网络，使用NTP服务器进行网络对时，最后添加u8g2软件包，驱动OLED显示数字时钟效果和温湿度效果。

> - [使用RT-Thread Studio DIY 迷你桌面时钟（一）| 基于STM32芯片创建工程](https://blog.csdn.net/Mculover666/article/details/104146623)
>- [使用RT-Thread Studio DIY 迷你桌面时钟（二）| 获取温湿度传感器数据（I2C设备驱动+SHT3x软件包）](https://mculover666.blog.csdn.net/article/details/104153715)
>- [使用RT-Thread Studio DIY 迷你桌面时钟（三）| 获取NTP时间（at_device软件包 + netutils软件包）](https://mculover666.blog.csdn.net/article/details/104418075)
> -[使用RT-Thread Studio DIY 迷你桌面时钟（四）| OLED显示时钟和温湿度（cpp组件 + u8g2软件包）](https://mculover666.blog.csdn.net/article/details/104422501)

本文中默认你已经了解Git的基本使用以及命令行操作，接下来讲述如何使用RT-Thread Studio内置的Git插件来管理整个项目。

# 2. Git插件配置
进入窗口->首选项，选择小组->Git->配置，可以看到当前Git工具的配置：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200221143355548.png)

如果没有，说明Git安装或者配置不成功，可以参考我的教程：

- [【Git & Github】（二）Git简介及其安装（Git是什么、Git的诞生、Git的优势、Git的安装、初次运行Git前的配置）](https://mculover666.blog.csdn.net/article/details/90034512)。

# 3. 建立并管理本地仓库
## 3.1. 建立仓库
在项目名称右键单击，按照图中所示点击共享项目：
![](https://img-blog.csdnimg.cn/20200221143735579.png)

选中第一个红框中的内容，点击Create Repository：
![](https://img-blog.csdnimg.cn/20200221143824708.png)
创建仓库之后，点击完成：
![](https://img-blog.csdnimg.cn/20200221143911199.png)
再次右键单击项目名称选择小组，可以看到当前所有可以进行的Git操作：
![](https://img-blog.csdnimg.cn/20200221144052288.png)

选择`Show in Repositories View`可进入仓库视图查看：
![](https://img-blog.csdnimg.cn/20200221144251504.png)
选择`Show in History`可进入历史提交信息视图查看：
![](https://img-blog.csdnimg.cn/20200221144332569.png)
## 3.2. 提交当前内容/修改
直接点击提交即可：
![](https://img-blog.csdnimg.cn/20200221144540501.png)
提交成功之后可以在之前的历史视图中查看到历史：
![](https://img-blog.csdnimg.cn/20200221144747839.png)
# 4. 配置并推送到远程库
## 4.1. 建立远程仓库
在Github建立一个新的空仓库过程此处不作赘述，如果不熟悉，请参考教程：

- [【Git & Github】（六）Git命令行操作 —— Github远程库操作（创建远程库、给远程库地址取别名、推送远程库、拉取远程库、克隆远程库）](https://mculover666.blog.csdn.net/article/details/90258680)

建立好之后的空仓库如图：
![](https://img-blog.csdnimg.cn/2020022114513740.png)

## 4.2. 配置远程库
在仓库视图中右击远程对象，选择新建远程对象：
![](https://img-blog.csdnimg.cn/20200221145252298.png)
这里我以建立推送对象为例：
![](https://img-blog.csdnimg.cn/20200221145402127.png)
填写远程库信息：
![](https://img-blog.csdnimg.cn/20200221145551723.png)
![](https://img-blog.csdnimg.cn/20200221145619376.png)
创建完成之后在仓库视图可以看到信息：
![](https://img-blog.csdnimg.cn/2020022114573123.png)
## 4.3. 推送提交信息到远程库
在小组中点击push到master分支：
![](https://img-blog.csdnimg.cn/20200221145825892.png)
按照如下步骤推送：
![](https://img-blog.csdnimg.cn/20200221145916553.png)
推送之后可以在Github仓库上看到：
![](https://img-blog.csdnimg.cn/20200221150006385.png)

还有一些另外的拉取远程库操作，分支操作内容不常用，暂不讲述，有兴趣可以自行研究。

<font color="red">**接收更多精彩文章及资源推送，欢迎订阅我的微信公众号：『mculover666』。**</font>
![](https://img-blog.csdnimg.cn/20200202092055136.png)



