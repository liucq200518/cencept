# 概述

一个服务存在多个实例时，`WebSocket`通过网关会被负载均衡连接到其中任意一个实例上。

而当一个服务实例发送消息时，连接另一个实例的客户端就会收不到消息

为了解决这个问题，该库提供了一种解决方案，开箱即用

只需要添加一个配置注解，就可以像单体应用一样使用`WebSocket`

也可以通过简单的自定义来支持更复杂的业务

本库同时兼容`Webmvc`和`Webflux`，使用方式上没有任何区别

# 2.x.x 新特性

注意事项：`2.x.x`版本与`1.x.x`版本不兼容，如有自定义组件的话升级需要注意

1. 支持`Redis(Redisson)/RabbitMQ/Kafka`的订阅转发方式

2. 订阅转发支持主从配置，当主订阅转发不可用时会自动切换到从订阅，当主订阅转发恢复可用时会切回主订阅

3. 提供发送重试配置（不过还是推荐使用自带的重试如`Kafka/RabbitMQ`）

# 集成

```grade
implementation 'com.github.linyuzai:concept-websocket-loadbalance-spring-boot-starter:2.0.0'

implementation 'org.springframework.boot:spring-boot-starter-websocket'//webmvc需要添加websocket依赖，webflux不需要
```

```xml
<dependency>
  <groupId>com.github.linyuzai</groupId>
  <artifactId>concept-websocket-loadbalance-spring-boot-starter</artifactId>
  <version>2.0.0</version>
</dependency>

<!--webmvc需要添加websocket依赖，webflux不需要-->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

# 使用

在启动类上添加注解`@EnableWebSocketLoadBalanceConcept`启用功能

```java
@EnableWebSocketLoadBalanceConcept
@SpringBootApplication
public class WsServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(WsServiceApplication.class, args);
    }
}
```

注入`WebSocketLoadBalanceConcept`即可跨实例发送消息

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

客户端的连接地址为`ws(s)://{服务的地址}/concept-websocket/{自定义路径}`

其中`concept-websocket`为默认的前缀，可在配置中自定义（JAVAX不支持自定义）

# 配置属性

```yaml
concept:
  websocket:
    type: auto #JAVAX/SERVLET/REACTIVE，AUTO自动适配，默认AUTO
    server: #服务配置
      default-endpoint: #默认端点
        enabled: true #是否启用默认端点，默认true
        path-selector: #Path选择器
          enabled: false #是否启用Path选择器，默认false
        user-selector: #User选择器
          enabled: false #是否启用User选择器，默认false
      message:
        retry:
          times: 0 #客户端重试次数，默认不重试
          period: 0 #客户端重试间隔，单位ms，默认0ms
      heartbeat: #心跳配置
        enabled: true #是否启用心跳，默认true
        period: 60000 #心跳间隔，单位ms，默认1分钟
        timeout: 210000 #超时时间，单位ms，默认3.5分钟，3次心跳间隔
    load-balance: #负载均衡（转发）配置
      subscriber-master: websocket #订阅者主通道，默认 websocket
      subscriber-slave: none #订阅者从通道，默认无从订阅者
      message:
        retry:
          times: 0 #转发重试次数，默认不重试
          period: 0 #转发重试间隔，单位ms，默认0ms
      monitor: #监控配置
        enabled: true #是否启用监控，默认true
        period: 30000 #轮训间隔，单位ms，默认30s
        logger: false #是否启用日志，默认false
      heartbeat: #心跳配置
        enabled: true #是否启用心跳，默认true
        period: 60000 #心跳间隔，单位ms，默认1分钟
        timeout: 210000 #超时时间，单位ms，默认3.5分钟，3次心跳间隔
    executor:
      thread-pool-size: 1 #线程池大小，默认1
```

# 原理

通过服务间的`websocket`连接或是`Redis`和`MQ`等中间件转发消息

# 连接域

由于本库支持多种连接（当前包括`WebSocket`和`Netty`）同时配置，所以引入连接域来进行限制。

在自定义组件时需要指定该组件所适配的连接类型（`NettyScoped.NAME/WebSocketScoped.NAME`）

### 事件监听器

可以实现`WebSocketEventListener`来监听事件

### 生命周期监听器

可以实现`WebSocketLifecycleListener`来监听生命周期（连接发布/连接关闭）

# 主从订阅

可以配置2种订阅转发方式提高容错

如以`Kafka`为主，`Redis`为从

当`Kafka`转发消息失败后，会切换到`Redis`重新转发，并开始对`Kafka`进行定时的的心跳检测

等到`Kafka`心跳检测正常，则重新切回到`Kafka`

### 配置

- `concept.websocket.load-balance.subscriber-master`用于配置主订阅器

- `concept.websocket.load-balance.subscriber-slave`用于配置从订阅器

