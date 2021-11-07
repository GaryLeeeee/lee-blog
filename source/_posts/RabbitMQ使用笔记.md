---
title: RabbitMQ使用笔记
date: 2021-11-06 13:51:29
tags: RabbitMQ
categories: RabbitMQ
---
## 一、依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```
## 二、配置
```yaml
spring:
  lee-rabbitmq:
    addresses: localhost:5672
    username: test_username
    password: test_password
    virtual-host: /
```
```java
public interface RabbitMqConstant {
    String EXCHANGE = "LEE_EXCHANGE";
    String ROUTING_KEY = "LEE_ROUTING_KEY";
    String QUEUE = "LEE_QUEUE";
}
```
```java
@Configuration
public class RabbitMqConfiguration {

    @Bean(name = "leeConnectionFactory")
    public ConnectionFactory leeConnectionFactory(
            @Value("${spring.lee-rabbitmq.addresses}") String addresses,
            @Value("${spring.lee-rabbitmq.username}") String username,
            @Value("${spring.lee-rabbitmq.password}") String password,
            @Value("${spring.lee-rabbitmq.virtual-host}") String virtualHost
    ){
        CachingConnectionFactory connectionFactory = new CachingConnectionFactory();
        connectionFactory.setAddresses(addresses);
        connectionFactory.setUsername(username);
        connectionFactory.setPassword(password);
        connectionFactory.setVirtualHost(virtualHost);
        return connectionFactory;
    }

    @Bean(name = "leeRabbitTemplate")
    public RabbitTemplate leeRabbitTemplate(
            @Qualifier("leeConnectionFactory") ConnectionFactory connectionFactory
    ){
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
        return rabbitTemplate;
    }

    @Bean(name = "leeQueue")
    public Queue leeQueue() {
        return new Queue(RabbitMqConstant.QUEUE);
    }

    @Bean(name = "leeExchange")
    public TopicExchange leeExchange() {
        return new TopicExchange(RabbitMqConstant.EXCHANGE);
    }

    @Bean
    Binding binding(@Qualifier("leeQueue") Queue leeQueue, @Qualifier("leeExchange") TopicExchange leeExchange) {
        return BindingBuilder.bind(leeQueue).to(leeExchange).with(RabbitMqConstant.ROUTING_KEY);
    }


    /**
     * 消费者配置
     */
    @Bean(name = "leeFactory")
    public SimpleRabbitListenerContainerFactory leeFactory(
            @Qualifier("leeConnectionFactory") ConnectionFactory connectionFactory
    ) {
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory);
        return factory;
    }

}
```

## 三、发送
```java
@Data
public class User {
    private int id;
    private String name;
    private int age;
}
```
```java
@Service
public class RabbitMqProvider {

    @Resource(name = "leeRabbitTemplate")
    private RabbitTemplate rabbitTemplate;

    public void send(User user){
        rabbitTemplate.convertAndSend(RabbitMqConstant.EXCHANGE, RabbitMqConstant.ROUTING_KEY, user);
    }
}
```
发送结果
![RabbitMQ创建Exchange](/images/RabbitMQ创建Exchange.png)
![RabbitMQ创建Queue](/images/RabbitMQ创建Queue.png)

## 四、接收
```java
@Slf4j
@Component
public class RabbitMqConsumer {

    @RabbitListener(queues = RabbitMqConstant.QUEUE, containerFactory = "leeFactory")
    public void process(Channel channel, Message message) {
        log.info("[消费MQ] message:{}",message);

        try {

            String messageVal = new String(message.getBody(), Charset.defaultCharset().name());
            User user = JSONObject.parseObject(messageVal, User.class);
            log.info("[消费MQ] user:{}", user);
        } catch (Exception e) {
            log.error("[消费MQ]消费异常", e);
        }

        try {
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
        } catch (IOException e) {
            log.error("[消费MQ]ACK异常", e);
        }
    }
}
```

## 五、相关代码地址
https://github.com/GaryLeeeee/lee-code-repository/tree/master/lee-mq-code/src/main/java/com/garylee/mq/rabbitmq

## 六、遇到的问题
### 1.Caused by: java.lang.ClassNotFoundException: com.fasterxml.jackson.databind.ObjectMapper
解决方案：添加依赖
```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```

### 2.Caused by: java.net.ConnectException: Connection refused: connect
解决方案：本地RabbitMQ的连接端口默认是5672，不是15672(后台管理页面端口)

### 3.Caused by: com.rabbitmq.client.AuthenticationFailureException: ACCESS_REFUSED - Login was refused using authentication mechanism PLAIN. For details see the broker logfile.
原因：guest账号出于安全考虑，不允许直接连接使用
解决方案：新增一个账号，并设置权限
![RabbitMQ添加账号](/images/RabbitMQ添加账号.png)
![RabbitMQ账号添加权限](/images/RabbitMQ账号添加权限.png)

