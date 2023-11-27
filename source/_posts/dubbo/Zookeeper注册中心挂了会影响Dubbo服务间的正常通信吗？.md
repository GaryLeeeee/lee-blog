---
title: Zookeeper注册中心挂了会影响Dubbo服务间的正常通信吗？
date: 2023-11-27 21:49:55
tags: [Zookeeper]
categories: [Dubbo]
---

## 一、背景
Dubbo服务在启动时都会从Zookeeper注册中心同步一份消费者/生产者的连接信息到本地，并通过Watch机制更新（如其中一台消费者挂了，就会通知生产者更新本地缓存的连接信息）。

## 二、 Zookeeper注册中心挂了会影响Dubbo服务间的正常通信吗？
答案是**不会**的。

从上面的背景上可以知道，我们的Dubbo服务本地还缓存着一份消费者/生产者的连接信息，当Zookeeper注册中心挂了，Dubbo服务仍然可以拿到本地缓存的连接信息进行访问（<font color=red>Zookeeper注册中心只是一个配置中心，并不充当Dubbo服务之间通信的媒介</font>）。

## 三、Q&A
### 1、如果Zookeeper注册中心挂了之后，其中一个Dubbo生产者也挂了会发生什么？
首先Dubbo生产者是无状态的，如果Dubbo消费者拿到了已经挂了的Dubbo生产者连接信息，那么会出现连接不通的问题，这时候就会继续查找下一个Dubbo生产者直到可用（如果Dubbo生产者全挂了，那么Dubbo消费者会无限次重连直到Dubbo生产者恢复）。