# rewin.ubsi.common.ScriptUtil 执行JS脚本

---

### 构造函数及成员变量

```
public class ScriptUtil {

    /** 输出信息 */
    public static class Message {
        public long     time;   // 时间戳，毫秒数
        public int      type;   // 消息类别，LogUtil.DEBUG | INFO | ERROR
        public String   text;   // 消息内容
    }

    /** API说明，格式：{ "方法": "说明" } */
    public static Map Api;
    
    /** 脚本的执行结果 */
    public Object  Result;

	/** 脚本执行过程中的输出消息记录 */
    public List<Message> Messages;

    /** 构造函数 */
    public ScriptUtil() {}
    /** 构造函数，指定服务容器的地址 */
    public ScriptUtil(String host, int port);
    /** 构造函数，指定日志记录的属性 */
    public ScriptUtil(String appTag, String appID, String tips);

}
```



### 获得JavaScript脚本执行引擎

```
public static ScriptEngine getEngine(ScriptUtil context, Map<String, Object> var);
```

参数：

- context - ScriptUtil对象实例，为JS脚本提供'$'对象
- var - 供JS脚本使用的其他环境变量

返回：

- javax.script.ScriptEngine对象



### 执行JavaScript脚本

```
public static Object runJs(ScriptEngine engine, String js) throws Exception;
```

参数：

- engine - JavaScript执行引擎
- js - JavaScript脚本代码

返回：

- JavaScript脚本代码的执行结果



### 执行JavaScript脚本

```
public static Object runJs(String js, ScriptUtil context, Map<String, Object> var) throws Exception;
```

参数：

- js - JavaScript脚本代码
- context - ScriptUtil对象实例，为JS脚本提供'$'对象
- var - 供JS脚本使用的其他环境变量

返回：

- JavaScript脚本代码的执行结果



