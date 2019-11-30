---
title: 【Hexo-04】换一个炫酷的响应式主题 —— Matery
tags: Hexo
categories: Hexo
summary: 最详细的Hexo博客搭建教程，没有之一
abbrlink: 853951676
date: 2019-11-04 10:06:52
---

![](http://mculover666.cn/blog/20191031/R4mWMXsrRKxu.png?imageslim)

<!--more-->

# 专栏回顾

- [【Hexo-01】本地建立 Hexo 站点](http://www.mculover666.cn/posts/3223194057/)
- [【Hexo-02】创建新文章并生成页面](http://www.mculover666.cn/posts/1228606410/)
- [【Hexo-03】Github + Coding 部署Hexo站点]()

# Matery主题

[Hexo官方站点](https://hexo.io/themes/)上有非常多的主题，可以选择一套自己喜欢的，并且这些主题都是开源的，基本参考readme文档就可以完成更换，如果你懂点前端知识的话，还可以进行修改，贡献代码。

这里我使用的 Matery 主题，是由`blinkfox`大佬开发的一款的响应式主题，具有以下特色：

- 首页轮播文章展示及每天动态更换 Banner 图片
- 瀑布流式的博客文章列表
- 时间轴式的归档页
- 词云的标签页和雷达图的分类页
- 支持在首页的音乐播放和视频播放功能
- ……

推荐这款主题的原因是因为：

**需要的功能它都有，不需要的它也有，很爽**！

好了，废话不多说，开始更换主题！

# 1. 下载Metery主题到本地

Hexo的所有主题源代码都是托管在Github的，更换主题第一步：**将该主题的源代码clone下来，放到本地Hexo站点根目录下的`themes`文件夹中**。

访问Metery主题的[Github仓库](https://github.com/blinkfox/hexo-theme-matery),复制仓库地址：

![](http://mculover666.cn/blog/20191105/nJCU2COHCI9p.png?imageslim)

然后在本地**站点根目录**打开git bash 命令行，进入`themes`文件夹，开始拉取代码到本地：

```
cd themes
git clone https://github.com/blinkfox/hexo-theme-matery.git
```

![](http://mculover666.cn/blog/20191105/NiNRwfQokdbk.png?imageslim)

![](http://mculover666.cn/blog/20191105/Ep7UeuiqxAJw.png?imageslim)

因为Hexo的部署是使用git推送的，这里我们不使用git子模块的方式，直接将主题文件的git仓库删除，可以避免很多问题：

![](http://mculover666.cn/blog/20191105/E5YjxB96Rzwu.png?imageslim)

# 2. 更换metery主题

下载Hexo的主题到`/themes`文件夹之后，要在**站点配置文件**中配置使用该主题：

![](http://mculover666.cn/blog/20191105/B1DHkEAzPqxO.png?imageslim)

这样就更换成功啦！清除生成部署三连，看看效果：
```
hexo clean
hexo g
hexo d
```

对于该主题，为了更好的显示效果，建议在**站点配置文件**中修改如下配置：

- 修改站点主url：

![](http://mculover666.cn/blog/20191105/TUwxAlNAOwNM.png?imageslim)

- 修改每页显示文章数目为6的倍数：

![](http://mculover666.cn/blog/20191105/3IQCSHvaJrKq.png?imageslim)

然后就是见证奇迹的时候了，访问站点看一下效果：

![](http://mculover666.cn/blog/20191105/ivpLYYeYqVCO.png?imageslim)

![](http://mculover666.cn/blog/20191105/MdAjt0P4zVrY.png?imageslim)

>如果碰到主页可以正常访问，但是文章访问不了的情况，应该是最近Metery主题的问题，我的解决方案是，使用命令`npm i --save hexo-wordcount`安装这个插件，然后重新清除生成部署三连即可。

![](http://mculover666.cn/blog/20191105/3o6lugbkbFbD.png?imageslim)

![](http://mculover666.cn/blog/20191105/4IXGt4FGwEGq.png?imageslim)

# 3. 配置metery主题

对于该主题的自定义配置修改，需要修改主题文件夹中的`_config.yml`文件，称之为**主题配置文件**：

![](http://mculover666.cn/blog/20191105/jFLupojDcmER.png?imageslim)

关于这些自定义配置，可以参考作者的文档，我差不多都跟着做了，没问题的，500多行的配置文件，我是真滴没法讲解，望大家谅解^_^：

>https://github.com/blinkfox/hexo-theme-matery/blob/develop/README_CN.md


这里我来说说最重要的一点 —— **评论系统**。

这款主题支持几乎所有主流的评论系统，如图：

![](http://mculover666.cn/blog/20191105/BwJxKkkWo528.png?imageslim)

这些评论我都试过来了，最终得出以下经验：

- gittalk是利用了github的issue功能，加载速度快，评论需要登录，方便管理，只支持Github登录；
- 来必力界面好看，评论时支持各种账号登录，有邮件通知，缺点是加载超级慢的；

gittalk的效果可以看主题作者的博客：

- https://blinkfox.github.io/

来必力的效果可以看我的博客效果：

- http://www.mculover666.cn/

评论系统就从这两个之间选吧，别的可以不用看哈哈~

关于文章永久链接也可以不用修改，后面我会在Hexo进阶教程中讲述如何设置链接为除数字编号，希望大家玩的愉快~

另外，大家如果搭出来，可以在我的博客留言板上留言，我添加友情链接到我的博客上，也欢迎互加友链，谢谢呀！


