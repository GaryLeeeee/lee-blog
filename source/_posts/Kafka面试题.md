---
title: Kafka面试题
date: 2023-07-12 22:07:40
tags: [Kafka]
categories: [Kafka]
---

## 一、Kafka面试题
### 1、Kafka的副本机制有什么用？
Kafka的副本机制是保证消息可靠性的重要手段之一，它可以：
* **提高可用性**：Kafka每个topic有多个分区，每个分区有多个副本，但其中只能有一个是leader副本，其余的都是follower副本，不像MySQL一样，Kafka只有leader副本才可以处理读写请求，follower副本只是从leader副本同步数据，并不会处理读写请求。<font color=red>当leader副本宕机时，会自动切换其中一个follower副本为新的leader副本，从而提高系统的可用性</font>
* **提高容错性**：能够防止数据丢失。在写入成功后，只有当数据被所有副本写入磁盘后，才认为该消息已经成功提交。如果某个节点出现故障导致不能正常同步，Kafka就会自动重试发送该消息，直到超时或成功为止

### 2、Kafka如何避免重复消费？
**背景**：
* Kafka broker的record都有一个offset标记（自己在分区中的位置信息）
* Kafka consumer也有个offset标记来维护当前已经消费数据的位置信息
* 每消费一批数据，broker就会更新offset的值，避免重复消费
* 默认record消费完会自动提交offset的值
    * **自动提交时机**：默认5s间隔提交（即5s后的下一次向broker拉取消息的时候提交）
    * **重复消费1**：当consumer消费时，如果程序宕机，那么会导致offset没提交，从而重复消费
* **rebalance机制**：把多个partition随机分配给多个消费者实例，如果其中某些partition不能正常使用，则会分配给其余正常的partition
    * **重复消费2**：如果consumer在默认的5分钟内处理不完从partition拉的消息，就会出发rebalance，从而导致offset自动提交失败。而rebalance后，新的consumer会从之前没提交的offset位置开始消费，从而导致重复消费

**避免方案**：
* 提高consumer的消费速率，如异步处理
* 减少一次性从broker拉取消息条数，减少问题出现
* 调整消息处理的超时时间
* **幂等性**：针对消息生成md5（或其他唯一标识），存到redis或mysql，每次消费消息都判断是否已消费过

### 3、Kafka如何保证消息消费的顺序性？
**背景**：
* Kafka通过partition来实现消息的物理存储，一个topic有多个partition
* 同一个partition的消息是有序的
* 同一个consumer有多个实例消费，每一个实例负责的partitin是有序的，但是每一个实例消费速率不同，导致可能宏观上看并不是有序的
* 如果用异步线程来消费数据，可以提高消费速率，但是每个线程的消息处理速率是不同的，所以单个partition的消费也可能会出现无序问题

**解决方案**：
* 调整路由策略，把指定key发送到同一个partition中（如用户行为可以通过uid去路由，这样就可以保证该用户是有序消费的）
* 把消息放到阻塞队列，然后用异步线程去阻塞队列获取消息消费

### 4、Kafka如何实现高吞吐量？
* **磁盘顺序读写**：日志追加写，减少随机I/O的耗时，提高了写入的吞吐量
* **零拷贝**：节约了用户空间和内核空间的切换
    * 一般文件拷贝：读取磁盘->写入内核缓冲区->拷贝到用户空间缓冲区->调用write等方法拷贝到内核空间的socket buffer->发送到网卡缓冲区->发送到目标服务器
    * Kafka拷贝：读取磁盘->写入内核缓冲区->拷贝到socket buffer->发送到网卡缓冲区->发送到目标服务器
* **批量处理**：Kafka允许生产者批量打包多个消息发送，减少网络传输的开销和提高I/O吞吐量，同时也允许消费者批量从broker拉取一批数据，并批量处理以提高效率

### 5、Kafka如何保证消息不丢失？
一般是通过三个方面来保证消息不丢失：
* **producer端**：保证消息不丢失，也就是保证数据发送成功
    * 失败重试：默认是异步发送，改为同步发送的话，producer可以感知消息发送的结果，并添加回调函数来监听消息发送结果，如果失败则重试
    * 调整retries参数：如果因为网络原因或broker故障等问题，producer会自动重试
    
* **broker端**：保证消息不丢失，也就是把数据持久化到硬盘即可
    * 背景：为了保证性能，Kafka采用了异步批量刷盘机制（达到一定消息量和时间间隔来刷盘），如果在刷盘之前系统崩溃，就会出现数据丢失
    * 解决方案：通过partition的副本机制+acks机制来解决
        * acks=0，表示producer不许等broker响应就认为发送成功，会存在<font color=red>消息丢失</font>
        * acks=1，表示producer会等leader返回确认就认为发送成功（不需要等follower同步完成），如果这时候leader挂了，就会存在<font color=red>消息丢失</font>
        * acks=-1，表示leader收到消息后，<font color=red>需要等待ISR列表的follower同步完成</font>，才会给producer返回确认，<font color=red>这个配置可以保证数据的可靠性</font>
    
* **consumer端**：保证消息不丢失，也就是保证数据正常消费并提交
    * 解决方案：手动提交offset，如果出现消息丢失的情况，则可以通过调整offset的值重新消费
    
### 6、Kafka的ISR、OSR有什么区别？
Kafka的ISR、OSR都是指副本机制中需要同步数据的副本，区别在于：
* **ISR**：即in-sync reply（同步中副本），指同步进度较好的副本，包含leader副本，在重新选举leader的时候会从ISR中选择
* **OSR**：即out-sync reply，指同步进度滞后的副本
