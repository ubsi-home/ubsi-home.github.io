# rewin.ubsi.common.JedisUtil 访问redis

------



### 检测redis是否可用

```
public static boolean isInited();
```

参数：

- 无

返回：

- 当前是否已经正常连接到了redis server



### 获取一个Jedis实例

```
public static Jedis getJedis();
```

参数：

- 无

返回：

- redis.clients.jedis.Jedis对象，后续可以使用这个对象访问redis；如果失败会抛出异常

注意：

* Jedis对象使用结束后，需要主动调用`close()`以释放资源，或者使用 try-with-resource 机制
* 如果操作中会切换redis的Database，需要在完成操作后切换回 JedisUtil.DATABASE

示例：

  ```
try (Jedis jedis = JedisUtil.getJedis()) {
	//do your work
	jedis.select(JedisUtil.DATABASE);	// 如果之前执行过select操作
}
  ```



### 发送一个广播消息

```
public static void publish(String channel, Object data);
```

参数：

- channel - 广播频道
- data - 消息数据

返回：

- 无

注意：

* 所有"订阅"了该频道的"监听器"都会收到这个广播



### 发送一个事件

```
public static void putEvent(String channel, Object data);
```

参数：

- channel - 事件频道
- data - 事件数据

返回：

- 无

注意：

- 在分布式环境下，即便存在多个订阅了该频道的监听器，但只有一个能收到这个事件（抢占式处理）



### 频道监听器

```
    public static abstract class Listener {
    
        /** 广播监听的回调入口 */
        public abstract void onMessage(String channel, Object message) throws Exception;
        
        /** 事件监听的回调入口 */
        public abstract void onEvent(String channel, Object event) throws Exception;

        /** 监听广播频道 */
        public void subscribe(String... channels);
        
        /** 监听广播频道（"*"表示通配符） */
        public void subscribePattern(String... patterns);
        
        /** 监听事件频道（在一个Java进程中，同一个事件频道只能存在一个监听器） */
        public void subscribeEvent(String... channels) throws Exception;
        
        /** 取消广播监听 */
        public void unsubscribe(String... channels);
        
        /** 取消广播监听 */
        public void unsubscribePattern(String... patterns);
        
        /** 取消事件监听 */
        public void unsubscribeEvent(String... channels);
        
    }
```



注意：

* 如果使用JedisUtil.publish()发布广播或者JedisUtil.putEvent()发送"事件"，必须使用JedisUtil.Listener来监听
* 发送的广播或者事件数据可以是任意数据类型，但是在监听器接收时，会转换为[UBSI基础数据类型](../overview/protocol.md)
* 在回调接口onMessage或onEvent中，如果需要进行高耗时的操作，应启动另外的任务线程进行处理，以免阻塞异步I/O



示例：

```
	if ( JedisUtil.isInited() ) {
		// 创建频道监听器
		JedisUtil.Listener listener = new JedisUtil.Listener() {
        	@Override
            public void onMessage(String channel, Object message) {
                System.out.print("\n~~~ receive message from [" + channel + "]: ");
                Request.printJson(message);
            }
            
            @Override
            public void onEvent(String channel, Object event) {
                System.out.print("\n~~~ receive event from [" + channel + "]: ");
                Request.printJson(event);
            }
		}
		
        listener.subscribe("msg1", "msg2");         // 监听广播频道
        listener.subscribeEvent("evt1", "evt2");    // 监听事件频道
	}
```

