---
title: Java字符串要注意哪些问题？
date: 2023-10-23 19:20:19
tags: [字符串]
categories: [Java]
---

## 一、String、StringBuffer、StringBuilder有什么区别？
`String`、`StringBuffer`、`StringBuilder`区别如下：
* **线程安全性**：`String`线程安全（因为不可变），`StringBuffer`线程安全（因为方法都加了`synchronized`锁），`StringBuilder`线程不安全（因为没有加锁）
* **性能**：`String`性能最差（每次更新都要创建一个新对象），`StringBuffer`性能一般（每次更新只需要原地操作），`StringBuilder`性能较好（相比`StringBuffer`较好，但是要承担线程不安全的风险）
* **可变性**：`String`不可变（重新赋值会创建新对象），`StringBuffer`和`StringBuilder`可变（都继承`AbstractStringBuilder`，提供了如`append`方法支持修改字符串）
```java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    char[] value;
    int count;
    
    public AbstractStringBuilder append(String str) {
        if (str == null)
            return appendNull();
        int len = str.length();
        // 检查数组容量是否足够，如果不够则扩容
        ensureCapacityInternal(count + len);
        // 复制str从0到len-1的字符串到value的count开始处
        str.getChars(0, len, value, count);
        // 修改字符串长度
        count += len;
        return this;
    }
}
```

## 二、String为什么是不可变的？
因为`String`的底层存储用的是`char[]`，<font color=red>而且用了`final`修饰，说明不可变</font>相关代码为：
```java
public final class String implements Serializable, Comparable<String>, CharSequence {
    private final char value[];
}
```

总结，`String`不可变的原因是：
* `value[]`被`private final`修饰，而`String`没有提供修改`value[]`的方法
* `String`被`final`修饰所以不能被继承，避免子类破坏`String`不可变

另外提一嘴，在JDK 9之后，`String`的底层存储由`char[]`改为了`byte[]`，目的是减少内存占用（具体为什么可百度）。

## 三、如何更好地拼接多个字符串？
一般情况下，如果要拼接多个字符串，我们是直接使用`+`来操作，如下：
```java
String str1 = "Hello";
String str2 = "World";
String str3 = "！";
String str4 = str1 + str2 + str3;
```

实际中，调用`+`操作时，底层还是通过`StringBuilder`的`append()`方法去拼接的，**但是会出现一个问题**：<font color=red>每拼接一个字符串都需要创建一个`StringBuilder`，如果拼接的字符串数量过大，那么会创建大量的`StringBuilder`，导致性能问题</font>

所以一般我们直接借用`StringBuilder`来进行字符串拼接，而不是`String`拼接`String`，那么就会解决这个问题了，代码如下：
```java
String str1 = "Hello";
String str2 = "World";
String str3 = "！";
//String str4 = str1 + str2 + str3;
StringBuilder s = new StringBuilder(;)
s.append(str1);
s.append(str2);
s.append(str3);
```

另外提一嘴，在JDK 9之后，使用`+`拼接字符串导致创建大量`StringBuilder`的问题已经被解决了（通过调用`makeConcatWithConstants()`方法）