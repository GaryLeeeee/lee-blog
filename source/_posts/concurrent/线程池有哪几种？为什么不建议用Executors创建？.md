---
title: 线程池有哪几种？为什么不建议用Executors创建？
date: 2023-10-03 11:43:40
tags: [线程池]
categories: [并发]
---

## 一、线程池有哪几种？
在Java中，线程池有4种，都是通过工具量Executors创建的，而线程池的最终实现类是ThreadPoolExecutor，这4种线程池分别是：
* **newCachedThreadPool（缓存线程池）**：可以用来处理突发流量，由于最大线程数没有限制，所以可以处理大量的任务，另外每个线程可以存活60s，使得这些线程可以缓存起来应对更对任务的处理
    * 最大线程数=Integer.MaxValue
    * 线程存活时间=60s
    * 阻塞队列=SynchronousQueue(不存任何元素的阻塞队列)
* **newFixedThreadPool（固定线程数线程池）**：一旦创建了线程池就是固定不变的，如果任务较多线程处理不过来，则加入到阻塞队列等待
    * 核心线程数=最大线程数
* **newSingleThreadExecutor（单一线程线程池）**：创建完就是只有一个线程在处理，无法动态修改，保证可以顺序执行
* **newSingleThreadPool（可延迟执行线程池）**：可用于定时调度

## 二、为什么不建议使用Executors来创建线程池？
* **配置不灵活**：Executors创建线程池大多数参数都是默认的，不方便我们定制化一些参数，比如线程池名字等
* **容易造成OOM**：Executors内部使用的阻塞队列是无界阻塞队列LinkedBlockingQueue，说明如果任务越多内存也会越多，最终导致OOM
```java
public class Executors {
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runable>());
    }
    
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService(
                new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runable>())
        );
    }
}
```

## 三、Q&A
### 1、为什么LinkedBlockingQueue是无界的？
因为它前面加了Linked，代表他是一个链表。链表跟数组不同的是，链表是动态添加元素的，不像数组一开始就初始化好数组大小。