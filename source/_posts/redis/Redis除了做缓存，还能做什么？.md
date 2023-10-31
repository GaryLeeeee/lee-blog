---
title: Redis除了做缓存，还能做什么？
date: 2023-10-31 15:18:01
tags:
categories: [Redis]
---

## 一、Redis除了做缓存，还能做什么？
Redis除了做缓存，还能做以下几种功能：
* **分布式锁**：具体可参考文章[《如何实现一个分布式锁？》](https://garyleeeee.github.io/2023/09/05/concurrent/ru-he-shi-xian-yi-ge-fen-bu-shi-suo/)
* **延迟队列**：具体可参考文章[《如何用Redis实现延迟队列？》](https://garyleeeee.github.io/2023/07/29/ru-he-yong-redis-shi-xian-yan-chi-dui-lie/)
* **消息队列**：Redis支持发布/订阅模式和Stream，可以作为轻量级消息队列使用
* **排行榜**：具体可参考文章[《如何实现一个分数相同则按时间排序的排行榜？》](https://garyleeeee.github.io/2023/08/20/scene/ru-he-shi-xian-yi-ge-fen-shu-xiang-tong-ze-an-shi-jian-pai-xu-de-pai-xing-bang/)
* **限流**：可以用来实现分布式限流，如令牌桶算法、漏桶算法
* ...