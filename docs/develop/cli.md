# 命令行工具

---

UBSI核心包除了服务容器及Consumer组件之外，还提供了几个常用的命令行工具，可以帮助开发者在未部署UBSI Web管理器的环境下，也能快速查看和访问微服务。



### rewin.ubsi.cli.Request

通过命令行发送一个UBSI服务请求：

```
java -cp rewin.ubsi.core-1.0.0-jar-with-dependencies.jar rewin.ubsi.cli.Request my.samples.count print "{\"entry1\":1,\"entry2\":2}"
```

在容器"localhost#7112"的控制台上会看到如下输出：

```
========
entry1的访问次数：1
entry2的访问次数：2
```



### rewin.ubsi.cli.Console

命令行交互工具：

```
java -cp rewin.ubsi.core-1.0.0-jar-with-dependencies.jar rewin.ubsi.cli.Console

UBSI Consumer Console v1.0.0, press ENTER for help

localhost#7112>

alone [on|off]                                           - show or set connect alone (use for router mode)
async [on|off]                                           - show or set receive result through Redis-MQ
call                                                     - request call to routed Container (switch to router mode)
config [router|log]                                      - show local config for Consumer [route|log]
direct [host [port]]                                     - request direct to host#port (switch to direct mode)
entry service [entry]                                    - show service's entry in Container (direct mode)
event channel data ...                                   - put event to channel
header [key [value]]                                     - show or set|clear request header's key-value
jedis                                                    - show Jedis pools
json                                                     - set JSON data format
publish channel data ...                                 - publish message to channel
register container|restful                               - show register data of Containers or Restful-Consumer
request service entry ...                                - send request synchronized
router service                                           - get service's routed path
service                                                  - show services in Container (direct mode)
statistics                                               - show request's statistics report
subscribe [channel|pattern#channel|event#channel ...]    - subscribe message or event channel
time                                                     - show request's result time (milli-seconds)
timeout [seconds]                                        - show or set request's timeout
tracelog [on|off]                                        - show or set access-log's setting
unsubscribe [channel|pattern#channel|event#channel ...]  - unsubscribe message or event channel
use service                                              - use spec service
version [min max release]                                - show or set request service's version
xml                                                      - set XML data format

localhost#7112>
```



比较常用的命令有：

* entry - 查看微服务的接口，例如：

  ```
  localhost#7112> entry my.samples.count
  
  entry1(): 接口方法1
  
  print(): 在服务端console输出访问计数
  参数：
    counts: java.util.Map, 访问计数的结构：发出请求时可以为Counts对象，收到参数时会被转换为Map
  
  getModels(): 获得数据模型的说明
  返回：
    java.util.Map<java.lang.String, java.util.Map<java.lang.String, java.lang.String>>, 数据模型的说明，格式：{ "模型1": { "字段1": "描述", ... }, ... }
  
  entry2(): 接口方法2
  
  localhost#7112> 
  ```

  

* request - 访问微服务的接口，例如：

  ```
  localhost#7112> request my.samples.count getModels
  
  {
    "my.service.samples.CountService$Counts: 访问计数": {
      "entry1": "long, entry1的访问数量",
      "entry2": "long, entry2的访问数量"
    }
  }
  
  localhost#7112>
  ```

  

### rewin.ubsi.cli.Stress

这是一个简单的性能测试工具，执行方式如下：

```
java -cp rewin.ubsi.core-1.0.0-jar-with-dependencies.jar rewin.ubsi.cli.Stress req.json 1000
```

参数req.json是数据文件，用来设置需要发送的服务请求，内容如下：

```
{
  "service": "my.samples.count",
  "entry": "getModels"
}
```

参数1000是一个连续发送请求的阈值，具体的工作机制是：

* 利用UBSI Consumer的非阻塞异步请求机制连续发送指定的请求，并实时计算总请求数量和总应答数量，如果发现二者的差值大于指定的阈值，就暂停发送，等待服务端进行处理，直到差值小于阈值再恢复发送

