---
title: Redis有哪几种常用的缓存读写策略？
date: 2023-11-08 16:41:09
tags:
categories: [Redis]
---

## 一、Redis有哪几种常见的缓存读写策略？
在Redis中，有3种常见的缓存读写策略，分别为：
* **Cache Aside Pattern（旁路缓存模式）**：同时更新数据库和缓存
* **Read/Write Through Pattern（读写穿透）**：只更新缓存，由缓存去同步更新缓存
* **Write Behind Pattern（异步缓存写入）**：只更新缓存，由缓存去异步更新缓存

### 1、Cache Aside Pattern（旁路缓存模式）
**介绍**：同时更新数据库和缓存
**适用场景**：适用读多写少的场景

**写场景步骤**：
1. 先更新数据库
2. 删除缓存

**读场景步骤**：
1. 先查询缓存，存在则直接返回
2. 不存在则查询数据库并返回
3. 更新缓存

不过这种模式在极端情况下可能会出现数据不一致，具体可参考文章[《Redis和MySQL如何保证数据一致性？》](https://garyleeeee.github.io/2023/07/23/redis-he-mysql-ru-he-bao-zheng-shu-ju-yi-zhi-xing/)
### 2、Read/Write Through Pattern（读写穿透）
**介绍**：通过缓存交互，客户端读写都是直接请求缓存即可，再由缓存跟数据库做数据同步
**适用场景**：适用读多写少、数据一致性不高的场景

**写场景步骤**：
1. 判断缓存是否存在，不存在则直接更新数据库
2. 如果缓存存在，则先更新缓存，再由缓存去更新数据库

**读场景步骤**：
1. 先查询缓存，存在则直接返回
2. 不存在则查询数据库并写到缓存
3. 返回数据


### 3、Write Behind Pattern（异步缓存写入）
**定义**：跟**Read/Write Through Pattern（读写穿透）**类似，但不同的是**Write Behind Pattern（异步缓存写入）**是异步更新数据库，而**Read/Write Through Pattern（读写穿透）**是同步更新数据库
**适用场景**：适用数据变化频繁、数据一致性不高的场景，如点赞量、浏览量