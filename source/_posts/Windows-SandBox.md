---
title: Win10沙盒开启方法
keywords: Win10沙盒
tags: 工具推荐
categories: 工具推荐
abbrlink: 2291178417
date: 2019-11-30 21:00:56
summary: 拥有一个秒启动的Win10环境
abbrlink: 2291178417
---

# Windows SandBox

Windows SandBox 是一个沙盒系统，可以在Windows系统下快速启动一个全新的Windows虚拟机。

Windows SandBox 有以下优势：

- Windows自带（专业版/企业版）
- 系统干净：每次启动时，系统都是全新的
- 启动速度快
- 一次性：关闭沙盒后，沙盒中的所有东西全部丢失
- 安全：沙盒提供了一个隔离环境，与主机分离

![Windows SandBox 界面](http://mculover666.cn/blog/20191126/nwG3SX5ciHhJ.png?imageslim)

# 开启条件

- `Windows系统专业版/企业版`

Windows系统专业版或者企业版中自带了Windows SandBox，无需下载直接开启，家庭版也可以安装，但是需要先下载安装文件，如有需要可自行搜索。

如何查看自己系统的版本呢？右击“此电脑”，选择“属性”即可查看：

![查看系统版本](http://mculover666.cn/blog/20191126/1iI9ayf0NuvK.png?imageslim)

- `OS版本号：18301或之后`

如何查看自己系统的版本号呢？使用 `win+R`打开命令行，输入winver查看：

![输入winver命令](http://mculover666.cn/blog/20191126/2mc2YdmG389Q.png?imageslim)

![查看OS版本号](http://mculover666.cn/blog/20191126/h6yKz3PN3hd4.png?imageslim)

如果满足这两个条件，快来跟我一起开启沙盒体验一下吧~

# 开启Windows SandBox功能

在搜索框搜索“启用和关闭windows功能”：

![打开windows功能开启或关闭页面](http://mculover666.cn/blog/20191126/sJaC92emywWl.png?imageslim)

找到 “windows 沙盒”，勾选确定：

![开启Windows沙盒](http://mculover666.cn/blog/20191126/iqLwxhSeCT6p.png?imageslim)

系统会提示重启电脑：


# 使用 Windows SandBox

在菜单栏选择`Windows SandBox`或者直接搜索`sandbox`，即可打开 Windows SandBox：

![Windows SandBox](http://mculover666.cn/blog/20191126/PnvLVugksspq.png?imageslim)


启动后的界面如下：

![Windows SandBox 界面](http://mculover666.cn/blog/20191126/nwG3SX5ciHhJ.png?imageslim)

进入Windows Sandbox，和进入主机的 Windows 系统中一样，进行软件的安装、浏览网页等等的操作。

# 主机与SandBox之间传输文件

Windows Sandbox和真实系统**共享剪贴板**，用户可以通过在主机系统中复制文件，然后在Windows Sandbox中直接粘贴即可。

# Windows SandBox使用注意事项

1. Windows SandBox比较迟内存，启动后占用内存情况如下，仅供参考：

![Win10 SandBox内存占用情况](http://mculover666.cn/blog/20191126/Bk6NVgJ05c6J.png?imageslim)

2. Windows SandBox是一次性的，关闭之后沙盒中所有东西都会消失，注意保存！

