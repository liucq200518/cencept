# 介绍

平时在协同开发时，可能会调用到别人的服务接口，导致开发调试的效率低下

为解决这一问题，该库提供了对某些接口指定服务实例的路由功能来提升效率

# 集成

```gradle
implementation 'com.github.linyuzai:concept-router-spring-boot-starter:1.0.0'
```

```xml
<dependency>
  <groupId>com.github.linyuzai</groupId>
  <artifactId>concept-router-spring-boot-starter</artifactId>
  <version>1.0.0</version>
</dependency>
```

# 功能

- gateway指定转发服务
- feign指定调用服务

只需引入包无需额外的配置

# 管理界面

`服务地址/concept-router/index.html`

如果对路径做了拦截记得将`/concept-router/**`加入白名单

# 判断是否生效

在`Gateway`转发或`Feign`调用时输出`Router >>`前缀的日志即路由生效

# 版本适配

- Spring Boot 2.6.x（loadbalancer）
- Spring Boot 2.2.x (ribbon)
- Spring Boot 2.0.x (ribbon)

如果集成之后出现报错或不生效请联系我并提供`Spring Boot`和`Spring Cloud`版本以及是否指定了负载均衡组件