这种简单的流量控制机制是为了防止服务端出现过载拒绝服务的情况，影响性能数据的准确测量。同时，为了配合性能测试，还需要调整服务容器的负载能力参数 - 在容器的工作目录下手工创建配置文件rewin.ubsi.container.json：

```
{
  "host": "my-pc",
  "port": 7112,
  "backlog": 128,
  "io_threads": 4,
  "work_threads": 20,
  "overload": 1000,
  "forward": 0
}
```

其中：

* host | port

  容器所在服务器的访问地址（建议使用DNS域名，不建议使用IP）和端口

* backlog

  建立socket连接的等待队列长度

* io_threads

  用来处理socket I/O的线程数，0表示默认设置

* work_threads

  处理服务请求的线程数量（注：并不是线程数越多并发处理能力就越强，请根据可用的CPU核数合理配置）

* overload

  等待处理的请求队列长度，如果等待队列已满，新的请求会被拒绝（注：在配合Stress测试时，这个值应该设置为Stress的阈值）

* forward

  如果请求的微服务不在本容器内，是否允许容器转发这个请求（0表示不转发）

> 注：手工修改配置后需要重启服务容器才能生效，建议使用UBSI Web管理器对容器节点进行管理，可以实现动态参数配置



另外，Stress工具使用了"路由"方式来访问微服务，与"直连"方式不同，"路由"方式可以保持socket长连接，并利用多路复用机制提高通讯效率，而"直连"方式每次请求都会单独建立一个socket连接，请求完成后关闭，这种方式效率较低，只建议用在特定的"测试"或"监控"场景。



通常情况下，"路由"方式应该配置"注册中心"，这样Consumer组件会自动发现可被访问的微服务实例，这种方式需要部署redis server；如果在一个"简单"的环境中，不需要"注册中心"，也可以通过配置静态路由的方式来指定服务路径 - 手工创建一个配置文件rewin.ubsi.router.json：

```
[
  {
    "Service": "my.samples.count",
    "Nodes": [
      {
        "Host": "my-pc",
        "Port": 7112,
        "Weight": 1
      }
    ]
  }
]
```

其中：

* Weight 参数表示容器节点的权重，如果某个服务有多个节点可选，路由算法会根据权重来动态分配请求



> 静态路由也可以跟"注册中心"提供的动态路由混合使用，在这种情况下，可以通过配置静态路由来实现根据"接口版本/发行状态/可用节点"等对请求进行限定，这种机制通常在"灰度发布"时使用。



OK！现在所有的准备工作完成，重新执行Stress测试工具，可以得到如下的结果：

```
java -cp rewin.ubsi.core-1.0.0-jar-with-dependencies.jar rewin.ubsi.cli.Stress req.json 1000

my.samples.count:getModels(): {
  "my.service.samples.CountService$Counts: 访问计数": {
    "entry1": "long, entry1的访问数量",
    "entry2": "long, entry2的访问数量"
  }
}

start stress testing ...

--- send: 118165, err: 0, ok: 117642, 15596/s

--- send: 85347, err: 0, ok: 84837, 23442/s

--- send: 83707, err: 0, ok: 83852, 24088/s

--- send: 78001, err: 0, ok: 77832, 23075/s

--- send: 70572, err: 0, ok: 70961, 23380/s

--- send: 83001, err: 0, ok: 82115, 23773/s
q
--- send: 59815, err: 0, ok: 59810, 23668/s

stress testing stopped!

main thread over: 581356 / 581356 / 0
```



Stress在验证服务请求可以成功返回后，开始连续发送请求并计算每秒的应答数量：

* 每次按"return"，都会显示"本段时间"内发送的请求数(send)、失败的请求数(err)、成功返回的请求数(应答数：ok)、每秒收到的应答数
* 按"q"表示退出测试
* 最后给出"发送"/"成功"/"失败"的请求总数



> 特别提示：
>
> 除非特殊情况，UBSI不建议手工配置任何服务容器/Consumer组件的运行参数，应该部署Web管理器来进行参数配置工作。

