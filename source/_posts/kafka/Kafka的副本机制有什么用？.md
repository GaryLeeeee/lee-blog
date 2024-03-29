---
title: Kafka的副本机制有什么用？
date: 2023-09-14 15:53:20
tags: [副本机制]
categories: [Kafka]
---

## 一、什么是Kafka的副本机制？
Kafka的副本机制，也就是备份机制，指在分布式集群中每个节点都同步了相同的数据备份。

## 二、Kafka的副本机制有什么用？
Kafka的副本机制是保证消息可靠性的重要手段之一，它可以：
* **提高可用性**：每个topic有多个partition，每个partition有多个副本，但其中只能有一个是leader副本，不像MySQL一样，Kafka只有leader副本才可以处理读写请求，follower副本只是被动同步leader副本数据，并不会处理读请求（<font color=red>Kafka 2.4开始可以通过配置参数允许follower副本提供读服务</font>）
* **提高容错性**：能够防止数据丢失，在写入成功后，只有当数据被所有副本写入磁盘后，才认为才消息已成功提交。如果出现某个节点故障不能正常同步，Kafka会自动重试发送该消息，直到超时或成功为止