---
title: 什么是AQS？
date: 2023-10-29 13:21:51
tags: [AQS]
categories: [并发]
---

## 一、什么是AQS？
AQS也就是`AbstractQueuedSynchronizer（抽象队列同步器）`，放在`java.util.concurrent.locks`包下。
![](/images/concurrent/AQS.png)

AQS是一个抽象类，大概定义如下：
```java
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer implements Serializable {
    
} 
```

它是很多同步器的基础框架，比如`ReentrantLock`、`CountDownLoatch`、`Semaphore`等。
![](/images/concurrent/AQS实现.png)

## 二、AQS实现原理
在AQS内部，维护了：
* **state变量**：用`volatile int`修饰，当state大于0的时候说明对象锁已经被占有了，state的修改动作是通过**CAS**来完成的
* **工作队列**：FIFO队列，用来实现线程的排队工作，当线程加锁失败时，该线程会封装成一个**Node节点**放在队列尾部
* **Node节点**：存放了`int waitStatus`、`Node prev`、`Node next`、`Thread thread`