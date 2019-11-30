---
title: 【Hexo-01】本地建立 Hexo 站点
tags: Hexo
categories: Hexo
summary: 最详细的Hexo博客搭建教程，没有之一
cover: true
coverImg: 'http://mculover666.cn/blog/20191031/R4mWMXsrRKxu.png?imageslim'
abbrlink: 3223194057
date: 2019-10-31 10:06:52
---

![](http://mculover666.cn/blog/20191031/R4mWMXsrRKxu.png?imageslim)

<!--more-->

# 效果展示

<iframe src="//player.bilibili.com/player.html?aid=73999970&cid=126588469&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>

# 安装Git、Nodejs、Hexo

## Git

参考文章：[【Git & Github】（二）Git简介及其安装](https://blog.csdn.net/Mculover666/article/details/90034512)

## Nodejs

Nodejs可以从官网（ https://nodejs.org/en ）下载LTS版本：

![](http://mculover666.cn/blog/20191030/iRwuHKdWVxi9.png?imageslim)

![](http://mculover666.cn/blog/20191030/whGi0PMz6aHT.png?imageslim)

![](http://mculover666.cn/blog/20191030/wwHAxxKKE4ok.png?imageslim)

安装之后检查一下是否正常输出版本信息：

![](http://mculover666.cn/blog/20191030/vMCw8uPGVKyY.png?imageslim)

## Hexo

>本文中所有的命令执行时，可以在Git bash中执行，但速度比较慢；如果要在cmd中执行，速度比较快，但要确保 git 已经添加到环境变量中！（即在cmd中执行 git --version 可以看到git版本信息）。

Hexo是一款快速简洁的博客框架，可以将 md 文档渲染为静态 HTML 页面，拥有非常多的主题和插件可以选择，安装过程如下：

```bash
npm install -g hexo-cli 
```

![](http://mculover666.cn/blog/20191030/PG748jWyKOFl.png?imageslim)

安装之后检查一下是否正常输出版本信息：

![](http://mculover666.cn/blog/20191030/AFBoAB1K0PU5.png?imageslim)

# 创建站点

## 初始化站点文件夹
```
hexo init <floder>
```
使用该命令会将Github上Hexo源码和默认主题源码拉取到本地，该文件夹即为**站点根目录**：

![](http://mculover666.cn/blog/20191030/QS02l4157fhc.png?imageslim)

![](http://mculover666.cn/blog/20191031/kE9J36TrVyJl.png?imageslim)

## 安装Hexo依赖模块

**后续所有的命令都是在站点根目录执行的**，所以在命令行中进入上一步Hexo创建的文件夹，：
```
cd <floder>
```
然后执行该命令，安装Hexo的依赖模块：
```
npm install
```

![](http://mculover666.cn/blog/20191030/TvLgAYNipcKP.png?imageslim)

这样 Hexo 站点就成功创建啦！

## 本地启动站点服务
```
hexo s
```
使用该命令，Hexo会在本地4000端口启动Web服务，供浏览器访问：

![](http://mculover666.cn/blog/20191030/bacz9ThXBtmI.png?imageslim)

## 访问本地站点

使用浏览器访问http://localhost:4000 即可：

![](http://mculover666.cn/blog/20191030/D2y2VptQejrS.png?imageslim)

本地启动和访问站点有什么用呢？

本地预览！

文章写好后，可以先在本地生成页面并启动服务，然后在浏览器中预览一下，确认没问题再推送到服务器上，方便很多。

# 修改站点配置

关于网站的所有自定义配置，都是在站点根目录下的`_config.yml`文件中配置，以后统称为**站点配置文件**：

![](http://mculover666.cn/blog/20191031/YR2liEgFSOgO.png?imageslim)

使用 VS Code 打开该文件，首先强调一下语法：

![](http://mculover666.cn/blog/20191031/LSFxL4j14tvM.png?imageslim)

这里面的配置项非常多，后续我们都会讲，本文中先讲第一块配置：

![](http://mculover666.cn/blog/20191031/Xm7XziSX01eQ.png?imageslim)

这些配置项自己修改，一定要注意语法，修改之后进行如下操作：

- 清除旧的生成页面
```
hexo clean
```
- 生成新的HTML页面
```
hexo g
```

![](http://mculover666.cn/blog/20191031/ywlNHleIgdqB.png?imageslim)

然后重新启动服务即可在本地看到效果：
```
hexo s
```

![](http://mculover666.cn/blog/20191031/R1RiAWDAhBBo.png?imageslim)


