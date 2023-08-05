---
title: 分页插件PageHelper的使用
date: 2022-10-19 22:36:15
tags: [PageHelper,Mybatis]
categories: [PageHelper]
---

## 一、介绍
插件地址：https://github.com/pagehelper/Mybatis-PageHelper/
**//TODO**

## 二、怎么使用？
### 1.引入依赖(maven)
```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>{pagehelper.version}</version>
</dependency>
```
如果是springboot项目，则改成引入
```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>{springbootPagehelperVersion}</version>
</dependency>
```

### 2.参数介绍
**//TODO**

### 3.如何在代码中使用
**//TODO**

## 三、Q&A
### 1.springboot使用PageHelper分页不生效怎么办？
正常分页使用
```
PageHelper.startPage(pageNo,pageSize)
```
排查后发现pom依赖错了，springboot环境下要用
```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>{springbootPageHelper.version}</version>
</dependency>
```
而不是原来的
```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>{pageHelper.version}</version>
</dependency>
```
因为上面的依赖缺少了
```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-autoconfigure</artifactId>
</dependency>
```

## 四、Mybatis-Plus如何使用分页插件？
**//TODO**