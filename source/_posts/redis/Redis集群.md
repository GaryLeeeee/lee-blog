---
title: Redis集群
date: 2023-08-08 17:56:58
tags: [Redis,集群]
categories: [Redis]
---

## 一、Redis集群有哪几种实现方式？
**Redis集群**有三种实现方式：
* 主从复制
* 哨兵模式
* Redis Cluster集群

### 1、主从复制
**优点**：
* 读写分离：master节点负责读写，slave节点负责读
* slave节点能够分担master节点的读操作压力

**缺点**：
* 不具备容错和恢复能力（需要等机器重启或手动切换master节点才能恢复）
* 一旦master节点挂了，不会自动切换新的master节点，导致后续的写请求都失败
* 存在在线扩容问题
* 所有slave节点的复制和同步都由master节点来处理，会增加master节点压力

### 2、哨兵模式（Sentinel）
**特点**：
* 一主多从
* 主从复制：所有节点存的是相同数据
* 适合读多于写场景

**优点**：
* 读写分离：master节点负责读写，slave节点负责读
* slave节点能够分担master节点的读操作压力
* 一旦master节点挂了，会从剩下的slave节点选举出新的master节点
* 具备容错和恢复能力

**缺点**：
* 存在在线扩容问题

### 3、Redis Cluster
**特点**：
* 实现了redis的分布式存储，每个节点存储不同的数据，实现数据的分片功能（Slot槽）
* 多主多从

**优点**：
* 一旦master节点挂了，会从剩下的slave节点选举出新的master节点
* 解决了在线扩容问题
* 具备故障转移能力

**缺点**：
* Slave节点只是实现冷备机制，不能分担redis读操作压力（只有在master宕机之后才会工作）
* 客户端实现更复杂

## 二、哨兵模式
**哨兵模式**由一个或多个sentinel实例组成sentinel集群，可以监视一个或多个主服务器和多个从服务器。适合读请求多于写请求的业务场景，比如在秒杀系统中缓存活动信息。如果写请求较多，当集群Slave节点多了，Master节点同步数据的压力会非常大

**检测主观下线状态**
Sentinel每秒一次向所有与它建立了命令连接的实例（主服务器、从服务器和其他Sentinel）发送PING命令，实例在`down-after-milliseconds`毫秒内返回无效回复，Sentinel就会认为该实例主观下线（SDown）

**检查客观下线状态**
当一个Sentinel将一个主服务器判断为主观下线后，Sentinel会向监控这个主服务器的所有其他Sentinel发送查询主机状态的命令，如果达到Sentinel配置中的数量的Sentinel实例都判断主服务器为主观下线，则该主服务器就会被判定为客观下线（ODown）

**选举Leader Sentinel**
当一个主服务器被判定为客观下线后，监视这个主服务器的所有Sentinel会通过选举算法（raft），选出一个Leader Sentinel去执行failover（故障转移）操作

**主服务器选择**
当选举出Leader Sentinel后，Leader Sentinel会根据以下规则从服务器中选择出新的主服务器：
* 过滤掉主观、客观下线的节点
* 选择配置`slave-priority`最高的节点，如果有则返回没有就继续选择
* 选择出复制偏移量最大的节点，因为复制偏移量越大则说明数据复制得越完整
* 选择runid最小的节点，因为runid越小说明重启次数越少

**故障转移**
当Leader Sentinel完成新的主服务器选择后，Leader Sentinel会对下线的主服务器执行故障转移操作
主要有三个步骤：
1. 它会将失效Master的其中一个Slave升级为新的Master，并让失效Master的其他Slave改为复制新的Master
2. 当客户端试图连接失效的Master时，集群会向客户端返回新Master的地址，使得集群当前状态只有一个Master
3. Master和Slave服务器切换后，Master的redis.conf、Slave的redis.conf和sentinel.conf的配置文件的内容都会发生相应的改变，即Master主服务器的redis.conf配置文件中会多一行`replicaof`的配置，sentinel.conf的监控目标也会随之调换

## 三、Redis哨兵模式和Redis Cluster有什么区别？
**读写**：
* 哨兵模式通过主从复制，实现读写分离，分担redis读操作压力
* Redis Cluster的Slave节点只是实现冷备机制，并不分担redis读操作压力（只有在master宕机之后才会工作）

**在线扩容**：
* 哨兵模式无法在线扩容，并发压力受限于单个服务器资源的配置
* Redis Cluster提供了Slot槽的数据分片机制，可以实现在线扩容

**集群架构**：
* 哨兵模式是一主多从
* Redis Cluster是多主多从