# DEMO服务

---

下面是一个完整的"访问计数"的服务示例。

### pom.xml：

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>my.service</groupId>
    <artifactId>my.service.samples.count</artifactId>
    <version>1.0.0-SNAPSHOT</version>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                    <encoding>utf-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <configuration>
                    <skipTests>true</skipTests>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <dependencies>
        <dependency>
            <groupId>rewin.ubsi</groupId>
            <artifactId>rewin.ubsi.core</artifactId>
            <version>1.0.0</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <distributionManagement>
        <repository>
            <id>nexus-release</id>
            <name>release</name>
            <url>http://192.168.1.116:8081/repository/maven-release</url>
        </repository>
        <snapshotRepository>
            <id>nexus-snapshot</id>
            <name>snapshot</name>
            <url>http://192.168.1.116:8081/repository/maven-snapshots</url>
        </snapshotRepository>
    </distributionManagement>

</project>

```



> 注意：&lt;distributionManagement&gt;用来描述项目build后jar包"发布"的URL地址（注意：这两个URL也应该配置到maven的统一资源路径 - 比如 http://192.168.1.116:8081/repository/maven-public/ 中），这样后续就可以使用UBSI部署工具来自动部署微服务了。



### CountService.java：

```
package my.service.samples;

import rewin.ubsi.annotation.*;
import rewin.ubsi.common.Codec;
import rewin.ubsi.common.Util;
import rewin.ubsi.container.ServiceContext;

import java.util.Map;
import java.util.concurrent.atomic.AtomicLong;

@UService(
	name = "my.samples.count",
	tips = "访问计数示例",        // 微服务的说明
	version = "1.0.0",          // 接口的版本号
	release = false             // 版本发行状态
)
public class CountService {

	@USNotes("访问计数")               // 标注一个数据模型
	public static class Counts {
		@USNote("entry1的访问数量")    // 标注模型中的属性
		public long entry1;
		@USNote("entry2的访问数量")
		public long entry2;
	}

	/** 运行信息：返回访问计数 */
	@USInfo
	public static Counts info(ServiceContext ctx) throws Exception {
		Counts counts = new Counts();
		counts.entry1 = entry1Count.get();
		counts.entry2 = entry2Count.get();
		return counts;	// 实际返回的数据会转换为Map
	}

	// 定义静态变量，记录访问次数
	static AtomicLong entry1Count = new AtomicLong(0);
	static AtomicLong entry2Count = new AtomicLong(0);

	@USEntry(
		tips = "获得数据模型的说明",
		result = "数据模型的说明，格式：{ \"模型1\": { \"字段1\": \"描述\", ... }, ... }"
	)
	public Map<String,Map<String,String>> getModels(ServiceContext ctx) {
		return Util.getUSNotes(Counts.class);	// 提取@USNotes注解的工具
	}

	@USEntry(
		tips = "接口方法1"
	)
	public void entry1(ServiceContext ctx) {
		entry1Count.incrementAndGet();
	}

	@USEntry(
		tips = "接口方法2"
	)
	public void entry2(ServiceContext ctx) {
		entry2Count.incrementAndGet();
	}

	@USEntry(
		tips = "在服务端console输出访问计数",
		params = {
			@USParam(
				name = "counts",
				tips = "访问计数的结构：发出请求时可以为Counts对象，收到参数时会被转换为Map"
			)
		}
	)
	public void print(ServiceContext ctx, Map counts) {
		// 将Map参数转换为Counts对象
		Counts countsData = Codec.toType(counts, Counts.class);
		System.out.println("========");
		System.out.println("entry1的访问次数：" + countsData.entry1);
		System.out.println("entry2的访问次数：" + countsData.entry2);
	}
}
```



最后，用 `mvn clean deploy` 命令来"发布"微服务的jar包。需要注意的是，向maven服务器"发布"jar包，还需要有相应的发布权限，权限的设置有两个步骤：

* 在nexus的管理工具中，创建开发者账号，并对"id"分别为nexus-release和nexus-snapshot的两个repository授予"发布"权限

* 在开发环境的maven配置文件settings.xml中，增加开发者账户，例如：

  ```
    <servers>
  	<server>
  		<id>nexus-release</id>
  		<username>{your account}</username>
  		<password>{your password}</password>
  	</server>
  	<server>
  		<id>nexus-snapshot</id>
  		<username>{your account}</username>
  		<password>{your password}</password>
  	</server>
    </servers>
  ```

