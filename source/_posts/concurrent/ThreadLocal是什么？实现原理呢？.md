---
title: ThreadLocal是什么？实现原理呢？
date: 2023-09-08 15:58:15
tags: [空间换时间]
categories: [并发]
---

## 一、ThreadLocal是什么？
ThreadLocal是java.lang包下的一个类，用来解决高并发下线程安全的问题，具体是通过为每一个线程创建一份共享变量的副本来保证各个线程之间的变量的访问和修改互不影响。

比如一次用户的页面请求操作，我们可以在最开始的filter处把用户信息存到ThreadLocal中，后面在同一个请求里，就可以直接通过ThreadLocal拿到该用户信息。

## 二、ThreadLocal的实现原理是什么？
ThreadLocal内部维护了一个ThreadLocalMap，对应的key是ThreadLocal对象，value是我们要保存的值。

## 三、ThreadLocal内存泄露
**背景**：ThreadLocalMap中的key为ThreadLocal的弱引用，而value是强引用
```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;
    
    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```
**问题**：key作为弱引用的对象在下一次垃圾回收的时候必然会被清理掉，而value是强引用不会被清理，这样一来就会出现key为null的value，如果不做任何措施的话那么该value将永远无法被垃圾回收掉，这时候线程如果长时间不被销毁，就可能会出现**内存泄露**
![ThreadLocal内存泄露](/images/concurrent/ThreadLocal内存泄露.png)
**解决**：ThreadLocalMap内部实现在调用set()、get()、remove()的时候，会自动清理掉key为null的记录（线性探测法）。但是如果在出现了null的记录后没有调用set()、get()、remove()方法，还是会出现内存泄露的，所以，<font color=red>我们在用完ThreadLocal方法后，最好手动调用remove()方法</font>