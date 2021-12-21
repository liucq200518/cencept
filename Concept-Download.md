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
  - `SpringBoot`自动配置模块
  - 包含 `core` `aop` `web-servlet` `source-classpath`

### `@Download` 注解说明

- `@Download(source = {})`
  - 需要下载的内容，但是优先级低于返回值
  - 如果方法返回值不为`null`则会使用返回值作为下载的内容
- `@Download(inline = false)`
  - 如果为`true`，可以直接在浏览器预览
  - 需要配合`contentType`，如图片或视频，默认`false`
- `@Download(filename = "")`
  - 指定下载时浏览器上显示的名称
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

### 整体流程

整个下载流程由`DownloadHandler`和`DownloadHandlerChain`实现链式处理

- `InitializeContextHandler`
  - 初始化下载上下文
- `CreateSourceHandler`
  - 解析适配各种类型的下载数据
- `LoadSourceHandler`
  - 针对一些网络资源或需要耗时处理的资源提前加载
- `CompressSourceHandler`
  - 压缩处理
- `WriteResponseHandler`
  - 写入响应
- `DestroyContextHandler`
  - 销毁下载上下文

##### 扩展流程

可以自定义实现`DownloadHandler`或`AutomaticDownloadHandler`

### 支持的下载类型

所有的下载对象最终都会通过`Source`体现，作为原始的下载数据的抽象

##### 默认支持

|类型|匹配|实现类|工厂|示例|依赖|
|-|-|-|-|-|-|
|文件|"file:"前缀的字符串|`FileSource`|`FilePrefixSourceFactory`|`file:/Users/Shared/README.txt`||
|文件|`File`对象|`FileSource`|`FileSourceFactory`|`new File("/Users/Shared/README.txt")`||
|user.home目录下的文件|"user.home:","user-home:","user_home:"前缀的字符串|`FileSource`|`UserHomeSourceFactory`|`user.home:/Public/README.txt`||
|classpath目录下的资源|"classpath:"前缀的字符串|`ClassPathResourceSource`|`ClassPathPrefixSourceFactory`|`classpath:/download/README.txt`|`source-classpath`|
|classpath目录下的资源|`ClassPathResource`对象|`ClassPathResourceSource`|`ClassPathResourceSourceFactory`|`new ClassPathResource("/download/README.txt")`|`source-classpath`|
|文本文件|任意的String对象|`TextSource`|`TextSourceFactory`|"任意的文本将会直接作为文本文件处理"||
|输入流|`InputStream`对象|`InputStreamSource`|`InputStreamSourceFactory`|任意的输入流||
|HTTP资源|http或https的url字符串|`OkHttpSource`|`OkHttpSourceFactory`|http://127.0.0.1:8080/concept-download/image.jpg|`source-okhttp`|

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

```java
/**
 * 数据源工厂 / Factory of download source
 */
public interface SourceFactory extends OrderProvider {

    /**
     * 是否支持某个对象 / Whether an object is supported
     *
     * @param source  需要下载的数据对象 / Object to download
     * @param context 下载上下文 / Context of download
     * @return 是否支持 / If supported
     */
    boolean support(Object source, DownloadContext context);

    /**
     * 创建 / Create
     *
     * @param source  需要下载的数据对象 / Object to download
     * @param context 下载上下文 / Context of download
     * @return 下载源 / Source
     */
    Source create(Object source, DownloadContext context);
}

```

### 网络资源的并发处理

针对一些网络资源，如HTTP、FTP等，需要进行并发的加载，通过`SourceLoaderInvoker`来实现

|类型|实现类|说明|依赖|
|-|-|-|-|
|串行|`SerialSourceLoaderInvoker`|按顺序加载，适用于本地文件||
|线程池|`ExecutorSourceLoaderInvoker`|依赖线程池加载，适合网络资源||
|协程|`CoroutinesSourceLoaderInvoker`|依赖协程加载，适合网络资源|`load-coroutines`|

每个`Source`都可以单独指定`asyncLoad`属性来控制是否需要异步加载，目前`OkHttpSource`默认为`true`，其他默认都为`false`

通过手动注入来切换不同的加载方式

```java
@Configuration
public class ConceptDownloadConfig {

    @Bean
    public CoroutinesSourceLoaderInvoker coroutinesSourceLoaderInvoker() {
        System.out.println("如果需要进行HTTP请求可以使用协程加载！");
        return new CoroutinesSourceLoaderInvoker();
    }

    //或者

    @Bean(destroyMethod = "shutdown")
    public ExecutorSourceLoaderInvoker executorSourceLoaderInvoker() {
        System.out.println("如果需要进行HTTP请求可以使用线程池加载！");
        return new ExecutorSourceLoaderInvoker(Executors.newFixedThreadPool(5));
    }
}

```

