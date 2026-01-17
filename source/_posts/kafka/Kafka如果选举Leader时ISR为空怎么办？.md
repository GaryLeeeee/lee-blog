---
title: Kafka如果选举Leader时ISR为空怎么办？
date: 2026-01-17 16:14:39
tags:
categories: [Kafka]
---

## 一、背景：什么时候会发生Leader选举？
当Controller检测到Leader不可用的时候，会重新选举新的Leader，其中Leader不可用的原因可能如下:
* **Leader节点宕机**：检测Leader所在Broker进程异常，导致心跳超时（可通过`zookeeper.session.timeout.ms`配置，默认6s）
* **网络不通**：当Leader与其他副本/Controller网络隔离，导致副本同步超时（可通过`replica.lag.time.max.ms`配置，默认30s）

## 二、去哪里选举Leader？
Kafka中默认会从ISR集合（同步进度比较完整的副本集合）去选举，具体还是通过`unclean.leader.election.enable`配置去决定：
* **false（默认）**：只能从ISR中选举，数据可靠性高，但是可用性差，适合电商等对数据一致性要求高的场景
* **true**：允许从OSR中选举（进度比较完整的），数据可靠性低，但是可用性比较好，适合日志收集等允许少量数据丢失的场景

## 三、什么情况ISR会为空？
* **副本同步滞后严重**：可能由于生生产者写入速率过高，副本来不及同步
* **网络异常**：当Leader和Follower中无法通信，Leader感知不到Follower的存在所以踢出ISR（可以调整超时时间避免轻微网络抖动带来的影响）

## 四、解决方案
* 增加副本，避免唯一副本不可用后ISR为空
* 如果业务场景允许少量数据丢失，可以临时开启`unclen.leader.election.enable=true`，允许选举OSR
* 如果业务场景不允许数据丢失，需要尽快修复故障副本（如重启宕机节点等），等副本同步完成加入ISR后即可恢复
* 调整同步延迟`replica.lag.time.max.ms`，避免轻微网络抖音导致Follower被踢出ISR