# 基础数据及编码方式

---

UBSI要求微服务接口的请求参数以及返回结果必须采用“标准”的基础数据类型，不能采用“语言相关”的数据结构，这样可以保证在向不同微服务发起请求时，不必依赖服务端定义的数据类型，同时这种机制也为API网关的构造以及以后多语言扩展提供了方便。



**UBSI支持的基础数据类型包括：**

| 类型   | 说明     | 编码   | Java类型             | XML方式                       | JSON方式                            |
| ------ | -------- | ------ | -------------------- | ----------------------------- | ----------------------------------- |
| null   | 空       | 0x00   | null                 | &lt;null/&gt;                     | null                                |
| bool   | 布尔     | ...... | boolean &#124; Boolean | &lt;bool&gt;true &#124; false&lt;/bool&gt; | true &#124; false                       |
| byte   | 单字节   | ...... | byte   |              &lt;byte&gt;ff&lt;/byte&gt;           | {"$t":"byte", "$v":-1}              |
| int    | 整数     | ...... | char &#124; short &#124; int |    &lt;int&gt;12&lt;/int&gt;           | 12                                  |
| long   | 长整数   | ...... | long  |               &lt;long&gt;123&lt;/long&gt;          | {"$t":"long", "$v":123}             |
| bigint | 大整数   | ...... | BigInteger  |         &lt;bigint&gt;12345&lt;/bigint&gt;    | {"$t":"bigint", "$v":12345}         |
| double | 浮点数   | ...... | float &#124; double |      &lt;double&gt;123.45&lt;/double&gt;   | 123.45 或 {"$t":"double", "$v":123} |
| bignum | 大浮点数 | ...... | BigNumber |           &lt;bignum&gt;123.45&lt;/bignum&gt;   | {"$t":"bignum", "$v":"123.45"}      |
| string | 字符串   | ...... | String  |             &lt;str&gt;hello, wor&lt;/str&gt;   | "hello, world"                      |
| map    | 键值     | ...... | Map     |             &lt;map&gt;&lt;...&gt;&lt;...&gt;&lt;/map&gt; | {...}                               |
| list   | 列表     | ...... | List    |             &lt;list&gt;&lt;...&gt;&lt;/list&gt;       | [...]                               |
| set    | 集合     | ...... | Set     |             &lt;set&gt;&lt;...&gt;&lt;/set&gt;         | {"$t":"set", "$v":[...]}            |
| bytes  | 多字节   | ...... | byte[]  |             &lt;bytes&gt;a0 b1&lt;/bytes&gt;      | {"$t":"bytes", "$v":"a0b1"}         |
| array  | 数组     | ...... | T[]    |              &lt;array&gt;&lt;...&gt;&lt;/array&gt;    | {"$t":"array", "$v":[...]}          |

&nbsp;

在请求/应答过程中，UBSI框架负责将数据按照对应的数据类型编码成字节数据进行传输，同样，对接收到的字节数据也会进行解码，形成"标准"类型的数据再传递给微服务或应用进行处理。

> XML和JSON表达方式可以在[命令行工具](../develop/cli.md)或通过[API网关](../gateway/readme.md)发起服务请求时使用。

&nbsp;

通常情况下，UBSI对数据的编/解码过程是透明的，不需要开发者进行处理。但下面几种情况需要Java开发者注意：

* char和short都会转换成int，float会转换为double
* int[]或String[]等非byte[]型的数组，都会转换为Object[]
* 非"标准"数据类型的Object，会将其非static的public成员变量提取出来，转换成Map


