---
title: Kafka使用笔记
date: 2021-11-07 23:25:22
tags: Kafka
categories: Kafka
---


## 常见问题
### 1.本地启动报错"Timeout expired while fetching topic metadata"
临时方案：如果不需要测试kafka可以临时注释掉listener的@Component或者@KafkaListener，使其不影响启动

### 2.消费者的LAG(消费滞后量，越大说明堆积越严重)一直在增加
* **排查**：
    * 配置中的`enable-auto-commit=false`代表消费者不会自动提交ack
    * 配置中的`ack-mode=MANUAL_IMMEDIATE`代表每一条mq消费之后需要立即手动提交ack
* **解决**：
    * **方案1**：配置`enable-auto-commit`改为`true`
    * **方案2**：消费入参引入`Acknowledgment ack`并在消费完mq后执行`ack.acknowledge()`
