# Gary's Blog
|Java|MySQL|Redis|JVM|并发|Kafka|Http|工具|场景题|代码优化|Spring|
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| [:strawberry:](#strawberry-Java) | [:floppy_disk:](#floppy_disk-MySQL) | [:cloud:](#cloud-Redis) | [:computer:](#computer-JVM) | [:dart:](#dart-并发) | [:art:](#art-Kafka) |[:memo:](#memo-Http)| [:wrench:](#wrench-工具&插件&框架) | [:book:](#book-场景题) | [:battery:](#battery-代码优化) | [:coffee:](#coffee-Spring) |

## :strawberry: Java
- [Java动态代理](https://garyleeeee.github.io/2023/08/12/java-dong-tai-dai-li/)

## :floppy_disk: MySQL
- [MySQL锁](https://garyleeeee.github.io/2023/07/06/mysql/mysql-suo/)
- [MySQL索引](https://garyleeeee.github.io/2023/07/03/mysql/mysql-suo-yin/)
- [MySQL事务](https://garyleeeee.github.io/2023/07/03/mysql/mysql-shi-wu/)
- [MVCC笔记](https://garyleeeee.github.io/2023/07/01/mysql/mvcc-bi-ji/)
- [MySQL日志篇](https://garyleeeee.github.io/2023/06/27/mysql/mysql-ri-zhi-pian/)
- [如何排查MySQL索引失效的问题？](https://garyleeeee.github.io/2023/08/15/mysql/ru-he-pai-cha-mysql-suo-yin-shi-xiao-de-wen-ti/)
- [什么是聚簇索引和非聚簇索引？](https://garyleeeee.github.io/2023/08/17/mysql/shi-me-shi-ju-cu-suo-yin-he-fei-ju-cu-suo-yin/)
- [什么是回表，怎么减少回表的次数？](https://garyleeeee.github.io/2023/08/17/mysql/shi-me-shi-hui-biao-zen-me-jian-shao-hui-biao-de-ci-shu/)
- [MySQL执行计划分析](https://garyleeeee.github.io/2023/08/18/mysql/mysql-zhi-xing-ji-hua-fen-xi/)
- [如何进行SQL调优？](https://garyleeeee.github.io/2023/08/19/mysql/ru-he-jin-xing-sql-diao-you/)
- [如何排查慢SQL问题？](https://garyleeeee.github.io/2023/08/19/mysql/ru-he-pai-cha-man-sql-wen-ti/)
- [limit 1000000,0加载很慢如何优化？](https://garyleeeee.github.io/2023/08/20/mysql/limit1000000-0-jia-zai-hen-man-ru-he-you-hua/)
- [InnoDB和MyISAM有什么区别？](https://garyleeeee.github.io/2023/09/05/mysql/innodb-he-myisam-you-shi-me-qu-bie/)

## :cloud: Redis
- [关于对缓存雪崩和缓存穿透的理解，以及如何避免？](https://garyleeeee.github.io/2023/07/24/guan-yu-dui-huan-cun-xue-beng-he-huan-cun-chuan-tou-de-li-jie-yi-ji-ru-he-bi-mian/)
- [Redis集群](https://garyleeeee.github.io/2023/08/08/redis/redis-ji-qun/)
- [Redis过期策略](https://garyleeeee.github.io/2023/08/16/redis/redis-guo-qi-ce-lue/)
- [Redis内存淘汰策略](https://garyleeeee.github.io/2023/08/16/redis/redis-nei-cun-tao-tai-ce-lue/)
- [如何处理Redis的大key问题？](https://garyleeeee.github.io/2023/08/19/redis/ru-he-chu-li-redis-de-da-key-wen-ti/)
- [Redis为什么这么快？](https://garyleeeee.github.io/2023/08/23/redis-wei-shi-me-zhe-me-kuai/)
- [如何实现多级缓存？](https://garyleeeee.github.io/2023/08/25/redis/ru-he-shi-xian-duo-ji-huan-cun/)
- [Redis的持久化机制是怎样的？](https://garyleeeee.github.io/2023/09/05/redis/redis-de-chi-jiu-hua-ji-zhi-shi-zen-yang-de/)
- [Redis中的Zset是怎么实现的？](https://garyleeeee.github.io/2023/09/10/redis/redis-zhong-de-zset-shi-zen-me-shi-xian-de/)

## :computer: JVM
- [GC学习笔记](https://garyleeeee.github.io/2023/07/12/jvm/gc-xue-xi-bi-ji/)
- [Java类加载机制](https://garyleeeee.github.io/2023/08/04/java-lei-jia-zai-ji-zhi/)
- [JVM运行时数据区域是怎样的？](https://garyleeeee.github.io/2023/09/06/jvm/jvm-yun-xing-shi-shu-ju-qu-yu-shi-zen-yang-de/)

## :dart: 并发
- [高可用学习笔记(科普向)](https://garyleeeee.github.io/2021/08/01/gao-ke-yong-xue-xi-bi-ji-ke-pu-xiang/)
- :trophy:[线程池学习笔记](https://garyleeeee.github.io/2021/07/28/xian-cheng-chi-xue-xi-bi-ji/)
- [如何限流？](https://garyleeeee.github.io/2023/07/19/ru-he-xian-liu/)
- [如何设计分布式ID？](https://garyleeeee.github.io/2023/07/19/ru-he-she-ji-fen-bu-shi-id/)
- [如何解决幂等性问题？](https://garyleeeee.github.io/2023/07/21/ru-he-jie-jue-mi-deng-xing-wen-ti/)
- [如何设计一个高并发系统？](https://garyleeeee.github.io/2023/07/19/ru-he-she-ji-yi-ge-gao-bing-fa-xi-tong/)
- :trophy:[Redis和MySQL如何保证数据一致性？](https://garyleeeee.github.io/2023/07/23/redis-he-mysql-ru-he-bao-zheng-shu-ju-yi-zhi-xing/)
- [什么是服务降级？](https://garyleeeee.github.io/2023/08/25/concurrent/shi-me-shi-fu-wu-jiang-ji/)
- [如何实现一个分布式锁？](https://garyleeeee.github.io/2023/09/05/concurrent/ru-he-shi-xian-yi-ge-fen-bu-shi-suo/)
- [ThreadLocal是什么？实现原理呢？](https://garyleeeee.github.io/2023/09/08/concurrent/threadlocal-shi-shi-me-shi-xian-yuan-li-ni/)

## :art: Kafka
- [Kafka面试题](https://garyleeeee.github.io/2023/07/12/kafka-mian-shi-ti/)
- [Kafka如何解决消息积压问题？](https://garyleeeee.github.io/2023/09/06/kafka/kafka-ru-he-jie-jue-xiao-xi-ji-ya-wen-ti/)

## :memo: Http
- [Http学习笔记](https://garyleeeee.github.io/2023/07/03/http-xue-xi-bi-ji/)

## :wrench: 工具
- [分页插件PageHelper的使用](https://garyleeeee.github.io/2022/10/19/fen-ye-cha-jian-pagehelper-de-shi-yong/)
- [Dubbo使用笔记](https://garyleeeee.github.io/2022/10/16/dubbo-shi-yong-bi-ji/)
- [hexo使用笔记](https://garyleeeee.github.io/2021/07/31/hexo-shi-yong-bi-ji/)
- [代码库](https://garyleeeee.github.io/2021/08/07/dai-ma-ku/)
- [RabbitMQ安装笔记](https://garyleeeee.github.io/2021/11/05/rabbitmq-an-zhuang-bi-ji/)
- [Vue项目创建方式](https://garyleeeee.github.io/2022/03/02/vue-xiang-mu-chuang-jian-fang-shi/)
- [k8s学习笔记](https://garyleeeee.github.io/2021/08/12/k8s-xue-xi-bi-ji/)
- [Mybatis-Plus使用笔记](https://garyleeeee.github.io/2021/11/09/mybatis-plus-shi-yong-bi-ji/)

## :book: 场景题
- :trophy:[如何设计一个秒杀系统？](https://garyleeeee.github.io/2023/07/17/ru-he-she-ji-yi-ge-miao-sha-xi-tong/)
- [如何用Redis实现延迟队列？](https://garyleeeee.github.io/2023/07/29/ru-he-yong-redis-shi-xian-yan-chi-dui-lie/)
- [如何排查CPU飙升问题？](https://garyleeeee.github.io/2023/07/29/ru-he-pai-cha-cpu-biao-sheng-wen-ti/)
- [如何实现一个点赞功能？](https://garyleeeee.github.io/2023/07/31/ru-he-shi-xian-yi-ge-dian-zan-gong-neng/)
- [如何实现一个扫码登录功能？](https://garyleeeee.github.io/2023/08/20/ru-he-shi-xian-yi-ge-sao-ma-deng-lu-gong-neng/)
- [如何实现一个分数相同则按时间排序的排行榜？](https://garyleeeee.github.io/2023/08/20/scene/ru-he-shi-xian-yi-ge-fen-shu-xiang-tong-ze-an-shi-jian-pai-xu-de-pai-xing-bang/)
- [如何防止库存超卖？](https://garyleeeee.github.io/2023/08/20/scene/ru-he-fang-zhi-ku-cun-chao-mai/)
- [如何设计一个购物车功能？](https://garyleeeee.github.io/2023/08/25/scene/ru-he-she-ji-yi-ge-gou-wu-che-gong-neng/)
- [如何提升接口的性能？](https://garyleeeee.github.io/2023/09/10/scene/ru-he-ti-sheng-jie-kou-de-xing-neng/)

## :battery: 代码优化
- [如何优化大量if-else代码结构？](https://garyleeeee.github.io/2023/07/31/ru-he-you-hua-da-liang-if-else-dai-ma-jie-gou/)

## :coffee: Spring
- [Spring如何解决循环依赖的问题？](https://garyleeeee.github.io/2023/08/01/spring-ru-he-jie-jue-xun-huan-yi-lai-de-wen-ti/)
- [介绍一下Spring的AOP](https://garyleeeee.github.io/2023/08/13/spring/jie-shao-yi-xia-spring-de-aop/)
- [Spring Bean的生命周期](https://garyleeeee.github.io/2023/08/22/spring/springbean-de-sheng-ming-zhou-qi/)
- [Spring事务](https://garyleeeee.github.io/2023/09/06/spring/spring-shi-wu/)
- [Spring中Bean的作用域有哪些？](https://garyleeeee.github.io/2023/09/08/spring/spring-zhong-bean-de-zuo-yong-yu-you-na-xie/)

## :mag: ElasticSearch
- [为什么要使用ElasticSearch？](https://garyleeeee.github.io/2023/08/26/wei-shi-me-yao-shi-yong-elasticsearch/)
- [什么是倒排索引？](https://garyleeeee.github.io/2023/08/26/shi-me-shi-dao-pai-suo-yin/)

## :triangular_ruler: MyBatis
- [Mybatis中有哪些占位符？区别是什么？](https://garyleeeee.github.io/2023/09/09/mybatis/mybatis-zhong-you-na-xie-zhan-wei-fu-qu-bie-shi-shi-me/)

## :books: 数据结构
- [什么是跳表？](https://garyleeeee.github.io/2023/09/10/data/shi-me-shi-tiao-biao/)