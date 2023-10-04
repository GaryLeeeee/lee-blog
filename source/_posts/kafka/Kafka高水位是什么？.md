---
title: Kafka高水位是什么？
date: 2023-10-03 22:55:55
tags:
categories: [高水位]
---

## 一、Kafka的高水位是什么？
在Kafka中，一条消息只有被成功提交了才能被消费者消费，而提交的进度就是用高水位来表示。

高水位也叫做HW，指的是未提交消息中最早的offset（意味着消费者只能消费offset之前的消息）。。

![](/images/kafka/Kafka高水位1.png)

## 二、高水位有什么用？
高水位作用主要有两个：
* 定义消息可见性，用来表示分区下的哪些消息是可以被消费者消费的
* 帮助Kafka完成副本同步

## 三、副本是什么？
副本机制是Kafka可靠性的重要手段之一，主要有以下几点：
* **Leader副本**：提供读写服务
* **Follower副本**：不提供读写服务，只被动同步Leader副本的数据
* **ISR**：包含Leader副本和同步进度完整的Follower副本（Leader选举所在集合）

## 四、高水位怎么存？
Kafka所有副本中（包括Leader副本和Follower副本）都保存有HW值和LEO值，其中Leader副本还存有其他Follower副本的LEO值（<font color=red>只存其他Follower副本的LEO值而不存HW值</font>）
* **HW**：即高水位（High Watermark）
* **LEO**：即日志末端offset+1（Log End Offset），也就是下一条插入消息对应的offset

![](/images/kafka/Kafka高水位2.png)

## 五、副本同步机制流程演示
1. 首先是初始状态，所有值都是0：
![](/images/kafka/Kafka高水位3.png)

2. 生产者发送一条消息后，Leader副本的LEO更新为1：
![](/images/kafka/Kafka高水位4.png)
   
3. Follower副本尝试从Leader副本同步消息，Leader副本返回LEO为1，Follower副本同步更新LEO为1：
![](/images/kafka/Kafka高水位5.png)
   
4. 前面的步骤Follower副本已经从Leader副本拉取了offset为0的消息，所以步骤4是要拉取offset为1的消息。Leader副本收到请求后，更新Remote LEO为1（即远程Follower副本为1），并比较所有副本的LEO得到最小值1作为高水位。同时发送HW为1给Follower副本，Follower副本也将自己的HW更新为1：
![](/images/kafka/Kafka高水位6.png)
   
5. end

## 六、什么是Leader Epoch？
**背景**：在步骤五中我们可以发现，Follower副本HW和LEO的更新是通过两次同步更新的，也就是第一次更新LEO，第二次更新HW，但是因为这两步不是原子性的，所以如果在异常情况下，是可能会出现数据丢失导致数据不一致的问题

**什么是Epoch**：Epoch是一个单调递增的版本号，当副本领导权发生变更时，会更新该版本号（版本号比较小的Leader被认为是过期的Leader）

### 1、举例分析数据丢失流程
1. 一开始，副本A和副本B都处于正常状态，副本A是Leader副本，副本B是Follower副本，这时生产者发送一条消息到副本A（只需要Leader返回确认即可），这时候副本B已经同步到副本A的LEO了，但还没进行第二次同步拿到HW（副本A的HW为2，副本B的HW为1）
2. 这时候副本B宕机了，发生重启，重启之后发生日志截断，将LEO值调整为之前的HW值，也就是1（这时候副本B的offset=1的消息丢失）
3. 副本B从副本A同步消息，假如这个时候副本A也宕机了，那么将会选举B为新的Leader
4. 副本A发生重启，重启之后也发生日志截断，将调整HW为副本B的HW值
5. 这时候offset=1的消息就丢失了
![](/images/kafka/Kafka高水位7.png)
   
### 2、Leader Epoch如何解决数据丢失？
Leader Epoch在上面的流程中做了一些处理：
1. 副本B重启之后，会向副本A请求Leader的LEO值（2）
2. <font color=red>判断步骤1的值（2）是否比当前副本B的LEO值（1）小，如果不是则不需要执行日志截断</font>
3. 副本A重启之后，会向副本B请求Leader的LEO值（2）
4. <font color=red>判断步骤3的值（2）是否比当前副本B的LEO值（2）小，如果不是则不需要执行日志截断</font>
5. 这时候offset=1的消息就不会丢失了

然后当后续生产者向副本B发送消息时，副本B会生成新的Leader Epoch条目为`[Epoch=1,Offset=2]`，用来保证后续判断是否需要进行日志截断。

![](/images/kafka/Kafka高水位8.png)
   
## 七、Q&A
### 1、高水位是什么时候更新的？
高水位更新需要区分Leader副本和Follower副本：
* **Leader副本**：收到生产者发送的消息后，存入本地磁盘，然后判断当前Leader副本和所有远程副本LEO的最小值作为Leader副本的高水位指
* **Follower副本**：从Leader副本拉取数据的时候，会同步设置Leader副本中对应全程Follower副本的LEO值，并拿到Leader副本返回的高水位值跟当前Follower副本的LEO值比较，取最小值作为Follower副本的高水位值

### 2、Leader副本为什么还要存远程Follower副本的LEO值？
Leader副本存远程Follower副本的LEO值，是为了计算确认当前Leader副本的高水位值，即分区高水位值。

### 3、Follower副本和Leader副本保持同步的条件是什么？
Follower副本和Leader副本保持同步的条件是：
* Follower副本要在ISR中
* Follower副本的LEO落后Leader副本LEO的时间不超过`replica.lag.time.max.ms`秒，默认是10秒（<font color=red>同时也用来判断是否能够进入ISR</font>）


