---
title: Mybatis的工作原理是什么？
date: 2023-11-21 19:06:27
tags:
categories: [Mybatis]
---

## 一、什么是Mybatis？
Mybatis是一个持久化框架，可以帮助我们快速做数据库交互开发。

常规的数据库交互开发，我们需要有下面几步：
* 创建Connection
* 创建Statement
* 代码中拼接SQL语句
* 返回结果集转换
* ...

而使用Mybatis，我们可以避免大部分的这些繁琐操作，比如支持：
* 配置Dao层提供接口使用，简化代码量
* 支持XML/注解配置，从而区分接口类和配置参数
* ...

所以使用Mybatis带来的优点是：
* **简单方便**：减少代码量，从而让我们能专注代码逻辑的开发和SQL语句的编写
* **灵活性**：支持XML/注解配置，可以通过修改配置文件达到修改SQL逻辑的效果（同时也提供了一些标签支持我们的复杂逻辑如`<if>`、`<resultMap>`等）
* **性能更好**：提供了缓存机制，可以提高查询性能

## 二、Mybatis的工作原理是什么？
Mybatis大概的工作原理如下：
1. 通过Dao层的方法调用找到XML映射文件对应id的Statement，找到对应的SQL语句（或者是预编译的）
2. 把传参跟SQL语句的占位符一一对应，得到最终的SQL语句
3. 执行查询
4. 将返回的结果集根据XML映射文件配置的`<resultMap>`转换为对应的Java对象
5. end

### 1、Dao层的工作原理是什么？
首先我们知道Dao层就是一个接口，不需要实现，代码如下：
```java
@Mapper
public interface TestMapper {
    void insert(User user);
    int deleteById(int id);
    User getById(int id);
}
```

然后我们会通过**JDK动态代理**生成一个代理对象，当我们调用Dao层的方法时，就会走到代理对象的`execute(...)`逻辑，再去解析XML映射文件的SQL语句等配置。

## 三、Q&A
### 1、Dao层支持重载方法吗？
支持的，但是多个重载方法只能对应同一个XML映射文件中的id（严格来说是只能有一个无参方法和一个有参方法，因为有参方法的话是会通过`getProperty(...)`方法来解析入参，当不存在时，就会报错）。

对应的例子代码如下：
```java
@Mapper
public interface TestMapper {
    User get();
    User get(int id);
}
```
对应的XML映射文件如下：
```xml
<select id="get" resultMap="UserMap">
    select id,name from User
    <where>
        <if test="id != null">
            id = #{id}
        </if>
    </where>
</select>
```

这样子当我们调用无参方法时，就不会命中`<if>`标签里的逻辑了，而调用有参方法时，就会命中`<if>`标签里的逻辑。

### 2、Mybatis用的是什么连接池？
Mybatis并不是直接通过JDBC去访问数据库的，而是通过连接池。

在Mybatis中默认的数据源有：
* Pooled
* Unpooled
* JNDI

同时Mybatis也是支持使用第三方数据源的，如我们常用的：
* Druid
* Hikari
* C3P0
* ...

为什么我们不用默认的数据源呢，而用常用的Druid、Hikari等？以Druid为例，原因有以下几点：
* **性能更好**：支持高并发下可以更快处理请求，支持自动检测和回收空闲连接，避免了连接资源的浪费，能够有效控制连接池的大小，防止连接过多或多少带来的问题
* **可配置性**：支持在配置文件中调整连接池的各个参数，从而应对不同的使用场景
* **多数据源**：支持使用多种数据库作为数据源，提高代码的灵活性

