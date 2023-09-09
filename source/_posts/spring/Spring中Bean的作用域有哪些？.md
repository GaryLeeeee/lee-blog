---
title: Spring中Bean的作用域有哪些？
date: 2023-09-08 14:19:55
tags:
categories: [Spring]
---

## 一、Spring中Bean的作用域
Spring中支持以下五种Bean的作用域：
* **singleton（默认）**：单例，也就是Spring容器中只会有一个Bean实例
* **prototype**：每次获取都会创建一个新的Bean实例
* **request**：每次HTTP请求都会创建一个新的Bean实例
* **session**：同一次HTTP session中共享同一个Bean实例
* **application**：全局HTTP session中共享同一个Bean实例

## 二、Q&A
### 1、如何保障线程安全？
**背景**：Spring中Bean的默认作用域是singleton，多个线程访问同一个Bean时会存在线程不安全问题
**解决**：在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在ThreadLocal中（具体可参考[《ThreadLocal是什么？实现原理呢》](https://garyleeeee.github.io/2023/09/08/concurrent/threadlocal-shi-shi-me-shi-xian-yuan-li-ni/)）