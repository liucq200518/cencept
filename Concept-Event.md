# 概述

该库支持在一个项目中（同时）配置多个 `Kafka` 和 `RabbitMQ`(还未集成)

如有必要还可扩展 `ActiveMQ` 或 `RocketMQ` 等任何符合事件模型的组件

同时以简单的事件模型作为抽象，支持不对任何中间件强绑定的场景

支持动态添加（未实现）

# 集成

```gradle
implementation 'com.github.linyuzai:concept-event-spring-boot-starter:0.1.0'
```

```xml
<dependency>
  <groupId>com.github.linyuzai</groupId>
  <artifactId>concept-event-spring-boot-starter</artifactId>
  <version>0.1.0</version>
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


# 接收事件