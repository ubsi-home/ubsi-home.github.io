# Java API

---

不管是开发UBSI的微服务还是WebApp，都可以利用UBSI核心包提供的API，常用的API包括：



* rewin.ubsi.consumer 包

  * [Context](Context.md) - 请求微服务

  * [ErrorCode](ErrorCode.md) - 错误代码

  * [Logger](Logger.md) - 日志输出



* rewin.ubsi.container 包

  * [ServiceContext](ServiceContext.md) - 访问微服务实例的运行环境



* rewin.ubsi.common 包

  * [Codec](Codec.md) - 处理数据编码解码

  * [JsonCodec](JsonCodec.md) - 处理数据的json格式编码

  * [XmlCodec](XmlCodec.md) - 处理数据的xml格式编码

  * [Crypto](Crypto.md) - 国密加解密算法

  * [JedisUtil](JedisUtil.md) - 访问redis的工具



另外，UBSI核心包还依赖下面的第三方jar包，这些包提供的API也可以在开发中直接使用：

```
        <dependency>
            <groupId>org.dom4j</groupId>
            <artifactId>dom4j</artifactId>
            <version>2.1.1</version>
        </dependency>
        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>2.8.6</version>
        </dependency>
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-all</artifactId>
            <version>4.1.43.Final</version>
        </dependency>
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>3.1.0</version>
        </dependency>
        <dependency>
            <groupId>org.mongodb</groupId>
            <artifactId>mongo-java-driver</artifactId>
            <version>3.11.2</version>
        </dependency>
        <dependency>
            <groupId>org.bouncycastle</groupId>
            <artifactId>bcprov-jdk15on</artifactId>
            <version>1.62</version>
        </dependency>
```

