---
title: Java是值传递还是引用传递？
date: 2023-10-24 19:04:17
tags:
categories: [Java]
---

## 一、什么是形参和实参？
形参和实参都指的是参数，他们的区别在于：
* **形参**：用于传递给方法的参数，需要赋值
* **实参**：用于方法定义来接收实参，不需要赋值

```java
public class TestClass {
    public static void main(String[] args) {
        // 这里str是实参
        String str = "abc";
        execute(str);
    }
    
    // 这里str是形参
    public static void execute(String str) {
        // ..
    }
}
```

## 二、什么是值传递和引用传递？
值传递和引用传递都是参数传递的方式，他们的区别在于：
* **值传递**：给方法传递的是参数**副本**（也就是**拷贝**，修改形参不会影响原值）
* **引用传递**：给方法传递的是参数**地址**（修改形参会影响原值）

## 三、为什么Java只有值传递？
### 1、基本数据类型传递
**代码**：
```java
public class TestClass {
    public static void main(String[] args) {
        int num1 = 1;
        int num2 = 2;
        swap(num1, num2);
        System.out.println("num1=" + num1);
        System.out.println("num2=" + num2);
    }
    
    public static void swap(int a, int b) {
        int temp = a;
        a = b;
        b = temp;
        System.out.println("a=" + a);
        System.out.println("b=" + b);
    }
}
```

**输出**：
```shell
a=2
b=1
num1=1
num2=2
```

**解析**：基本数据类型的传递是值拷贝，如`int a; a=num1;`。也就是`swap()`方法中的`a`、`b`参数跟外层调用传递的`num1`、`num2`不是同一个实例，只是值相等。所以修改方法内的形参并不会影响实参。

### 2、对象传递
**代码**：
```java
public class User {
    private String name;
    // ...
}

public class TestClass {
    public static void main(String[] args) {
        User gary = new User("gary");
        User lee = new User("lee");
        swap(gary, lee);
        System.out.println("gary=" + gary.getName());
        System.out.println("lee=" + lee.getName());
    }
    
    public static void swap(User user1, User user2) {
        User temp = user1;
        user1 = user2;
        user2 = temp;
        System.out.println("user1=" + user1.getName());
        System.out.println("user2=" + user2.getName());
    }
}
```

**输出**：
```shell
user1=lee
user2=gary
gary=gary
lee=lee
```

**解析**：为什么对象传递修改了也没有影响实参？是因为这里传递的是`gary`和`lee`的**地址**，跟上面**基本数据类型同理**，也是一样`User user1; user1 = gary;`。所以修改方法内的形参并不会影响实参。

## 四、总结
在Java中只有值传递，只不过传递的是对象的引用/地址而已。

## 五、Q&A
### 1、Java只有值传递，那为什么修改传递对象的属性会影响原对象呢？
首先Java只有值传递，而且传递的是对象的地址。

举个例子，你有一把钥匙和一间房子，你通过**值传递**复制了一把钥匙给你的朋友（这时候两把钥匙都**指向**同一个房子）。如果你朋友使用钥匙打开房子并把冰箱拿走了（**形参修改**），那么对于你来说你的房子也是发生了变化了，

所以说，虽然是值传递，但是实参和形参都是指向同一个地址，那么同一个地址的变化是一定会同时影响实参和形参的。

### 2、Java为什么不引入引用传递？
因为考虑到安全问题，对于调用者来说，要传什么类型的参数是确定的，但是这个方法里做了什么事情我们是不知道的，所以避免这种情况出现，就不引入引用类型。