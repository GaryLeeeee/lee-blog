---
title: Vue项目创建方式
date: 2022-03-02 22:52:12
tags: [vue]
categories: [vue]
---

## 一、相关链接
参考链接：https://blog.csdn.net/ccf19881030/article/details/105358242

## 二、创建步骤
### 1.全局安装vue-cli脚手架
```shell
npm install vue-cli -g
```

### 2.开始创建项目
* 使用vue初始化基于webpack的新项目
```shell
vue init webpack test-project
```
创建过程中会提示是否安装eslint，可以跳过不安装(否则项目编译过程中会出现各种代码格式的问题)

* 创建完成后，安装基础模块
```shell
cd test-project
npm install
```

* 安装完成后，可在开发模式下运行项目并预览项目效果
```shell
npm run dev
```

![运行成功](/images/Vue项目创建方式1.png)

* 访问http://localhost:8080
![项目效果](/images/Vue项目创建方式2.png)