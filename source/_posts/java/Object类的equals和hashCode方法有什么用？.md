---
title: Object类的equals和hashCode方法有什么用？
date: 2023-10-19 20:57:35
tags:
categories: [Java]
---

## 一、背景
众所周知，在Java中Object类是所有类的父类，也就是说所有类都会有equals和hashCode方法。

## 二、equals()方法
### 1、代码实现
```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

### 2、特点
equals()方法有以下几个特点：
* **自反性**：`x.equals(x)`必须为`true`
* **对称性**：`x.equals(y)`与`y.equals(x)`的返回值必须相等
* **传递性**：`x.equals(y)`为`true`，`y.equals(z)`也为`true`，那么`x.equals(z)`也必须为`true`
* **非NULL**：当x为NULL时，`x.equals(y)`会报NPE。当y为NULL时，`x,equals(y)`必须为`false`

## 三、hashCode()方法
### 1、代码实现
```java
/** 返回对象内存地址，保证不同对象的hashCode也不同 **/
public native int hashCode();
```

### 2、特点
hashCode()方法有以下几个特点：
* 如果`x.equals(y)`，那么x和y的hashCode也一定相等
* 如果`!x.equals(y)`，那么x和y的hashCode有可能相等也有可能不相等（但是尽可能越散列越好）

### 3、作用
一般用在哈希表中，如HashSet、HashMap等

插入步骤：
1. 调用hashCode()方法计算得到Object的哈希码
2. 通过哈希码取余定位到下标
3. 判断下标处是否为空，如果为空则直接插入，如果不为空则遍历链表（HashMap实现）
4. 遍历链表调用equals()方法比较对象是否相等，如果相等则说明已存在，如果不相等则插入到链表尾部

## 四、String中equals和hashCode的实现
```java
public class String {
    private final char value[];
}
```

