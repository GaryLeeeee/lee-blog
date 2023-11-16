---
title: 什么是SpringBoot的Starter？如何自定义一个Starter？
date: 2023-11-16 22:49:39
tags: [SpringBoot]
categories: [Spring]
---

## 一、什么是SpringBoot的Starter？
SpringBoot的Starter说白了就是一系列依赖关系的集合，它可以让我们的依赖关系变得更简单。

Starter有以下几个特点：
* **自动配置**：Starter提供了相关的自动配置类，可以根据应用程序的依赖和配置，自动配置必要的依赖和设置
* **自定义参数**：Starter提供了默认的依赖和配置参数，但是我们也是可以在配置文件配置自定义参数来覆盖
* **提供预配置依赖**：Starter集合了一系列依赖关系，用来启用特定场景的功能，我们可以根据特定场景引入特定的Starter而不用单独处理每个依赖项的版本和配置
* **避免依赖冲突**：Starter会自动处理相关依赖的版本兼容性，避免依赖冲突
* **约定大于配置**：Starter遵循约定大于配置的原则，约定了如何将依赖和配置组合在一块，使得应用程序开发能更加一致和高效

### 1、举个例子说明为什么要有Starter？
在没有SpringBoot的Starter之前，如果我们要开发Web应用程序，我们需要引用多个依赖如`Spring MVC`、`Tomcat`、`Jackson`等。

但这样会带来一系列问题：
* **复杂、麻烦**
* **容易出现依赖冲突**

但是有了SpringBoot的Starter之后，我们只需要引入一个依赖`spring-boot-starter-web`即可，它里面就会自动帮我们集成了多个依赖如`Spring MVC`、`Tomcat`、`Jackson`等，具体代码如下：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### 2、常见的Starter有哪些？
|Starter|作用|集成依赖|
|--|--|--|
|**spring-boot-starter-web**|用于开发Web应用程序，提供了处理HTTP请求和响应的功能|SpringMVC、Tomcat等|
|**spring-boot-starter-data-jpa**|用于与关系型数据库交互，提供了常见的操作方法|Spring Data JPA、Hibernate等|
|**spring-boot-starter-security**|用于添加安全性功能，提供了身份验证、授权和安全配置的功能|Spring Security、OAuth2|
|**spring-boot-starter-test**|用于编写单元测试和集成测试，提供了测试框架和工具|Junit、Spring Test等|
|**spring-boot-starter-data-redis**|用于与Redis数据库交互，提供了常见的操作方法|Spring Data Redis等|
|...|...|...|

### 3、如何自定义配置参数？
SpringBoot的Starter支持我们去修改我们的默认配置，如：
* `@ConditionalOnClass(A.class)`：代表如果存在`A.class`，该配置类才会生效
* `@ConditionalOnMissingBean(B.class)`：代表如果不存在`B.class`对应的bean，那么该配置类将会创建默认`B.class`对应的bean
* `@ConfigurationProperties(prefix = "gary.test")`：代表如果存在前缀为`gary.test`的配置项（存在`application.yml`中），则该配置类生效（同时我们也支持修改参数，如`gary.test.name=xxx`）
* `@EnableConfigurationProperties(C.class)`：配置`@ConfigurationProperties`使用，代表启用该配置项

## 二、如何自定义一个Starter？

### 1、如何给Starter命名？
一般官方的Starter命名规则都是`spring-boot-starter-xxx`，而第三方的Starter命名规则应该是`xxx-spring-boot-starter`，如`mybatis-spring-boot-autoconfigure`。

### 2、自定义步骤
自定义一个Starter的步骤如下：
a. 创建一个maven项目，命名为`xxx-spring-boot-starter`
b. 在`pom.xml`文件中添加`spring-boot-starter-parent`、`spring-boot-starter`依赖
c. 创建一个自动配置类，用`@Configuration`和`@EnableConfigurationProperties`注解
```java
@Configuration
@EnableConfigurationProperties(MyStartProperties.class)
@ConditonOnClass(MyStarter.class)
@ConditionOnProperty(prefix = "gary.starter", value = "enabled", matchIfMissing = true)
public class MyStarterAutoConfiguration {
    @Autowired
    private MyStartProperties myStartProperties;
    
    @Bean
    @ConditionOnMissingBean(MyStarter.class)
    public MyStarter myStarter() {
        return new MyStarter(myStartProperties);
    }
}
```

d. 在`src/main/resources/META_INF/spring.factories`添加自动配置类的全限定名（SpringBoot自动装配机制才会生效）
```yaml
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.gary.test.MyStarterAutoConfiguration
```
e. 打包部署成jar包即可