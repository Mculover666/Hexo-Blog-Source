---
title: 【Hexo进阶教程-03】自有云图床配合Mpic，轻松解决md插图问题
tags: Hexo
categories: Hexo
summary: 最详细的Hexo博客搭建教程，没有之一
abbrlink: 1543687885
date: 2019-11-13 10:06:52
---

![](http://mculover666.cn/blog/20191031/R4mWMXsrRKxu.png?imageslim)

<!--more-->

# 教程汇总

- [【Hexo基础教程-01】本地建立 Hexo 站点](http://www.mculover666.cn/posts/3223194057/)
- [【Hexo基础教程-02】创建新文章并生成页面](http://www.mculover666.cn/posts/1228606410/)
- [【Hexo基础教程-03】Github + Coding 部署Hexo站点](http://www.mculover666.cn/posts/3223194057/)
- [【Hexo基础教程-04】换一个炫酷的响应式主题 —— Matery](http://www.mculover666.cn/posts/3223194057/)

- [【Hexo进阶教程-01】优化文章永久链接为数字编号]()
- [【Hexo进阶教程-02】使用Appveyor备份并持续集成博客]()

# 1. 待优化问题

在使用 md 写文章的时候，图片的路径是一个很大的问题，很多人使用**相对路径**来引用图片，一方面是占用存储资源，另一方面是一旦路径修改，图片路径全部失效，很不方便。

还记得在CSDN里写博客是如何插图的吗？

截图或者复制一张图片之后，在 CSDN 的 down 编辑器中直接按`Ctrl+V`复制，编辑器中就会自动将图片上传到图床，并生成一个图片的url链接，俗称图片外链。

本文接下来将带你拥有一个自己的云图床，并配合MPic工具实现类似CSDN这样的快速插图。

# 2. 拥有一个自有云图床

所谓图床，其实就是云服务厂商提供的对象存储服务，简称OBS服务，各大云厂商都有该服务，收费情况不一样，这里我强烈建议：**七牛云**！

来看看七牛云的免费额度：

![](http://mculover666.cn/blog/20191113/DBYTA42ukGFh.png?imageslim)

这个免费的量，够用吗？

咱用数据来说话，到目前为止， 我写了有50多篇博客，都是以步骤为主，图片比较多，来看看我的用量：

![](http://mculover666.cn/blog/20191113/bFUfexiHwXkf.png?imageslim)

10GB用来放图片，足够了，可千万别放视频哦，不然100GB都不够用的~

言归正传，开干！

## 注册登录七牛云

七牛云的网址是：[https://www.qiniu.com/](https://www.qiniu.com/)，访问后先注册登录七牛云，然后在右上角**进入管理控制台**。

## 创建OBS存储空间

点击对象存储立即添加按钮，创建一个OBS存储空间：

![](http://mculover666.cn/blog/20191113/DSV0L1hYmEel.png?imageslim)

填写OBS存储空间信息：

![](http://mculover666.cn/blog/20191113/82njP0JsQzIx.png?imageslim)

填写完成之后点击页面下方的`立即创建`，创建成功后如图所示：

![](http://mculover666.cn/blog/20191113/KtTgeKs5iyA1.png?imageslim)

## 手动体验一下云图床

在内容管理页面，点击`上传文件`：

![](http://mculover666.cn/blog/20191113/hjvocHp3ACM8.png?imageslim)

上传一张本地图片：

![](http://mculover666.cn/blog/20191113/Pg6bAK3nsQ2I.png?imageslim)

上传成功：

![](http://mculover666.cn/blog/20191113/eNFcX8WTywVI.png?imageslim)

点击`复制外链`：

![](http://mculover666.cn/blog/20191113/qvDORonAMi31.png?imageslim)

复制出来的外链如下，直接插入到md文件中即可：
```
http://q0w2lvoj6.bkt.clouddn.com/Logo.jpg
```

然后将其写在downw文件中即可：

```
![logo](http://q0w2lvoj6.bkt.clouddn.com/Logo.jpg)
```

效果如下：

![logo](http://q0w2lvoj6.bkt.clouddn.com/Logo.jpg)

## 关于外链域名

创建OBS的时候，七牛云会默认给一个测试域名，这个域名有效期30天：

![](http://mculover666.cn/blog/20191113/yK8ucKCtFJRV.png?imageslim)

所以之前的教程里我大概提了一下，拥有一个自己的域名会很方便。

因为七牛云的OBS进行了CDN加速，所以绑定的域名必须**备案通过**，这里我也有个小经验分享：

域名备案时一般都是一级域名，比如`mculover666.cn`，所以在云图床绑定的域名必须是该域名，可是这样博客就没有域名了呀~

别慌！一级域名没有了，二级域名任你玩！

在DNS解析的时候，所有加了前缀的域名都是二级域名，可以随意解析，比如你的博客域名可以是：

- `www.mculover666.cn`
- `blog.mculover666.cn`
- ……

# 3. MPic图床神器

在上一步中我们体验了如何手动上传一张图片并生成图片url外链，这样的过程未免过于麻烦，下面拉出我们的图床神器 —— MPic。

MPic可以直接在其官网 [http://mpic.lzhaofu.cn/](http://mpic.lzhaofu.cn/) 下载：

![](http://mculover666.cn/blog/20191113/qR7M4Do0VPIk.png?imageslim)

该神器的界面如下：

![](http://mculover666.cn/blog/20191113/DebbyXWBO2Sv.png?imageslim)

## 绑定七牛云授权信息

首次运行时，需要绑定七牛云的一些授权信息，点击左上角的`设置账号`：

>这里我展示的是我自己的配置，需要全部修改为你的配置！

![](http://mculover666.cn/blog/20191113/6XNsKsG9hBTK.png?imageslim)

七牛空间名和域名对应你自己的配置即可，AccessKey和SecretKey的获取方式如下：

进入`秘钥管理`页面：

![](http://mculover666.cn/blog/20191113/CLl0HUc53MlL.png?imageslim)

随便选择一对秘钥即可：

![](http://mculover666.cn/blog/20191113/Tn0M1WCQQDCm.png?imageslim)

## 开始使用

**该工具界面会置顶显示，可以点击关闭按钮，该工具会在后台保持运行**！

复制一张图片或者截图，之后该工具就会自动上传，不用管，上传完成之后会在屏幕右上角给出上传成功的信息，这时直接按`Ctrl+V`就会粘贴该图片的md外链，是不是很方便呢？

















