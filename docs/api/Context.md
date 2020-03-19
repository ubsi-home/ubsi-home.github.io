# rewin.ubsi.consumer.Context 请求微服务

------



### 初始化UBSI Consumer运行环境（静态方法）

```
public static void startup(String workPath) throws Exception;
```

参数：

- workPath - 指定工作目录（用来查找配置文件）

返回：

- 无



### 关闭UBSI Consumer运行环境（静态方法）

```
public static void shutdown();
```

参数：

- 无

返回：

- 无



### 设置应用的属性（静态方法）

```
public static void setLogApp(String appAddr, String appTag);
```

参数：

- appAddr - 应用的位置
- appTag - 应用的分类标签

返回：

- 无

注意：

* UBSI Consumer本身的日志输出信息中会使用这些属性



### 获得应用使用的日志记录器（静态方法）

```
public static Logger getLogger(String appTag, String appID);
```

参数：

- appTag - 应用的分类标签

- appID - 应用ID

返回：

- rewin.ubsi.consumer.Logger对象

注意：

- 如果是在开发微服务，请务必使用ServiceContext的getLogger()来获得Logger对象



### 得到微服务请求的统计数据（静态方法）

```
public static Register.Statistics getStatistics(String service, String entry);
```

参数：

- service - 服务名字
- entry - 接口名字

返回：

- rewin.ubsi.consumer.Register.Statistics对象，结构定义如下：

  ```
      /** 请求统计 */
      public static class Statistics {
          public long     request;            // 计数器：总请求次数
          public long     result;             // 计数器：总返回次数
          public long     success;            // 计数器：总成功次数
          public long     max_time;           // 计时器：最长的处理时间（毫秒）
          public String   req_id;             // 最长处理时间的请求ID
      }
  ```



### 得到微服务请求的统计数据（静态方法）

```
public static Map<String, Register.Statistics> getStatistics(String service);
```

参数：

- service - 服务名字

返回：

- 指定服务的各个接口的分立统计，格式：{ "接口名字" : &lt;Statistics&gt; }



### 得到全部微服务请求的统计数据（静态方法）

```
public static Map<String, Map<String, Register.Statistics>> getStatistics();
```

参数：

- 无

返回：

- 所有服务的各个接口的分立统计，格式：{ "服务名字" : { "接口名字" : &lt;Statistics&gt; } }



### 创建微服务请求对象（静态方法）

```
public static Context request(String service, Object... entryAndParams) throws Exception;
```

参数：

- service - 服务名字
- entryAndParams - 接口名字及参数列表

返回：

- rewin.ubsi.consumer.Context请求对象

注意：

- 如果是在开发微服务，请务必使用ServiceContext的request()来创建请求对象



### 获得请求ID

```
public String getReqID();
```

参数：

- 无

返回：

- 请求ID



### 获得请求的服务名字

```
public String getService();
```

参数：

- 无

返回：

- 服务名字



### 获得请求的接口名字

```
public String getEntry();
```

参数：

- 无

返回：

- 接口名字



### 获得请求参数的数量

```
public int getParamCount();
```

参数：

- 无

返回：

- 请求参数的数量



### 获得请求参数的值

```
public Object getParam(int index);
```

参数：

- index - 参数的序号，从0开始

返回：

- 请求参数的值



### 重新设置请求参数

```
public void setParam(Object... o);
```

参数：

- o - 请求的参数列表

返回：

- 无



### 获取请求的Header数据项

```
public Object getHeader(String key);
```

参数：

- key - 数据项的名字

返回：

- 数据项的值



### 获取请求的Header

```
public Map<String,Object> getHeader();
```

参数：

- 无

返回：

- 请求的Header



### 设置请求Header的数据项

```
public Context setHeader(String key, Object value);
```

参数：

- key - 数据项的名字
- value - 数据项的值

返回：

- 本对象



### 设置请求Header

```
public Context setHeader(Map<String,Object> header);
```

参数：

- header - 请求的Header

返回：

- 本对象



### 设置目标微服务的版本

```
public Context setVersion(int min, int max, int release);
```

参数：

