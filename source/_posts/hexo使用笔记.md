---
title: hexo使用笔记
date: 2021-07-31 14:37:45
tags: 学习笔记
categories: hexo
---

## 1.前提工作
* github(本文针对github，其他如gitee类似)
* node.js、npm
* git客户端
* hexo(本文会介绍怎么安装)

## 2.搭建博客项目(h5)
* 创建github仓库
新建一个名为 **{用户名}.github.io**的仓库
  
* 在settings-pages可以看到博客部署的地址
  ![](https://lee-blog-picture.oss-cn-shenzhen.aliyuncs.com/github%20page.png)
  
## 3.如何使用hexo
### 3.1 install(安装)
`npm install -g hexo`
  
### 3.2 hexo init(初始化)
随便找一个目录放代码(后面发博客内容等需要修改) 如d:/hexo
并初始化为hexo项目
`cd d:/hexo`    
`hexo init`
这时候会自动生成一些文件
  ![hexo init](https://lee-blog-picture.oss-cn-shenzhen.aliyuncs.com/hexo-init.png)

### 3.3 hexo g(生成)
hexo g是hexo generate的缩写，用于在public文件夹生成最终的文件(.md文件会被生成.html文件)
![hexo g](https://lee-blog-picture.oss-cn-shenzhen.aliyuncs.com/hexo-g.png)

### 3.4 hexo s(启动服务)
hexo s是hexo server的缩写，用于本地启动服务，可以通过http://localhost:4000访问
![hexo s](https://lee-blog-picture.oss-cn-shenzhen.aliyuncs.com/hexo-s.png)

### 3.5 重要目录/文件的作用
|文件夹|作用|
|-------------|-------------|
|source|存放静态文件的文件夹(包括md、图片等)|
|themes|模板库(可以网上下载到这里)|
|_config.yml|全局配置(如username等)|

  
## 4.使用主题并写博客
一般我们下载网上的主题，就可以直接修改配置文件(_config.yml)来自定义页面内容
* 找一个主题并下载到`/themes`目录下
比如我的主题是https://github.com/blinkfox/hexo-theme-matery
  
* 修改`_config.yml`的`theme: landscape`为`theme: matery`
  
## 5.如何部署到远程{username.github.io}
### 5.1 配置git账户信息
deploy:
  type: git
  repository: git@github.com:GaryLeeeee/garyleeeee.github.io.git
  branch: master

### 5.2 安装deployer插件
`npm install hexo-deployer-git --save`
如果没安装的话`hexo d`一般会报错"Deployer not found: github"或者"Deployer not found: git"

### 5.3 hexo d(部署)
hexo d是hexo doploy的缩写，用来部署服务到远程

## 6.如何写博客
### 6.1 hexo new xxx(新建博客)
通过hexo new 'my-first-blog'即可创建一个名字为my-first-blog的微博文件
![](http://image.liuxianan.com/201608/20160823_183325_470_9306.png)

### 6.2 hexo new page xxx(新建页面)
与new不同的是他是生成一个页面，并不会以一个博客显示在目录中

### 6.3 博客配置
|属性|作用|
|-------------|-------------|
|title|博客名|
|date|创建日期|
|tags|标签名(后面有讲)|
|categories|分类名(后面有讲)|

## 7.常见问题
### matery主题如果没有tags/categories/about页面？
新增标签tags页面(categories/about类似)
`tags`页是用来展示所有标签的页面，如果在你的博客`source`目录下还没有`tags/index.md`文件，那么你就需要新建一个，命令如下：
>hexo new page "tags"

并且在生成的md文件中加上type和layout，如下
>title: tags
date: 2021-07-31 16:40:30
type: "tags"
layout: "tags"

### 部署远程后没更新
删除本地.deploy_git目录，重新执行`hexo d -g`

