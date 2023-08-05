---
title: Java类加载机制
date: 2023-08-04 22:00:10
tags: [双亲委派模型]
---

## 一、背景

## 二、Java中有哪些类加载器？
JDK有三个**类加载器**，分别为：
* **启动类加载器（BootStrapClassLoader）**：负责加载Java运行时环境核心类库（即`%JAVA_HOME%/lib`下的jar包和class类），是`ExtClassLoader`的父类加载器，
* **扩展类加载器（ExtClassLoader）**：负责加载Java运行时环境扩展类库（即`%JAVA_HOME%/lib/ext`下的jar包和class类），是`AppClassLoader`的父类加载器
* **应用类加载器（AppClassLoader）**：负责加载classpath下的类文件，是自定义类加载器的父类加载器

## 三、类加载过程
**类加载过程**其实就是把class文件装载到JVM内存中得到一个Class对象，然后我们就可以通过new关键字来实例化这个对象，主要有加载、验证、准备、解析、初始化这几个阶段
![](/images/jvm/类加载过程.png)
**加载阶段**：
1. 根据类的全限定名来查找并读取类的二进制字节流
2. 将这个字节流转换为内部数据结构（即方法区的运行时数据结构）
3. 在JVM堆中创建一个Class对象代表这个类（作为方法区这些数据的访问入口）
```java
public class TestClass {
    Class<?> clazz = CLass.forName("com.test.TestClass");
}
```

**验证阶段**：
1. 文件格式验证：是否符合Class文件格式的规范，并且能被当前版本的虚拟机处理
2. 元数据验证：是否符合Java语言规范要求
3. 字节码验证：保证被校验类的方法在运行时不会做出危害虚拟机安全的行为
4. 符号引用验证：对类自身以外（常量池中的各种符号引用）的信息进行匹配性校验，确保解析动作能正常执行

**准备阶段**：为类变量（静态变量）分配内存空间，并设置默认初始值
```java
public class TestClass {
    public static int testStaticVariable;
}
```

**解析阶段**：
* 目的：将符号引用解析为直接引用
* 符号引用：用来引用类、字段、方法等
* 直接引用：用来直接指向内存中的数据结构的指针或偏移量
```java
public class TestClass {
    public static void testStaticMethod() {
        // 静态方法的解析
    }
}
```

**初始化阶段**：类加载过程的最后一步，也是类被真正使用之前的最后准备工作。主要是对类的静态变量进行赋值和静态代码块的执行
```java
public class TestClass {
    public static int testStaticVariable = 10;
    
    static {
        // do sth
    }
}
```

## 四、双亲委派机制
![](/images/jvm/类加载双亲委派模型.png)
### 1、介绍
* 每一个类都有一个对应它的类加载器，系统中的类加载器在协同工作的时候会默认使用**双亲委派模型**
* 类加载的时候，系统会先判断该类是否被加载过
    * 如果被加载过，会直接返回
    * 如果没被加载过，会尝试加载
* 加载流程：
    1. 首先会把请求委派给父类加载器的loadClass()处理，因此所有的请求最终都会委派给最顶层的`BootstrapClassLoader`
    2. 然后如果父类加载器无法处理时，就会交回自己来处理，如果能处理则直接加载完返回
* 如果父类加载器为`null`时，会使用`BootstrapClassLoader`作为父类加载器

### 2、使用双亲委派机制的好处
使用**双亲委派机制**的好处为：
* 保证JDK核心类的优先加载
* 避免类的重复加载
* 保证Java的核心API不被篡改

如果不用**双亲委派模型**，而是每个类加载器加载的话就会出现一些问题，比如我们编写一个java.lang.Object类时，那么程序运行的时候，系统就会出现多个不同的Object类

### 3、破坏双亲委派模型
**破坏双亲委派模型**的方式有：
* **自定义类加载器**：继承ClassLoader抽象类，重写loadClass方法（可以自定义要加载的类使用的类加载器）
* **使用线程上下文加载器**：可以通过`java.lang.Thread`类的setContextClassLoader()方法来设置当前

### 4、为什么要破坏双亲委派模型？
**为什么要破坏双亲委派模型？**：由于加载范围的限制，顶层的ClassLoader无法访问底层ClassLoader所加载的类，此时就需要破坏双亲委派模型

