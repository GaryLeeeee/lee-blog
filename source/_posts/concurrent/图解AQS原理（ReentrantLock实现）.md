---
title: 图解AQS原理（ReentrantLock实现）
date: 2023-10-30 22:23:57
tags: [AQS]
categories: [并发]
---

## 一、什么是AQS？
具体可参考文章[《什么是AQS？》](https://garyleeeee.github.io/2023/10/29/concurrent/shi-me-shi-aqs/)

## 二、AQS原理
在AQS内部，维护了：
* **state变量**：用`volatile int`修饰，当`state`大于0的时候说明对象锁已经被占有了，`state`的修改动作是通过`CAS`来完成的
* **等待队列**：即CLH队列（FIFO），用来实现线程的排队工作，当线程加锁失败时，该线程会封装成一个**Node节点**放在队列尾部
* **Node节点**：存放了`int waitStatus`、`Node prev`、`Node next`、`Thread thread`

### 1、state变量
`state`变量由`volatile int`修饰，并提供多个方法使用，大概代码如下：
```java
// 共享变量，使用volatile修饰保证线程可见性
private volatile int state;

//返回同步状态的当前值
protected final int getState() {
    return state;
}
// 设置同步状态的值
protected final void setState(int newState) {
    state = newState;
}
//原子地（CAS操作）将同步状态值设置为给定值update如果当前同步状态的值等于expect（期望值）
protected final boolean compareAndSetState(int expect, int update) {
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

### 2、等待队列
如图所示，对于某一资源，每一线程会先添加到等待队列，再按照顺序去尝试获得资源的对象锁。

如果当前线程所在的Node节点获得锁，那么它会释放前驱节点，将`head`指向当前占有锁的线程所在的Node节点。
![](/images/concurrent/AQS等待队列.png)

## 三、图解AQS原理（ReentrantLock实现+非公平锁）
本文主要是来模拟多个线程通过竞争锁、释放锁的场景来解释AQS原理（ReentrantLock实现），具体场景为：**三个线程同时竞争锁、释放锁**。

### 步骤1：线程一加锁成功，线程二和线程三加锁失败
三个线程并发加锁，结果如下：
* **线程一**加锁成功（通过`CAS`将`state`从`0`改为`1`）
* **线程二**和**线程三**加锁失败，加入到**FIFO队列**
![](/images/concurrent/图解AQS原理1.png)
  
此时**AQS队列同步器**数据如下：
* **线程一**通过`CAS`将`state`从`0`改为`1`，并设置当前独占线程`exclusiveOwnerThread`为**线程一**
![](/images/concurrent/图解AQS原理2.png)

此时**FIFO队列**数据如下：
* **线程二**和**线程三**依次添加到**FIFO队列**的队尾
* **双向链表设计**：如**线程二**后置节点是**线程三**、**线程三**前驱节点是**线程二**
* 设置`head`为第一个节点、`tail`为最后一个节点
* （<font color=red>具体为什么**线程二**之前还有一个节点？后续源码分析再说。。。</font>）
![](/images/concurrent/图解AQS原理3.png)

### 步骤2：线程一释放锁，线程二尝试加锁
结果如下：
* **线程一**释放锁，并唤醒`head`节点的后置节点（即**线程二**）
* **线程二**尝试加锁，并成功加锁
![](/images/concurrent/图解AQS原理4.png)

在**线程一**释放锁后，**线程二**加锁前的数据如下：
* **线程一**释放锁，`state`置为0（代表可被加锁），并设置当前独占线程`exclusiveOwnerThread`为null
* `head`节点的后置节点（即**线程二**）被唤醒
![](/images/concurrent/图解AQS原理5.png)

唤醒**线程二**后的数据如下：
* **线程二**被唤醒后，通过`CAS`尝试获得锁（如果没获得就继续被挂起）
* 修改`head`为当前节点（即**线程二**），并回收原`head`节点
![](/images/concurrent/图解AQS原理6.png)

### 步骤3：线程二释放锁，线程三尝试加锁
<font color=red>同理步骤2</font>

## 四、非公平锁有什么问题？什么是公平锁？
像上面的图解是基于**非公平锁**来实现的，也是`ReentrantLock`的默认实现。

**什么是公平锁？什么是非公平锁？**： 简单来说，公平锁就是线程按顺序获得锁（先等待先获得），非公平锁就是随机获得锁（可能造成某个线程等待太久）。

### 1、非公平锁带来的问题
我们先来看下**非公平锁**的流程：
![](/images/concurrent/图解AQS原理7.png)

**正常步骤**：
1. **线程二**释放锁，并唤醒**线程三**
2. **线程三**尝试加锁成功
3. **线程四**尝试加锁失败，并加入到**FIFO队列**

**异常步骤**：
1. **线程二**释放锁，并唤醒**线程三**
2. **线程四**尝试加锁成功
3. **线程三**尝试加锁失败，仍然挂起

**解析**：当释放锁的线程唤醒FIFO队列的线程去尝试加锁时，被新加入的线程抢先加锁并成功了，导致FIFO队列的线程尝试加锁失败并继续挂起

### 2、公平锁
我们先来看下**公平锁**的流程：
![](/images/concurrent/图解AQS原理8.png)

**步骤**：
1. **线程二**释放锁，并唤醒**线程三**
2. **线程四**判断**FIFO队列**是否为空，如果为空则尝试加锁，如果不为空则加入
3. **线程三**尝试加锁成功

**解析**：**公平锁**解决了**非公平锁**中出现的非顺序获得锁的问题（具体实现是通过分开实现`tryAcquire()`方法，<font color=red>具体后面源码分析再说。。。</font>）