---
title: synchronized和ReentrantLock有什么区别？
date: 2023-09-10 19:30:09
tags: [锁]
categories: [并发]
---

## 一、synchronized和ReentrantLock有什么区别？
synchronized和ReentrantLock的区别有以下几点：
* **底层实现**：synchronized是Java关键字，ReentrantLock是Java API
* **手动获取/释放**：synchronized不需要手动获取/释放锁，ReentrantLock需要手动获取/释放锁
* **是否公平锁**：synchronized只有非公平锁，ReentrantLock有公平锁和非公平锁
* **实现原理**：synchronized的实现涉及到锁的升级，具体为无锁、偏向锁、轻量级锁、重量级锁等；ReentrantLocal的实现是通过CAS自旋机制保证线程操作的原子性和volatile保证数据可见性以实现锁的功能
* **是否可中断**：synchronized是不可中断类型的锁，ReentrantLock则可以中断，可通过`trylock(long timeout, TimeUnit unit`设置超时方法或者将lockInterruptibly()放到代码块中，调用interrupt方法进行中断
* **是否可重入**：synchronized和ReentrantLock都是可重入的
* ...

## 二、Q&A
### 1、什么是可重入锁？
可重入锁，也叫做递归锁，指的是一个线程在获取一个对象锁后，如果在还没释放锁之前还想再获得该锁，那么直接可以获得。相反的，如果不是可重入锁，那么重复获得锁的时候就会出现**死锁**。

`synchronized`和`ReentrantLock`都是可重入锁，简单举个例子：
```java
public class SynchronizedDemo {
    public synchronized void method1() {
        method2();
    }
    
    public synchronized void method2() {
        // ...
    }
}
```

**解析**：`method1()`和`method2()`都用了`synchronized`修饰，所以他们锁住的是当前对象`this`，当`method1()`调用`method2()`时，判断两个方法都是被当前对象占有，所以可以直接获得锁并执行

### 2、什么是公平锁和非公平锁？
简单来说，公平锁就是线程按顺序获得锁（先等待先获得），非公平锁就是随机获得锁（可能造成某个线程等待太久）。

`synchronized`是非公平锁，`ReentrantLock`是公平锁，通过`ReentrantLock(boolean fair)`构造方法来指定。