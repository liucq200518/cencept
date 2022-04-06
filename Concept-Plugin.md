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

# 集成

