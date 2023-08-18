---
title: MySQL执行计划分析
date: 2023-08-18 12:10:24
tags: [索引]
categories: [MySQL]
---

## 一、执行计划字段介绍
```sql
+----+-------------+-------+------------+-------+---------------+----------+---------+------+------+----------+--------------------------+                                           
| id | select_type | table | partitions | type  | possible_keys | key      | key_len | ref  | rows | filtered | Extra                    |                                           
+----+-------------+-------+------------+-------+---------------+----------+---------+------+------+----------+--------------------------+                                           
|  1 | SIMPLE      | t2    | NULL       | index | NULL          | idx_abcd | 198     | NULL |    5 |    20.00 | Using where; Using index |                                           
+----+-------------+-------+------------+-------+---------------+----------+---------+------+------+----------+--------------------------+
```

一个执行计划中，共有12个字段，分别为：
* **id**：代表每个查询的唯一标识符（自增）
* **select_type**：表示查询类型，包括SIMPLE、PRIMARY、SUBQUERY、UNION等
* **table**：查询哪张表
* **partitions**：坐落分区（一般为NULL）
* **type**：表示索引类型，包括ALL、index、range、ref、eq_ref、const等
* **possible_keys**：表示可能命中的索引
* **key**：表示实际命中的索引
* **key_len**：表示索引的长度（如id类型为long，占用8字节，所以key_len=8），索引长度越短，查询效率越高
* **ref**：表示连接操作所使用的索引（可以是某一列、也可以是常数）
* **rows**：表示本次查询扫描行数（即扫描表中多少行才能得到结果，不一定等于最终结果数，扫描行数越少越好）
* **filter**：表示本次查询过滤掉的行数占扫描行数的百分比（百分比越大，说明查询结果越准确）
* **Extra**：表示其他额外的信息，包括Using index、Using filesort、Using temporary等

### 1、type
type取值包括ALL、index、range、ref、eq_ref、const等，举例说明：
```sql
 CREATE TABLE `t` (
     `id` INT(11),
     `a` varchar(64) NOT NULL,
     `b` varchar(64) NOT NULL,
     `c` varchar(64) NOT NULL,
     `d` varchar(64) NOT NULL,
     `f` varchar(64) DEFAULT NULL,
     PRIMARY KEY(id),
     UNIQUE KEY `f` (`f`),
     KEY `idx_abcd` (`a`,`b`,`c`, `d`)
 ) ENGINE=InnoDB;
```
* **ALL**：表示全表查询，如`explain select * from t where d = 'test'';`
* **index**：表示全索引查询（不包括最左前缀匹配的查询，和`ALL`区别是按照索引顺序查询），如`explain select c from t where b= 'test';`
* **range**：表示范围查询，如`explain select * from t where a between 'min' and 'max';`
* **ref**：表示非唯一索引查询，如`explain select * from t where a = 'test';`
* **eq_ref**：表示唯一索引查询，用来做等值连接查询，如`explain select * from t1 join t2 on t1.id = t2.id where t1.f = 'test';`
* **const**：表示常数索引查询（唯一索引），如`explain select * from t where f = 'test';`

**总结**：在效率上，`const`>`eq_ref`>`ref`>`range`>`index`>`ALL`

### 2、Extra
Extra表示查询时的一些额外信息，包括Using index、Using filesort、Using temporary等
* **Using where**：表示MySQL在索引检索后，再通过条件过滤（where）；或者查询的列未被索引覆盖，用联合索引的非前导列查询，通过条件过滤（where）
    * **场景1：非索引字段查询**：`explain select * from t where d = 'test';`
    * **场景2：未被覆盖索引**：`explain select d from t where b = 'test';`
    
* **Using index**：表示MySQL通过索引查询时使用了<font color=red>覆盖索引</font>优化，只需要扫描索引（非聚簇索引），而无需回表到数据表（聚簇索引）中查找
    * **覆盖索引**：`explain select b,c from t where a = 'test';`
    
* **Using index condition**：表示MySQL通过索引查询时无法使用覆盖索引，需要回表到数据表（聚簇索引）中查找
    * **索引下推**：`explain select d from t where a = 'test' and b like 't%';`
  
* ...

## 二、如何判断sql有没有命中索引？
判断sql有没有命中索引，通常需要看key有没有值，有的话则说明用到了索引，但是具体怎么用的，还需要看type和extra，举例说明：

>例子1：`explain select b from t where a in ('a','b');`
```sql
+----+-------+---------------+----------+--------------------------+                                           
| id | type  | possible_keys | key      | Extra                    |                                           
+----+-------+---------------+----------+--------------------------+                                           
|  1 | index | NULL          | idx_abcd | Using where; Using index |                                           
+----+-------+---------------+----------+--------------------------+ 
```
**解析**：`key=idx_abcd`说明用到了索引，但是没有遵守最左前缀匹配，或者遵守最左前缀匹配但是使用a进行了范围查询，导致`extra=Using where;Using index;`。所以这条sql<font color=red>走了索引，但是效率不高</font>

>例子2：`explain select * from t where a = 'test';`
```sql
+----+-------+---------------+----------+--------------------------+                                           
| id | type  | possible_keys | key      | Extra                    |                                           
+----+-------+---------------+----------+--------------------------+                                           
|  1 | ref   | idx_abcd      | idx_abcd | NULL                     |                                           
+----+-------+---------------+----------+--------------------------+ 
```
**解析**：`type=ref`表示用的是非唯一索引，且该非唯一索引是`key=idx_abcd`

>例子3：`explain select * from t where f = 'f';`
```sql
+----+-------+---------------+----------+--------------------------+                                           
| id | type  | possible_keys | key      | Extra                    |                                           
+----+-------+---------------+----------+--------------------------+                                           
|  1 | const | f             | f        | NULL                     |                                           
+----+-------+---------------+----------+--------------------------+ 
```
**解析**：`type=const`表示用的是唯一索引，且该唯一索引是`key=f`

>例子4：`explain select b,c from t where a = 'test';`
```sql
+----+-------+---------------+----------+--------------------------+                                           
| id | type  | possible_keys | key      | Extra                    |                                           
+----+-------+---------------+----------+--------------------------+                                           
|  1 | ref   | idx_abcd      | idx_abcd |  Using index             |                                           
+----+-------+---------------+----------+--------------------------+ 
```
**解析**：`type=ref`表示用的是非唯一索引，且该非唯一索引是`key=idx_abcd`，另外`Extra=Using index`表示用到了覆盖索引，不需要回表

>例子5：`explain select b,c from t where d = 'test';`
```sql
+----+-------+---------------+----------+--------------------------+                                           
| id | type  | possible_keys | key      | Extra                    |                                           
+----+-------+---------------+----------+--------------------------+                                           
|  1 | ALL   | NULL          | NULL     |  Using where             |                                           
+----+-------+---------------+----------+--------------------------+ 
```
**解析**：`type=ALL,key=NULL`表示没有用到索引

