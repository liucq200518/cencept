# 概述

一个服务存在多个实例时，`websocket`通过网关会被负载均衡连接到其中任意一个实例上

而当一个实例发送消息时，连接另一个实例的客户端就会收不到消息

为了解决这个问题，该库提供了一种解决方案

只需要添加一个配置注解，就可以像单体应用一样使用`websocket`，开箱即用

也可以通过简单的自定义来支持更复杂的业务

本库同时兼容`Web`和`Webflux`，使用方式上没有任何区别

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

# 原理

通过服务间进行相互的`WebSocket`连接来实现消息转发

//TODO tu

# 连接订阅

将消息转发抽象为对其他服务实例消息的订阅

通过`ConnectionSubscriber`来和其他服务实例建立连接

默认实现了`WebSocket`的双向连接

也可以自定义使用`MQ`或`HTTP`等其他消息转发方式

需要注意，该流程在服务启动后触发，通过`ApplicationRunner`实现

### `WebSocket`双向连接

通过注册中心获得其他服务实例的信息（默认通过注册中心获取，同时支持自定义服务列表）并向这些服务实例发起连接

连接成功后发送自身服务实例信息，其他服务实例通过收到的服务实例信息进行反向连接

### 连接订阅监控

存在一个定时任务定时检查（默认30s）服务实例间的连接是否完整

每个服务实例会检查自己需要连接的其他服务实例

当发现与某个服务实例不存在连接或连接已经死亡（超过一定时间没有心跳时间更新）会尝试连接

# 服务信息

通过`ConnectionServerProvider`来获取服务实例的信息（从抽象的层面上讲并不一定局限于同服务的实例）

默认通过`Spring Cloud`的服务发现来获得注册中心上维护的同服务的实例信息

# 消息

通过`Message`体现

### 消息工厂

通过`MessageFactory`将我们传入的对象封装成消息对象

### 消息编码器

### 消息解码器

### 消息发送

当一个消息发送时将会适配连接选择器`ConnectionSelector`

连接选择器将会根据消息返回需要发送该消息的连接

消息类`Message`的`headers`字段用于自定义消息头

连接类`Connection`的`metadata`字段用于自定义元数据

所以可以通过匹配两者来筛选连接以支持复杂的业务场景

### 消息接收

通过实现`MessageHandler`接收客户端发送的消息

# 事件


