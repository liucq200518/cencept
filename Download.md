### 示例说明

用简单的方式实现一个下载接口，直观感受如下

```java
@Download(source = "classpath:/download/README.txt")
@GetMapping("/classpath")
public void classpath() {
}

@Download
@GetMapping("/file")
public File file() {
    return new File("/Users/Shared/README.txt");
}

@Download
@GetMapping("/http")
public String http() {
    return "http://127.0.0.1:8080/concept-download/image.jpg";
}
```

借助`@Download`注解，你可以把被下载的资源写在注解的`source`参数中或者作为方法的返回值

两者没有任何区别，只是返回值支持动态的对象

### 集成

当前版本

```gradle
//包含了core，aop，web-servlet，source-classpath模块
implementation 'com.github.linyuzai:concept-download-spring-boot-starter:version'

//如果需要支持http请求则需要手动依赖此模块
implementation 'com.github.linyuzai:concept-download-source-okhttp:version'

//如果需要使用Kotlin协程来来做HTTP请求的并发需要手动依赖此模块，core也提供了线程池的方式需要手动配置
implementation 'com.github.linyuzai:concept-download-load-coroutines:version'
```

```maven
```

模块：

- concept-download-core(核模块)
- concept-download-aop(切面模块)
- concept-download-web-servlet(Servlet支持模块，Webflux暂未支持，如有需要再加)
- concept-download-source-classpath(ClassPathResource支持)
- concept-download-source-okhttp(基于OkHttp的HTTP资源支持)
- concept-download-load-coroutines(基于Kotlin协程的I/O请求支持)
- concept-download-spring-boot-starter(SpringBoot自动配置模块，包含部分模块)

### 支持的下载类型
