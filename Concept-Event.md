# 概述

该库支持在一个项目中（同时）配置多个 `Kafka` 和 `RabbitMQ`(还未集成)

如有必要还可扩展 `ActiveMQ` 或 `RocketMQ` 等任何符合事件模型的组件

同时以简单的事件模型作为抽象，支持不对任何中间件强绑定的场景

支持动态添加（未实现）

# 集成

```gradle
implementation 'com.github.linyuzai:concept-event-spring-boot-starter:0.4.0'
```

```xml
<dependency>
  <groupId>com.github.linyuzai</groupId>
  <artifactId>concept-event-spring-boot-starter</artifactId>
  <version>0.4.0</version>
</dependency>
```

在项目中使用`@EnableEventConcept`启用功能

```java
@EnableEventConcept
@SpringBootApplication
public class ConceptSampleApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConceptSampleApplication.class, args);
    }
}
```

# 配置Kafka

```yaml
concept:
  event:
    kafka:
      endpoints: #在该节点下配置多个kafka，属性同spring.kafka
        kafka1: #端点名称-kafka1
          inherit: parent #继承名称为parent的端点配置
          bootstrap-servers:
            - 192.168.30.100:9092
          consumer:
            group-id: kafka1
        kafka2: #端点名称-kafka2
          inherit: parent #继承名称为parent的端点配置
          bootstrap-servers:
            - 192.168.30.101:9092
          consumer:
            group-id: kafka2
        parent: #作为其他端点的父配置
          enabled: false #是否启用该端点，这里作为父配置不启用
          producer:
            retries: 0
            acks: 1
          consumer:
            enable-auto-commit: false
            auto-offset-reset: earliest
          template:
            default-topic: sample
          listener:
            ack-mode: manual_immediate
```

`kafka`的配置属性同`spring.kafka`

额外提供配置继承可将一些相同的配置提取出来，使用`inherit`属性指定继承的端点

# 发布事件

### 简单方式

```java
@RestController
@RequestMapping("/concept-event")
public class EventController {

    @Autowired
    private EventConcept concept;

    @GetMapping("/send")
    public void send() {
        concept.event("123").publish();
    }
}
```

需要注意该方式需要配置一些默认的组件

### 自定义方式

@RestController
@RequestMapping("/concept-event")
public class EventController {

    @Autowired
    private EventConcept concept;

    @GetMapping("/send")
    public void send() {
        concept.event("123")
                .exchange(EventExchange) //指定发布到哪些端点（多个Kafka中哪几个）
                .context(Map) //配置上下文（用于满足自定义数据传递）
                .encoder(EventEncoder) //指定事件编码器（如把对象转成json）
                .error(EventErrorHandler) //指定异常处理器（发布异常的后续操作）
                .publish(EventPublisher); //指定事件发布器（如使用KafkaTemplate给指定的Topic发消息）
    }
}
```

##### 示例

# 接收事件

### 简单方式

```java
@Configuration
public class EventSubscriberRegister implements ApplicationRunner {

    @Autowired
    public EventConcept concept;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        concept.event().subscribe();//指定事件订阅器
    }
}
```

需要注意该方式需要配置一些默认的组件

### 自定义方式

```java
@Configuration
public class EventSubscriberRegister implements ApplicationRunner {

    @Autowired
    public EventConcept concept;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        concept.event(Type) //指定事件类型（通过解码器处理）
                .exchange(EventExchange) //指定订阅哪些端点（多个Kafka中哪几个）
                .context(Map) //配置上下文（用于满足自定义数据传递）
                .decoder(EventDecoder) //指定事件解码器（如把json转成对象）
                .error(EventErrorHandler) //指定异常处理器（订阅或消费异常的后续操作）
                .subscribe(EventSubscriber); //指定时间订阅器（如订阅哪个Topic）
    }
}
```

##### 示例
