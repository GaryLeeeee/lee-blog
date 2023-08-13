---
title: Java动态代理
date: 2023-08-12 22:32:14
tags: [动态代理]
categories: [Java]
---

## 一、什么是代理？
**代理**是一种设计模式（代理模式），当我们要访问某个目标类的时候，不是直接访问目标类，而是通过访问代理类，然后代理类调用目标类来完成的，也就是<font color=red>直接调用变成间接调用</font>

### 1、代理的好处
* **解耦**
* **方便扩展**：不修改目标类逻辑的前提下，起到增强逻辑的作用

### 2、使用场景
可以在代理类调用目标类之前和之后去添加一些预处理和后处理的操作，达到扩展一些不属于目标类功能的目的，比如：
* 可以在方法开始和结束前打印日志，如`Logger.info("method before")`
* 可以在方法执行前进行参数校验，如`Assert.notNull(req)`
* 可以在方法执行前进行权限校验，如`checkUserAuth()`
* ...

### 3、代理的类型
**代理**分为：
**静态代理**：在程序运行前，就已经给目标类编写了代理类的代码（提前编译了代理类并生成对应的字节码）
```java
interface Animal {
    void sound();
}

class Dog implements Animal {
    public void sound() {
        System.out.println("汪");
    }
}

class StaticProxy implements Person {
    private Animal animal;
    
    public StaticProxy(Animal animal) {
        this.animal = animal;
    }
    
    @Override
    public void sound() {
        System.out.println("method before");
        animal.sound();
        System.out.println("method after");
    }
}

public class Test {
    @Test
    public void testStaticProxy() {
        // 正常情况下
        Animal dog = new Dog();
        dog.sound();
        // 静态代理下
        StaticProxy staticProxy = new StaticProxy(dog);
        staticProxy.sound();
    }
}
```
**JDK Proxy动态代理**：不需要提前创建代理类，而是通过反射自动生成代理对象（在Java中只需要实现InvocationHandler接口，然后实现invoke()方法即可）
```java
class DynamicProxy implements InvocationHandler {
    private Object targetObj;
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("method before");
        Object invoke = method.invoke(targetObj, args);
        System.out.println("method after");
    }
    
    // 创建代理对象实例
    public Object getInstance(Object targetObj) {
        this.targetObj = targetObj;
        return Proxy.newProxyInstance(targetObj.getClass().getClassLoader(),    // 目标类加载器
                targetObj.getClass().getInterfaces(),                           // 目标类接口
                this                                                            // 代理处理类
        );      
    }
}

public class Test {
    @Test
    public void testDynamicProxy() {
        List<String> dynamicProxyList = (List) new DynamicProxy().getInstance(new ArrayList<String>());
        dynamicProxyList.add("test");

        System.out.println(dynamicProxyList);
    }
}
```
**CGLib动态代理**：不需要提前创建代理类，只需要实现MethodInterceptor接口，然后实现intercept()方法即可）
```java
class CglibProxy implements MethodInterceptor {
    private Class<?> target;
    
    public CglibProxy(Class<?> target) {
        this.target = target;
    }

    /**
     * 
     * @param obj       代理类（子类）
     * @param method    调用的方法
     * @param args      入参
     * @param proxy     MethodProxy代理对象
     * @return
     */
    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) {
        System.out.println("method before");
        proxy.invokeSuper(obj, args);
        System.out.println("method after");
        return obj;
    }
}

public class Test {
    @Test
    public void testCglibDynamicProxy() {
        // 实例化增强器
        Enhancer enhancer = new Enhancer();
        // 设置需要代理的目标类
        enhancer.setSuperClass(Dog.class);
        // 设置拦截对象回调的实现类
        enhancer.setCallback(new CglibProxy(Dog.class));
        // 调用create()方法生成代理类
        Dog dog = (Dog) enhancer.create();
        
        dog.sound();
    }
}
```

## 二、为什么要用动态代理？
**静态代理**的缺点：
* 需要提前创建，一个目标类就要对应有一个代理类，造成代码冗余
* 同一个代理类下需要实现接口的所有方法，编码效率低下

**动态代理**的优点：
* 减少代理对象个数，降低程序复杂度
* 方便动态扩展

## 三、动态代理的基本实现（JDK Proxy）
动态代理的基本实现为：
1. 拿到目标类的引用，通过反射获取它所有的接口
2. 通过JDK Proxy类重新生成一个新的类，实现了目标类所有接口的方法
3. 动态生成Java代码，把增强逻辑加入到新生成代码中
4. 动态编译新生成Java代码的class文件
5. 加载并重新运行新的class，得到全新类

## 四、JDK Proxy和CBLib的区别？
**实现方式**：JDK Proxy是实现了目标类的接口，CGLib底层是继承了目标类（所以如果目标类被final修饰，那么它是无法使用CBLib做动态代理的）
**生成字节码**：JDK Proxy和CGLib都是在运行时生成字节码
**调用方式**：JDK Proxy是通过反射机制调用，CGLib是通过FastClass机制调用（子类调用父类）
**class文件**：JDK Proxy只会生成一个class文件，CGLib会生成多个class文件（代理类+代理类对应的fastclass文件+目标类对应的fastclass文件）
**性能**：JDK1.8之前CGLib性能更好，因为JDK1.8之前JDK Proxy要通过JDK内部反射逻辑去找到目标类，而JDK1.8之后对反射性能做了优化（跟CGLib差不多）
**目标类调用本类方法处理**：JDK Proxy目标类调用本类方法，只会代理一次（<font color=red>所以会导致AOP失效</font>），而CGLib由于是继承了代理类，通过invokeSuper()方法调用父类方法，在目标类中调用本类方法会再一次代理（如果只需要代理一次可以用invoke()方法）
```java
public class Dog {
    public void method1() {
        method2(); //CGLib动态代理会执行子类method2()方法，因为method1()也是子类触发调用的
        //...
    }
    
    public void method2() {
        //...
    }
}
```


## 五、其他
### 1、如何查看动态代理生成出来的代理类文件？
如果用的是JDK Proxy，可以通过设置环境变量`sun.misc.ProxyGenerator.saveGeneratedFiles`为`true`，会自动生成一个`$Proxy`代理类文件到根目录里
```java
public class Test {
    @Test
    public void test() {
        System.setProperty("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");
        //...
    }
}
```

如果用的是CGLib，可以通过设置环境变量`DebuggingClassWriter.DEBUG_LOCATION_PROPERTY`为`./`，会自动生成多个`xx$$EnhancerByCGLIB%%xx`代理类文件到根目录里
```java
public class Test {
    @Test
    public void test() {
        System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "./");
        //...
    }
}
```

### 2、如何选择用JDK Proxy还是CGLib？
如果目标类实现了接口，则用JDK Proxy（使用Spring AOP会自动选择更优的，SprintBoot 2.x的AOP默认使用的是CGLib）

//TODO：代理流程图