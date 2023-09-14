---
title: Kafka如何保证数据不丢失？
date: 2023-09-14 17:03:06
tags:
categories: [Kafka]
---

## 一、Kafka中数据是怎么传输的？
Kafka中数据传输分为三步：
* Producer端将消息发送给Broker
* Broker端将消息进行同步并持久化
* Consumer端从Broker将消息拉取并消费

## 二、Kafka如何保证数据不丢失？
因为Kafka中数据是分为三步传输的，所以我们也需要分别从这三步里去分析如何保证数据不丢失。

### 1、Producer
* **同步发送+重试**：异步发送改为同步发送，监听callback回调，并进行重试
* **acks机制**：配置acks机制，保证所有副本都同步完数据才算成功
    * acks=0：表示Producer不需要等Broker返回确认就认为发送成功，<font color=red>会存在消息丢失</font>
    * acks=1：表示Producer会等Leader副本返回确认就认为发送成功（不需要等Follower副本同步完成），如果这时候Leader副本挂了，<font color=red>会存在消息丢失</font>
    * acks=-1：表示Producer会等Leader副本和ISR列表里的Follower副本都同步完成才认为发送成功，<font color=red>不存在消息丢失，但是性能比较差</font>
    
相关配置参数如下：
```yaml
acks: -1                // 确认机制
retries: 3              // Producer重试次数
retry.backoff.ms: 300  // 消息发送超时或失败后的间隔重试时间
```

### 2、Broker
* **ISR机制**：ISR列表指同步进度较好的副本（包括Leader副本和Follower副本），每个partition都有多个副本分布在不同节点上，当一个节点宕机时，其他节点的副本仍然可以提供服务，保证数据不丢失
* **持久化存储**：Kafka会将消息写入磁盘，防止节点宕机导致数据丢失（为了保证性能，Kafka采用了异步批量刷盘机制，即达到一定数据量或时间间隔才会刷盘）

相关配置参数如下：
```yaml
replication.factor: 3                  // 表示分区副本数量（一般大于1才能保证Leader副本宕机后能正常选举新的Leader副本）
min.insync.replices: 3                 // 表示ISR最少的副本数量
unclean.leader.election.enable: false  // 是否可选举非ISR列表中的副本做Leader副本
```

### 3、Consumer
* **手动提交offset**：关闭自动提交offset功能，防止消费者还没正常消费完就提交了offset，导致数据丢失

相关配置参数如下：
```yaml
enable.auto.commit: false // 不自动提交offset
```