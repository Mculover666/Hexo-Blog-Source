---
title: 【Hexo进阶教程-02】使用Appveyor备份并持续集成博客
tags: Hexo
categories: Hexo
summary: 最详细的Hexo博客搭建教程，没有之一
abbrlink: 3365570122
date: 2019-11-12 14:06:52
---

![](http://mculover666.cn/blog/20191031/R4mWMXsrRKxu.png?imageslim)

<!--more-->

# 教程汇总

- [【Hexo基础教程-01】本地建立 Hexo 站点](http://www.mculover666.cn/posts/3223194057/)
- [【Hexo基础教程-02】创建新文章并生成页面](http://www.mculover666.cn/posts/1228606410/)
- [【Hexo基础教程-03】Github + Coding 部署Hexo站点](http://www.mculover666.cn/posts/3223194057/)
- [【Hexo基础教程-04】换一个炫酷的响应式主题 —— Matery](http://www.mculover666.cn/posts/3223194057/)

- [【Hexo进阶教程-01】]()

# 1. 待优化问题

使用`hexo d`命令部署Hexo博客时，在Github仓库上传的只是 public 文件夹中生成的页面内容，这样就带来了一些问题：

- 本地博客文章的md源文件没有备份，哪天硬盘挂了可咋整？
- 博客是在家里的电脑上更新发布的，换到公司的电脑上想发布一篇，咋整？

别慌，小问题，**用持续集成服务**就可以解决这些问题，先放上一张使用持续集成服务之后的整体架构图：

![](http://mculover666.cn/blog/20191112/nfH4OPHoIYBV.png?imageslim)

看不懂不要紧，这篇文章的目的就是让你看懂这张图！

# 2.什么是持续集成服务
持续集成服务的全名叫做`Continuous Integration`，简称 CI，它其实是一个云端自动化构建（编译）代码的服务。

**它绑定 Github 上面的项目，只要有新的代码，就会自动抓取。然后，提供一个运行环境，执行测试，完成构建，还能部署到服务器。**

C语言的源文件是`.c`文件，使用`gcc`命令编译出`.out`可执行文件，这个过程称为编译构建。

同样可以类比一下，Hexo博客的源文件是`.md`文件，使用`hexo g`命令即可生成`html`页面，这个过程也可以叫做编译构建。

那么，这样的一个云端自动化构建服务，为什么称为持续集成呢？

因为Github仓库中的代码只要有一点点变更，该服务就会自动运行构建和测试，反馈运行结果，确保符合预期以后，再将新代码"集成"到主干，所以该服务称为“持续”“集成”。提供持续集成服务的工具非常多，因为大多数用户都是在Windows下，所以在本文中我们使用持续集成服务工具[appveyor](https://www.appveyor.com/)。

接下来进行一个简单的分析，如何将持续集成服务应用到Hexo博客上？

首先分析一下本地Hexo的目录：

![](http://mculover666.cn/blog/20191112/abiUmkeCqn4u.png?imageslim)

可以在Github新建一个仓库，作为Hexo源文件的备份仓库，将上图中标示出需要备份的文件上传，这样就达到了备份的目的。

然后对该仓库编写脚本进行持续集成：

## 在云端建立环境的脚本代码

- 在windows下安装nodejs环境；
- 安装hexo博客框架；
- 安装nodejs依赖模块；
- 安装hexo插件（如果有的话，比如abbrlink插件）；

## 在云端进行构建的脚本代码

- 执行`hexo d`命令生成HTML页面，即public文件夹；

## 在云端部署HTML页面

- 将public文件夹部署到Hexo站点仓库；

# 3. 上传Hexo博客源码到新的备份仓库

在Github上新建一个远程仓库：

![](http://mculover666.cn/blog/20191112/Pp7bHJXqbMVq.png?imageslim)


然后在站点根目录**初始化本地仓库**，并且与远程仓库关联起来
```
git init
git remote add origin https://github.com/Mculover666/Hexo-Blog-Source.git
```

![](http://mculover666.cn/blog/20191112/1d7WS3zTcIdo.png?imageslim)

因为不需要备份所有文件，所以修改`.gitignore`文件来说明忽略的文件和文件夹，通常需要备份的文件有：

- source文件夹的所有内容
- scaffolds文件夹的所有内容
- 站点配置文件：`_config.yml`
- themes文件夹的所有内容

综上，`.gitignore`文件的内容如下：
```
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
public/
package-lock.json
```

![](http://mculover666.cn/blog/20191112/LijA0jw771Fu.png?imageslim)

然后执行如下git命令开始推送：
```
git add .
git commit -m "hexo source"
git push origin master
```

# 4. 使用AppVeyor建立CI

访问[AppVeyor登陆页面](https://ci.appveyor.com/login)，使用`GitHub账号`登陆即可：

![](http://mculover666.cn/blog/20191112/afzfqvDtXja0.png?imageslim)

然后创建新的项目：

![](http://mculover666.cn/blog/20191112/XeUy5LOGvlAy.png?imageslim)

选择需要绑定的Github仓库：

![](http://mculover666.cn/blog/20191112/3fW0wUq3t1g7.png?imageslim)

# 5.新建Access Token并加密

因为Appveyor需要向Github上的仓库提交文件，所以需要在Github生成一个token给appveyor，可是该脚本是公开的，肯定不能直接把token写进去，所以appveyor提供了一个加密的功能，可以将加密后的token放到脚本里公开。

首先到Github生成`Access Token`，生成方法可以参考：

- [Creating a personal access token for the command line](https://help.github.com/en/articles/creating-a-personal-access-token-for-the-command-line)。

在GitHub生成好`Access Token`之后，在[AppVeyor加密页面](https://ci.appveyor.com/tools/encrypt)把Access Token加密之后再替换`Your GitHub Access Token`：

![](https://img-blog.csdnimg.cn/20190709092050268.png)

# 6. 添加自动化构建文件

创建appveyor项目的时候绑定了GIthub仓库，所以需要在该仓库中存放`appveyor.yml`脚本文件，一旦该仓库有变化，就会执行该脚本的内容：

在源文件中手动添加`appveyor.yml`文件：

![](http://mculover666.cn/blog/20191112/YHF5gweUaMv0.png?imageslim)

该文件的内容如下：

>建议直接复制过去，这些文件内容中只需要替换`Your GitHub Access Token`为第4步中生成并加密的token即可，不需要搞懂！

```
clone_depth: 5
environment:
access_token:
    secure: [Your GitHub Access Token]
install:
	- node --version
	- npm --version
	- npm install
	- npm install hexo-cli -g
build_script:
	- hexo generate
artifacts:
	- path: public
on_success:
	- git config --global credential.helper store
	- ps: Add-Content "$env:USERPROFILE\.git-credentials" "https://$($env:access_token):x-oauth-basic@github.com`n"
	- git config --global user.email "%GIT_USER_EMAIL%"
	- git config --global user.name "%GIT_USER_NAME%"
	- git clone --depth 5 -q --branch=%TARGET_BRANCH% %STATIC_SITE_REPO% %TEMP%\static-site
	- cd %TEMP%\static-site
	- del * /f /q
	- for /d %%p IN (*) do rmdir "%%p" /s /q
	- SETLOCAL EnableDelayedExpansion & robocopy "%APPVEYOR_BUILD_FOLDER%\public" "%TEMP%\static-site" /e & IF !ERRORLEVEL! EQU 1 (exit 0) ELSE (IF !ERRORLEVEL! EQU 3 (exit 0) ELSE (exit 1))
	- git add -A
	- if "%APPVEYOR_REPO_BRANCH%"=="master" if not defined APPVEYOR_PULL_REQUEST_NUMBER (git diff --quiet --exit-code --cached || git commit -m "Update Static Site" && git push origin %TARGET_BRANCH% && appveyor AddMessage "Static Site Updated")
```

>如果有安装了插件，比如abbrlink，可以在install部分中现有的命令之后继续添加，如下是我的配置（举个例子，说明如何添加）：

![](http://mculover666.cn/blog/20191112/u8TO4YWnxCcu.png?imageslim)

# 7. 设置Appveyor环境变量
添加好`appveyor.yml`之后，再到Appveyor portal添加以下四个变量：

- STATIC_SITE_REPO：博客站点Github仓库地址；
- TARGET_BRANCH：博客站点仓库的branch（默认是master）
- GIT_USER_EMAIL：GitHub账号的信息；
- GIT_USER_NAME：GitHub账号的信息；

![](https://img-blog.csdnimg.cn/20190707203912918.png)

# 8. 完成
至此配置完成，以后只需要执行：`git push`将源代码上传到仓库后，Appveyor就会检测到变化，然后自动完成推送和部署。

## 执行推送
首先暂存所有更改并提交：
```bash
git add -A
git commit -m "first"
```
然后推送到远程仓库：
```bash
git push origin master
```

![](http://mculover666.cn/blog/20191112/rcjI3C3bvKR6.png?imageslim)

## 观察自动化脚本运行情况
登录Appveyor网站，在`current build`中即可看到当前构建情况：

![](http://mculover666.cn/blog/20191112/UTcukwd6s7DE.png?imageslim)

![](http://mculover666.cn/blog/20191112/HkWDRa3dLy0A.png?imageslim)

可以看到自动化脚本运行成功，站点部署成功，可以再去看看博客站点仓库是否更新。

![](http://mculover666.cn/blog/20191112/XVzw7jiTM8Jt.png?imageslim)

# 9. 关于更换电脑的操作

使用自动构建服务后，**Hexo站点仓库(mculover666.github.io)和我们没关系了，我们只需要管理Hexo源码仓库(Hexo-Blog-Source)即可**。

**在写好一篇文章或者做了任何修改之后，不需要进行任何操作，只需`git push`到源码仓库**，自动构建服务会自动检测到修改，然后生成页面，部署页面到站点仓库，是不是很方便呢？ 

所以，更换电脑之后，首先将`Hexo源码`仓库拉取下来，然后修改或者添加新的文章进去，最后`git push`到源码仓库，ok！剩下的一堆事情，交由自动构建服务去做吧~

整个系统的架构可以调整为：

![](http://mculover666.cn/blog/20191112/NyPUDxW2IT5n.png?imageslim)






