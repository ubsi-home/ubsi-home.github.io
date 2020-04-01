# rewin.ubsi.container.ServiceContext 微服务环境

------



### 构造函数

```
public ServiceContext(String name);
```

参数：

- name - 微服务名字

返回：

- 无



### 获得服务的配置文件的存放目录

```
public String getLocalPath();
```

参数：

- 无

返回：

- 微服务配置文件所在的本地路径



### 获得访问者的网络地址

```
public InetSocketAddress getConsumerAddress();
```

参数：

- 无

返回：

- 访问者的网络地址



### 获得请求ID

```
public String getRequestID();
```

参数：

- 无

返回：

- 请求ID



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



### 修改请求Header的数据项

```
public void setHeader(String key, Object value);
```

参数：

- key - 数据项的名字
- value - 数据项的值

返回：

- 无



### 获得服务名字

```
public String getServiceName();
```

参数：

- 无

返回：

- 微服务实际部署的服务名字

注意：

* 微服务在部署时可以指定服务名字，未必使用@UService中声明的名字，这种机制使得同一个微服务的class可以部署为不同的名字。



### 获得服务的状态

```
public int getServiceStatus();
```

参数：

- 无

返回：

- 服务的运行状态，0:未启动，1:运行中，-1:暂停



### 获得请求的接口名字

```
public String getEntryName();
```

参数：

- 无

返回：

- 请求的接口名字



### 获得请求接口的注解

```
public USEntry getEntryAnnotation();
```

参数：

- 无

返回：

- 请求接口的注解



### 获得服务容器ID

```
public String getContainerId();
```

参数：

- 无

返回：

- 服务容器ID，格式："host#port"



### 获得服务容器的版本

```
public int getContainerVersion();
```

参数：

- 无

返回：

- 服务容器的版本，格式："1.2.3" => 1002003，版本分3段，每段占3个整数位（取值：0~999）



### 获得服务容器的发行状态

```
public boolean getContainerRelease();
```

参数：

- 无

返回：

- 服务容器的发行状态：是否Release



### 检查Container是否开始服务

```
public boolean isContainerReady();
```

参数：

- 无

返回：

- 服务容器是否已经进入正常服务状态



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



### 获得请求的标志

```
public byte getRequestFlag();
```

参数：

- 无

返回：

- 按位表示的请求标志，0x01：是否丢弃处理结果，0x02：是否通过广播消息返回结果，0x80：是否产生请求的跟踪日志



### 暂停服务

```
public void pause();
```

参数：

- 无

返回：

- 无



### 重启服务

```
public void restart();
```

参数：

- 无

返回：

- 无



### 获得处理数量

```
public long[] getStatistics(String service, String entry);
```

参数：

- service - 服务名字
- entry - 接口名字

返回：

- 处理数量，格式：[ 处理的请求总量，处理失败的总量 ]

注意：

* 数量中不包含"待处理"或"处理中"的请求



### 获得各个接口的处理数量

```
public Map<String, long[]> getStatistics(String service);
```

参数：

- service - 服务名字

返回：

- 各接口的处理数量，格式：{ "接口名字" : [ 处理的请求总量，处理失败的总量 ] }



### 获得所有服务的处理数量

```
public Map<String, Map<String, long[]>> getStatistics();
```

参数：

- 无

返回：

- 所有服务的处理数量，格式：{ "服务名字" : { "接口名字" : [ 处理的请求总量，处理失败的总量 ] } }



### 设置成功的处理结果

```
public void setResultData(Object data);
```

参数：

- data - 处理结果

返回：

- 无

注意：

* 此接口通常会在容器过滤器@USFilter中使用，详见 [@USFilter的说明](../develop/annotation.md)



### 设置错误结果

```
public void setResultError(String msg);
```

参数：

- msg - 错误信息

返回：

- 无



### 是否有处理结果

```
public boolean hasResult();
```

参数：

- 无

返回：

- 是否已经处理完成 或 设置了处理结果



### 获得结果代码

```
public int getResultCode();
```

参数：

- 无

返回：

- 结果代码，ErrorCode.OK（0）表示成功



### 获得结果数据

```
public Object getResultData();
```

参数：

- 无

返回：

- 结果数据，对于void类型的服务接口，返回的结果数据为null



### 请求是否被转发处理

```
public boolean isForwarded();
```

参数：

- 无

返回：

- 是否被转发



### 创建请求对象

```
public Context request(String service, Object... entryAndParams) throws Exception;
```

参数：

- service - 服务名字
- entryAndParams - 接口名字及请求参数

返回：

- rewin.ubsi.consumer.Context对象

注意：

* 在微服务中应该使用ServiceContext的request()而不是Context.request()来创建请求对象，这样可以保证：
  * 正确的服务依赖
  * 持续的请求链路跟踪



### 获得日志对象

```
public Logger getLogger();
```

参数：

- 无

返回：

- rewin.ubsi.consumer.Logger对象

