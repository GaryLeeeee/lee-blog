---
title: Java数据类型和包装类型
date: 2023-10-20 23:13:49
tags:
categories: [Java]
---

## 一、Java中有哪些基本数据类型？
|基本数据类型|位数|字节|默认值|取值范围|
|---|---|---|---|---|
|byte|8|1|0|-128~127|
|short|16|2|0|-32768~32767（-2^15~2^15-1）|
|int|32|4|0|-2147483648~2147483647|
|long|64|8|0L|-9223372036854775808~9223372036854775807（-2^63~2^63-1）|
|char|16|2|'u0000'|0~65535（0~2^16-1）|
|float|32|4|0f|3.4e-38~3.4e38|
|double|64|8|0d|1.7e-308~1.7e308|
|boolean|1||false|true、false|

**为什么有的最大值都需要-1？**：因为在二进制补码中，最高位是用来表示符号的（0表示整数、1表示负数）。所以我们要表示最大的整数，需要把除了最高位之外的所有位都设为1（如果再加1，就会溢出，变成负数）

**其他注意点**：
* 在Java中`long`类型的数值需要在后面加上L，否则会被视作`int`解析
* `char a = 'h'`用到的是单引号，`String s = "hi"`用到的是双引号

## 二、基本数据类型和包装类型有什么区别？
基本数据类型对应的包装类型如下：

|基本数据类型|包装类型|
|---|---|
|byte|Byte|
|short|Short|
|int|Integer|
|long|Long|
|char|Char|
|float|Float|
|double|Double|
|boolean|Boolean|

基本数据类型和包装类型的区别如下：
* **默认值**：基本数据类型都有对应的默认值，而包装类型默认为`null`
* **比较方式**：基本数据类型通过`==`比较值，包装类型通过`equals`比较值（包装类型通过`==`比较的是对象内存地址）
* **占用空间**：基本数据类型占用空间相比包装类型的要小
* **JVM存储**：基本数据类型的局部变量存放在JVM栈中的局部变量表中，<font color=red>成员变量存放在JVM的堆中</font>；包装类型都存放在JVM的堆中（因为都是对象）

## 三、包装类型的缓存机制
Java基本数据类型的包装类型大部分都提供了缓存机制来提升性能，比如：
* `Byte`、`Short`、`Integer`、`Long`缓存了范围`[-128, 127]`的数据
* `Character`缓存了范围`[0,127]`的数据
* `Boolean`直接返回`TRUE`or`FALSE`

**`Integer`缓存实现**：
```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}

private static class IntegerCache {
    static final int low = -128;
    static final int high;
    
    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;
        // ...
    }
}
```

**`Character`缓存实现**：
```java
public static Character valueOf(char c) {
    if (c <= 127) { // must cache
        return CharacterCache.cache[(int)c];
    }
    return new Character(c);
}

private static class CharacterCache {
    private CharacterCache() {}
    static final Character cache[] = new Character[127 + 1];
    static {
        for (int i = 0; i < cache.length; i++)
            cache[i] = new Character((char)i);
    }
}
```

**`Boolean`缓存实现**：
```java
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}
```

超过范围的话则还是会创建新的对象，这个范围的大小是在性能和资源之间做权衡得出的。

浮点型的包装类型`Float`、`Double`并没有实现缓存机制
```java
Integer i1 = 50;
Integer i2 = 50;
System.out.println(i1 == i2); // 输出true

Float f1 = 50f;
Float f2 = 50f;
System.out.println(f1 == f2); // 输出false

Double d1 = 5.5;
Double d2 = 5.5;
System.out.println(d1 == d2); // 输出false
```

如果我们用new的方式声明对象，那么它将会创建一个新的对象，不会使用缓存
```java
Integer i1 = 50;
Integer i2 = new Integer(50);
System.out.println(i1 == i2); // 输出false
```