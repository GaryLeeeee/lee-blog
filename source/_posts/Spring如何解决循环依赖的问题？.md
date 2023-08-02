---
title: Spring如何解决循环依赖的问题？
date: 2023-08-01 15:57:52
tags:
---

## 一、什么是循环依赖？
**循环依赖**其实就是两个对象互相依赖，包括直接和间接，比如
* **直接**：A依赖B，B依赖A
* **间接**：A依赖B，B依赖C，C依赖A
```java
// A依赖B
class A {
  public B b;
}
// B依赖A
class B {
  public A a;
}
```
在Spring中，一个对象经过一系列的**Bean的生命周期**，就会出现**循环依赖**问题

## 二、Bean的生命周期（**简易版**）
Bean就是被Spring管理的对象，生成步骤如下：
1. **生成bean**：执行getBean()，根据BeanDefinition生成bean
2. **实例化**：根据其构造方法（默认是无参）反射创建一个对象
3. **填充属性**：也就是**依赖注入**
4. **AOP**：如果该对象某个方法被AOP了，那么就要基于该对象生成一个代理对象
5. **放入单例池**：也叫做一级缓存（singletonObjects），后续getBean直接从单例池取即可

## 三、为什么会出现循环依赖？
**一般情况**：
1. A创建了一个原始对象（类似new）
2. 给属性b赋值，根据B的类型和属性名去BeanFactory获取B类对应的单例bean 、
   2.1 如果存在B类对应的单例bean， 则直接赋值给b
   2.2 如果不存在B类对应的单例bean，则先创建该bean，再赋值给b
3. ...
```java
class A {
    public B b;
}
```

**特殊情况（循环依赖）**：如果A在填充b属性的时候，要去创建B对象，但是B又需要填充a属性，此时A还在创建中，导致A和B都创建不了
```java
class A {
    public B b;
}
class B {
    public A a;
}
```

而在Spring中，使用了**三级缓存**来解决**循环依赖**的问题

## 四、三级缓存
**三级缓存**分别为：
* **一级缓存**：即源码中的singletonObjects，也叫做单例池。存放的对象是<font color=red>完成了完整生命周期的bean对象</font>
* **二级缓存**：即源码中的earlySingletonObjects。存放的对象是<font color=red>早期的bean对象，即没完成完整生命周期的bean对象</font>
* **三级缓存**：即源码中的singletonFactories，也叫做对象工厂。存放的对象是<font color=red>创建对象的lambda表达式</font>，用来创建某个对象的


//TODO

## 笔记
* bean工厂：ObjectFactoru
* bean仓库/单例池：SingletonBeanRegistry

bean创建步骤：
* 执行getBean
* 实例化：createBeanInstance，用构造方法
* 填充属性：populateBean
* 初始化：执行initializeBean

a依赖b的话：
* 执行getBean("a")
* 实例化：createBeanInstance("a")   //放在半成品池
* 填充属性：populateBean("a")
    * 执行getBean("b")
    * 实例化：createBeanInstance("b")
    * 填充属性：populateBean("b")
    * 初始化：执行initializeBean("b")
* 初始化：执行initializeBean("a")     //放在单例池

后面还有工厂池

一级缓存：singletonObjects，用于保存实例化、注入、初始化完成的bean实例
二级缓存：earlySingletonObject，用于保存实例化完成的bean实例
三级缓存：singletonFactories，用于保存bean创建工厂，便于后续创建代理对象


场景：
A依赖A
A依赖B，B依赖A
A依赖B，B依赖C，C依赖A

顺序加载，如TestService1会先比TestService2加载

上课笔记：
* @Autowired的reequired为true，说明一定要赋值，不能为null
* 赋值给@Autowired对象的是service的普通对象还是代理对象？  其实是代理对象
* 三级缓存存的lambda表达式，判断是否要aop，如果不要就返回普通对象，如果要就返回代理对象
* 为什么会有提前aop和后续aop？
* 用构造方法去创建对象，如果自定义了构造方法，那么默认的无参构造方法就不生效了，如果这个自定义的有参构造方法包含了循环依赖的service类，那么就会报错，因为生成不了普通对象。加@Lazy可解决


为什么@Lazy可以？
* 生成形参的代理对象：比如public AService(BService bService)，则自动生成bService代理对象
* 




