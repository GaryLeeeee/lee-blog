---
title: 如何排查慢SQL问题？
date: 2023-08-19 16:44:31
tags: [SQL]
categories: [MySQL]
---

## 一、如何发现慢SQL问题？
慢SQL指的是数据库中查询时间超过指定阈值的SQL，比如阈值设置为1s，那么说明如果一条SQL执行时间超过1s，则认为这是一条慢SQL

通常在实际开发中，如果有成熟的监控体系的话，那么会自动把慢SQL统计出来，并通过告警通知开发人员处理

如果没有的话，自己也是可以配置慢SQL日志并查询的，步骤如下：
1. 找到MySQL的配置文件`my.cnf`/`my.ini`（通常放在MySQL目录的etc或conf文件夹里）
2. 在配置文件中配置开启慢查询日志
```sql
slow_query_log = 1  --是否开启慢查询日志
slow_query_log_file = /path/to/slow-query.log  --慢查询日志地址
long_query_time = 1  --慢查询时间阈值（超过该值则为慢查询）
```
3. 保存文件并重试MySQL，使配置生效
4. *执行时间超过`long_query_time`的SQL，会自动打印慢SQL日志*
5. 查看慢查询日志：`cat /path/to/slow-query.log`
6. 分析慢查询日志
```sql
# Time: 2023-08-01T00:00:00.00Z
# User@Host: test[192.168.0.1]:3306
# Query_time: 1.23456  Lock_time: 0.012345 Rows_sent: 10  Rows_examined: 100
SET timestamp=1234567890;
SELECT * FROM user WHERE age > 18;  --慢查询SQL
```

## 二、如何定位并解决慢SQL问题？
一般来说，慢SQL问题都是由索引造成的，少数情况可能是多表查询、深度分页等造成的，具体可以参考[如何进行SQL调优？](https://garyleeeee.github.io/2023/08/19/mysql/ru-he-jin-xing-sql-diao-you/)