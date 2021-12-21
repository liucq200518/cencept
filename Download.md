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

借助`@Download`注解，你可以把被下载的资源写在注解的`source`参数中或者作为方法的返回值`return`

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

- concept-download-core
  - 核心模块
- concept-download-aop
  - 切面模块
- concept-download-web-servlet
  - `Servlet`支持模块
  - `Webflux`暂未支持，有需要再加
- concept-download-source-classpath
  - `ClassPathResource`支持
- concept-download-source-okhttp
  - 基于`OkHttp`的HTTP资源支持
- concept-download-load-coroutines
  - 基于`Kotlin`协程的I/O请求支持
- concept-download-spring-boot-starter
  - `SpringBoot`自动配置模块，包含部分模块

### `@Download` 注解说明

- `@Download(source = {})`
  - 需要下载的内容，但是优先级低于返回值
  - 如果方法返回值不为`null`则会使用返回值作为下载的内容
- `@Download(inline = false)`
  - 如果为`true`，可以直接在浏览器预览
  - 需要配合`contentType`，如图片或视频，默认`false`
- `@Download(filename = "")`
  - 指定下载的文件名
  - 如果不指定则会获取下载内容的名称，如文件则使用文件名
- `@Download(contentType = "")`
  - 默认`application/octet-stream`
- `@Download(compressFormat = "")`
  - 压缩格式，默认`zip`
- `@Download(forceCompress = false)`
  - 强制压缩
  - 如果为`true`，不管下载的对象有几个都会压缩
  - 如果为`false`，有多个下载对象时压缩，只有一个下载对象时不压缩
- `@Download(charset = "")`
  - 如果下载包含中文的文本文件出现乱码，可以尝试指定编码
- `@Download(headers = {})`
  - 统一的响应头，每2个为一组
- `@Download(extra = "")`
  - 额外的数据，当需要自行编写额外流程业务时可能会用到

### 支持的下载类型

##### 默认支持

|类型|匹配|实现类|工厂|示例|依赖|
|-|-|-|-|-|-|
|文件|"file:"前缀的字符串|`FileSource`|`FilePrefixSourceFactory`|`file:/Users/Shared/README.txt`||
|文件|`File`对象|`FileSource`|`FileSourceFactory`|`new File("/Users/Shared/README.txt")`||
|user.home目录下的文件|"user.home:","user-home:","user_home:"前缀的字符串|`FileSource`|`UserHomeSourceFactory`|`user.home:/Public/README.txt`||
|classpath目录下的资源|"classpath:"前缀的字符串|`ClassPathResourceSource`|`ClassPathPrefixSourceFactory`|`classpath:/download/README.txt`|`source-classpath`|
|classpath目录下的资源|`ClassPathResource`对象|`ClassPathResourceSource`|`ClassPathResourceSourceFactory`|`new ClassPathResource("/download/README.txt")`|`source-classpath`|
|文本文件|任意的String对象|`TextSource`|`TextSourceFactory`|"任意的文本将会直接作为文本文件处理""||
|输入流|`InputStream`对象|`InputStreamSource`|`InputStreamSourceFactory`|任意的输入流||
|HTTP资源|http或https的url字符串|`OkHttpSource`|`OkHttpSourceFactory`|"http://127.0.0.1:8080/concept-download/image.jpg"|`source-okhttp`|

**同时支持上述类型任意组合的数组或集合**

```java
@Download(filename = "压缩包.zip")
@GetMapping("/list")
public List<Object> list() {
    List<Object> list = new ArrayList<>();
    list.add(new File("/Users/Shared/README.txt"));
    list.add(new ClassPathResource("/download/image.jpg"));
    list.add("http://127.0.0.1:8080/concept-download/video.mp4");
    return list;
}
```

##### 自定义支持

实现`SourceFactory`来自定义支持任意的类型和对象

### 网络资源的并发处理

