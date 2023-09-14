---
title: Kafka如何保证顺序消费？
date: 2023-09-14 23:56:26
tags:
categories: [Kafka]
---

## 一、背景
Kafka中一个topic有多个partition，在同一个partition中的消息是有序的。为什么呢？因为Kafka会将消息**<font color=red>追加写**</font>**到指定partition的日志文件中，所以消费消息的话也是按offset顺序消费的。

但是如果多partition的情况下，如何保证全局顺序消费呢？

## 二、如何保证顺序消费？
* **单partition**：前面说到同一个partition中的消息是顺序消费的，所以这种方案是可以保证顺序消费的
    * **缺点**：<font color=red>吞吐量低，不推荐</font>
* **指定场景有序**：比如用户行为、频道信息变更等场景，都是只需要保证同一用户、同一频道的消息可以顺序消费即可，所以我们可以按照用户id、频道id做hash，来指定发送的目标partition
    * **缺点**：可能导致partition消息不平均（比如用户A有100w条消息，用户B只有100条消息），可以适当调大hash的key粒度（比如用户id改为用户id+类型id）