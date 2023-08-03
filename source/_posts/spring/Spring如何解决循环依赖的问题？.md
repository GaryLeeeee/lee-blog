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

### 1、一级缓存（单例池）
**什么时候使用一级缓存？**：当一个bean完成了完整生命周期，就会放进**一级缓存**，但下次该bean被其他bean依赖注入时，直接从**一级缓存**取出赋值即可
**问题**：如果两个/多个bean相互依赖，那么在依赖注入时，由于分别都没完成完整生命周期，所以**一级缓存**里找不到对应的bean，导致出现死循环
![spring循环依赖](/images/spring/spring循环依赖.png)

### 2、二级缓存（早期bean对象）
**背景**：为了解决循环依赖，我们引入了**二级缓存**，允许一个bean创建后（依赖注入前）<font color=red>提前暴露</font>到缓存中，当该bean被其他bean依赖注入时，先从**一级缓存**找，找不到再从**二级缓存**找，这时候就能取到其原始对象并赋值即可
**问题**：由于从**二级缓存**取出的对象是<font color=red>提前暴露的原始对象</font>，并不是最终的bean。这时候如果该原始对象进行了AOP产生了一个代理对象，这时候就会出现冲突
![spring循环依赖二级花奴才能](/images/spring/spring循环依赖二级缓存.png)
如果所示，A生成了一个原始对象，B依赖注入A时赋值了该原始对象，而A如果在进入单例池前进行了AOP（BeanPostProcessor），那么就会出现矛盾：
* 对于A而言：A的bean是AOP后的代理对象
* 对于B而言，B依赖注入的A是原始对象，而不是AOP后的代理对象

所以这时候，经过了Spring的BeanPostProcessor对bean的加工，**B依赖的A和最终的A不是同一个对象**，我们可以通过一段代码证明一下：
```java
@Component 
public class A {
}

@Component
public class TestBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        // 打印前面的值
        System.out.println(bean);

        if (beanName.equals("a")) {
            // 这里生成了一个新的对象
            return new A();
        }
        return bean;
    }
}

public class Test {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(XXX.class);
        
        A a = context.getBean("a", A.class);
        // 打印后面的值
        System.out.println(a);
    }
}
```

运行main方法，得到打印如下：
```
com.test.service.A@4b0393b9
com.test.service.A@2e052e20
```
所以，BeanPostProcessor中可以完全替换掉某个beanName对应的bean对象：
* 循环依赖注入的对象是发生在填充属性过程中的
* BeanPostProcessor的执行是发生在填充属性过程之后的

<font>这种情况下的**循环依赖**，Spring是解决不了的，因为在填充属性时，Spring也不知道该对象后续会经过哪些BeanPostProcessor以及会对A对象做什么处理</font>
虽然这种情况可能发生，但是肯定发生得很少，我们一般在开发中不会这样做。但是，有种情况，某个beanName对应的最终对象和原始对象不是同一个对象却经常出现，这就是**AOP**

### 3、AOP
**AOP**是通过一个BeanPostProcessor来实现，也就是`AnnotationAwareAspectJAutoProxyCreator`，它继承了`AbstractAutoProxy`
在Spring中AOP利用的要么是JDK动态代理，要么是CGLib的动态代理，所以<font color=red>如果给一个类中的某个方法设置了切面，那么这个类最终就需要生成一个代理对象</font>

（上面有提到）Spring中AOP的一般流程：生成bean->实例化->填充属性->AOP->放入单例池

而要解决AOP导致的前后对象不一致的问题，就需要引入**三级缓存**

### 4、三级缓存（对象工厂）
**三级缓存**对应源码的`singletonFactories`，在bean生成完原始对象后，会构造一个ObjectFactory存入`singletonFactories`
`ObjectFactory`是一个函数式接口，所以支持lambda表达式：`() -> getEarlyBeanReference(beanName, mbd, bean)`

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




