---
title: MySQL中一条SQL的执行过程是怎样的？
date: 2023-09-11 23:43:35
tags:
categories: [MySQL]
---

### 一、SQL的执行过程
![](/images/mysql/sql执行过程.png)

执行SQL语句：
```sql
select * from student A where age = 18 and name = 'Gary';
```
步骤如下：
1. 通过**客户端/服务器通信协议**与MySQL建立连接，并查询是否有权限
2. MySQL 8.0之前**检查是否开启缓存**，开启了Query Cache且命中完全相同的SQL语句，则直接返回查询结果给客户端
3. 由**解析器**进行语法解析，并生成解析树，如查询是`select`、表名`student`、条件是`age='18' and name = 'Gary'`。同时**预处理器**会根据MySQL规则进一步检查解析树是否合法，如检查要查询的数据或数据列是否存在等
4. **查询优化器*生成执行计划，根据索引查看是否可以优化
5. **查询执行引擎**执行SQL语句，根据MySQL的存储引擎类型，得到查询结果。若开启了Query Cache，则缓存，否则直接返回