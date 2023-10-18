---
title: ElasticSearch的基本概念
date: 2023-10-18 16:27:21
tags:
categories: [ElasticSearch]
---

## 一、ElasticSearch的基本概念
ElasticSearch中有一些基本概念如下：
* **Index（索引）**：指同一类有相似特征的文档的集合，类似MySQL中的表，如用户索引、频道索引等
* **Document（文档）**：指可搜索的最小单位，一个文档由一个或多个字段组成（如整型、布尔值、字符串、日期等）
* **Type（字段类型）**：每个文档都需要对应有一个Type。在7.0之前，一个Index可以有多个Type。6.0开始，Type已经被Deprecated。7.0开始，一个Index只可以有一个Type（即`_doc`）。8.0之后，Type被完全删除
* **Mapping（映射）**：定义字段名称、数据类型、优化信息（比如是否索引）、分词器，类似MySQL中的表结构定义。一个Index对应一个Mapping
* **Node（节点）**：一个Node对应一个Es实例，多个Node组成一个集群
* **Cluster（集群）**：多个Node的集合，用来解决单个节点无法处理的搜索需求和数据存储需求
* **Shard（分片）**：Index可以被分为多个Shard分散到多个不同的Node节点上的分片中（建议分片为10个），以提高吞吐量和性能
* **Replica（副本）**：每个Index有一个或多个副本，用来提高吞吐量
* **DSL（查询语句）**：基于JSON的查询语句，类似于MySQL中的SQL语句

## 二、ElasticSearch的基本概念与MySQL简单类比
|MySQL|ElasticSearch|
|---|---|
|Table（表）|Index|
|Row（行）|Document|
|Column（列）|Field|
|Schema（约束）|Mapping|
|SQL（查询语句）|DSL|