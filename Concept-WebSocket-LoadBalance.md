# 概述

一个服务存在多个实例时，`websocket`通过网关会被负载均衡连接到其中任意一个实例上

而当一个实例发送消息时，连接另一个实例的客户端就会收不到消息

为了解决这个问题，该库提供了一种解决方案

只需要添加一个注解，就可以像单体应用一样使用`websocket`，开箱即用

也可以通过简单的自定义来支持更复杂的业务

# 集成（未发布）

```gradle
implementation 'com.github.linyuzai:concept-websocket-spring-boot-starter:1.0.0'
```

```xml
<dependency>
  <groupId>com.github.linyuzai</groupId>
  <artifactId>concept-websocket-spring-boot-starter</artifactId>
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

```yaml
concept:
  websocket:
    type: auto #JAVAX/SERVLET/REACTIVE，AUTO自动适配，默认AUTO
    server: #服务配置
      default-endpoint: #默认端点
        enabled: true #是否启用默认端点，默认true
        path-selector: #Path选择器
          enabled: true #是否启用Path选择器，默认false
      heartbeat: #心跳配置
        enabled: true #是否启用心跳，默认true
        period: 60000 #心跳间隔，单位ms，默认1分钟
        timeout: 210000 #超时时间，单位ms，默认3.5分钟，3次心跳间隔
    load-balance: #负载均衡（转发）配置
      protocol: ws #服务间连接协议，默认ws
      logger: true #是否启用日志，互相连接的日志打印，默认true
      monitor: #监控配置
        enabled: true #是否启用监控，默认true
        period: 30000 #轮训间隔，单位ms，默认30s
        logger: true #是否启用日志，默认true
      heartbeat: #心跳配置
        enabled: true #是否启用心跳，默认true
        period: 60000 #心跳间隔，单位ms，默认1分钟
        timeout: 210000 #超时时间，单位ms，默认3.5分钟，3次心跳间隔
```