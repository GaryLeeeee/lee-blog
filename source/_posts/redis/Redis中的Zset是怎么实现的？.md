---
title: Redis中的Zset是怎么实现的？
date: 2023-09-10 00:15:40
tags: [跳表]
categories: [Redis]
---

## 一、什么是Zset？
Zset（也叫Sorted Set）是Redis中的一种数据结构，支持传member和score两个字段，同时支持按照score排序。

**常见命令**：
* **添加成员**：`zadd <key> <score> <member>`
* **删除成员**：`zrem <key> <member>`
* **查询成员数**：`zcard <key>`
* **增量加分**：`zincrby <key> <increment> <member>`
* **范围查询**：`zrange <key> <start> <stop> [withscores]`
* **查询指定成员排名**：`zrank <key> <member>`
* **查询指定成员分数**：`zscore <key> <member>`
* ...

**使用场景**：
* **排行榜**：把用户分数存到score，可以按照分数高低查询（如TopN）
* **延迟队列**：把延迟队列的执行时间戳存到score，可以轮询取出历史延迟队列并消费

## 二、Zset底层实现
Zset底层实现一般分为跳表（skiplist）和压缩列表（ziplist），但是在Redis 7.0后就不再使用ziplist了，而是用紧凑列表（listpack）替代。

### 1、使用场景
* 当Zset数据量比较少时，Redis采用ziplist/listpack实现（通过连续存储元素来节约内存空间）
* 当Zset数据量比较多时，Redis自动将ziplist/listpack转换为skiplist实现（保证元素的有序性和支持范围查询操作）

### 2、跳表（skiplist）
skiplist是一种基于并联链表的数据结构，实现简单，插入、删除、查找的复杂度均为O(logN)
//TODO：实现



