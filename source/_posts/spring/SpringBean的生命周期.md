---
title: Spring Bean的生命周期
date: 2023-08-22 16:19:18
tags:
categories: [Spring]
---

## 一、Spring Bean的生命周期
Spring Bean的生命周期主要分为五个阶段：
* 准备阶段
* 实例化（Instantiation）
* 属性赋值（Populate）
* 初始化（Initialization）
* 销毁（Destruction）

### 1、准备阶段
在准备阶段，主要是为了加载bean的定义，大概流程为：
* 扫描xml/注解配置的bean
* 生成对应的BeanDefinition
* 将BeanDefinition放入beanDefinitionMap

### 2、实例化
在实例化阶段，主要是创建一个普通对象，大概流程为：
* 扫描beanDefinitionMap
* 遍历拿到BeanDefinition通过反射拿到类的构造方法
* 根据构造方法创建一个普通对象（只new没赋值）

### 3、属性赋值
在属性赋值阶段，主要是对bean的属性进行依赖注入，大概流程为：
* 扫描@Autowired/@Resource注解的属性
* 对扫描出来需要赋值的属性进行依赖注入

另外，Spring是通过三级缓存来解决循环依赖的问题，具体可看[《Spring如何解决循环依赖的问题？》](https://garyleeeee.github.io/2023/08/01/spring/spring-ru-he-jie-jue-xun-huan-yi-lai-de-wen-ti/)

### 4、初始化
在初始化阶段，主要是执行一些aware、init等初始化的工作，大概流程为：
* aware：执行实现了BeanNameAware、BeanFactoryAware、ApplicationContextAware等接口的方法，如set
* initMethods：
    * 前置处理，如@PostConstuct
    * 执行实现了InitializingBean接口的afterPropertiesSet方法
    * 执行BeanPostProcessor的自定义initMethod方法
    * 后置处理，如@PreDestory
    * 实现DisposableBean，用于后面销毁调用
* 加入单例池

### 5、销毁
* 执行postProcessBeforeDestruction（销毁前处理器）：会执行bean中@PreDestory注释的方法
* 执行destroyBeans：销毁单例池里的bean
* 执行invokeCustomDestroyMethod（执行客户自定义销毁方法）：如实现了DisposableBean的destroyMethod方法





