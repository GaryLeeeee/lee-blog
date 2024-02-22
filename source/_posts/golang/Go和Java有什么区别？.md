---
title: Go和Java有什么区别？
date: 2024-02-22 19:12:07
tags: [Golang, Java]
categories: [Golang]
---

## 一、Go和Java有什么区别？
### 1、接口
* Go：只要实现了某个接口的所有方法，那么就认为它实现了该接口
```go
package main

import "fmt"

type Animal interface {
	// 叫声
	shout()
}
type Cat struct {
}
func (Cat) shout() {
	fmt.Println("喵")
}
type Dog struct {
}
func (Dog) shout() {
	fmt.Println("汪")
}
```
* Java：需要显式指定自己实现了某个接口
```java
public interface Animal {
    void shout();
}
public class Cat implements Animal {
    @Override
    public void shout() {
        System.out.println("喵");
    }
}
public class Dog implements Animal {
    @Override
    public void shout() {
        System.out.println("汪");
    }
}
```
