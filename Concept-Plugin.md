# 概述

目前主要用于动态加载外部`jar`中的`Class`

当我们遇到一些插件化的需求时可能就会想到通过如动态加载类的方式来实现

`java`中自带有`spi`，不过功能有限，是以类加载作为基础概念

而本库是以插件作为基础概念，类加载作为一种插件的具体实现方式

插件可以是一个`jar`文件，一段`java`代码，一个`Excel`文件...

由于`jar`文件相对`java`来说可能更适合作为插件的载体

所以具体实现了`jar`文件作为插件

# 示例说明

```java
public class ConceptPluginSample {

    /**
     * 插件提取配置
     */
    private final JarPluginConcept concept = new JarPluginConcept.Builder()
            //添加类提取器
            .addExtractor(new ClassExtractor<Class<? extends CustomPlugin>>() {

                @Override
                public void onExtract(Class<? extends CustomPlugin> plugin) {
                    //回调
                }
            })
            .build();

    /**
     * 加载一个 jar 插件
     *
     * @param filePath jar 文件路径
     */
    public void load(String filePath) {
        concept.load(filePath);
    }
}
```

创建一个`JarPluginConcept`并添加一个类提取器`ClassExtractor`，指定提取`CustomPlugin.class`或是其子类

调用`load`方法传入文件地址就会回调`jar`中匹配到的类，如果没有匹配到则不会触发回调

当然如果存在多个符合条件的`Class`可以直接指定集合类型

```java
public class ConceptPluginSample {

    /**
     * 插件提取配置
     */
    private final JarPluginConcept concept = new JarPluginConcept.Builder()
            //添加类提取器
            .addExtractor(new ClassExtractor<List<Class<? extends CustomPlugin>>>() {

                @Override
                public void onExtract(List<Class<? extends CustomPlugin>> plugin) {
                    //回调
                }
            })
            .build();
}
```

# 集成（还未发布）

```gradle
implementation 'com.github.linyuzai:concept-plugin-jar:1.0.0'
```

```xml
<dependency>
  <groupId>com.github.linyuzai</groupId>
  <artifactId>concept-plugin-jar</artifactId>
  <version>1.0.0</version>
</dependency>
```

# 支持类型

|类型|说明|数据结构|
|-|-|-|
|类|支持提取`Class`|`Map<String, Class<CustomPlugin>>`<br>`List<Class<CustomPlugin>>`<br>`Set<Class<CustomPlugin>>`<br>`Collection<Class<CustomPlugin>>`<br>`Class<CustomPlugin>[]`<br>`Class<CustomPlugin>`|
|实例|支持提取实例，支持能够使用无参构造器实例化的类|`Map<String, CustomPlugin>`<br>`List<CustomPlugin>`<br>`Set<CustomPlugin>`<br>`Collection<CustomPlugin>`<br>`CustomPlugin`<br>`CustomPlugin[]`|
|`Properties`文件|支持提取后缀为`.properties`的文件|`Map<String, Properties>`<br>`List<Properties>`<br>`Set<Properties>`<br>`Collection<Properties>`<br>`Properties[]`<br>`Properties`<br>`Map<String, Map<String, String>>`<br>`List<Map<String, String>>`<br>`Set<Map<String, String>>`<br>`Collection<Map<String, String>>`<br>`Map<String, String>[]`<br>`Map<String, String>`|
|任意文件内容|支持提取任意的文件内容（`jar`中会排除`.class`和`.properties`）|`Map<String, byte[]>`<br>`List<byte[]>`<br>`Set<byte[]>`<br>`Collection<byte[]>`<br>`byte[][]`<br>`byte[]`<br>`Map<String, InputStream>`<br>`List<InputStream>`<br>`Set<InputStream>`<br>`Collection<InputStream>`<br>`InputStream[]`<br>`InputStream`<br>`Map<String, String>`<br>`List<String>`<br>`Set<String>`<br>`Collection<String>`<br>`String[]`<br>`String`<br>|
|插件对象|可以获得类加载器，`URL`等数据|`Plugin`<br>`JarPlugin`|
|上下文|插件加载时的中间数据等|`PluginContext`|

当使用`Map`时，对应的`key`将返回文件的路径和名称，如`com/github/linyuzai/concept/sample/plugin/CustomPluginImpl.class`

支持泛型写法

- `List<Class<? extends CustomPlugin>>`
- `Set<? extends CustomPlugin>`
- ...

# 插件动态匹配

动态匹配可以支持任意类型与数量的插件匹配

```java
public class ConceptPluginSample {

    /**
     * 插件提取配置
     */
    private final JarPluginConcept concept = new JarPluginConcept.Builder()
            //添加类提取器
            .extractTo(this)
            .build();

    @OnPluginExtract
    public void onPluginExtract(Class<? extends CustomPlugin> pluginClass, Properties properties) {
        //任意一个参数匹配上都会触发回调
    }

    /**
     * 加载一个 jar 插件
     *
     * @param filePath jar 文件路径
     */
    public void load(String filePath) {
        concept.load(filePath);
    }
}
```

当我们既要获得某些指定的类又想要同时获得配置文件（假设包里定义了一个`properties`文件）

可以直接定义一个方法，设置参数为我们需要提取的类和配置文件，再在方法上标注`@OnPluginExtract`

然后使用`extractTo`将定义了上述方法的对象传入就行了

### 注解支持

动态匹配还提供了更精准化的注解配置

|注解|说明|
|-|-|
|`@PluginPath`|路径匹配|
|`@PluginName`|名称匹配|
|`@PluginProperties`|`properties`文件匹配|
|`@PluginPackage`|包名匹配|
|`@PluginClassName`|类名匹配|
|`@PluginClass`|类匹配|
|`@PluginAnnotation`|类上注解匹配|

其中`@PluginProperties`可以单独指定`key`

- `@PluginProperties("concept-plugin.a")`可以直接得到对应的`String`值（只能是`String`没有做类型转换）
- `@PluginProperties("concept-plugin.map.**")`可以获得`concept-plugin.map`为前缀的`Map<String, String>`

当存在多个能匹配上的对象（类，配置文件等等）时，可以通过上述注解保证唯一或是使用集合类

由于匹配字符串使用的都是`Spring`中的`AntPathMatcher`，所有注解都支持通配符，如`@PluginPackage("com.github.linyuzai.concept.**.plugin")`

# 插件自动加载

支持通过监听本地文件目录变化来自动加载插件

默认使用`WatchService`来监听文件目录，提供了`JarNotifier`来自动触发加载卸载

```java
@Slf4j
public class ConceptPluginSample {

    //自动加载器
    private final PluginAutoLoader loader = new WatchServicePluginAutoLoader.Builder()
            .locations(new PluginLocation.Builder()
                    //监听目录
                    .path("/Users/concept/plugin")
                    //所有jar
                    .filter(it -> it.endsWith(".jar"))
                    .build())
            //指定线程池
            .executor(Executors.newSingleThreadExecutor())
            //增删改时触发自动加载，自动重新加载，自动卸载
            .onNotify(new JarNotifier(concept))
            //异常回调
            .onError(e -> log.error("Plugin auto load error", e))
            .build();

    /**
     * 开始监听
     */
    @PostConstruct
    private void start() {
        loader.start();
    }

    /**
     * 结束监听
     */
    @PreDestroy
    private void stop() {
        loader.stop();
    }
}
```

# 插件加载流程

# 插件提取器

# 插件过滤器

# 插件解析器

### 动态解析

# 插件匹配器

# 插件转换器

# 插件格式器

# 插件事件

# 插件类加载器

# 插件需要依赖其他`jar`时的注意事项