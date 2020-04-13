# Maven环境及依赖

---

开发UBSI微服务需要依赖UBSI的核心包rewin.ubsi.core，UBSI的发行包都托管在github packages上，仓库路径是：ubsi-home/maven。



UBSI强烈建议在开发环境中使用maven来管理项目并为自己的开发组织配置一个"私有"的maven服务器，然后将UBSI发行包所在的github仓库的URL https://maven.pkg.github.com/ubsi-home/maven/ 配置到maven私服的资源路径中（同时建议将国内的maven镜像也配置进去，这样可以提高获取"中心"jar包的速度，比如：http://maven.aliyun.com/nexus/content/groups/public/ ）。



简单介绍一下搭建一个Maven私服的步骤：（以docker为例）

* 获取并启动nexus3

  ```
  docker pull sonatype/nexus3
  docker run --name nexus3 --restart=always -p 8081:8081 -d sonatype/nexus3
  ```

* 通过 http://192.168.1.116:8081 （假设是在主机192.168.1.116上启动的nexus3）来配置maven的服务资源（例如：最终配置的统一资源路径为 http://192.168.1.116:8081/repository/maven-public/ ），具体过程请自行参考相关教程或文档。需要注意的是：在将 https://maven.pkg.github.com/ubsi-home/maven/ 配置为nexus3的proxy repository时，需要使用github的token：

  ```
  username: liuxd0911
  password: 40c2b60b2ecbb4ae7b25344d15d5eedcac196405  
  ```

  

然后需要对开发设备上的maven环境进行配置，修改maven的配置文件settings.xml，示例如下：

```
	<profile>
		<id>dev</id>
		<repositories>
			<repository>
				<id>maven-public</id>
				<url>http://192.168.1.116:8081/repository/maven-public/</url>
				<releases>
					<enabled>true</enabled>
				</releases>
				<snapshots>
					<enabled>true</enabled>
					<updatePolicy>always</updatePolicy>
				</snapshots>
			</repository>
		</repositories>
		<pluginRepositories>
			<pluginRepository>
				<id>maven-public</id>
				<url>http://192.168.1.116:8081/repository/maven-public/</url>
				<releases>
					<enabled>true</enabled>
				</releases>
				<snapshots>
					<enabled>true</enabled>
					<updatePolicy>always</updatePolicy>
				</snapshots>
			</pluginRepository>
		</pluginRepositories>
	</profile>
  </profiles>

  <activeProfiles>
	<activeProfile>dev</activeProfile>
  </activeProfiles>
```



当maven环境建设完成后，在Java项目的pom.xml中增加下面的依赖，就可以正常使用UBSI的发行包了：

```
<dependency>
	<groupId>rewin.ubsi</groupId>
	<artifactId>rewin.ubsi.core</artifactId>
	<version>1.0.0</version>
</dependency>
```

