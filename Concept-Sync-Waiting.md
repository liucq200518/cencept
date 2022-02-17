# 概述

主要将异步回调的功能场景封装成同步返回

# 示例说明

以我们现在物联网开发为例，或多或少都会接触到物理设备的对接，甚至需要对设备进行控制等操作

但是有很大一部分设备不会同步返回结果，而是另外上报一条数据

所以在下发了一条命令之后无法直接获得对应的结果

而该库能十分方便的将异步回调转换成同步返回

```java
@RestController
@RequestMapping("/concept-sync-waiting")
public class SyncWaitingController {

    /**
     * 新建一个 {@link SyncWaitingConcept} 对象
     * 也可以直接在 Spring 容器中注入一个全局使用
     */
    private final SyncWaitingConcept concept = new ConditionSyncWaitingConcept();

    /**
     * 下发命令，阻塞线程直到数据上报或超时
     *
     * @param key 每条命令唯一的id
     * @return 设备上报的数据
     */
    @RequestMapping("/send")
    public String send(@RequestParam String key) {
        try {
            return concept.waitSync(key, new SyncCaller() {
                @Override
                public void call(Object k) {
                    //在这里下发命令
                }
            }, 5000);
        } catch (SyncWaitingTimeoutException e) {
            return "下发命令超时";
        }
    }

    /**
     * 接收设备上报的数据，唤醒下发命令的线程
     *
     * @param key   一般需要从上报数据中附带命令id
     * @param value 上报数据
     */
    @RequestMapping("/receive")
    public void receive(@RequestParam String key, @RequestParam String value) {
        concept.notifyAsync(key, value);
    }
}
```

# 集成

当前版本`1.0.0`

```gradle
implementation 'com.github.linyuzai:concept-sync-waiting:version'
```

```xml
<dependency>
  <groupId>com.github.linyuzai</groupId>
  <artifactId>concept-sync-waiting</artifactId>
  <version>version</version>
</dependency>
```

# 使用

