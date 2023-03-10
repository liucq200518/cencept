# 概述

主要用于简化请求和响应的拦截扩展

封装`Spring Boot`提供的切面功能

同时支持`webmvc`和`webflux`

# 集成（未发布）

```gradle
implementation 'com.github.linyuzai:concept-cloud-web:1.0.0'
```

或者

```xml
<dependency>
  <groupId>com.github.linyuzai</groupId>
  <artifactId>concept-cloud-web</artifactId>
  <version>1.0.0</version>
</dependency>
```

# 使用

### 注解方式

```java
@Component
public class Intercepts {

    @OnRequest
    public void request() {
        System.out.println("request");
    }

    @OnResponse
    public void response() {
        System.out.println("response");
    }
}
```