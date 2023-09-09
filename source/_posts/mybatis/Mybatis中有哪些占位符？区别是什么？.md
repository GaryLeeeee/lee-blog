---
title: Mybatis中有哪些占位符？区别是什么？
date: 2023-09-09 17:10:41
tags: 
categories: [Mybatis]
---

## 一、Mybatis中有哪些占位符？
Mybatis有两个占位符，分别是#{}和${}

## 二、Mybatis中#{}和${}有什么区别？
```sql
-- #{}
select id from #{table} where id = #{id}

-- ${}
select id from ${table} where id = ${id};
```
区别如下：
* **防止SQL注入**：#{}类似jdbc中的PreparedStatement，对于传入的参数会在预编译阶段用`？`代替，<font color=red>可以有效防止SQL注入</font>。而${}是直接将参数拼到SQL里，Mybatis不会对它进行特殊处理
* **使用场景**：#{}尽量能用到的地方都用上吧，而${}一般用在动态排序字段上（如`order by ${sortParam}`）