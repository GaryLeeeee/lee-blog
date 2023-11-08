---
title: Redis如何做性能优化？
date: 2023-11-08 13:37:27
tags:
categories: [Redis]
---

## 一、Redis如何做性能优化？
### 1、批量操作减少网络传输
**背景**：如果我们一次要执行多条命令，如果是常规的`set`、`get`等命令，那么执行多少条命令就涉及多少次网络调用，所以如果我们把这些命令打包就会减少网络调用次数

**批量操作减少网络传输**有以下几种方案：
* **使用原生批量操作命令**：如`mget`、`sadd`等命令，支持批量操作的同时，也保证了原子性
* **管道pipeline传输**：对于不支持批量操作的命令，我们可以使用`pipeline`打包命令（但没办法保证原子性）
* **Lua脚本**：一段Lua脚本可以视为一条命令执行，所以它保证了原子性（但要注意Redis Cluster下无法保证，同时执行异常也无法回滚）

### 2、大key问题
具体可参考文章[《如何处理Redis的大key问题？》](https://garyleeeee.github.io/2023/08/19/redis/ru-he-chu-li-redis-de-da-key-wen-ti/)

### 3、缓存雪崩、缓存穿透问题
具体可参考文章[《关于对缓存雪崩和缓存穿透的理解，以及如何避免？》](https://garyleeeee.github.io/2023/07/24/guan-yu-dui-huan-cun-xue-beng-he-huan-cun-chuan-tou-de-li-jie-yi-ji-ru-he-bi-mian/)

### 4、热key问题
具体可参考文章[《如何解决Redis的热点key问题？》](https://garyleeeee.github.io/2023/09/12/redis/ru-he-jie-jue-redis-de-re-dian-key-wen-ti/)

### 5、慢查询问题
我们应该减少一些慢查询命令，他们的时间复杂度是O(n)，可能会导致全表扫描，比如：
* `KEYS *`
* `HGETALL`
* `SMEMBERS`
* ...

和MySQL的慢查询类似，Redis也有对应的慢查询日志、慢查询阈值参数等，比如：
* `slowlog-log-slower-than`参数设置慢查询阈值
* `slowlog-max-len`参数设置慢查询最大记录条数
* `slowlog get n`命令获取慢查询日志（默认返回最近10条）

### 6、内存碎片
当我们频繁对数据进行修改/删除操作时，可能会出现内存碎片，从而减少我们的内存利用率。

## 二、Q&A
### 1、pipeline和Redis事务有什么区别？
pipeline和Redis事务的区别如下：
* pipeline是非原子操作，Redis事务是原子操作
* pipeline可以同时执行，Redis事务不能同时执行
* pipeline只需要发送一次打包后的命令到服务端，Redis事务需要每个事务都发送到服务端

### 2、什么时候用pipeline？什么时候用Lua脚本？
当我们需要有上下文依赖的时候（如第一条命令的结果要用到第二条命令的判断），应该用Lua脚本。如果每一条命令都是独立无依赖关系的，那么我们也可以使用pipeline。






