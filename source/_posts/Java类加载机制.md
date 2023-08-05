---
title: Java类加载机制
date: 2023-08-04 22:00:10
tags:
---

## 一、背景

## 二、Java中有哪些类加载器？
JDK有三个类加载器，分别为：
* **启动类加载器（BootStrapClassLoader）**：负责加载Java运行时环境核心类库（即%JAVA_HOME%/lib下的jar包和class类），是`ExtClassLoader`的父类加载器，
* **扩展类加载器（ExtClassLoader）**：负责加载Java运行时环境扩展类库（即%JAVA_HOME%/lib/ext下的jar包和class类），是`AppClassLoader`的父类加载器
* **应用类加载器（AppClassLoader）**：负责加载classpath下的类文件，是自定义类加载器的父类加载器

## 三、类加载过程
//TODO
**类加载过程**主要有加载、验证、准备、解析、初始化
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
