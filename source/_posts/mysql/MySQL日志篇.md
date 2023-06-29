---
title: MySQL日志篇
date: 2023-06-27 17:40:29
tags: [MySQL,日志,主从复制]
categories: [MySQL]
---

## 一、常见的日志类型
* 错误日志(error log)：对MySQL的启动、运行和关闭进行了记录
* **二进制文件**(binary log/binlog)：记录了更改数据库数据的SQL语句（不包括select、show等查询语句）
 * 一般查询日志(general query log)：记录客户端发送给MySQL服务器的所有SQL语句（因为量很大，所以默认不开启，也不建议开启）
* 慢日志查询(slow query log)：执行时间超过`long_query_time`的语句（一般排查慢查询问题的时候用到）
* **事务日志**(redo log和undo log)：redo log是重做日志，undo log是回滚日志
* 中继日志(relay log)：relay log是复制过程中的日志，跟binary log猜不到，不过relay log针对的是主从复制中的从库
* DDL日志(metadata log)：DDL语句执行的与数据操作

## 二、binlog介绍
### 1、概念
**定义**：binlog（即binary log，二进制日志文件），记录了对MySQL数据库执行了更改的所有操作（所有DDL和DML语句）
* DDL（即数据定义语句）：包括create、alter、drop等
* DML（即数据修改语句）：包括insert、update、delete等

**模式**：
* **Statement模式**：只记录DML语句（MySQL 5.7.7之前默认使用）
* **Row模式**：记录DDL+DML语句（MySQL 5.7.7之后默认使用）
* **Mixed模式**：Statement模式+Row模式组合（默认使用Statement模式，少数情况下自动切到Row模式）
* 如何查看使用的模式：`show variables like `%binlog_format%`;`
**场景**：**主从复制**，<font color=red>依赖binlog去同步数据，保证数据一致性</font>

### 2、主从复制
**参考文档**：https://javaguide.cn/high-performance/read-and-write-separation-and-library-subtable.html
**原理图**：
 ![](/images/mysql/mysql主从复制流程图.png)
**步骤**：
1. 主库将数据变化写入binlog
2. 从库连接主库
3. 从库创建一个I/O线程请求主库的binlog
4. 主库创建一个binlog dump线程来发送binlog，从库的I/O线程负责接收
5. 从库的I/O线程把接收到的binlog写入到relay log
6. 从库的SQL线程读取relay log数据到本地（也就是再执行一遍SQL）


### 3、binlog什么时候写入？
* 对于InnoDB存储而言，在执行一个事务时，会把日志写到binlog cache，等事务完成了再把binlog cache的日志持久化到binlog中，这样做也是为了考虑性能
* **为什么要有binlog cache呢？**因为一个事务的binlog不能被拆开，无论这个事务多大，都要保证一次性写入，所以才会有binlog cache的存在，我们可以通过设置`binlog_cache_size`来控制单个线程binlog cache的大小（如果超过改大小，就要<font color=red>暂存到磁盘中</font>）
* **binlog是什么时候刷新到磁盘中的？**可以通过sync_binlog来控制binlog的刷盘时机：`show variables like 'sync_binlog'`
    * sync_binlog=0：不强制要求，由系统自行判断刷盘时机（MySQL5.7之前的默认值）
    * sync_binlog=1：每次提交事务的时候都会刷盘（MySQL5.7之后的默认值）
    * sync_binlog=N：每N个事务，才会刷盘
    * 设置建议：不建议设置为0，如果对性能要求比较高/出现I/O瓶颈的画，可以适当增大sync_binlog，不过这样会增加数据丢失的可能（因为间隔变久了）
    
### 4、什么时候会重新生成binlog？
下面这几个情况，MySQL会重新生成新的binlog，并且序号递增：
* MySQL停止或重启
* 使用`flush logs`命令后
* binlog文件大小超过`max_binlog_size`变量的阈值后


## 三、redo log介绍
### 1、背景
* InnoDB引擎是以页（数据页）为单位来存储数据的，也就是我们往MySQL插入的所有数据最终都会写入到这个页中
* 为了减少IO开销，中间会有一个Buffer Pool的区域存在内存中，如果Buffer Pool中没有我们想要的数据，MySQL就会把页中的数据缓存到Buffer Pool中，这样我们就直接操作缓存即可，<font color=red>大大提高了读写性能</font>

### 2、Buffer Pool存在带来的问题
如果一个事务提交了，在Buffer Pool没来得及刷新（即持久化）回磁盘，MySQL就突然宕机了，这时候这个事务就消失了呢？结论是不会的，这样就违反了事务的持久性。<font color=red>MySQL InnoDB引擎用redo log来保证事务的持久性</font>

### 3、redo log是什么？
* <font color=red>redo log是InnoDB引擎独有的，用来保证事务的持久性</font>
* 主要是记录页的修改，比如哪个页某个偏移量处修改了几个字节的值以及具体被修改的值是什么
* redo log的每一条记录包括：
    * 表空间号
    * 数据页号
    * 偏移量
    * 具体修改的数据
    
### 4、redo log有什么用？
在事务提交时，我们会将redo log按照刷盘策略刷到磁盘上去，这样即使MySQL宕机了，重启之后也能恢复没有写入磁盘的数据，从而保证数据的持久性

