---
title: Spring事务
date: 2023-09-06 00:01:17
tags: [事务]
categories: [Spring]
---

## 一、什么是事务？
参考[《MySQL事务》](https://garyleeeee.github.io/2023/07/03/mysql/mysql-shi-wu/)

## 二、Spring支持哪些事务管理类型？
Spring支持两种类型的事务管理：
* **编程式事务管理**：用户需要通过编程的方式手动管理事务的开启、提交、回滚等操作（比较灵活，但是比较难维护）
* **声明式事务管理**：用户可以通过注解或XML配置来管理事务（方便，解耦，能将业务代码和事务管理分离）

## 三、Spring有哪几种事务实现方式？
### 1、基于transactionTemplate、PlatformTransactionManager编程式
//TODO：编程式实现方式

### 2、基于Aspectj AOP声明式（即基于<tx>和<aop>命名空间）
```xml
<!-- 配置事务 -->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="*" propagation="REQUIRED" rollback-for="Exception"/>
    </tx:attributes>
</tx:advice>

<!-- 配置织入 -->
<aop:config>
    <!-- 扫描com.yy.service路径下所有的方法，并加入事务处理 -->
    <aop:pointcut id="tx" expression="exection(* com.yy.service.*.*(..))" />
    <aop:advisor advice-ref="txAdvice" pointcut-ref="tx"/>
</aop:config>
```
### 3、基于注解@Transactional声明式
```java
@Transactional
public int save(User user) {
    return userDao.save(user);
}
```

```xml
<!-- 开启事务注解 -->
<tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
```

## 四、Spring的事务传播行为
### 1、什么是Spring的事务传播行为？
Spring的事务传播行为是用于控制在多个事务方法相互调用时事务的行为，它解决的核心问题是，多个声明了事务的方法相互调用的时候存在事务嵌套问题，那么这个事务的行为应该如何进行传递？
**举个例子**：如下，如果methodA()调用methodB()，两个方法都开启了事务，那么methodB()是开启一个新事物，还是继续在methodA()事务中执行？这就取决于**事务传播行为**
```java
@Transaction(Propagation = REQUIRED)
public void methodA() {
    methodB();
    //doSth
}

@Transaction(Propagation = REQUIRED_NEW)
public void methodB() {
    //doSth
}
```

### 2、Spring中事务的传播行为有哪些？
Spring中事务的传播行为有7种，分别是：
* **REQUIRED（默认）**：保证只有一个事务在执行，即如果存在事务则加入之前的事务，不存在事务则开启一个新的事务
* **REQUIRED_NEW**：每次执行都开启新的事务
* **SUPPORTS**：表示支持该事务，即有事务则加入，没有事务则正常执行
* **NOT_SUPPORTED**：表示不支持该事务，即有事务则暂停该事务，没有事务则正常执行
* **NEVER**：表示不用事务，如果有事务则抛出异常
* **MANDATORY**：表示强制用事务，如果没有事务则抛出异常
* **NESTED**：表示嵌套事务，即如果存在事务则嵌套之前的事务中执行，如果不存在事务则新建一个事务（嵌套事务回滚不影响父事务，反之父事务回滚影响嵌套事务）