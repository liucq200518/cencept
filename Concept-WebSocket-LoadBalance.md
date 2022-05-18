# 概述

一个服务存在多个实例时，`websocket`通过网关会被负载均衡连接到其中任意一个实例上

而当一个实例发送消息时，连接另一个实例的客户端就会收不到消息

为了解决这个问题，该库提供了一种解决方案

只需要添加一个注解，就可以像单体应用一样使用`websocket`，开箱即用

也可以通过简单的自定义来支持更复杂的业务

# 集成

```gradle
implementation 'com.github.linyuzai:concept-connection-loadbalance-spring-boot-starter:1.0.0'
```

```xml
<dependency>
  <groupId>com.github.linyuzai</groupId>
  <artifactId>concept-connection-loadbalance-spring-boot-starter</artifactId>
  <version>1.0.0</version>
</dependency>
```

在启动类上添加注解`@EnableWebSocketLoadBalanceConcept`启用功能

```java
@EnableWebSocketLoadBalanceConcept
@EnableDiscoveryClient
@SpringBootApplication
public class AServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(AServiceApplication.class, args);
    }
}
```

# 使用

注入`WebSocketLoadBalanceConcept`就可以跨实例发送消息

```java
@RestController
@RequestMapping("/ws")
public class WsController {

    @Autowired
    private WebSocketLoadBalanceConcept concept;

    @RequestMapping("/send")
    public void send(@RequestParam String msg) {
        concept.send(msg);
    }
}
```

# 配置文件

