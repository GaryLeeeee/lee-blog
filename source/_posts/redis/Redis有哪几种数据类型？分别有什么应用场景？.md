---
title: Redis有哪几种数据类型？分别有什么应用场景？
date: 2023-11-07 17:17:07
tags:
categories: [Redis]
---

## 一、Redis有哪几种数据类型？
Redis常见的数据类型有以下几种：
* **String（字符串）**：一般用于缓存JSON对象
* **List（列表）**：有序、可重复
* **Set（集合）**：无序、不重复
* **Hash（散列）**：
* **Zset（有序集合）**：有序、不重复
* **HyperLogLog（基数统计）**
* **Bitmap（位图）**
* **Geospatial（地理位置）**
* ...

## 二、Redis的数据类型分别有什么应用场景？
**String（字符串）**：
* **分布式锁**：具体可参考文章[《如何实现一个分布式锁？》](https://garyleeeee.github.io/2023/09/05/concurrent/ru-he-shi-xian-yi-ge-fen-bu-shi-suo/)
* **单一对象缓存**：存储单一对象用（如商品信息的key为商品id，value为商品信息）
* **计数**：通过`incr`命令实现原子性自增，可用于如文章阅读量等计数场景
---

**List（列表）**：
* **消息队列**：可以通过`LPUSH`、`RPOP`实现消息队列的入队和出队，也可以通过`BRPOP`实现阻塞入队和出队
* **列表**：如文章列表、图片列表等
---

**Set（集合）**：
* **去重**：可以利用Set去重特性做数据存储，如文章点赞列表、用户关注列表等
* **需要用到交集、并集等场景**：`Set`提供了交集`SINTER`、并集`SUNION`等命令，可用在一些需要计算的场景，如共同关注（交集）、共同好友（交集）等
---

**Hash（散列）**：
* **对象信息缓存**：`Hash`查询效率高，用于一些kv类型的场景查询，如用户信息（用户id->用户消息）、购物车信息（商品id->商品信息）等
---

**Zset（有序集合）**：
* **排行榜**：具体可参考文章[《如何实现一个分数相同则按时间排序的排行榜？》](https://garyleeeee.github.io/2023/08/20/scene/ru-he-shi-xian-yi-ge-fen-shu-xiang-tong-ze-an-shi-jian-pai-xu-de-pai-xing-bang/)
* **延迟队列**：具体可参考文章[《如何用Redis实现延迟队列？》](https://garyleeeee.github.io/2023/07/29/ru-he-yong-redis-shi-xian-yan-chi-dui-lie/)
---

**BitMap（位图）**：
* **签到统计**：根据用户每天的签到情况，在`BitMap`上做几路（用1和0）。优点是内存使用少