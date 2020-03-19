# UBSI Web管理器

---

UBSI Web管理器是一个基于UBSI微服务架构的Web应用（WebApp）。

### Web管理器的构成

UBSI Web管理器由以下几个部分构成：

* Web前端

  采用react+antd构建的Web前端应用，提供基于Web的UI交互页面

* Web后端服务

  采用SpringBoot构建的HTTP服务，通过restful-api向前端提供json数据以及操作接口，并通过UBSI Consumer访问"基础微服务"完成对业务数据的逻辑操作，并且处理用户认证/鉴权

* 基础微服务

  封装了数据存储、数据模型及其操作逻辑的若干基础微服务，供Web后端调用，这些微服务包括：

  * rewin.ubsi.repo

    "服务仓库"管理，可以管理"注册"的微服务，查看服务接口的定义，管理默认配置以及直接部署
    ```
    groupId: rewin.service.ubsi
    artifactId: rewin.service.ubsi.repo
    version: 1.0.0
    service class: rewin.service.ubsi.repo.Service
    ```

  * rewin.ubsi.logger

    日志服务，负责统一接收/存储日志数据，并提供查询/统计接口
    ```
    groupId: rewin.service.ubsi
    artifactId: rewin.service.ubsi.log
    version: 1.0.0
    service class: rewin.service.ubsi.log.Service
    ```

  * rewin.ubsi.tester

    功能/性能测试，可以创建/管理测试的脚本及方案，并且执行这些测试
    ```
    groupId: rewin.service.ubsi
    artifactId: rewin.service.ubsi.test
    version: 1.0.0
    service class: rewin.service.ubsi.test.Service
    ```

  * rewin.ubsi.gateway

    API网关的配置管理服务
    ```
    groupId: rewin.service.ubsi
    artifactId: rewin.service.ubsi.gateway
    version: 1.0.0
    service class: rewin.service.ubsi.gateway.Service
    ```

  * rewin.ubsi.scheduler

    服务编排及调度服务

    ```
    groupId: rewin.service.ubsi
    artifactId: rewin.service.ubsi.schedule
    version: 1.0.0
    service class: rewin.service.ubsi.schedule.Service
    ```

  * rewin.common.favor

    配置管理，可以用来保存各种配置数据
    ```
    groupId: rewin.service.common
    artifactId: rewin.service.common.favor
    version: 1.0.0
    service class: rewin.service.common.favor.Service
    ```

  * rewin.common.user

    用户管理，可以用来管理"用户"的数据模型，并提供"角色/权限"等管理机制
    ```
    groupId: rewin.service.common
    artifactId: rewin.service.common.user
    version: 1.0.0
    service class: rewin.service.common.user.Service
    ```

  * rewin.user.auth

    用户认证，提供简单的用户密码管理以及验证机制
    ```
    groupId: rewin.service.user
    artifactId: rewin.service.user.auth
    version: 1.0.0
    service class: rewin.service.user.auth.Service
    ```

  > 注：这些微服务的发行包都已经托管到github的ubsi-home/maven仓库中，也可以在  https://github.com/orgs/ubsi-home/packages 页面中直接查看。

* 数据库

  UBSI的基础微服务统一采用了MongoDB数据库
  

### Web管理器的部署

部署一套完整运行环境的步骤如下：

* 部署MongdoDB

  ```
  docker pull mongo
  docker run --name mongo -p 27017:27017 -d mongo
  ```

* 部署redis（可选）

  ```
  docker pull redis
  docker run --name redis -p 6379:6379 -d redis
  ```

* 部署一个服务容器

  ```
  java -jar rewin.ubsi.core-1.0.0-jar-with-dependencies.jar
  ```

  > 注意：如果部署了redis，需要先手工配置rewin.ubsi.consumer.json和rewin.ubsi.container.json，在rewin.ubsi.consumer.json中配置redis访问地址，在rewib.ubsi.container.json中配置容器的访问地址（详细内容请参阅 [部署注册中心](readme.md) ）

* 通过命令行部署工具向服务容器部署需要的基础微服务
  
  以rewin.ubsi.repo为例，创建部署文件deploy.json，内容如下：
  
  ```
  {
    "maven_url": "http://192.168.1.116:8081/repository/maven-public/",
    "service_name": "rewin.ubsi.repo",
    "service_class": "rewin.service.ubsi.repo.Service",
    "jar_group": "rewin.service.ubsi",
    "jar_artifact": "rewin.service.ubsi.repo",
    "jar_version": "1.0.0",
    "config": {
      "maven_url": "http://192.168.1.116:8081/repository/maven-public/",
      "mongo_servers": [
        {
          "server": "{mongo-server-host}",
          "port": 27017
        }
      ]
    }
  }
  ```
  > 注意：其他微服务的config中不需要配置maven_url，只需要配置mongo_servers
  
  然后执行部署命令：
  
  ```
  java -jar rewin.service.ubsi.repo-1.0.0-jar-with-dependencies.jar deploy.json {container-host} 7112
  ```
  
* 部署Web后端rest服务

  1. 在  https://ubsi-home.github.io/download 页面中下载rewin.rest.ubsi.admin-1.0.0.jar

  2. 如果配置了redis，则手工创建配置文件rewin.ubsi.consumer.json和rewin.ubsi.rest.json，其中rewin.ubsi.rest.json是WebApp的配置，内容如下：
  
     ```
     {
       "url": "http://{rest-server-host}:{rest-port}"
     }
     ```
  
     > url是WebApp中Consumer组件的配置管理服务接口的访问路径，每个WebApp的后端服务实例也会将自己"注册"到redis注册中心，以供Web管理器"发现"并进行"在线"配置，详情请见 [WebApp](../webapp/readme.md)
  
  3. 如果未配置redis，则手工创建静态路由文件rewin.ubsi.router.json，内容如下：
  
     ```
     [
       {
         "Service": "rewin.*",
         "Nodes": [
           {
             "Host": "{container-host}",
             "Port": 7112
           }
         ]
       }
     ]
     ```
  
  4. 启动Web服务：
      ```
      java -jar rewin.rest.ubsi.admin-1.0.0.jar
      ```
      
  
* 部署Web前端

  {待补充：部署nginx代理服务器，配置nginx的代理路径，下载前端页面文件}

* 部署完成，Web管理器的URL地址：

  http://{nginx-server-host}:{nginx-port}



