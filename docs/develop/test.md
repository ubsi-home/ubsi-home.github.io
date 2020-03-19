# 访问并测试服务接口

---

UBSI的Consumer组件提供了访问微服务的上下文环境以及API接口，这个组件也包含在了UBSI核心包中。



访问微服务接口的简单示例如下：

```
package my.service.samples;

import rewin.ubsi.cli.Request;
import rewin.ubsi.consumer.Context;

public class DemoClient {

	public static void main(String[] args) throws Exception {
		// 初始化UBSI Consumer的Context环境，参数用来指明"工作路径"
		Context.startup(".");
	
		try {
            // 构造服务请求，指定："服务名字"、"接口名字"、"参数列表"
            Context context = Context.request("ServiceName", "EntryName", "ParamsList");
            // 将请求发送到微服务所在的服务容器（地址,端口），并获得返回结果
            Object res = context.direct("localhost", 7112);
            // 以json方式在console输出结果数据
            Request.printJson(res);
		} catch(Exception e) {
			e.printStackTrace();
		}
		
		// 关闭UBSI Consumer
		Context.shutdown();
	}

}
```

在实际访问之前，必须先保证微服务已经加载到指定的服务容器中。如何启动一个服务容器并加载微服务请参见[微服务的发布及部署](deploy.md)。

但是在微服务开发过程中，如果需要不断启停服务来进行接口调试，会严重影响开发效率，我们可以利用JUnit单元测试工具来更高效率地进行服务接口的开发及测试。

以前面的"访问计数"CountService为例，用JUnit4构造一个完整的测试用例：

```
package my.service.samples;

import org.junit.Test; 
import org.junit.Before; 
import org.junit.After;
import rewin.ubsi.cli.Request;
import rewin.ubsi.common.Codec;
import rewin.ubsi.consumer.Context;
import rewin.ubsi.container.Bootstrap;

import java.util.Map;

public class CountServiceTest {

	@Before
	public void before() throws Exception {
    	// 在本机启动一个服务容器，默认服务端口为7112，同时会：
    	//   加载rewin.ubsi.module.json配置文件指定的微服务
    	//   初始化consumer的Context环境
    	Bootstrap.start();
	}

	@After
	public void after() throws Exception {
    	// 关闭容器
    	Bootstrap.stop();
	}

	static String SERVICE_NAME = "my.samples.count";	// 要访问的微服务名字
	static String SERVER_HOST = "localhost";            // 微服务所在容器的地址
	static int SERVER_PORT = 7112;                      // 微服务所在容器的端口

	// 定义在访问端使用的数据结构
	public static class EntryCounts {
		public long entry1;		// entry1接口的访问计数
		public long entry2;		// entry2接口的访问计数
	}
	// 获取微服务当前的运行状态（访问计数），在console输出并返回
	EntryCounts printCounts() throws Exception {
		// 构造一个"获取指定微服务当前运行状态"的监控请求：
		//   名字为""的微服务是特指"服务容器的控制器",这是一个容器内置的特殊微服务，提供监控接口；
		//   "getRuntime"接口表示获取当前运行状态，容器会调用指定微服务的@USInfo接口；
		//   参数SERVICE_NAME表示指定的微服务；
		Context ctx = Context.request("", "getRuntime", SERVICE_NAME);
		// 将监控请求发送到指定的服务容器，容器会调用微服务的@USInfo接口并返回结果
		Map res = (Map)ctx.direct(SERVER_HOST, SERVER_PORT);
		// 将返回结果转换为EntryCounts对象
		EntryCounts counts = Codec.toType(res, EntryCounts.class);
		System.out.println("~~~~~~~~");
		Request.printJson(counts);  // 在本地console输出json文本
		return counts;
	}

	// 测试用例
	@Test
	public void testEntry() throws Exception {
    	// 在本地输出当前的访问计数
    	printCounts();

		for ( int i = 0; i < 3; i ++ ) {
			// 构造对my.samples.count服务的entry1接口的访问请求
			Context context = Context.request(SERVICE_NAME, "entry1");
			// 将请求发送到指定的服务容器
			context.direct(SERVER_HOST, SERVER_PORT);
		}

		for ( int i = 0; i < 5; i ++ ) {
			// 构造对my.samples.count服务的entry2接口的访问请求
			Context context = Context.request(SERVICE_NAME, "entry2");
			// 将请求发送到指定的服务容器
			context.direct(SERVER_HOST, SERVER_PORT);
		}

		// 再次输出当前的访问计数
		EntryCounts counts = printCounts();
		
		// 构造对my.samples.count服务的print接口的访问请求
		Context context = Context.request(SERVICE_NAME, "print", counts);
		// 将请求发送到指定的服务容器：在服务端输出访问计数
		context.direct(SERVER_HOST, SERVER_PORT);
	}

} 
```



然后在项目目录（即运行时的"工作目录"）下手工创建配置文件rewin.ubsi.module.json，用来指定在服务容器启动时需要加载的微服务：

```
{
  "services": {
    "my.samples.count": {
      "class_name": "my.service.samples.CountService",
      "startup": true
    }
  }
}
```



用JUnit执行上面的用例，可以得到如下输出：

```
[INFO]	2019-09-03 13:47:32.826	my-pc#7112	rewin.ubsi.container	rewin.ubsi.container	rewin.ubsi.container.Bootstrap#start()#138	startup	"my-pc#7112"
~~~~~~~~
{
  "entry1": 0,
  "entry2": 0
}
~~~~~~~~
{
  "entry1": 3,
  "entry2": 5
}
========
entry1的访问次数：3
entry2的访问次数：5
[INFO]	2019-09-03 13:47:34.587	my-pc#7112	rewin.ubsi.container	rewin.ubsi.container	rewin.ubsi.container.Bootstrap#stop()#172	shutdown	"my-pc#7112"
```



> 注意：
>
> * 两条[INFO]的输出是服务容器"启动"和"关闭"时的提示信息，"my-pc#7112"表示容器所在的主机名字及端口
> * 这个测试用例使用了"服务容器控制器"提供的"getRuntime"监控接口，获得微服务的运行信息
> * 访问服务接口使用的是direct()"直连"方式，这种方式需要显式指定服务的位置，通常用于测试或监控等特定场景；在配置了"路由"或"注册中心"的运行环境下，应该使用call()"路由"方式，示例：
>
> ```
> // 构造服务请求
> Context context = Context.request("ServiceName", "EntryName", "ParamsList");
> // 通过路由算法将请求发送到合适的服务容器，并获得返回结果
> Object res = context.call();
> ```
>
> * 服务请求的Context实例对象，在发送完成后应该被丢弃，不可以重复发送

