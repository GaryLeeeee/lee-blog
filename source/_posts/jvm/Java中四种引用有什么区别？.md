---
title: Java中四种引用有什么区别？
date: 2023-11-12 20:56:11
tags:
categories: [JVM]
---

## 一、Java中四种引用分别是什么？
Java中四种引用分别是：
* **强引用**：默认引用，使用后不会被垃圾回收器回收（如果内存不足，则直接报错OOM）。例子代码为`String[] arr = new String[]{"a","b"};`
* **弱引用**：如果一个对象只有弱引用，那么无论内存充足与否，都会被垃圾回收器回收。例子代码为`WeakReference<String[]> weakReference = new WeakRerence<String []>(new String[]{"a","b"});`
* **软引用**：如果一个对象只有软引用，那么当内存不足时才会被垃圾回收器回收（内存充足时不会回收，所以是最大可能性地保证不被回收）。例子代码为`SoftReference<String[]> softReference = new SoftReference<String[]>(new String[]{"a","b"});`
* **虚引用**：只是一个引用，并没有具体实现，如果一个对象只有虚引用，那么就跟没有引用一样，在任何时候都可能被垃圾回收器回收（虚引用主要用来跟踪对象被垃圾回收的活动）

对于弱引用的例子，可以参考文章[《ThreadLocal是什么？实现原理呢？》](https://garyleeeee.github.io/2023/09/08/concurrent/threadlocal-shi-shi-me-shi-xian-yuan-li-ni/)

## 二、Java中四种引用有什么区别？
||生命周期|OOM前被清理|GC前被潜力|
|--|--|--|--|
|强引用|最长|否|否|
|软引用|次于强引用|是|否|
|弱引用|次于软引用|是|是|
|虚引用|次于弱引用|是|是|