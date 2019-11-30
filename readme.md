# 简介
该仓库是我的个人博客本地所有文件，作以备份。

博客地址：[www.mculover666.cn](http://www.mculover666.cn)

# 注意
该仓库中的文件不包含hexo的安装文件。

# 1.安装
## git
[官网下载安装](https://git-scm.com)

## nodejs
[官网下载安装](https://nodejs.org)

## hexo
```
npm install -g hexo-cli
```

# 2.初始化博客文件夹
```
hexo init <name>
cd <name>
npm install
```
# 3.安装需要的插件
## RSS
```
npm install hexo-generator-feed --save
```
## Localsearch
```
npm install hexo-generator-searchdb --save
```
# 4.使用
```
hexo new <layout> "<title>"
hexo clean
hexo g
hexo s
hexo d
```