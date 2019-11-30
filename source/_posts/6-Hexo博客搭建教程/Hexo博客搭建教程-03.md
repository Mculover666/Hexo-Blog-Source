---
title: 【Hexo-03】Github + Coding 部署Hexo站点
tags: Hexo
categories: Hexo
summary: 最详细的Hexo博客搭建教程，没有之一
abbrlink: 764890890
date: 2019-11-04 10:06:52
---

![](http://mculover666.cn/blog/20191031/R4mWMXsrRKxu.png?imageslim)

<!--more-->

# 专栏回顾

-[【Hexo-01】本地建立 Hexo 站点](http://www.mculover666.cn/posts/3223194057/)
-[【Hexo-02】创建新文章并生成页面](http://www.mculover666.cn/posts/1228606410/)

>本文同时并行讲解了如何在Github部署和在Coding部署，Github面向国际，但是速度慢，Coding速度快，但是不稳，选择一个自己喜欢的平台就好~

# 1. 何为部署？

之前我们在本地使用`hexo s`启动服务，然后浏览器访问`http://localhost:4000`即可访问到博客，但是博客搭建好之后总不能只有我们自己可以用，所以需要部署Hexo站点。

何为部署？

就是把 Hexo 生成的 HTML 页面放到一个具有公网ip的服务器上，这样大家都可以访问到博客站点了。

服务器最好是在各大云厂商买一台，1核2G的学生机配置足够，这样用着也舒服，然而，不想花钱怎么办？

天下有没有免费的午餐？当然有！

类似Github这样的代码托管平台提供了**静态Page服务**，也就是说我们**直接将HTML页面上传到Github仓库中**，并且开启该仓库的静态页面服务，这样大家都可以访问到你的站点，并且Github是全球性的，你的站点会收到来自世界各个角落的访问~

# 2. Github部署

## 创建空的仓库

![](http://mculover666.cn/blog/20191104/hnkyKxJEQeyH.png?imageslim)

创建之后仓库应该是这样的，复制仓库的 HTTPS 地址：

![](http://mculover666.cn/blog/20191104/K2hi5CfzT3xU.png?imageslim)

>补充说明一下：因为本地使用的是Windows系统，所以直接使用HTTPS地址即可，本地第一次访问Github时会要求输入Github密码，然后Windows的凭据管理器就会记住密码，之后不用再次输入。

## 配置Hexo

打开Hexo的**站点配置文件**（站点根目录下的`_config.yml`文件），找到`deploy`选项，填写 type选项的配置为`git`，然后在`repo`选项粘贴你刚刚复制的github仓库地址：

![](http://mculover666.cn/blog/20191104/iJxD3lqUIaRD.png?imageslim)

## 安装git部署插件

在站点根目录下打开命令行，执行如下命令安装`hexo-deployer-git`插件：

```
npm install hexo-deployer-git --save
```

![](http://mculover666.cn/blog/20191104/8rKOORyw0mAf.png?imageslim)

## 大功告成，部署站点

在站点根目录下执行命令，部署Hexo：

```
hexo d
```

>首次执行的时候可能会要求输入Github密码，输入即可！

![](http://mculover666.cn/blog/20191104/BwL6alw7sp3A.png?imageslim)

执行完成后再来看看Github仓库，部署成功：

![](http://mculover666.cn/blog/20191104/cXFhUhkFMBc7.png?imageslim)

## 访问站点

接下来就是见证奇迹的时刻了，访问**设置的仓库名**（域名就是仓库名）即可看的你的博客：

![](http://mculover666.cn/blog/20191104/hmpqtvmm3rVF.png?imageslim)

# 3. Coding部署

[Coding](https://coding.net/)（选择个人版登陆）的部署方式和Github的部署方式相同，因为Coding已经升级为腾讯云开发者平台，所以这里我们在腾讯云开发者平台操作：

## 创建空的仓库

![](http://mculover666.cn/blog/20191104/BRuEKjbRFmJK.png?imageslim)

在新建立项目的时候，**项目名称必须和用户名称完全一致**，否则之后部署Pages服务时就会出现静态资源加载失败，网页样式丢失的情况：

![](http://mculover666.cn/blog/20191104/qDA4dbBDfXhT.png?imageslim)

创建之后复制你的仓库地址：

![](http://mculover666.cn/blog/20191104/qAdk5UOCoRHh.png?imageslim)

## 配置Hexo

打开Hexo的**站点配置文件**（站点根目录下的`_config.yml`文件），找到`deploy`选项，填写 type选项的配置为`git`，然后在`repo`选项粘贴你刚刚复制的coding仓库地址：

![](http://mculover666.cn/blog/20191104/cFtkglHDiusA.png?imageslim)

## 安装git部署插件

参考Github部署的这一章节，操作相同，已经安装过的不用再次安装：

```
npm install hexo-deployer-git --save
```

## 部署站点

在站点根目录下执行部署命令：

```
hexo d
```

>首次执行的时候可能会要求输入Coding密码，输入即可！

![](http://mculover666.cn/blog/20191104/7swY51QxLADF.png?imageslim)

执行之后来看看Coding仓库，文件上传成功：

![](http://mculover666.cn/blog/20191104/v6IMQopO3w1z.png?imageslim)

## 开启Page服务

Coding的Page服务需要手动开启：

![](http://mculover666.cn/blog/20191104/fuI1zg5A3WvD.png?imageslim)

![](http://mculover666.cn/blog/20191104/y0EBnQMzy1SG.png?imageslim)

## 访问Coding站点

访问Coding Page中给出的域名即可看到站点：

![](http://mculover666.cn/blog/20191104/ua18oXshiR4t.png?imageslim)

# 4. 自定义域名

## 购买域名

>个人经验

- 非常有必要拥有一个自己的域名，可以做很多事；
- 域名越短越好，要非常容易让别人记住最好；
- 域名.com太贵，尽量买.cn，别买乱七八糟的.xyz，.io啥的，备案不方便；


要绑定域名，首先你得拥有一个域名呀哈哈~

域名可以在各大云厂商买，这里我建议阿里云：

![](http://mculover666.cn/blog/20191104/Nf5I1KnqCQ3e.png?imageslim)

选一个自己喜欢的购买即可，这里我注册的是：`muclover666.top`。

![](http://mculover666.cn/blog/20191104/0UnMXCAgTGcJ.png?imageslim)

## 设置Github自定义域名

首先在阿里云控制台设置域名DNS解析，解析到你的Github站点域名：

![](http://mculover666.cn/blog/20191104/ox0oBVFQ8Czj.png?imageslim)

然后在Hexo本地文件夹的`source`目录下，创建名为`CNAME`文件，不要后缀，写上自定义的域名：

![](http://mculover666.cn/blog/20191104/gGVeqt1wtdOv.png?imageslim)

然后重新生成页面，并部署：
```
hexo clean
hexo g
hexo d
```

等待部署完成后，在Github设置自定义域名：

![](http://mculover666.cn/blog/20191104/dMBsPG2HHpbJ.png?imageslim)

![](http://mculover666.cn/blog/20191104/Qf15PRHNcUEp.png?imageslim)

这样就可以了，访问自己的域名：

![](http://mculover666.cn/blog/20191104/fIFGiTbYD4a2.png?imageslim)

最后再来ping一下，看看域名解析到的地址：

![](http://mculover666.cn/blog/20191104/q8rcW4CP2mxL.png?imageslim)


## 设置Coding自定义域名

>请先在阿里云控制台删除该域名到Github的DNS解析设置！

首先在阿里云控制台设置域名DNS解析，解析到你的Coding站点域名：

![](http://mculover666.cn/blog/20191104/m24RmiXxIu2B.png?imageslim)

在Coding设置自定义域名：

![](http://mculover666.cn/blog/20191104/4cEFxRVvQ2vo.png?imageslim)

![](http://mculover666.cn/blog/20191104/8znxoRDW0kgB.png?imageslim)

这样就可以了，访问自己的域名：

![](http://mculover666.cn/blog/20191104/5SmxagbOXHaG.png?imageslim)

最后再来ping一下，看看域名解析到的地址：

![](http://mculover666.cn/blog/20191104/ochcJN2FesY7.png?imageslim)

# 5. Github + Coding双平台部署并分流解析


>**这样设置后Coding会经常莫名其妙容易挂掉，所以非常不建议这样设置，一心还是不要二用，脚踩两只船，迟早翻船，对吧择Github或者Coding单一平台部署即可!这个了解一下就好**。

2和3中讲述的都是如何部署到单一平台，如何选择呢？

小孩子才做选择，我全都要！

修改一下Hexo站点配置项即可完成双平台部署：

![](http://mculover666.cn/blog/20191104/gOcdqYoGN6NM.png?imageslim)

这样执行`hexo d`部署命令之后，就会将HTML文件同时上传到Github仓库和Coding仓库。

![](http://mculover666.cn/blog/20191104/NqwFyjxzNqIU.png?imageslim)

域名分流解析是指：

- 国内解析到 `mculover666.coding.me`
- 国外解析到 `mculover666.github.io`

这个直接在阿里云DNS解析控制台添加解析时选择国内或国外，即可设置：

![](http://mculover666.cn/blog/20191104/Y6PTodgr0hQH.png?imageslim)








