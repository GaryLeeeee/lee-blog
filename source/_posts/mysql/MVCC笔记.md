---
title: MVCC笔记
date: 2023-07-01 11:50:55
tags: [MySQL,MVCC]
categories: [MySQL]
---

## 一、介绍
* MVCC机制，全程（Multi-Version Concurrency Control）多版本并发控制，是确保在高并发下，多个事务读取数据时不加锁也可以多次读取相同的值
* MVCC在RC（读已提交）、RR（可重复读）的事务隔离级别下才生效
* MVCC在RR（可重复读）的事务隔离级别下，可以解决脏读、不可重复读等问题

## 二、实现原理
MVCC的实现原理主要依赖于记录中的几个属性实现的：
* 三个隐藏字段（即除了我们自定义的字段外，数据库默认的隐式定义字段，也就是存在但无感知）：
  * DB_TRX_ID：即最近修改事务id，记录创建这条记录或最后一次修改该记录的事务id 
  * DB_ROLL_PTR：即回滚指针，指向这条记录的上一个版本，与undo log配合使用
  * DB_ROW_ID：即隐藏主键，如果MySQL表没有主键，那么InnoDB就会自动生成一个row_id
* undo log：即回滚日志，记录了数据的历史版本
* read view：即读视图，用来保证多个并行事务之间能否互相读到数据


