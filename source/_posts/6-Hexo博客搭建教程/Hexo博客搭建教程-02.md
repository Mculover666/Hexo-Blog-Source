---
title: 【Hexo-02】创建新文章并生成页面
tags: Hexo
categories: Hexo
summary: 最详细的Hexo博客搭建教程，没有之一
abbrlink: 1228606410
date: 2019-11-01 10:06:52
---

![](http://mculover666.cn/blog/20191031/R4mWMXsrRKxu.png?imageslim)

<!--more-->

# 专栏回顾

-[【Hexo-01】本地建立 Hexo 站点](http://www.mculover666.cn/posts/3223194057/)

# 创建新文章

在网站根目录下打开命令行，使用如下命令创建新文章：
```
hexo new <title>
```

![mark](http://mculover666.cn/blog/20191101/sYuMstaKkVSx.png?imageslim)

执行该命令，Hexo会在`/source/_posts`目录下创建一篇新的文章：

![mark](http://mculover666.cn/blog/20191101/R0uxfYA2K3G0.png?imageslim)

接下来在这篇文章里使用 MarkDown 语法编写文章即可。

# Front-matter

打开 Hexo 创建的文章可以看到，文章开头有这样一段：

![mark](http://mculover666.cn/blog/20191101/onwM2cboNJ7G.png?imageslim)

这个使用`---`包括起来的内容称之为`Front-matter`，即前置信息，用于给 Hexo 渲染该 md 文档，除了这三项，还有很多的配置项可以自己添加：

|配置项|意义|
|:-:|:-:|
|title|网页文章标题|
|date|文章创建如期|
|comments|文章评论功能是否启动|
|tags|文章标签|
|categories|文章分类|
|keywords|文章关键字|

这里我用一张图来说明如何使用：

![mark](http://mculover666.cn/blog/20191101/Frt5YATy4uCc.png?imageslim)

# MarkDown内容

Hexo 支持大多数的 MD 语法，如果对MD语法还不熟悉，可以查看该教程：

- [菜鸟教程-Markdown 教程](https://www.runoob.com/markdown/md-tutorial.html)

![mark](http://mculover666.cn/blog/20191101/Dk5GMS4QsPKr.png?imageslim)

这里我编写一些简单的内容，作为测试使用：

```
# 一级标题

代码测试：
\```py
print("Hello")
\```

注意：这里因为我放在md文件中的，所以加上了\，不解析```，实际测试时请去掉\。

图片测试：

![](http://mculover666.cn/blog/20191031/R4mWMXsrRKxu.png?imageslim)

引用测试：

>这是一条引用

## 二级标题

无序列表测试：

- 哈哈
- 嘿嘿
- 吼吼

### 三级标题

#### 四级标题
```


# 生成文章

文章写好之后，首先清除掉旧的数据：

```
hexo clean
```
这个命令会清除掉之前生成的网页，即站点根目录下的`public`文件夹：

![mark](http://mculover666.cn/blog/20191101/GqpGTOJNTC2H.png?imageslim)

然后使用如下命令生成新的页面：

```
hexo g
```

这个命令会将`source`文件夹下所有的md文件进行渲染，生成HTML页面，存放在`public`文件夹下：

![mark](http://mculover666.cn/blog/20191101/wcBOvAUO1fz3.png?imageslim)

![mark](http://mculover666.cn/blog/20191101/nLoYlk1g66u8.png?imageslim)

>特别提醒：
每次修改文章后，都要执行这两条命令，清除掉旧的数据，然后重新生成页面。

# 预览文章

重新生成页面后，我们可以在本地开启服务器，预览一下文章是否满意：

```
hexo s
```

![mark](http://mculover666.cn/blog/20191101/ET589FlJg8Xp.png?imageslim)

![mark](http://mculover666.cn/blog/20191101/EMFGnhgs9xQG.png?imageslim)

好啦，今天的教程就到这儿，快去上手试试吧~







