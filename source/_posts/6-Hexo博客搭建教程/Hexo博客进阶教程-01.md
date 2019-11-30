---
title: 【Hexo进阶教程-01】优化文章永久链接为数字编号
tags: Hexo
categories: Hexo
summary: 最详细的Hexo博客搭建教程，没有之一
abbrlink: 3778924399
date: 2019-11-11 14:06:52
---

![](http://mculover666.cn/blog/20191031/R4mWMXsrRKxu.png?imageslim)

<!--more-->

# 基础教程

- [【Hexo-01】本地建立 Hexo 站点](http://www.mculover666.cn/posts/3223194057/)
- [【Hexo-02】创建新文章并生成页面](http://www.mculover666.cn/posts/1228606410/)
- [【Hexo-03】Github + Coding 部署Hexo站点](http://www.mculover666.cn/posts/3223194057/)
- [【Hexo-04】换一个炫酷的响应式主题 —— Matery](http://www.mculover666.cn/posts/3223194057/)

# 1. 待优化问题

Hexo默认使用的文章永久链接格式是：
```bash
year/:month/:day/:title/
```
这种链接，如果遇上个中文标题，简直要爆炸，如下：

![](http://mculover666.cn/blog/20191111/p96q1BfurUDz.png?imageslim)

而且这种中文链接，由于编码的问题，在分享文章链接的时候往往变成：

![](http://mculover666.cn/blog/20191111/V9qcj9xaVHCq.png?imageslim)

就问你难受不难受？

这种方式不仅导致链接变得非常长，而且一旦修改文章发布日期或者标题，链接立马失效，造成大量死链，所以：

不换掉它准备留着过年？

`abbrlink`插件可以帮助我们很好的解决这个问题，Github仓库如下：

- [https://github.com/rozbo/hexo-abbrlink](https://github.com/rozbo/hexo-abbrlink)

# 2. 安装abbrlink插件
在博客根目录（执行hexo命令的地方）安装插件：
```bash
npm install hexo-abbrlink --save
```

![](http://mculover666.cn/blog/20191111/T87GYbyrR0gp.png?imageslim)

# 3. 编辑站点配置文件
打开博客根目录下的`站点配置文件_config.yml`，修改如下配置：
```bash
#permalink: :year/:month/:day/:title/
#permalink_defaults:
permalink: posts/:abbrlink/
abbrlink:
  alg: crc32 #support crc16(default) and crc32
  rep: dec   #support dec(default) and hex
```
下面解释说明一下：

首先指定Hexo文章永久链接的格式，接下来两个是abbrlink插件的设置：
```bash
alg -- Algorithm (currently support crc16 and crc32, which crc16 is default)
rep -- Represent (the generated link could be presented in hex or dec value)
```
这两个设置的示例如下：
```bash
crc16 & hex
https://post.zz173.com/posts/66c8.html

crc16 & dec
https://post.zz173.com/posts/65535.html
```
```bash
crc32 & hex
https://post.zz173.com/posts/8ddf18fb.html

crc32 & dec
https://post.zz173.com/posts/1690090958.html
```
# 4. 重新生成部署
使用`hexo clean && hexo g`重新生成博客，在博客源文件可以看到**自动生成**的abbrlink编号：

![](http://mculover666.cn/blog/20191111/m1qeIgJsAHmx.png?imageslim)

接下来使用`hexo  d`部署，使用浏览器查看：

![](http://mculover666.cn/blog/20191111/mfRypx3keLb9.png?imageslim)

这样优化后的优点有以下几个：

- 只要abbrlink编号不变，该文章的url就不变，可以随意修改文件名，文章标题；
- 纯英文和数字链接非常有利于SEO；
