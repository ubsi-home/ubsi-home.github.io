# rewin.ubsi.common.Codec 数据编码

---



### 将Java数据转换为UBSI基础数据对象

```
public static Object toObject(Object value);
```

参数：

- value - Java数据对象

返回：

- [UBSI基础数据对象](../overview/protocol.md)



### 将数据对象转换为指定的数据类型

```
public static <T> T toType(Object obj, Type type, Type... typeArguments);
```

参数：

- obj - Java数据对象
- type - 目标数据类型
- typeArguments - 如果目标数据类型是"泛型"，指明泛型需要的数据类型

返回：

- 指定数据类型的对象

示例：

```
List<Integer> value = Codec.toType(new Object[] { 1, 2, 3 }, ArrayList.class, Integer.class);
```



### 将数据对象编码为base64编码的字符串

```
public static String encode(Object data);
```

参数：

- data - Java数据对象

返回：

- base64编码的字符串



### 将base64编码的字符串解码为数据对象

```
public static Object decode(String data) throws Exception;
```

参数：

- data - base64编码的字符串

返回：

- Java数据对象



### 将数据对象编码为字节数据

```
public static byte[] encodeBytes(Object data);
```

参数：

- data - Java数据对象

返回：

- 字节数据



### 将字节数据解码为数据对象

```
public static Object decodeBytes(byte[] data) throws Exception;
```

参数：

- data - 字节数据

返回：

- Java数据对象