### 美枚举

|配置|说明|
|-|-|
|websocket|每个服务实例ws双向连接，只能配置为主订阅器，且不支持主从切换|
|websocket_ssl|每个服务实例wss双向连接，只能配置为主订阅器，且不支持主从切换|
|kafka_topic|kafka转发|
|rabbit_fanout|rabbit转发|
|redis_topic|redis发布订阅|
|redis_topic_reactive|redis发布订阅|
|redisson_topic|redisson发布订阅|
|redisson_topic_reactive|redisson发布订阅|
|redisson_shared_topic|redisson发布订阅|
|redisson_shared_topic_reactive|redisson发布订阅|
|none|不转发|

# 幂等转发

通过`Kafka/RabbitMQ`转发消息时可能会重复消费

提供`MessageIdempotentVerifier`抽象`messageId`生成以及验证是否重复

默认缓存所有`messageId`在内存中（存在缓存越来越大的问题，建议自定义使用`Redis`或数据库存储）

可通过`MessageIdempotentVerifierFactory`自定义并注入`Spring`生效

# 消息发送

可通过自定义`ConnectionSelector`来实现消息的准确发送，如发送给某个路径或带有某个参数的连接

### 给指定路径的客户端发送消息

假设前端连接的`WebSocket`地址为`ws://localhost:8080/concept-websocket/sample`，`sample`为我们自定义路径

在配置中启用路径选择器

```yaml
concept:
  websocket:
    server: 
      default-endpoint: 
        path-selector: 
          enabled: true #启用Path选择器
```

使用`PathMessage`给所有的`sample`客户端发送消息

```java
@RestController
@RequestMapping("/ws")
public class WsController {

    @Autowired
    private WebSocketLoadBalanceConcept concept;

    @RequestMapping("/send-path")
    public void sendPath(@RequestParam String msg) {
        concept.send(new PathMessage(msg, "sample"));
    }
}
```

### 给指定用户发送消息

假设前端连接的`WebSocket`地址为`ws://localhost:8080/concept-websocket/user?userId=1`

其中`userId`为固定参数名

在配置中启用路径选择器

```yaml
concept:
  websocket:
    server: 
      default-endpoint: 
        user-selector: 
          enabled: true #启用用户选择器
```

使用`UserMessage`给指定的用户发送消息

```java
@RestController
@RequestMapping("/ws")
public class WsController {

    @Autowired
    private WebSocketLoadBalanceConcept concept;

    @RequestMapping("/send-user")
    public void sendUser(@RequestParam String msg) {
        concept.send(new UserMessage(msg, "1"));
    }
}
```

### 组合条件

如果既有路径条件，又有用户条件

```java
@RestController
@RequestMapping("/ws")
public class WsController {

    @Autowired
    private WebSocketLoadBalanceConcept concept;

    @RequestMapping("/send-path-user")
    public void sendUser(@RequestParam String msg) {
        Message message = new ObjectMessage(msg);
        PathMessage.condition("sample").apply(message);
        UserMessage.condition("1").apply(message);
        concept.send(message);
    }
}
```

### 选择过滤器

默认情况下，`ConnectionSelector`对于每次发送消息都只会有一个生效（只能生效一种过滤条件）

对于`ConnectionSelector`提供扩展`FilterConnectionSelector`

将会作为一个过滤器来支持多种条件，即[组合条件](#组合条件)模式

# 消息接收

实现`WebSocketMessageHandler`来处理客户端发送的消息

# 编解码链

# 组件说明

所有组件均可自定义扩展（可能需要指定`Order`来保证自定义的组件生效）

### 连接仓库

`ConnectionRepository`用于缓存连接实例

默认使用`Map<String, Map<Object, Connection>> connections = new ConcurrentHashMap<>();`缓存在内存中

通过`ConnectionRepositoryFactory`自定义并注入`Spring`容器

### 连接服务管理器

`ConnectionServerManager`用于获取其他服务实例信息（`ws`双向连接中使用）和自身服务信息

默认使用`DiscoveryClient`和`Registration`来获得信息

通过`ConnectionServerManagerFactory`自定义并注入`Spring`容器

### 连接订阅器

`ConnectionSubscriber`用于订阅其他服务的消息

可在配置文件中配置主从订阅器

- `concept.websocket.load-balance.subscriber-master`

- `concept.websocket.load-balance.subscriber-slave`

也可以通过`ConnectionSubscriberFactory`或`MasterSlaveConnectionSubscriberFactory`自定义并注入`Spring`容器

### 连接工厂

`ConnectionFactory`用于扩展`Connection`（如`WebSocketConnection/NettyConnection`）

通过`ConnectionFactory`自定义并注入`Spring`容器

### 连接选择器

`ConnectionSelector`用于在发送消息时选择发送给哪些连接

