---
title: Kafka中Partition和Consumer数量应该怎么分配？
date: 2023-10-02 21:19:38
tags:
categories: [Kafka]
---

## 一、背景
Kafka中各个概念关系如下：
* 一个Topic有多个Partition
* 一个Consumer Group有多个Consumer
* 一个Consumer可以监听多个Partition

## 二、Kafka中Partition和Consumer数量应该怎么分配？
Kafka中Partition和Consumer分配情况可以大概分为下面几种：
* **Partition多，Consumer少**：意味着其中一个或多个Consumer要负责多个Partition，会出现消费不平衡的情况（<font color=red>解决方案：将Partition数设置为Consumer数的整数倍，保证每个Consumer分配到相等的Partition数，如Partition数为8，Consumer数为4</font>）
* **Partition少，Consumer多**：会出现部分Consumer没有分配到Partition而造成浪费
* **Partition和Consumer相等**：可以保证Consumer消费的消息是有序的（因为单个Partition下是有序的，但不能保证多个Partition是全局有序的）