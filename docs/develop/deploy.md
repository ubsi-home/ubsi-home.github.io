# 服务的部署方式

---

微服务可以单独部署，但是不能独立运行，必须要部署到UBSI服务容器中才能运行。具体的部署方式可以有下面几种：



### 手工部署

在没有maven私服、没有治理工具的简单环境下，可以用纯手工方式配置并启动服务容器及微服务：

* 准备一个目录作为UBSI服务容器的工作目录

* 下载UBSI核心jar包：在  https://ubsi-home.github.io/download 页面中下载 rewin.ubsi.core-1.0.0-jar-with-dependencies.jar

* 准备微服务的jar包，包括第三方依赖

* 手工创建配置文件rewin.ubsi.module.json，用来指定需要加载的微服务：

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

* 启动容器及微服务：

  ```
  java -cp rewin.ubsi.core-1.0.0-jar-with-dependencies.jar:my.service.samples.count-1.0.0-SNAPSHOT.jar rewin.ubsi.container.Bootstrap
  ```



### 部署工具

通常情况下，我们会把服务容器的部署跟微服务的部署分开，容器的部署是独立的，可以通过手工或者docker的方式快速部署及启动。



服务容器可以"零配置"启动，不需要任何配置文件，启动后通过管理工具进行监控和配置。执行下面的命令可以手工启动一个"干净"的服务容器：

```
java -jar rewin.ubsi.core-1.0.0-jar-with-dependencies.jar
```



有了"活动"的服务容器，并且已经通过mvn deploy命令将微服务的jar包"发布"到了maven服务器，就可以通过UBSI部署工具来部署微服务了。

> 注意：maven服务器必须正确配置自己的"统一资源路径"，比如 http://192.168.1.116:8081/repository/maven-public/ ，以保证部署工具能够通过这个单一地址获得微服务的jar包及其所有依赖。



* **命令行部署工具**

  UBSI的基础微服务rewin.ubsi.repo提供了一个工具，可以帮助开发者通过命令行方式来部署微服务。

  * 获取命令行工具
  
    在  https://ubsi-home.github.io/download 页面中下载 rewin.service.ubsi.repo-1.0.0-jar-with-dependencies.jar

  * 创建部署文件 deploy.json
    
    ```
    {
      "maven_url": "http://192.168.1.116:8081/repository/maven-public/",
      "service_name": "my.samples.count",
      "service_class": "my.service.samples.CountService",
      "jar_group": "my.service",
      "jar_artifact": "my.service.samples.count",
      "jar_version": "1.0.0-SNAPSHOT",
      "config": null
    }
    ```
    
  * 执行部署命令
  
    ```
    java -jar rewin.service.ubsi.repo-1.0.0-jar-with-dependencies.jar deploy.json localhost 7112
    ```
    
    部署工具会将微服务"my.samples.count"的jar包及其依赖包从maven服务器上download下来，然后上传到指定的服务容器（localhost#7112）中，进行微服务的安装和配置，然后"启动"这个微服务。部署命令的执行结果如下：
    
    ```
    000.051 开始运行，启动UBSI Consumer
    000.502 读取部署文件
    000.518 获取最新的JAR包
    005.428 查找JAR包的依赖关系
    159.947 检查目标容器：localhost#7112
    162.866 部署my.service.samples.count : 1.0.0-SNAPSHOT
    162.881 上传my.service.samples.count-1.0.0-SNAPSHOT.jar
    162.921 注册my.service.samples.count-1.0.0-SNAPSHOT.jar
    162.964 添加微服务my.samples.count
    163.018 启动微服务my.samples.count
    164.840 部署成功!
    ```
    
    
  
* **Web管理器**

  UBSI Web管理器的"服务仓库"可以帮助完成微服务的注册、配置以及部署等操作。

  ![](../overview/repo.png)

