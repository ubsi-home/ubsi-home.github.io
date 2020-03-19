# rewin.ubsi.common.XmlCodec 数据编码xml

------



### 将UBSI格式的xml字符串解码为Java数据对象

```
public static Object decode(String str) throws Exception;
```

参数：

- str - UBSI格式的xml字符串，详见 [UBSI数据编码](../overview/protocol.md)

返回：

- Java数据对象



### 将Java数据对象编码为UBSI格式的xml字符串

```
public static String encode(Object obj, boolean strCData, boolean filterHeader) throws Exception;
```

参数：

- obj - Java数据对象
- strCData - 是否将String内容放在&lt;![CDATA[...]]&gt;中
- filterHeader - 是否滤掉&lt;?xml version="1.0" encoding="UTF-8"?&gt;

返回：

- UBSI格式的xml字符串



