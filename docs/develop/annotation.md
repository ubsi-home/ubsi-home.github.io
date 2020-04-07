# 服务注解

---

UBSI通过一系列预定义的注解来声明微服务及其接口，这些注解包括：

### @UService

用来标注Java Class，将其声明为一个微服务，例如：

```
package my.service.samples;

import rewin.ubsi.annotation.*;
import rewin.ubsi.container.ServiceContext;

@UService(
	name = "my.samples.demo",           // 微服务的名字，缺省为""
	tips = "测试服务",                   // 微服务的说明，缺省为""
	version = "1.0.0",                  // 接口的版本号，缺省为"0.0.1"
	release = false,                    // 版本发行状态：true 或 false，缺省为false
	depend = {                          // 依赖的其他微服务
		@USDepend(                      // 第一个依赖，可以有多个
			name = "my.samples.xxx",    // 依赖的微服务的名字
			version = "",               // 该微服务的最小版本
			release = false             // 该微服务是否必须是"release"状态
		)
	}
)
public class DemoService {
	// 这是一个UBSI微服务，请注意：
	//   class的声明必须是public，并且有无参数的构造函数
}
```



### @USFilter

用来声明一个UBSI微服务的过滤器，除了没有"name"属性，其他定义都跟@UService相同，示例如下：

```
/** 这是一个UBSI Filter */
@USFilter(
	tips = "这是一个filter"
)
public class DemoFilter {
	@USBefore
	public void before(ServiceContext ctx) throws Exception {
		ctx.getLogger().info("服务容器开始处理一个服务请求");
	}
	
	@USAfter
	public void after(ServiceContext ctx) throws Exception {
		ctx.getLogger().info("服务容器已经完成一个服务请求的处理");
	}
}
```

@USFilter与@UService都是由服务容器(Container)加载运行的Class，但与@UService不同，@USFilter不提供对外的访问接口，而是可以通过@USBefore/@USAfter定义的入口拦截本容器所有的服务请求，从而可以记录或改变处理行为。

### @USDepend

用在@UService/@USFilter注解的depend属性中，声明依赖的其他微服务，示例请见@UService注解。

### @USBefore | @USAfter 

可以用在@UService或@USFilter声明的Class中，定义在一个服务请求"开始"或"结束"时的拦截动作，例如：

```
@UService(
	name = "my.samples.demo"
)
public class DemoService {
	@USBefore(
		timeout = 1		// 超时时间（秒数），缺省为1
	)
	public void before(ServiceContext ctx) throws Exception {
		ctx.getLogger().info("开始处理一个请求");
	}
	
	@USAfter(
		timeout = 1		// 超时时间（秒数），缺省为1
	)
	public void after(ServiceContext ctx) throws Exception {
		ctx.getLogger().info("完成了一个请求的处理");
	}
}
```

注意：

* 被标注的方法只能有一个ServiceContext参数，可以通过该参数获得请求的内容或者容器的上下文环境，更多详情可以参见[ServiceContext的API](../api/ServiceContext.md)
* @UService的before/after只能拦截对本服务的请求，@USFilter可以拦截本容器内所有微服务的请求

### @USEntry

用来声明微服务的接口，例如：

```
@UService(
	name = "my.samples.demo"
)
public class DemoService {
	@USEntry(
		tips = "回显",			// 接口的说明
		params = {
			@USParam(			 // 第一个参数，可以有多个
				name = "args",	 // 参数的名字
				tips = "参数"		// 参数的说明
			)
		},
		result = "返回传入的参数",	// 结果的说明
		readonly = true,		 // 是否是"只读"接口，缺省为true
		timeout = 1				 // 超时时间（秒数），缺省为1
	)
	public Object echo(ServiceContext ctx, Object args) {
		return args;
	}
	
	@USEntry(
		tips = "输出一条\"hello world!\"日志"
	)
	public void hello(ServiceContext ctx) {
		ctx.getLogger().info("hello, world!");
	}
}
```

注意：

* 被标注方法的第一个参数必须是ServiceContext（通过该参数可以获得请求的上下文），后续可以附加任意数量的参数
* 附加参数的说明应该放在@USEntry注解的params属性中，按照顺序一一对应，以帮助生成正确的接口文档
* 附加参数以及返回结果的数据类型必须是UBSI支持的基础数据类型，详见["UBSI数据类型"](../overview/protocol.md)
* readonly属性用来声明该接口是否会改变微服务的运行状态或数据，UBSI服务容器可以根据这个属性来设置访问权限
* timeout属性用来声明该接口"正常"的处理时间，UBSI监控工具可以根据这个属性来发现处理超时的服务请求
* 被标注的方法必须是public的，并且不能重名
* 在运行时，每次服务请求都会使用一个新的@UService实例，所以接口方法不需要考虑并发重入造成的冲突（除非是对静态数据的访问）

