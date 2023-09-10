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
* ...