##### 自定义并发处理

可以自定义实现`SourceLoaderInvoker`或`ParallelSourceLoaderInvoker`或`ConcurrentSourceLoaderInvoker`

```java
/**
 * 下载源加载器的调用器 / Invoker to invoke SourceLoader
 */
public interface SourceLoaderInvoker {

    /**
     * 调用加载器 / Invoke loader
     *
     * @param loaders 加载器 / Loaders
     * @param context 下载上下文 / Context of download
     * @return 加载结果 / Results of loadings
     * @throws IOException I/O exception
     */
    Collection<SourceLoadResult> invoke(Collection<? extends SourceLoader> loaders, DownloadContext context) throws IOException;
}

```

通过调用对应的方法触发加载，并返回加载结果

```java
SourceLoadResult result = SourceLoader.load(context);
```

##### 异常处理

默认实现为`RethrowLoadedSourceLoadExceptionHandler`将在加载结束时进行异常判断

可以自定义实现`SourceLoadExceptionHandler`

```java
/**
 * 下载源加载的异常处理器 / Handler to handle exception when source loading
 */
public interface SourceLoadExceptionHandler {

    /**
     * 每个异常都会回调 / Each exception will be called back
     * 可能是在线程池中的某个线程中回调 / It may be a callback in a thread in the thread pool
     *
     * @param e 异常 / exception
     */
    void onLoading(SourceLoadException e);

    /**
     * 加载结束后，如果有异常将会回调 / If there is any exception, it will be called back after loading
     * 会在触发下载的主线程回调 / Will call back in the main thread of the download
     *
     * @param exceptions 一个或多个异常 / One or more exceptions
     */
    void onLoaded(Collection<SourceLoadException> exceptions);
}

```

### 网络资源的缓存处理

##### 配置文件

```yaml
concept:
  download:
    source:
      cache:
        enabled: true //是否启用
        path: / //缓存目录
        delete: false //下载结束后是否删除
```

##### 代码全局配置

```java
@Configuration
public class ConceptDownloadConfig {

    @Bean
    public DownloadConfigurer downloadConfigurer() {
        return new DownloadConfigurer() {
            @Override
            public void configure(DownloadConfiguration configuration) {
                System.out.println("可以在这里设置或覆盖配置文件的配置！");
                configuration.getSource().getCache().setEnabled(true);
                configuration.getSource().getCache().setPath("/");
                configuration.getSource().getCache().setDelete(false);
            }
        };
    }
}
```

##### 注解配置单个方法

```java
@Download(filename = "压缩包.zip")
@SourceCache(group = "source")
@GetMapping("/source-cache")
public String[] sourceCache() {
    return new String[]{
          "http://127.0.0.1:8080/concept-download/text.txt",
          "http://127.0.0.1:8080/concept-download/image.jpg",
          "http://127.0.0.1:8080/concept-download/video.mp4"
    };
}
```

使用`@SourceCache`注解配合`@Download`实现下载资源的缓存处理

##### `@SourceCache` 注解说明

- `@SourceCache(enabled = true)`
  - 是否启用缓存
- `@SourceCache(group = "")`
  - 分组，会在缓存目录下额外创建一个对应的目录作为实际的缓存目录
  - 考虑到不同功能出现相同名称的文件等冲突问题
  - 默认空，不创建，及直接使用配置的缓存目录
- `@SourceCache(delete = false)`
  - 下载结束后是否删除缓存文件

### 资源压缩

默认情况下，如果是单个资源则不会压缩，如果是多个资源或者是一整个文件夹则会压缩处理

可以使用`@Download(forceCompress = true)`强制压缩

目前只实现了Java自带的Zip压缩`ZipSourceCompressor`

##### 自定义压缩

可以自定义实现`SourceCompressor`或`AbstractSourceCompressor`

```java
/**
 * 压缩器 / Compressor to compress source
 */
public interface SourceCompressor extends OrderProvider {

    /**
     * 判断是否支持对应的压缩格式 / Judge whether the corresponding compression format is supported
     *
     * @param format  压缩格式 / Compression format
     * @param context 下载上下文 / Context of download
     * @return 是否支持该压缩格式 / If support this compressed format
     */
    boolean support(String format, DownloadContext context);

    /**
     * 如果支持对应的格式就会调用该方法执行压缩 / This method will be called to perform compression if the corresponding format is supported
     *
     * @param source    {@link Source}
     * @param writer    {@link DownloadWriter}
     * @param cachePath 缓存路径 / The path of cache
     * @param context   Context of download
     * @return An specific compression
     * @throws IOException I/O exception
     */
    Compression compress(Source source, DownloadWriter writer, String cachePath, DownloadContext context) throws IOException;
}

```

### 压缩缓存