> 以JDBC为例，讲一下为什么要破坏双亲委派模型
#### a、JDBC不使用Java SPI
```java
public class TestSqlClass {
    public static void main(String[] args) {
        // 加载驱动类
        Class.forName("com.mysql.jdbd.Driver");
        // 获取数据库连接
        Conncetion connection = DriverManager.getConnect("jdbc:mysql://xxx", "xxx", "xxx");
    }
}
```

#### b、JDBC使用Java SPI
在JDBC4.0之后，开始支持使用SPI的方式来注册Driver，具体做法是在MySQL的jar里的`META-INF/services/java.sql.Driver`文件中指定当前使用的Driver是哪个
**什么是SPI？**：SPI就是策略模式，根据配置来决定运行时接口的实现类是哪个
![](/images/jvm/类加载JDBC.png)
用这种方式的话，当使用不同的驱动时，我们不需要手动通过Class.forName加载驱动类，而是引入对应的jar包即可，所以上面的代码可以改成如下形式
```java
public class TestSqlClass {
  public static void main(String[] args) {
    Connection connection = DriverManager.getConnect("jdbc:mysql:/xxx", "xxx", "xxx");
  }
}
```

> 对应的驱动类是如何加载的？

* 从META-INF/services/java.sql.Driver文件中获取具体的实现类`com.mysql.jdbd`
* 通过`Class.forName("com.mysql.jdbc.Driver")`将这个类加载进来

> 存在什么问题呢？

* `DriberManager`位于rt.jar包中，所以是通过`BootStrapClassLoader`加载的
* 而`Class.forName()`加载用的是调用者的ClassLoader
如果用`BootStrapClassLoader`去加载`com.mysql.jdbc.Driver`，是肯定加载不到的（<font color=red>因为一般情况下启动类加载器只加载rt.jar包中的类</font>）
  
> 如何解决这个问题呢？

想让顶层的ClassLoader加载底层的ClassLoader，只能**破坏双亲委派机制**
DriverManager在加载时，会先执行静态代码块，在静态代码块中，会执行loadInitialDrivers()方法，这个方法会加载对应的驱动类
```java
public class DriverManager {
    // ...
    static {
        loadInitialDrivers();
        println("JDBC DriverManager initialized");
    }
    
    // ...
    private static void loadInitialDrivers() {
        //...
        AccessController.doPrivileged(new PrivilegedAction<Void>() {
            public void run() {
                // 根据配置文件加载驱动实现类
                ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
                Iterator<Driver> driversIterator = loadedDrivers.iterator();
                
                try {
                  while (driversIterator.hasNext()) {
                      driversIterator.next();
                  }
                } catch (Throwable t) {
                    //Do nothing
                }
                return null;
            }
        });

        println("DriverManager.initialize: jdbc.drivers = " + drivers);
        //...
    }
}
```
进入ServiceLoader的load()方法，可以看到通过执行`Thread.currentThread().getContextClassLoader()`获取了线程上下文加载器
```java
public static <S> ServiceLoader<S> load(Class<S> service) {
    ClassLoader cl = Thread.currentThread().getContextClassLoader();
    return ServiceLoader.load(service, cl);
}
```
线程上下文类加载器可以通过`Thread.setContextClassLoader()`方法设置（默认是`AppClassLoader`，通过Launcher类创建，ExtClassLoader同理）
```java
public class Launcher {
    //...
    public Launcher() {
        Launcher.ExtClassLoader var1;
        try {
            var1 = Launcher.ExtClassLoader.getExtClassLoader();
        } catch (IOException var10) {
            throw new InternalError("Could not create extension class loader", var10);
        }
        
        try {
            this.loader = Launcher.AppClassLoader.getAppClassLoader(var1);
        } catch (IOException var9) {
            throw new InternalError("Could not create application class loader", var9);
        }
        
        // 设置上下文类加载器为AppClassLoader
        Thread.currentThread().setContextClassLoader(this.loader);
        
        //...
    }
}
```
可以看到，线程上下文类加载器让父类加载器能通过调用子类加载器来加载类，达到<font color=red>打破双亲委派模型</font>的目的