---
title: GC学习笔记
date: 2023-07-12 16:50:22
tags: [JVM,GC]
categories: [JVM]
---

## 一、介绍
**背景**：我们在Java开发中，会不断创建很多的对象，这些对象会占用系统内存，如果得不到有效的管理，内存的占用会越来越多，甚至会出现内存溢出的情况，所以，我们需要进行对内存合理地释放，

**介绍**：GC，也就是Garbage Collection（垃圾收集），是JVM提供的一种对内存回收的机制，它一般会在内存空闲或者内存占用过高的时候对那些没有任何引用的对象不定时地进行回收

## 二、垃圾回收算法
### 1、标记清除算法（Mark-Sweep）
**介绍**：标记所有需要回收的对象，并清理
**缺点**：
* 会产生大量内存碎片，导致后续可能大对象找不到连续可用空间
* 标记、清理效率不高，需要扫描所有对象（堆越大，gc越慢）
![标记清除算法](/images/jvm/gc标记清除算法.png)

### 2、标记复制算法（Mark-Copy）
**介绍**：把内存分为等大小的两块，每次只使用其中一块，当这一块内存满后将存活对象复制到另一块去，然后清理，反过来亦然
**优点**：
* 不会产生内存碎片

**缺点**：
* 可用空间缩小为原来一半（即需要双倍空间）
* 在对象存活率高时，效率低下
![标记复制算法](/images/jvm/gc标记复制算法.png)

### 3、标记整理算法（Mark-Compact）
**介绍**： 标记后，将存活对象整理到一起，然后再清理
**优点**：
* 不会产生内存碎片
* 不用额外的内存空间

**缺点**：
* 相比标记清除算法耗时更多
![标记整理算法](/images/jvm/gc标记整理算法.png)


## 三、CMS垃圾收集器
### 1、介绍
* CMS（Concurrent Mark Sweep）收集器是一种以<font color=red>获取最短回收时间</font>为目标的收集器
* 面向老年代
* 基于标记清除算法实现

### 2、步骤
* **初始标记**：标记gc roots直接关联的对象，这个过程需要STW
* **并发标记**：标记所有对象，这个过程会和用户线程并发执行
* **重新标记**：由于`并发标记`是并发的，可能会有部分引用异常，所以需要重新标记，来标记`并发标记`中变动过的少数对象，这个过程需要STW
* **并发清除**：并发清理，这个过程会和用户线程并发执行

## 四、常见问题
### 1、如何判断哪些垃圾能回收？
**可达性分析算法**：从整个堆内存的根对象出发，看看哪些对象是可达的，哪些对象是不可达的
**哪些可以作为根对象（GC ROOTS）**：
* 栈中引用的对象（如局部变量等）
* 类静态属性引用的对象（如java类的引用类型静态变量）
* 常量引用的对象（如字符串常量池中的引用）
* synchronized修饰的对象
* native修饰的对象
* ...


