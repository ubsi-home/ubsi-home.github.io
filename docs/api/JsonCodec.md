# rewin.ubsi.common.JsonCodec 数据编码json

------



### 将UBSI格式的json字符串解码为Java数据对象

```
public static Object fromJson(String str) throws Exception;
```

参数：

- str - UBSI格式的json字符串，详见 [UBSI数据编码](../overview/protocol.md)

返回：

- Java数据对象



### 将Java数据对象编码为UBSI格式的JsonElement

```
public static JsonElement toJson(Object obj) throws Exception;
```

参数：

- obj - Java数据对象

返回：

- com.google.gson.JsonElement对象



