---
title: Java工具类
date: 2022-10-16 20:39:56
tags: [Java,Http,工具]
categories: [Java]
---

# JSON
## 一、Q&A
### 1.http接口想返回自定义格式字符串(原Date类型)，如输出"2022-10-16 20:00:00"
* **方法**：添加注解`@JsonFormat`
* **用途**：将Date转换成String，一般是后台传值给前台
```java
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone="GMT+8")
private Date startTime;
```

### 2.http接口想接收自定义格式字符串(新Date类型)，如输入"2022-10-16 20:00:00"
* **方法**：添加注解`@DateTimeFormat`
* **用途**：将String转换成Date，一般是前台传值给后台
```java
@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
private Date startTime;
```
