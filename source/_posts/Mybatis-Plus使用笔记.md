---
title: Mybatis-Plus使用笔记
date: 2021-11-09 12:59:01
tags: [Mybatis,Mybatis-Plus]
categories: [Mybatis,Mybatis-Plus,SQL,MySQL]
---

## 一、官方文档
https://baomidou.com/

## 二、怎么使用
### 1.添加依赖
```xml
<!-- MybatisPlus -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.2.0</version>
</dependency>
```

### 2.添加配置
```yaml
# Mybatis-plus
mybatis-plus:
  # 放在resource目录 classpath:/mapper/*Mapper.xml
  mapper-locations: classpath:/mybatis/mapper/*Mapper.xml
  # 实体扫描，多个package用逗号或者分号分隔
  typeAliasesPackage: com.yy.mobilevoice.svc.diversion.model.domain
  global-config:
    # 主键类型  0:"数据库ID自增", 1:"用户输入ID",2:"全局唯一ID (数字类型唯一ID)", 3:"全局唯一ID UUID";
    id-type: 2
    # 字段策略 0:"忽略判断",1:"非 NULL 判断",2:"非空判断"
    field-strategy: 2
    # 驼峰下划线转换
    db-column-underline: true
    # 刷新mapper 调试神器
    refresh-mapper: true
    # 数据库大写下划线转换
    #capital-mode: true
    # 逻辑删除配置（下面3个配置）
    logic-delete-value: 0
    logic-not-delete-value: 1
    # SQL 解析缓存，开启后多租户 @SqlParser 注解生效
    sql-parser-cache: true
  configuration:
    map-underscore-to-camel-case: true
    cache-enabled: false
```

### 3.常见注解(更多查看文档)
|注解|作用|
|-------------|-------------|
|@MapperScan("com.demo.mapper")|扫描该目录下的mapper接口都会自动生成实现类(等同于一个个加@Mapper)|
|@TableName("tableName")|用于标识数据库表对应的实体类||

## 三、demo代码地址
https://github.com/GaryLeeeee/lee-code-repository/tree/master/lee-db-code/src/main/java/com/garylee/repository/mybatisplus

