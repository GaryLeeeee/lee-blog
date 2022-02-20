---
title: Mybatis-Plus使用笔记
date: 2021-11-09 12:59:01
tags: [Mybatis,Mybatis-Plus]
categories: [Mybatis,Mybatis-Plus,SQL,MySQL]
---

## 一、官方文档
https://baomidou.com/

## 二、怎么使用
### 1.添加依赖
```xml
<!-- MybatisPlus -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.2.0</version>
</dependency>
```

### 2.添加配置
```yaml
# Mybatis-plus
mybatis-plus:
  # 放在resource目录 classpath:/mapper/*Mapper.xml
  mapper-locations: classpath:/mybatis/mapper/*Mapper.xml
  # 实体扫描，多个package用逗号或者分号分隔
  typeAliasesPackage: com.yy.mobilevoice.svc.diversion.model.domain
  global-config:
    # 主键类型  0:"数据库ID自增", 1:"用户输入ID",2:"全局唯一ID (数字类型唯一ID)", 3:"全局唯一ID UUID";
    id-type: 2
    # 字段策略 0:"忽略判断",1:"非 NULL 判断",2:"非空判断"
    field-strategy: 2
    # 驼峰下划线转换
    db-column-underline: true
    # 刷新mapper 调试神器
    refresh-mapper: true
    # 数据库大写下划线转换
    #capital-mode: true
    # 逻辑删除配置（下面3个配置）
    logic-delete-value: 0
    logic-not-delete-value: 1
    # SQL 解析缓存，开启后多租户 @SqlParser 注解生效
    sql-parser-cache: true
  configuration:
    map-underscore-to-camel-case: true
    cache-enabled: false
```

### 3.常见注解(更多查看文档)
|注解|作用|
|-------------|-------------|
|@MapperScan("com.demo.mapper")|扫描该目录下的mapper接口都会自动生成实现类(等同于一个个加@Mapper)|
|@TableName("tableName")|用于标识数据库表对应的实体类||

### 4.如何使用多数据源？
#### a.添加依赖
```xml
<dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>dynamic-datasource-spring-boot-starter</artifactId>
  <version>${version}</version>
</dependency>
```

#### b.添加数据源
```yaml
spring:
  datasource:
    dynamic:
      primary: master #设置默认的数据源或者数据源组,默认值即为master
      strict: false #严格匹配数据源,默认false. true未匹配到指定数据源时抛异常,false使用默认数据源
      datasource:
        master:
          url: jdbc:mysql://xx.xx.xx.xx:3306/dynamic
          username: root
          password: 123456
          driver-class-name: com.mysql.jdbc.Driver # 3.2.0开始支持SPI可省略此配置
        slave_1:
          url: jdbc:mysql://xx.xx.xx.xx:3307/dynamic
          username: root
          password: 123456
          driver-class-name: com.mysql.jdbc.Driver
        slave_2:
          url: ENC(xxxxx) # 内置加密,使用请查看详细文档
          username: ENC(xxxxx)
          password: ENC(xxxxx)
          driver-class-name: com.mysql.jdbc.Driver
       #......省略
       #以上会配置一个默认库master，一个组slave下有两个子库slave_1,slave_2
```
```yaml
# 多主多从
spring:
  datasource:
    dynamic:
      datasource:
        master_1:
        master_2:
        slave_1:
        slave_2:
        slave_3:
                                                                                      
# 纯粹多库（记得设置primary）
spring:
  datasource:
    dynamic:
      datasource:
        mysql: 
        oracle:
        sqlserver:
        postgresql:
        h2:
# 混合配置
spring:
  datasource:
    dynamic:
      datasource:
        master:
        slave_1:
        slave_2:
        oracle_1:
        oracle_2:
```

#### c.使用 **@DS** 切换数据源
|注解|作用|
|-------------|-------------|
|没有@DS|默认数据源(即`spring.datasource.dynamic.primary`)|
|@DS("name")|`name`可以为组别/某个库名(如`master`)||

**@DS 可以注释在方法上或类上，同时存在就近原则，即方法上注解优先于类上注解(如下面代码`selectByCondition()`函数优先使用`slave_1`数据源而不是`slave`数据源)**
```java
@Service
@DS("slave")
public class UserServiceImpl implements UserService {

  @Autowired
  private JdbcTemplate jdbcTemplate;

  public List selectAll() {
    return  jdbcTemplate.queryForList("select * from user");
  }
  
  @Override
  @DS("slave_1")
  public List selectByCondition() {
    return  jdbcTemplate.queryForList("select * from user where age >10");
  }
}
```


## 三、demo代码地址
https://github.com/GaryLeeeee/lee-code-repository/tree/master/lee-db-code/src/main/java/com/garylee/repository/mybatisplus

