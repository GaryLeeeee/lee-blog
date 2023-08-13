---
title: 介绍一下Spring的AOP
date: 2023-08-13 16:22:00
tags: [AOP,动态代理]
categories: [Spring]
---

## 一、什么是AOP？
AOP（Aspect-Oriented Programming），一般称为面向切面编程，用来将与业务无关，但却对多个对象产生影响的公共行为和逻辑，抽取并封装成一个可重用的模块，这个模块就叫做**切面**
**AOP的优点**：
* 减少系统中的重复代码
* 降低模块间的耦合度
* 提高系统的可维护性

**AOP的场景**：
* 权限认证
* 日志处理
* 事务处理
* ...

## 二、Spring AOP的一些名词
* **切面（Aspect）**：切面由切点和通知组成，在Spring AOP中切面就是切面类，如日志处理和事务处理就可以理解为两个切面
* **连接点（JoinPoint）**：指的是被增强的业务方法
* **切点（PointCut）**：决定通知应该作用于哪些方法（充当where角色，即在哪里做）
* **通知（Advice）**：指的是切面的具体行为（充当what角色，即做什么）
* **目标对象（Target）**：指的是增强的对象（目标对象）
* **织入（Weaving）**：指的是将切面和业务逻辑对象连接，并创建通知代理的过程（Spring AOP的织入方式为动态代理）

## 三、Spring有哪些通知类型？
Spring有5种通知类型，分别为：
* **前置通知（Before）**：在目标方法调用之前通知
* **后置通知（After）**：在目标方法完成之后通知（不关心是否出现异常）
* **返回通知（After-returning）**：在目标方法成功执行之后通知（不出现异常）
* **异常通知（After-throwing）**：在目标方法抛出异常后通知（出现异常）
* **环绕通知（Around）**：在目标方法调用之前和调用之后通知

执行顺序：
* **正常**：@Before -> 方法 -> @AfterReturning -> @After
* **异常**：@Before -> 方法 -> @AfterThrowing -> @After

## 四、Spring AOP的流程
//TODO

## 五、Q&A
### 1、AOP和OOP有什么区别？
AOP和OOP是面向不同领域的两种设计思想，具体为：
**OOP（面向对象编程）**：针对业务处理过程的实体及其属性和行为进行抽象封装，以获得更加清晰高效的逻辑单元划分
**AOP（面向切面编程）**：作为OOP的一种补充，针对业务处理过程中的切面进行提取，以达到业务代码和公共代码之间低耦合性的隔离效果

### 2、Spring AOP和AspectJ AOP有什么区别？
* Spring AOP基于动态代理来实现，致力于解决企业级开发中普遍的AOP需求（方法织入）。默认地，如果使用接口，用JDK提供的动态代理实现，如果没有接口，使用CGLib实现，
* AspectJ AOP基于静态代理来实现，是AOP编程的完全解决方案

### 3、JDK动态代理和CGLib动态代理有什么区别？
[Java动态代理](https://garyleeeee.github.io/2023/08/12/java-dong-tai-dai-li/)

### 4、什么情况下AOP会生效？
* 目标类没有配置为bean
* 方法用private修饰（需要改用public）
* 切点表达式配置错误
* ...
//TODO
  
### 5、Spring AOP是在哪里进行动态代理的？
正常的Bean会在Bean生命周期的初始化后，通过`BeanPostProcessor.postProcessAfterInitialization`创建AOP的动态代理
参考：[Spring如何解决循环依赖的问题？](https://garyleeeee.github.io/2023/08/01/spring/spring-ru-he-jie-jue-xun-huan-yi-lai-de-wen-ti/)