# 概述

一个服务存在多个实例时，`WebSocket`通过网关会被负载均衡连接到其中任意一个实例上

而当一个实例发送消息时，连接另一个实例的客户端就会收不到消息

为了解决这个问题，该库提供了一种解决方案

只需要添加一个配置注解，就可以像单体应用一样使用`WebSocket`，开箱即用

也可以通过简单的自定义来支持更复杂的业务

本库同时兼容`Web`和`Webflux`，使用方式上没有任何区别

# 集成

```gradle
implementation 'com.github.linyuzai:concept-websocket-loadbalance-spring-boot-starter:1.0.3'
```

```xml
<dependency>
  <groupId>com.github.linyuzai</groupId>
  <artifactId>concept-websocket-loadbalance-spring-boot-starter</artifactId>
  <version>1.0.3</version>
</dependency>
```

# 使用

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

客户端的连接地址为`ws://{服务的地址}/concept-websocket/{自定义路径}`

其中`concept-websocket`为默认的固定前缀

# 配置文件

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
        logger: false #是否启用日志，默认false
      heartbeat: #心跳配置
        enabled: true #是否启用心跳，默认true
        period: 60000 #心跳间隔，单位ms，默认1分钟
        timeout: 210000 #超时时间，单位ms，默认3.5分钟，3次心跳间隔
```

# 原理

通过服务间进行相互的`WebSocket`连接来实现消息转发

![原理](https://user-images.githubusercontent.com/18523183/170614224-6c33e2f6-9a89-446d-b053-17464beb64f5.svg)

### 订阅流程

![订阅](https://user-images.githubusercontent.com/18523183/171103206-4b61b056-2192-4696-aa8a-94b560767a86.svg)

### 连接管理

![管理](https://user-images.githubusercontent.com/18523183/170669799-466776f6-d591-414c-891d-f17449c853ec.svg)

### 消息发送

![消息](https://user-images.githubusercontent.com/18523183/170673572-f5a62d61-379a-4e69-ac2c-d408076bfa6a.svg)

# 连接订阅

将消息转发抽象为对其他服务实例消息的订阅

通过`ConnectionSubscriber`来和其他服务实例建立连接

默认实现了`WebSocket`的双向连接

可以自定义`ConnectionSubscriber`使用`MQ`或`HTTP`等其他消息转发方式

需要注意，该流程在服务启动后触发，通过`ApplicationRunner`实现

### `WebSocket`双向连接

通过注册中心获得其他服务实例的信息（默认通过注册中心获取，同时支持自定义服务列表）并向这些服务实例发起连接

连接成功后发送自身服务实例信息，其他服务实例通过收到的服务实例信息进行反向连接

### 连接订阅监控

存在一个定时任务定时检查（默认30s）服务实例间的连接是否完整

每个服务实例会检查自己需要连接的其他服务实例

当发现与某个服务实例不存在连接会尝试连接，或连接已经死亡（超过一定时间没有心跳时间更新）会重新连接

# 服务信息

通过`ConnectionServerProvider`来获取服务实例的信息（从抽象的层面上讲并不一定局限于同服务的实例）

默认通过`Spring Cloud`的服务发现来获得注册中心上维护的同服务的实例信息

# 消息

通过`Message`体现

### 消息工厂

通过`MessageFactory`将我们传入的任意对象封装成消息对象

默认使用`ObjectMessageFactory`讲对象封装成`ObjectMessage`

可以针对部分类型数据自定义消息工厂

### 消息编码器

通过`MessageEncoder`在发送消息时进行编码

### 消息解码器

通过`MessageDecoder`在接收消息时进行解码

### 消息编解码适配器

通过`MessageCodecAdapter`统一编解码器的入口

- 普通客户端收发消息的编解码器
- 服务实例间订阅消息的编解码器
- 消息转发的编解码器

可以自定义编解码适配器用于特定的消息编解码

### 消息发送

默认编码为`json`字符串

当一个消息发送时将会适配连接选择器`ConnectionSelector`

连接选择器将会根据消息返回需要发送该消息的连接

消息类`Message`的`headers`字段用于自定义消息头

连接类`Connection`的`metadata`字段用于自定义元数据

可以通过匹配两者来筛选连接以支持复杂的业务场景

##### 给指定路径的客户端发送消息

假设前端连接的`WebSocket`地址为`ws://localhost:8080/concept-websocket/sample`

其中`concept-websocket`为默认的固定前缀，`sample`为我们自定义路径

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

##### 给指定用户发送消息

假设前端连接的`WebSocket`地址为`ws://localhost:8080/concept-websocket/user?userId=1`

其中`userId`为固定参数名

在配置中启用路径选择器

```yaml
concept:
  websocket:
    server: 
      default-endpoint: 
        user-selector: 
          enabled: true #启用Path选择器
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

### 消息接收

通过实现`MessageHandler`接收客户端发送的消息

# 生命周期监听

通过实现`LifecycleListener`监听生命周期

# 异常处理

通过实现`ErrorHandler`处理异常，默认将会通过`logger`打印

# 事件

|事件|说明|
|-|-|
|`ConnectionLoadBalanceConceptInitializeEvent`|`ConnectionLoadBalanceConcept`初始化|
|`ConnectionLoadBalanceConceptDestroyEvent`|`ConnectionLoadBalanceConcept`销毁|
|`ConnectionEstablishEvent`|连接建立|
|`ConnectionCloseEvent`|连接关闭|
|`ConnectionCloseErrorEvent`|连接关闭异常|
|`ConnectionErrorEvent`|连接异常|
|`ConnectionSubscribeErrorEvent`|连接订阅异常|
|`MessagePrepareEvent`|消息准备|
|`MessageSendEvent`|消息发送|
|`MessageSendErrorEvent`|消息发送异常|
|`DeadMessageEvent`|当一个消息不会发送给任何一个连接|
|`MessageDecodeErrorEvent`|消息解码异常|
|`MessageReceiveEvent`|消息接收|
|`EventPublishErrorEvent`|事件发布异常|
|`UnknownCloseEvent`|未知的连接关闭|
|`UnknownErrorEvent`|未知的连接异常|
|`UnknownMessageEvent`|未知的消息|

# 默认服务端点配置

可以自定义`DefaultEndpointCustomizer`来配置

`Servlet`环境下会回调`WebSocketHandlerRegistration`

`Reactive`环境下会拿回调`ReactiveWebSocketLoadBalanceHandlerMapping`