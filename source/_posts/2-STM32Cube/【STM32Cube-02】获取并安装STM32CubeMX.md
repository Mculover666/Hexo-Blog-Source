---
title: 【STM32Cube_02】获取并安装STM32CubeMX
tags: STM32CubeMX
categories: STM32CubeMX
abbrlink: 2106737533
date: 2019-07-23 10:48:56
---

本篇文章主要介绍如何获取并安装STM32CubeMX。
<!--more-->

本文中涉及到的安装包可以在官网下载到，速度比较慢，为了方便大家，我已上传到百度网盘，为了确保资源的更新，我没有直接放上链接，可以在**文末**关注我的微信公众号，回复“**STM32CubeMX**”获取，敬请原谅~

![mark](http://mculover666.cn/image/20190814/gubaOwmETp1w.png?imageslim)

# 1.安装Java环境（JRE）
因为STM32CubeMX是采用Java语言编写的，所以需要**先在电脑上安装Java运行环境**（JRE，Java runtime Environment），安装JRE时建议**选择Java 8或者以后的版本**。

安装JRE有两种方式：

- 单独的安装Jre；
- 直接安装开发者套件JDK，其中就包括了JRE，这样以后还能用于开发Java。

这里我选择第二种，安装JDK8不是本文的重点，具体请参考我的Java系列文章或者网上自行搜索。

# 2.获取STM32CubeMX
STM32CubeMX可以访问STM32官网( https://www.st.com/en/development-tools/stm32cubemx.html )获取：

![mark](http://mculover666.cn/image/20190811/bWTWrfuJnhX5.png?imageslim)

然后同意下载协议，填写一些信息，ST会向填写的邮箱中发送一封邮件，点击邮件中的链接即可下载。

# 3.安装STM32CubeMX
解压下载的压缩包，其中包含三个平台的安装包和一个发布说明，这里我们选择Windows平台的安装包：

![mark](http://mculover666.cn/image/20190811/JmkwIDTsSWo1.png?imageslim)

双击运行安装程序，安装过程如下：

![mark](http://mculover666.cn/image/20190811/QRiGcKFGel1T.png?imageslim)

选择是否同意许可协议：

![mark](http://mculover666.cn/image/20190811/P8S8GriWkoyG.png?imageslim)

![mark](http://mculover666.cn/image/20190811/1c4D5wBFVV9e.png?imageslim)

选择STM32CubeMX安装目录，注意**不能有中文或者空格**：

![mark](http://mculover666.cn/image/20190811/YH9UCGaHSIX7.png?imageslim)

如果这个目录不存在，会提示是否创建该目录：

![mark](http://mculover666.cn/image/20190811/3933VlaP8fHp.png?imageslim)

选择创建桌面快捷方式：

![mark](http://mculover666.cn/image/20190811/xwgbSVuGHgdb.png?imageslim)

安装中...

![mark](http://mculover666.cn/image/20190811/aAvl1nIYwYBj.png?imageslim)

安装完成后如图：

![mark](http://mculover666.cn/image/20190811/DoLwzbrlsDT6.png?imageslim)

最后选择是否记录刚刚安装的过程，并创建一个安装脚本，下次直接执行脚本就可以完成这个安装过程了，这里选择不创建：

![mark](http://mculover666.cn/image/20190811/VcnP9F8j04bS.png?imageslim)

桌面图标如下：

![mark](http://mculover666.cn/image/20190811/1EcyTM3NGxL7.png?imageslim)

# 4. 获取并安装STM32Cube MCU Packages
STM32Cube MCU Packages的安装方式有两种：

- 在STM32CubeMX中**在线安装**；
- 在ST官网获取STM32Cube MCU Packages，然后**离线安装**；

## 在线安装STM32Cube MCU Packages
打开STM32CubeMX，选择`Help`->`Manage embedded software packages`：

![mark](http://mculover666.cn/image/20190811/28Oclq0pEvP3.png?imageslim)

在packages列表中勾选上需要的 `MCU Packages`， 点击`Install Now` ：

![mark](http://mculover666.cn/image/20190811/plS5zNQiSzAo.png?imageslim)

安装中（下载速度还比较快哈哈哈）：

![mark](http://mculover666.cn/image/20190811/FSVTgvBenHS5.png?imageslim)

## 离线安装STM32Cube MCU Packages
首先在[ST官网的packages列表](https://www.st.com/content/st_com/en/stm32cube-ecosystem.html)找到需要的packages，点击名字即可跳转：

![mark](http://mculover666.cn/image/20190811/ts9NYzyepz08.png?imageslim)

跳转过去之后点击获取，开始下载packages：

![mark](http://mculover666.cn/image/20190811/kuiYBrE14cre.png?imageslim)

下载之后如图（**不要解压**）：

![mark](http://mculover666.cn/image/20190811/nflkfFhiwRq3.png?imageslim)

### 导入package到STM32CubeMX
打开STM32CubeMX的包管理器（方法同上），点击`From Local`：

![mark](http://mculover666.cn/image/20190811/i384oOghVbBU.png?imageslim)

选择打开要导入的package：

![mark](http://mculover666.cn/image/20190811/bDoB6uLWc3rR.png?imageslim)

点击打开即可成功导入。

至此，我们已经安装好了STM32CubeMX和STM32L4的MCU Package，下一节中讲述如何使用STM32CubeMX快速生成MDK的工程，点亮一个LED。

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![mark](http://mculover666.cn/image/20190814/NQqt1eRxrl1K.png?imageslim)