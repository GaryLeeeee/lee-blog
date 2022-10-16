---
title: Dubbo使用笔记
date: 2022-10-16 21:16:14
tags: [Dubbo]
categories: [Dubbo]
---

## Q&A
### 1.项目启动时出现循环依赖(A依赖B，B依赖A)，哪一方启动都会报错找不到消费者，如何解决？
**分析**：Dubbo默认会在启动时检查依赖的服务是否可用，不可用时会抛出异常(No provider...)
**解决**：关闭启动时依赖检查
* **方案a：配置spring配置文件**
    * 关闭某个服务的启动时检查(没有提供者时报错)
      ```<dubbo:reference interface="com.demo.UserService" check="false" />```
    * 关闭所有服务的启动时检查(没有提供者时报错)
      ```<dubbo:consumer check="false" />```
    * 关闭注册中心启动时检查(注册订阅失败时报错)
      ```<dubbo:registry check="false" />```
* **方案b：配置dubbo.properties**
```
dubbo.reference.com.demo.UserService.check=false
dubbo.reference.check=false
dubbo.consumer.check=false
dubbo.registry.check=false
```
* **方案c：配置`-D`参数**
```
java -Ddubbo.reference.com.demo.UserService.check=false
java -Ddubbo.reference.check=false
java -Ddubbo.consumer.check=false 
java -Ddubbo.registry.check=false
```