- min - 最小版本号，-1表示不改变此项（缺省为0）
- max - 最大版本号，-1表示不改变此项（缺省为0，表示不限）
- release - 发行状态，1:必须为Release版；0:必须为非Release版；-1:不限（缺省为-1）

返回：

- 本对象



### 设置请求的超时时间

```
public Context setTimeout(int timeout);
```

参数：

- timeout - 超时时间，秒数，0表示不限

返回：

- 本对象



### 设置是否使用独立连接发送请求

```
public Context setConnectAlone(boolean alone);
```

参数：

- alone - 是否使用独立连接发送请求（缺省为false）

返回：

- 本对象

注意：

* UBSI底层通讯框架采用了多路复用机制，在"路由"模式下，多个请求会复用同一个socket连接。当某个请求需要传递大量数据的时候，会占用通讯链路，影响其他请求的处理。这种情况下，可以`setConnectAlone(true)`，单独创建socket连接来处理这个请求。



### 获取请求的处理时间

```
public long getResultTime();
```

参数：

- 无

返回：

- 请求的处理时间，毫秒数，0表示还未发送请求，<0表示请求还未返回（绝对值表示当前耗时）



### 获取结果代码

```
public int getResultCode();
```

参数：

- 无

返回：

- 结果代码，ErrorCode.OK（0）表示成功返回



### 获取结果数据

```
public Object getResultData();
```

参数：

- 无

返回：

- 结果数据，对于void类型的服务接口，返回的结果数据为null



### 设置处理结果

```
public void setResult(int code, Object data);
```

参数：

- code - 结果代码，ErrorCode.OK表示处理成功
- data - 结果数据

返回：

- 无

注意：

* 此接口通常会在请求过滤器中使用，过滤器的定义如下：

  ```
      /** 请求过滤器 */
      public static interface Filter {
          /** 前置接口，返回：0-正常，-1-拒绝，1-降级（使用Mock数据） */
          public int before(Context ctx);
          /** 后置接口 */
          public void after(Context ctx);
      }
  ```



### 向指定的服务容器直接发送请求（同步方式）

```
public Object direct(String host, int port) throws Exception;
```

参数：

- host - 容器的主机地址
- port - 容器的监听端口

返回：

- 处理结果，对于void类型的服务接口，返回的结果数据为null

注意：

* 直接指定容器发送请求通常用在下面的场景：
  * 对该容器进行配置或监控
  * 测试该容器中部署的微服务实例
* 每次直接发送请求会单独新建socket连接，不会使用"多路复用"机制



### 向指定的服务容器直接发送请求（异步方式）

```
public void directAsync(String host, int port, ResultNotify notify, boolean message) throws Exception;
```

参数：

- host - 容器的主机地址
- port - 容器的监听端口
- notify - 处理结果的监听器，null表示不需要容器回传结果
- message - 是否通过redis广播消息回传结果

返回：

- 无

注意：

- 异步方式在请求发送后立即返回，不会阻塞等待，结果数据通过监听器获取，监听器的定义如下：

  ```
      /** 异步方式得到请求结果的回调接口 */
      public static interface ResultNotify {
          /**
           * 回调入口，code:结果代码(ErrorCode.OK表示成功），result:结果（失败时为错误信息）
           * 注：如果需要进行高耗时的操作，应启动另外的任务线程进行处理，以免阻塞异步I/O
           */
          public void callback(int code, Object result);
      }
  ```

  

### "路由"方式发送服务请求（同步方式）

```
public Object call() throws Exception;
```

参数：

- 无

返回：

- 处理结果，对于void类型的服务接口，返回的结果数据为null

注意：

- "多路复用"、"动态路由" 机制的请求方式，正常情况下应该使用这种方式请求微服务



### "路由"方式发送服务请求（异步方式）

```
public void callAsync(ResultNotify notify, boolean message) throws Exception;
```

参数：

- notify - 处理结果的监听器，null表示不需要容器回传结果
- message - 是否通过redis广播消息回传结果

返回：

- 无

注意：

* 当请求需要服务端进行长时间处理时（例如批处理任务），可以考虑采用message为true的请求方式

