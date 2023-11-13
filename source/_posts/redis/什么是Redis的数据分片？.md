---
title: 什么是Redis的数据分片？
date: 2023-11-13 17:21:40
tags:
categories: [Redis]
---

## 一、什么是Redis的数据分片？
**Redis的数据分片（Sharding）**指的是将数据集分为多个子集，分别存储在不同的Redis节点上，提高了Redis集群的性能和扩展性。

## 二、如何实现Redis的数据分片？
**Redis的数据分片**实现一般是按照某种规则（如取模）分配到不同的节点上，当客户端要访问某个key时的步骤如下：
1. 客户端通过规则（如取模）计算该key位于哪个Redis节点中
2. 客户端连接该节点并执行命令

### 1、Redis Cluster
在Redis集群模式中，**Redis Cluster**就是用了客户端分片（即哈希槽hash slot）的方式进行数据切片，将数据集分为多个槽，一个Redis节点分配多个槽。客户端访问某个key时，就是根据Redis Cluster内置的路由规则计算到对应的槽，然后就可以连接访问了。

另外，Redis Cluster还提供了故障转移、在线扩容等功能，具体可参考文章[《Redis集群》](https://garyleeeee.github.io/2023/08/08/redis/redis-ji-qun/)。

## 三、Q&A
### 1、除了Redis Cluster的客户端分片，还有其他数据分片方式吗？
除了Redis Cluster的客户端分片，还有**服务端分片**。

**服务端分片**指的是使用代理层来进行维护我们的路由规则，客户端直接跟代理层通信即可，然后代理层根据路由规则计算好数据整合返回给客户端。