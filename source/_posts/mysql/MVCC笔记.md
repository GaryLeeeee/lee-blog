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
  * DB_TRX_ID：即最近修改事务id，<font color=red>记录创建这条记录或最后一次修改该记录的事务id</font> 
  * DB_ROLL_PTR：即回滚指针，指向这条记录的上一个版本，与undo log配合使用
  * DB_ROW_ID：即隐藏主键，如果MySQL表没有主键，那么InnoDB就会自动生成一个row_id
* undo log：即回滚日志，记录了数据的历史版本
* read view：即读视图，是<font color=red>事务进行快照读操作时候产生的读视图</font>，在该事务执行快照读的那一刻，会生成一个数据系统当前的快照，记录并维护系统当前活跃事务的id（事务id是递增的）
  * **作用**：当作条件来判断当前事务能够看到哪个版本的数据，有可能是最新的数据，也有可能是当前行记录的undo log中某个版本的数据
  

## 三、Read View详解
### 1、属性
Read View包含三个全局属性：
* m_ids：当前活跃的事务编号集合
* min_trx_id：最小活跃事务编号
* max_trx_id：预分配事务编号（当前最大事务编号+1）
* creator_trx_id：ReadView创建者的事务编号

### 2、可见性判断规则
* a、首先判断`DB_TRX_ID`<`min_trx_id`
  * **如果成立**：说明当前事务可以看到`DB_TRX_ID`所在的记录
  * **如果不成立**：则走下一个判断
* b、然后判断`DB_TRX_ID`>=`max_trx_id`
  * **如果成立**：说明当前事务不可以看到`DB_TRX_ID`所在的记录（因为`DB_TRX_ID`所在的记录是在Read View生成后才出现的，所以不可见） 
  * **如果不成立**：则走下一个判断
* c、最后判断`DB_TRX_ID`存在于`m_ids`
  * **如果成立**：说明当前事务不可以看到`DB_TRX_ID`所在的记录（因为Read View生成时刻，<font color=red>该事务还是活跃状态，没有commit</font>，所以修改的数据，当前事务是不可见的）
  * **如果不成立**：说明当前事务可以看到`DB_TRX_ID`所在的记录（因为该事务在Read View生成之前就已经commit，所以修改的数据，当前事务是可见的）
  
### 3、RC、RR级别下的InnoDB快照读有什么不同？
生成时机不同：
* RC是每次进行快照读都会生成一个新的Read View
* RR是只有第一次进行快照读才会生成一个新的Read View（后续快照读都是同一个，所以解决了<font color=red>不可重复读</font>的问题）

### 4、快照读和当前读有什么区别？
* **快照读**：select
* **当前读**：insert、update、delete、select ... for update、select ... lock in share mode

## 四、undo log详解
### 1、如果undo log被删除了，版本链不就断了吗？
undo log版本链不是立即删除，MySQL确保版本链数据不再被<font color=red>引用</font>后再进行删除

//TODO

## 五、问题
### 1、RR隔离级别下使用MVCC能避免幻读吗？
能，但不完全能。RR用的是快照读，仅第一次生成，后面使用的ReadView会复用第一次的
**特例**：当两次<font color=red>快照读</font>之间存在<font color=red>当前读</font>，ReadView会重新生成，从而产生幻读