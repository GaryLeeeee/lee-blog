---
title: MongoDB和MySQL有什么区别？
date: 2023-09-11 20:30:12
tags:
categories: [MongoDB]
---

## 一、MongoDB和MySQL有什么区别？
||MongoDB|MySQL|
|---|---|---|
|数据库类型|非关系型|关系型|
|存储方式|JSON|列+行存储|
|扩展性|支持分布式部署，可以动态扩展集群|需要横向分表或纵向扩容来解决数据量过大的问题|
|数据一致性|较差|较高，支持ACID|
|查询语句|类似JavaScript函数的Mongo Shell命令|SQL语句|
|事务操作|仅支持单文档事务操作|支持事务操作|
|占用空间|占用空间大|占用空间小|
|如何使用|直接插入JSON即可，比较灵活|需要先定义表结构再使用|

## 二、MongoDB和MySQL属性对应表
|MongoDB|MySQL|
|---|---|
|table（表）|collection（集合）|
|row（行）|document（文档）|
|column（列）|field（字段）|
|index（索引）|index（索引）|
|join（主外键查询）|embedded Document（嵌套文档）|
|primary key（指定1至N列做主键）|primary key（指定_id field做主键|

## 三、MongoDB和MySQL使用场景比较
MongoDB使用场景有：
* **非结构化数据存储**：如日志、博客等
* **支持分布式部署和水平扩展**：可以很好应对高并发和大数据量场景
* ...

MySQL使用场景有：
* **结构化数据存储**：如电商、金融等
* **支持事务处理**：如电商，处理订单、支付等操作
* **支持数据关系处理**：如人力资源系统，处理员工、薪资等操作
* ...

## 四、举例子
比如需要设计一个用户表，并关联用户的兴趣、社交账号

### 1、MySQL实现
**用户表**：

|id|name|
|---|---|
|10001|Gary|

**兴趣表**：

|id|user_id|hobby|
|---|---|---|
|101|10001|basketball|
|101|10001|football|

**社交账号表**：

|id|user_id|type|account|
|---|---|---|---|
|201|10001|wechat|Gary2333|
|202|10001|qq|123456|

### 2、MongoDB实现
```json
{
  "_id": 10001,
  "name": "Gary",
  "hobbies": ["bastketball", "football"],
  "accounts": [
    {
      "type": "wechat",
      "account": "Gary2333"
    },
    {
      "type": "qq",
      "account": "123456"
    }
  ]
}
```

可以看到，MongoDB支持数组和嵌套，可以在一个JSON里关联所有数据，比较灵活