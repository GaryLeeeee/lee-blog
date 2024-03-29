---
title: synchronized的锁优化是怎样的？
date: 2023-09-11 16:46:25
tags: [锁]
categories: [并发]
---

## 一、synchronized的锁优化包括哪些？
synchronized的锁优化主要包括：
* 自旋锁
* 锁清除
* 锁粗化
* ...

## 二、synchronized的锁优化过程
### 1、自旋锁
**例子**：比如我们去银行取钱，我们有两种方式如下：
* **站在取款机前排队等待**：相当于自旋锁，需要经常留意（CAS）是否轮到自己了
* **取号后去休息区等待叫号**：相当于阻塞锁，需要等待叫号（被唤醒）

**总结**：
* 自旋锁可以避免等待竞争锁进入阻塞挂起状态被唤醒造成的内核态和用户态之间切换的损耗
* 具体实现是通过自旋的方式，但避免如果锁被其他线程长时间占用而导致自旋过久，所以会有一个最大自旋次数（默认为10，动态根据上一次自旋变化），达到了就停止自旋
* 自旋锁和阻塞锁的最大区别是要不要放弃CPU的执行时间，阻塞锁是放弃CPU，进入了等待区，等待被唤醒；而自旋锁是占有CPU，通过CAS时刻检查是否可获取锁

### 2、锁消除
**例子**：比如我们去银行取钱，如果办理业务的人不多，那么根本不用取号（加锁），直接去进行业务办理即可
**总结**：
* 锁消除是JIT编译器对内部锁的一种优化，通过**逃逸分析**判断锁是否只能被当前线程访问，如果是的话则会做锁消除
```java
// 优化前
public void removeLock() {
    // 由于锁对象是局部变量，属于栈内线程私有
    Object lockObj = new Object();
    synchronized(lockObj) {
        // do sth    
    }
}

// 优化后
public void removeLock() {
    // 由于锁对象是局部变量，属于栈内线程私有
    Object lockObj = new Object();
    // do sth
}
```

### 3、锁粗化
**例子**：比如我们去银行取钱，要从两张卡里取两次钱，如果我们每次取钱都去取号、排队，那么会浪费很多时间，最好的做法是一次取号排队（拿锁）后，就把多次取钱业务给完成了就行
**总结**：
* 锁粗化的目的是减少在一段代码中反复对同一对象加锁解锁，通过增大锁的范围，减少性能消耗
```java
// 优化前
public void incrLock() {
    for(int i=0;i<10000;i++) {
        synchronized(this) {
            // do sth
        }    
    }
}

// 优化后
public void incrLock() {
    synchronized(this) {
        for(int i=0;i<10000;i++) {
            // do sth
        }
    }
}
```

* 当然不是锁的粒度越大越好，一些场景下还是要减少锁粒度的，比如我们去银行取钱，排到我们时，我们因为忘记了密码所以花了十分钟时间去记起来，那么如果我们可以在取号前就记起密码，那么就不会浪费太多时间在办理业务上，从而导致后面的人等太久