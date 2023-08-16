---
title: 如何排查MySQL索引失效的问题？
date: 2023-08-15 14:40:43
tags: [MySQL,索引]
categories: [MySQL]
---

## 一、索引失效分析
MySQL索引失效是一个比较常见的问题，这种情况一般会在发生慢SQL时需要考虑是否存在索引失效的问题

一般MySQL索引失效的原因有：
* **索引列参与计算**：如... where a + 1 = 2
* **对索引列进行函数操作**：如... where YEAR(a)=2023
* **使用or**：如... where a = 1 or b < 2
* **使用like**：如... where a like '%xxx'
* **隐式类型转换**：如... where a = 1 //a是varchar类型
* **使用!=或者**<>：如... where a != 1
* **使用is not null**：如... where a is not null

## 二、索引失效原因
```sql
create table user_table (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `name` varchar(50) NOT NULL,
    `age` int(11) DEFAULT NULL,
    `create_time` datetime DEFAULT NULL,
    PRIMARY KEY (`id`),
    UNIQUE KEY `name` (`name`),
    KEY `age` (`age`),
    KEY `create_time` (`create_time`)
) ENGINE=InnoDB DEFAULT CHARSET = utf8mb4;
```

### 1、索引列参与计算
可以走索引的情况：
```sql
select * from user_table where age = 18;
select * from user_table where age = 18 + 1;
```
不可以走索引的情况：
```sql
select * from user_table where age + 1 = 18;
```

### 2、对索引列进行函数操作
可以走索引的情况：
```sql
select * from user_table where create_time = '2023-06-01 00:00:00';
```
不可以走索引的情况：
```sql
select * from user_table where YEAR(create_time) = 2023;
```

### 3、使用or
可以走索引的情况：
```sql
select * from user_table where name = 'gary' and age = 18;
-- OR两边都是=时，索引不会失效
select * from user_table where name = 'gary' or age = 18;
```
不可以走索引的情况：
```sql
-- OR两边存在<或者>时，索引会失效
select * from user_table where name = 'gary' or age > 18;
```

### 4、使用like
可以走索引的情况：
```sql
select * from user_table where name like 'gary%';
select * from user_table where name like 'ga%ry';
```
不可以走索引的情况：
```sql
select * from user_table where name like '%gary%';
select * from user_table where name like '%gary';
```

### 5、隐式类型转换
可以走索引的情况：
```sql
-- age是int类型，条件查询用的是字符串，MySQL会将其转化为int类型，不会导致索引失效
select * from user_table where age = "1";
```
不可以走索引的情况：
```sql
-- 由于name是varchar类型，条件查询用的是int类型，会导致索引失效
select * from user_table where name = 1;
```

### 6、使用!=或者<>
不可以走索引的情况：
```sql
select * from user_table where age != 18;
```

### 7、使用is not null
不可以走索引的情况：
```sql
select * from user_table where name is not null;
```

### 