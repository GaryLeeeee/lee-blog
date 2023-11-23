---
title: 线上MySQL如何给千万级大表加索引？
date: 2023-11-22 14:02:26
tags: [MySQL]
categories: [场景题]
---

## 一、背景
如果我们直接在线上MySQL给千万级大表加索引，会导致长时间卡死（锁表）甚至崩溃，所以我们不能直接在原表上加索引，如：
```sql
ALTER TABLE `user` ADD INDEX `test_idx`(`name`);
```

## 二、解决方案
### 1、方案a：拷贝新表加索引后再交换命名（MySQL层面）
**思路**：因为不能直接操作原表，所以我们可以拷贝一个新表，然后在新表上加索引（此时新表没有被访问），再拷贝数据过去即可，最后再交换命名，让请求都打到新表上（请求方无感知）
大概代码如下：
```sql
-- 拷贝表结构到新表（先不拷贝数据）
CREATE TABLE `new_user` like `user`;
-- 新表加索引
ALTER TABLE `new_user` ADD INDEX `test_index`(`name`);
-- 同步数据
INSERT INTO new_user select * from user;
-- 交换表命名
ALTER TABLE user RENAME TO bak_user;
ALTER TABLE new_user RENAME TO user;
```

**缺点**：因为交换表这个动作不是原子性，所以可能出现这中间有数据变化导致数据丢失/不一致问题

### 2、方案b：拷贝新表加索引后再交换命名（业务逻辑层面）
**思路**：针对方案a的缺点，我们可以使用**双写**的方式防止数据丢失
**步骤**：
1. 拷贝表结构到新表（先不拷贝数据）
2. 新表加索引
3. 同步数据
4. 启用双写（即同时往旧表和新表写相同的数据）
5. **请求方将查询数据库表改为新表（如user改为new_user）**

**缺点**：需要改业务代码

### 3、方案c：pt-online-schema-change
**思路**：使用pt-online-schema-change的触发器（INSERT/UPDATE/DELETE触发器）可以解决新表和旧表同步数据可能存在的数据丢失问题（影子策略），使新表和旧表在同步时的数据变化也能得到同步
**步骤**：
1. 拷贝表结构到新表（先不拷贝数据）
2. 新表加索引
3. 在旧表添加**3个触发器**（INSERT/UPDATE/DELETE），用来同步复制表数据时的数据变化
4. 复制旧表数据到新表（使用数据库chunk方式）
5. 交换表命名
6. 删除触发器