---
title: Redis内存淘汰策略
date: 2023-08-16 19:45:43
tags: [Redis]
categories: [Redis]
---

## 一、定义
Redis内存淘汰策略，指的是内存的使用率达到`maxmemory`上限时的一种内存释放行为

## 二、Redis有哪些内存淘汰算法？
Redis内存淘汰算法主要有五种，分别为：
* **LRU算法（Least Recently Used）**：移除最近最少使用的key
* **LFU算法（Least Frequently Used）**：移除最近最少使用的key（是**LRU算法**的优化增加了数据的访问频次，从而确保淘汰的数据是非热点的）
* **Random算法**：随机移除key
* **TTL算法**：在设置了过期时间的key中，优先移除即将过期的key

### 1、LRU算法
LRU算法是比较常见的一种内存淘汰算法，它是在Redis中维护了一个大小为16的候选池（双向链表，数据按时间排序），然后每次取出5个key放入候选池里，等候选池满了，候选池里最早的key就会被淘汰掉（新的key插入到头部）

**存在问题**：假如一个key访问频次很低，但是最近一次偶尔被访问到，那么LRU算法会认为它是热点数据，而不会被淘汰
**解决方案**：使用**LRU算法**

### 2、LFU算法
LFU算法是LRU算法的优化，区别是LRU算法注重的是最近是否使用过，LFU算法注重的是频次
另外在实现层面上，LRU算法用的是一个双向链表，而LFU算法用的是<font color=red>两个HashMap和多个双向链表，插入数据用尾插法，淘汰数据从链表头淘汰</font>

>举个例子

**前提**：
* HashMap：key=redis key，value=对应节点
* HashMap_Freq：key=频次，value=双向链表(存节点)

<font color=red>注意：为了避免混淆，redis key用字母a、b、c等表示，对应节点用A、B、C等表示。另外链表容量为3</font>
>步骤1：插入a、b、c

这时候的数据如下：
* HashMap：`[a=A,b=B,c=C]`
* HashMap_Freq：`[1={a,b,c}]`
* 移除：

**解析**：
* HashMap存放了数据的key和对应节点，由于是第一次操作，所以在HashMap_Freq中a、b、c都是第一次，存到了key=1的双向链表中

>步骤2：插入d

这时候的数据如下：
* HashMap：`[d=D,b=B,c=C]`
* HashMap_Freq：`[1={b,c,d}]`
* 移除：`a`

**解析**：
* 插入d后HashMap_Freq大小超过容量了，所以要从最小频次（1）拿到最早key（a）移除
* 由于插入了d、移除了a，所以在HashMap中d代替了a的位置

>步骤3：插入e

这时候的数据如下：
* HashMap：`[d=D,e=E,c=C]`
* HashMap_Freq：`[1={c,d,e}]`
* 移除：`b`

**解析**：
* 插入e后HashMap_Freq大小超过容量了，所以要从最小频次（1）拿到最早key（b）移除
* 由于插入了e、移除了b，所以在HashMap中e代替了b的位置

>步骤4：查询d

这时候的数据如下：
* HashMap：`[d=D,e=E,c=C]`
* HashMap_Freq：`[1={c,e},2={d}]`

**解析**：
* 由于是查询操作，HashMap不变
* 查询d，d在HashMap_Freq对应的频次（1）要加1（2）

> 步骤5：插入f

这时候的数据如下：
* HashMap：`[d=D,e=E,f=F]`
* HashMap_Freq：`[1={e,f},2={d}]`
* 移除：`c`

**解析**：
* 插入f后HashMap_Freq大小超过容量了，所以要从最小频次（1）拿到最早key（c）移除
* 由于插入了f、移除了c，所以在HashMap中f代替了c的位置

---
**总结LRU算法的关键逻辑**：
* 使用一个HashMap存放key和对应节点，一个HashMap_Freq存放频次和对应key双向列表
* 插入数据时如果LFU容量已满，则找到HashMap_Freq频次最低的双向链表，删除链表头部，并插入新数据到链表尾部
* 查询数据时如果数据存在双向链表中，则该数据对应的节点需要移动到+1频次的双向链表尾部去（如频次1的数据查询后就放到频次2中）