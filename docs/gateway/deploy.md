# 部署API Gateway

---

UBSI API Gateway可以独立部署，但是在运行时需要微服务rewin.ubsi.gateway提供配置数据，这个微服务通常是在部署UBSI Web管理器时一起部署的。



假设我们的环境中已经部署了redis注册中心以及Web管理器，可以继续部署API Gateway，步骤如下：

* 下载jar包

  在  https://ubsi-home.github.io/download 页面中下载 rewin.rest.ubsi.gateway-1.0.0.jar



* 运行配置

  API Gateway缺省的配置参数 application.properties 如下：

  ```
  ### 初始的URL路径(SpringBoot2)
  server.servlet.context-path=/
  
  spring.mvc.view.prefix=/
  spring.mvc.view.suffix=.html
  
  ### Web Server监听端口
  server.port=8080
  
  ### 网关实例的分组
  ug.group=ubsi-gate
  ### 应用令牌的过期时间(分钟数)
  ug.token-expire=720
  ### 默认的远程主机访问权限
  ug.acl.remote=false
  ### 默认的服务访问权限
  ug.acl.service=false
  ### 是否验证Token的合法性，false表示Token的值就是AppID
  ug.token.check=true
  ```

  另外，还需要在rewin.ubsi.consumer.json中配置redis的访问地址：

  ```
  {
    "redis_host": "{redis-server-host}",
    "redis_port": 6379
  }
  ```



* 启动API Gateway

  ```
  java -jar rewin.rest.ubsi.gateway-1.0.0.jar
  ```



部署完成后，访问 http://{api-gateway-host}:8080/swagger-ui.html 可以查看restful-api的接口文档。

