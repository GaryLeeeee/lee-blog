---
title: 什么是volatile？能保证原子性吗？
date: 2023-10-27 16:10:38
tags:
categories: [并发]
---

## 一、什么是volatile？
`volatile`是Java中一个关键字，用来修饰变量。可以用来保证多线程下访问的变量的可见性，但是不能保证原子性。

## 二、volatile如何保证可见性？
`volatile`翻译成**不稳定的**，一般我们的变量存在主内存里，但为了提高读写效率，每个线程都有自己的工作内存，存放着从主内存同步过来的变量副本。

![](/images/concurrent/volatile关键字.png)

`volatile`被比喻成轻量的`synchronized`，`volatile`只能保证变量的可见性，而不能保证变量的原子性，`synchronized`能保证变量的可见性和原子性。

## 三、如何禁止指令重排？
`volatile`除了保证变量的可见性外，还能防止JVM的指令重排。

举个例子（典型的单例模式）：
```java
public class Singleton {
    private volatile static Singeton instance;
    
    private Singleton() {
        
    }
    
    public static Singleton getInstance() {
        // 双重检验
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

实际上，`instance = new Singleton()`这一步是分为三步执行：
1. 为`instance`分配内存
2. 初始化`instance`
3. 将·instance`指向分配的内存地址

但是由于JVM的指令重排，执行顺序可能变成`1->3->2`，这样子如果有另外一个线程来请求`getInstance()`方法，那么第一个`instance == null`将会返回`true`，但是此时`instance`还没初始化完，所以调用就会有问题。

## 四、volatile能保证原子性吗？
举个例子先，启动10个线程，每个线程执行10次自增操作，代码如下：
```java
public class VolatileAtomicDemo {
    public volatile static int inc = 0;
    
    public static void increase() {
        inc++;
    }

    public static void main(String[] args) {
        ExecutorService thrreadPool = Executors.newFixedThreadPool(5);
        for(int i=0;i<10;i++) {
            thrreadPool.execute(() -> {
                for(int j=0;j<10;j++) {
                    increase();
                }
            });
        }

        System.out.println("inc:" + inc);
    }
}
```

**输出**：
```shell
inc:98
```

为什么inc不是100呢？不是说加了`volatile`就能保证变量的可见性了吗？

其实是因为`inc++`并不是原子性的，它的执行分为了三步：
1. 读取inc的值
2. 对inc加1
3. 将inc的值写回内存

所以这个场景就可能会出现以下情况：
1. 线程1读取到`inc`的值为0，还没来得及修改，线程2也读取到`inc`的值为0，并执行`inc++`后并写回内存（此时内存里`inc`的值为1）
2. 这时线程开始执行`inc++`，并将结果值（1）写回内存（这时候就会出现内存中`inc`的值1被1覆盖，也就是2次++操作其实只生效了1次）

那么要如何解决这个问题呢？有以下三种方案：
* 使用`synchronized`
* 使用`Lock`
* 使用`AtomicInteger`

**使用synchronized**：
```java
public synchronized int increase() {
    inc++;
}
```

**使用AtomicInteger**：
```java
public AtomicInteger inc = new AtomicInteger();

public void increase() {
    inc.getAndIncrement();
}
```

**使用Lock**：
```java
Lock lock = new ReentrantLock();
public void increase() {
    lock.lock();
    try {
        inc++;
    } finally {
        lock.unlock();
    }
}
```