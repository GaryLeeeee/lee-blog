---
title: 如何用Redis实现延迟队列？
date: 2023-07-29 14:43:37
tags: [场景题,Redis,延迟队列,RocketMQ]
categories: [场景题]
---

## 一、什么是延迟队列？
**延迟队列**是一种特殊类型的消息队列，它允许把消息发送到队列中，但不立即投递给消费者，而是在一定时间后再将消息投递给消费者

## 二、使用场景
**延迟队列**适用于需要在未来的某个时间执行某个任务的场景，如：
* **订单的超时处理**：如电商交易场景，订单下单后暂未支付，此时不可以直接关闭订单，而是需要等待一段时间后才能关闭订单
* **定时任务**：如每天5点执行文件清理
* **游戏中定时事件触发**：如第2分钟要发一次消息推送
* ...

## 三、实现原理
**延迟队列**是通过Redis的Zset这个有序集合来实现的，具体步骤为：
1. **写入**：使用`ZADD`命令把消息写入到sorted set中，并将<font color=red>当前时间作为score</font>，对应命令为`ZADD delay-queue <timestamp> <message>`
2. **读取**：启动一个消费者线程，使用`ZRANGEBYSCORE`命令获取定时从Zset中获取<font color=red>当前时间之前的所有消息</font>，对应命令为`ZRANGEBYSCORE delay-queue 0 <current_time> WITHSCORES LIMIT 0 <batch_size>`
3. **移出**：消费者处理完消息后，可以从有序集合中删除这些消息，对应命令为`ZREMRANGEBYSCORE delay-queue 0 <current_time>`

## 四、缺陷
由于消费者线程需要不断向Redis发起轮询，所以会出现两个问题：
* 轮询存在时间间隔，所以延时消息的实际消费时间会大于设定的时间
* 大量轮询会对Redis服务造成压力，所以考虑到性能问题的话，可以考虑MQ，也是支持延时队列

## 五、RocketMQ实现延迟队列
**介绍**：RocketMQ的定时消息和延时消息本质相同，都是服务端根据消息设置的定时时间在某一固定时刻将消息投递给消费者消费
**优势**：传统的数据库扫描方式较为复杂，需要轮询，容易产生性能瓶颈，而RocketMQ具有高并发和水平扩展的能力
**参考链接**：
https://rocketmq.apache.org/zh/docs/featureBehavior/02delaymessage/
https://blog.csdn.net/qq_19734597/article/details/129194011

## 六、代码实现(Go)
https://github.com/GaryLeeeee/lee-code-repository/tree/master/lee-rd-code-go/delay_queue
