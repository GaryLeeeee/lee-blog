---
title: Spring Bean的生命周期
date: 2023-08-22 16:19:18
tags:
categories: [Spring]
---

## 一、Spring Bean的生命周期
**Spring Bean的生命周期**主要分为五个阶段：
* 准备阶段
* 实例化（Instantiation）
* 属性赋值（Populate）
* 初始化（Initialization）
* 销毁（Destruction）

### 1、准备阶段
在准备阶段，主要是为了在开始加载Bean之前，从Spring配置中解析并查找Bean有关的配置内容，加载顺序为：
1. 实例化BeanFactoryPostProcessor（BeanFactory后置处理器）
2. 实例化BeanPostProcessor（Bean后置处理器）
3. 实例化BeanPostProcessor实现类
4. 实例化InstantiationAwareBeanPostProcessorAdapter实现类（实例化感知的Bean后置处理器）
5. 执行InstantiationAwareBeanPostProcessor的postProcessBeforeInstantiation()方法（在实例化之前的后置处理）

以上这些类或者配置是Spring提供给开发者，用来实现Bean加载过程中的扩展机制，在很多和Spring继承的中间件经常使用，如Dubbo

另外，在准备阶段最重要的是会加载Bean的定义，流程如下：
1. 扫描xml/注解配置的bean
2. 生成对应的BeanDefinition
3. 将BeanDefinition放入beanDefinitionMap

### 2、实例化
在实例化阶段，主要是创建一个普通对象，大概流程为：
1. 扫描beanDefinitionMap
2. 遍历拿到BeanDefinition通过反射拿到类的构造方法
3. 根据构造方法创建一个普通对象（只new没赋值）
4. 执行InstantiationAwareBeanPostProcessor的postProcessPropertyValues()方法

### 3、属性赋值（依赖注入）
在属性赋值阶段，主要是对bean的属性进行依赖注入，大概流程为：
1. 扫描@Autowired/@Resource等注解的属性
2. 对扫描出来需要赋值的属性进行依赖注入

另外，Spring是通过三级缓存来解决循环依赖的问题，具体可看[《Spring如何解决循环依赖的问题？》](https://garyleeeee.github.io/2023/08/01/spring/spring-ru-he-jie-jue-xun-huan-yi-lai-de-wen-ti/)
### 4、初始化
在初始化阶段，主要是执行一些aware、init等初始化的工作，大概流程为：
1. aware：执行实现了BeanNameAware、BeanFactoryAware、ApplicationContextAware等接口的方法，如set
2. initMethods：
    a. 前置处理，如@PostConstuct
    b. 执行实现了InitializingBean接口的afterPropertiesSet方法
    c. 执行BeanPostProcessor的自定义initMethod方法
    d. 后置处理，如@PreDestory
    e. 实现DisposableBean，用于后面销毁调用
3. 加入单例池

### 5、销毁
1. 执行postProcessBeforeDestruction（销毁前处理器）：会执行bean中@PreDestory注释的方法
2. 执行destroyBeans：销毁单例池里的bean
3. 执行invokeCustomDestroyMethod（执行客户自定义销毁方法）：如实现了DisposableBean的destroyMethod方法

---
//TODO待优化