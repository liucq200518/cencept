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

未发布

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

NettyEventListener/NettyLifecycleListener

# 主从订阅

# 幂等转发

# 消息接收

# 选择器链

user/path

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