### @USParam

用在@USEntry注解的params属性中，声明接口的参数，示例请见@USEntry注解。

### @USInit | @USClose

微服务/过滤器的初始化动作，示例：

```
@UService(
	name = "my.samples.demo"
)
public class DemoService {
	@USInit
	public static void init(ServiceContext ctx) throws Exception {
		ctx.getLogger().info("微服务启动，进行初始化");
	}

	@USClose
	public static void close(ServiceContext ctx) throws Exception {
		ctx.getLogger().info("微服务关闭，进行清理");
	}
}
```

注意：

* 被标注的必须是public static方法，且只有一个ServiceContext参数
* 当开始加载微服务/过滤器，或者是监控工具"停止 | 启动"服务时，容器会调用这两个方法
* 如果没有必要，可以不使用这两个注解

### @USConfigGet | @USConfigSet

UBSI微服务可以通过这两个注解实现运行时的动态参数配置，示例：

```
	/** 返回配置参数 */
	@USConfigGet
	public static Object getConfig(ServiceContext ctx) throws Exception {
		return "这是配置参数";
	}

	/** 设置配置参数 */
	@USConfigSet
	public static void setConfig(ServiceContext ctx, String json) throws Exception {
		//todo 处理传入的配置参数
	}
```
注意：

* 被标注的必须是public static方法

* @USConfigGet可以返回任意数据结构的配置参数，UBSI配置管理工具会将其转换为json格式的字符串展示给管理员，并将修改后的配置（json格式字符串）传递给@USConfigSet进行处理。UBSI Web管理器可以通过这两个接口对微服务进行动态配置，为了适应不同微服务的不同配置方式，Web管理器对@USConfigGet返回的数据格式有个默认约定，示例如下：

  ```
  {
    "param1": "配置参数的当前值",
    "param1_restart": "配置参数的配置值（可能需要微服务重启后才能生效）",
    "param1_comment": "配置参数的说明",
    
    "param2": { ... },
    "param2_restart": { ... },
    "param2_comment": { "参数属性的名字": "参数属性的说明", ... }
  }
  ```

  其中"\_restert"和"\_comment"是默认的后缀，分别表示该项参数的"配置值"和"说明"；如果某项参数是一个"键值对"，则其对应的"\_comment"为该参数各个键值属性的说明

* 如果需要将配置参数保存为本地的配置文件，@USConfigSet可以通过ServiceContext提供的API获得本地配置文件的存放路径等环境信息；通常情况下，@USConfigSet设置的都是服务实例的运行参数，不同的服务实例可以有不同的配置

* 这两个接口不能被外部直接访问，而是通过UBSI容器封装的监控接口来调用

* 如果没有必要，可以不使用这两个注解

### @USInfo

用来向UBSI监控工具报告运行信息的接口，示例：

```
	/** 返回运行信息 */
	@USInfo
	public static Object info(ServiceContext ctx) throws Exception {
		return "当前的运行数据，可以是自定义的数据结构";
	}
```

注意：

- 被标注的必须是public static方法
- @USInfo接口不能被外部直接访问，而是通过UBSI容器封装的监控接口来调用
- 如果没有必要，可以不使用这个注解

### @USNotes | @USNote

如果微服务的输入参数或返回结果需要有特定的结构时，可以通过这两个注解对自定义数据结构进行描述，以方便开发人员对数据进行处理。示例：
```
	@USNotes("访问计数")				 // 标注一个数据模型
	public static class Counts {
		@USNote("entry1的访问数量")		// 标注模型中的属性
		public long entry1;
		@USNote("entry2的访问数量")
		public long entry2;
	}
	
	/* 通过接口返回标注的数据模型，供开发者查看 */
	@USEntry(
		tips = "获得数据模型的说明",
		result = "数据模型的说明，格式：{ \"模型1\": { \"字段1\": \"描述\", ... }, ... }"
	)
	public Map<String,Map<String,String>> getModels(ServiceContext ctx) {
		return Util.getUSNotes(Counts.class);	// 提取@USNotes注解的工具
	}
```

注意：

- 被标注的必须是public class或成员变量
- 如果需要对开发者可见，必须定义一个服务接口来返回标注的模型

