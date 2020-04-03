# WebApp的开发及治理

---

WebApp的前端开发不在UBSI的讨论范围之内，可以根据实际情况选择react-js/vue-js、react native/kotlin/swift/flutter等不同的前端框架，在这里我们重点关注后端Web服务的开发。



在Java环境下，UBSI强烈建议采用SpringBoot2框架开发restful风格的Web服务。UBSI为SpringBoot应用提供了rewin.ubsi.rest组件，这个组件通过一组预置的restful-api，使得Web服务能够被UBSI Web管理器进行配置和管理。



如果要使用rewin.ubsi.rest组件，需要在SpringBoot项目的pom.xml中添加如下依赖：

```
<dependency>
	<groupId>rewin.ubsi</groupId>
	<artifactId>rewin.ubsi.rest</artifactId>
	<version>1.0.0</version>
</dependency>
```

> 注意：
>
> * rewin.ubsi.rest依赖的SprintBoot版本是2.2.6.RELEASE
> * rewin.ubsi.rest已经依赖了rewin.ubsi.core，不必重复添加



另外，还需要在SpringBoot项目的主程序入口处添加启动代码，示例如下：

```
@SpringBootApplication(
		scanBasePackages = {
				"rewin.ubsi.rest", 
				"{your package}"
		},
		exclude = {
				MongoAutoConfiguration.class, 
				MongoDataAutoConfiguration.class
		}	// 取消MongoDB的自动配置（rewin.ubsi.core已经包含了MongoDB的驱动，避免冲突）
)
public class Application {

	public static void main(String[] args) throws Exception {
		SpringApplication.run(Application.class, args);

		// 启动rewin.ubsi.rest，也可以在单独的ApplicationRunner中进行启动
		rewin.ubsi.rest.Starter.startup(".");	// 参数是项目的运行目录，用来查找配置文件
	}

}
```



rewin.ubsi.rest在启动时会自动加载UBSI Consumer组件，如果在配置文件rewin.ubsi.consumer.json中配置了redis，则会将自己作为一个WebApp的实例注册到redis注册中心，随后UBSI Web管理器就可以自动发现这个WebApp，并通过rest组件提供的api对其进行配置管理。



rewin.ubsi.rest在向redis注册时，需要声明自己的URL访问路径，这个路径应该配置在rewin.ubsi.rest.json文件中，示例如下：

```
{
  "url": "http://{web-server-host}:{web-server-port}",
  "gateway": true
}
```

> 注：gateway表示是否允许Web服务直接转发UBSI服务请求



如果Web服务未进行任何手工配置，也可以在启动后，通过UBSI Web管理器的"手工"发现机制，将运行实例的URL手工添加，然后再进行配置或监控。



rewin.ubsi.rest为Web服务提供了两组预置的服务接口，分别是：

* /controller

  用于Web管理器进行监控，比如 [GET] http://{web-server-host}:{web-server-port}/controller/info 可以获得rewin.ubsi.rest的运行信息

* /request

  用于转发UBSI服务请求，可以通过 [POST] http://{web-server-host}:{web-server-port}/request?help 得到帮助

