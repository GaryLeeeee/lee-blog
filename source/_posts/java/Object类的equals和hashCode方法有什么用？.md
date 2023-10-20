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

### 1、代码实现
```java
public class String {
    private final char value[];
    private int hash; // Default to 0
    
    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }

        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if(n == anObject.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                while(n-- != 0) {
                    if(v1[i] != v2[i]) {
                        return false;
                    }
                }
                return true;
            }
        }
        // 如果长度不一致则一定不相等
        return false;
    }
    
    public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;
            
            for(int i=0;i<value.length;i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        
        return h;
    }
}
```

### 2、解析
String代码实现可以看出以下几点：
* String的底层存储char[]是final修饰的，说明String一旦创建就不可修改（比如`String s = "abc"`，如果继续执行`s = "bbq`，那么将是会新建一个String对象，并且s指向新的String对象）
* hashCode执行结果存到成员变量hash字段中，提高查询性能，避免每次都去计算哈希码
* String的equals方法判断的是两个对象长度相同的前提下每个字符都相同（并不要求一定是同个String对象）

## 五、如何重写hashCode()
### 1、重写hashCode()的原则
* 由于“如果`x.equals(y)`，那么x和y的hashCode也一定相等”，所以如果我们重写了equals()，那么我们也应该重写hashCode()
* hashCode()实现尽量分散，减少出现哈希冲突的可能
* hashCode()实现尽量不要太复杂，避免影响性能

### 2、hashCode()重写方法
/** 尽可能用到equals()里所用到的所有对象变量，从而减少哈希冲突**/

## 六、Q&A
### 1、为什么String的hashCode计算用到了数字31？
String的hashCode()方法代码如下：
```java
public class String {
    public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;

            for(int i=0;i<value.length;i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }

        return h;
    }
}
```

可以得到String的hashCode=s[0]*31^(n-1)+s[1]*31^(n-2)+...+s[n-1]

为什么用到了数字31呢？原因是：
* 31是质数，与其他数字相乘得到的计算结果唯一概率更大（比如62=31*2，但是64=32*2=16*4=8*8）
* 但是质数中为什么选择31呢？因为质数越大，哈希冲突的概率就越小，所以在哈希冲突和性能的权衡下选择了31
* JVM会对31做优化：`31*i=(i<<5)-i`







