---
title: Kafka使用笔记
date: 2021-11-07 23:25:22
tags: Kafka
categories: Kafka
---


## 常见问题
### 1.本地启动报错"Timeout expired while fetching topic metadata"
临时方案：如果不需要测试kafka可以临时注释掉listener的@Component或者@KafkaListener，使其不影响启动


