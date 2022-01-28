# PinPoint安装配置

## 简介

![1](https://qiniu.oneone.life/img/1.png)

<br />pinpoint是开源在github上的一款全链路APM监控工具，提供了无侵入式的调用链监控、方法执行详情查看、应用状态信息监控等功能。基于GoogleDapper论文进行的实现，与另一款开源的全链路分析工具Zipkin类似，但相比Zipkin提供了无侵入式、代码维度的监控等更多的特性。 Pinpoint支持的功能比较丰富，可以支持如下几种功能：
<!--more-->
- 服务拓扑图：

    对整个系统中应用的调用关系进行了可视化的展示，单击某个服务节点，可以显示该节点的    详细信息，比如当前节点状态、请求数量等

- 实时活跃线程图：

    监控应用内活跃线程的执行情况，对应用的线程执行性能可以有比较直观的了解

- 请求响应散点图：
    
    以时间维度进行请求计数和响应时间的展示，拖过拖动图表可以选择对应的请求查看执行的详细情况

- 请求调用栈查看：
    
    对分布式环境中每个请求提供了代码维度的可见性，可以在页面中查看请求针对到代码维度的执行详情，帮助查找请求的瓶颈和故障原因。

- 应用状态、机器状态检查：
    
    通过这个功能可以查看相关应用程序的其他的一些详细信息，比如CPU使用情况，内存状态、垃圾收集状态，TPS和JVM信息等参数。

总体来说，使用pinpoint的一些优点：

- 可以掌握系统的整体响应速度情况，对系统运行情况能比较清晰

- 可以掌握各节点的响应速度情况，比如第三方服务接口，redis，mysql等

- 可以掌握单次请求的具体服务链路耗时情况，定位性能瓶颈

- 单次请求的具体服务链路请求信息，对于排查问题能提供帮助

- 监控各服务的JVM、线程池、数据库连接池使用情况，尤其是对分布式服务系统来说


<br />**github地址：**<br />​

项目地址：[https://github.com/pinpoint-apm/pinpoint](https://github.com/pinpoint-apm/pinpoint)<br />官方docker部署地址：[https://github.com/pinpoint-apm/pinpoint-docker](https://github.com/pinpoint-apm/pinpoint-docker)
<a name="kOl6Q"></a>

## 架构组成

<br />Pinpoint 主要由 3 个组件外加 Hbase 数据库组成，三个组件分别为：Agent、Collector 和 Web UI<br />​<br />

- Agent组件：
    
    用于收集应用端监控数据，无侵入式，只需要在启动命令中加入部分参数即可

- Collector组件：
   
    数据收集模块，接收Agent发送过来的监控数据，并存储到HBase

- WebUI：
    
    监控展示模块，展示系统调用关系、调用详情、应用状态等，并支持报警等功能
    
![2](https://qiniu.oneone.life/img/2.png)<br />

<a name="TRU4Z"></a>
## 安装Collector组件

<br />使用docker-compose安装最新版本：
```shell=
# 克隆官方的docker部署git
git clone https://github.com/naver/pinpoint-docker.git

# 进入clone的目录
cd pinpoint-docker

# 使用docker-compose拉取镜像和运行服务。注意docker-compose是需要单独安装的
docker-compose pull && docker-compose up -d

```

如果想安装历史或指定版本，可以直接指定git tags：
```shell=
# 克隆官方的docker部署git
git clone https://github.com/naver/pinpoint-docker.git

# 进入clone的目录
cd pinpoint-docker

# 使用git命令切换到指定版本的tag
git checkout {tag}

# 使用docker-compose拉取镜像和运行服务。注意docker-compose是需要单独安装的
docker-compose pull && docker-compose up -d
```


<br />如果部署的时候想使用一些自定义配置，例如指定的端口，可以通过修改 `docker-compose.yml` 和 `.env` 中对应的配置来实现，**建议优先通过修改.env文件来做自定义配置**


修改端口可以直接修改.env文件，默认端口是Pinpoint-Web：8079，QuickStart：8000和Flink：8081
    
![3](https://qiniu.oneone.life/img/3.png)

部署完成之后，使用`docker-compose ps`命令即可查看所有容器：

![4](https://qiniu.oneone.life/img/4.png)

访问 http://xx.xx.xx.xx:8079/ 即可浏览pinpoint页面
![5](https://qiniu.oneone.life/img/5.png)



## 安装配置Agent组件

从`github`中`pinpoint`的[Release发布页面](https://github.com/pinpoint-apm/pinpoint/releases)中下载和Collector组件版本一致的Agent程序包

![6](https://qiniu.oneone.life/img/6.png)


将Agent程序包上传到需要监控的服务器上，解压压缩包，修改`pinpoint.config`配置文件，一般只需修改文件中的`profiler.collector.ip=127.0.0.1`为前面所部署的Collector组件所在IP地址

![7](https://qiniu.oneone.life/img/7.png)

![](https://upyun.oneone.life/upyun-img/1637221323192-df61d479-750c-4f3c-acb7-6e8d22d100c9.png)

- 配置文件分为`local`和`Release`版本，选择任一一个修改即可，但是需要在`pinpoint-root.config`文件中指定对应的配置版本
![](https://upyun.oneone.life/upyun-img/1637220823104-c25a8f32-39bd-4c39-9d1a-923c77c090ac.png)
- 如果在多台服务器上部署了应用，那么就需要在多台机器上部署Agent组件

### 配置监听应用

pinpoint的监控完全是无侵入式的，配置起来也很简单，只需要在java应用的启动命令增加几个参数即可：

```shell=
# ${pinpointPath}是agent组件存放的路径，类似于JAVA_HOME
-javaagent:${pinpointPath}/pinpoint-bootstrap-2.3.1.jar

#'dpccb'可自定义命名，在pinpoint页面上显示的名称
-Dpinpoint.applicationName=dpccb 

#id可自定义命名，可以和Name一样
-Dpinpoint.agentId=dpccb

```

启动应用程序，打开应用页面访问一下，然后登陆pinpoint的Web页面即可看到相关的监控内容了


如果应用是通过容器方式运行的，也可以通过修改启动脚本和Dockerfile的方式来配置agent的监听

//修改启动命令
![](https://upyun.oneone.life/upyun-img/1637223237860-a6bbf591-0c2b-43ab-a793-5638823776e8.png)

//把agent的存放目录挂载到应用的容器内
![](https://upyun.oneone.life/upyun-img/1637223390002-c6ff6722-09a0-426c-bd0d-9cf2cbfc1136.png)





## pinpoint功能介绍




<u>**详细使用教程可以参考这篇文章：[pinpoint使用详解（图文版）](https://blog.csdn.net/weixin_43931358/article/details/107671436)**</u>


### 首页

选择一个需要查看的应用，即可在首页中以图形化的方式展示用户请求、服务间的调用关系等信息。另外还有响应时间分析图、调用散点图、响应时长分布、等待时长分布等。在右侧的调用统计图中用鼠标左键框选(如图上红框)即可查看选中部分的调用详情

![](https://upyun.oneone.life/upyun-img/1637223668576-e2d68eab-7b95-456f-890a-caf23463f0e8.png)
### 调用详细信息

详情页，选择一个请求后下方会显示其详细信息，包含响应时间，请求过程中涉及的代码方法，sql语句等

![](https://upyun.oneone.life/upyun-img/1637224277566-811864f4-b821-4141-a4c5-d34a3e8f167f.png)

选择混合视图后还可以同时查看调用树、服务器性能占用、server map等信息
![](https://upyun.oneone.life/upyun-img/1637224646777-27af178d-42fa-414e-94ac-bbd324e877ec.png)

### 应用检查工具

查看应用的其他细节，如CPU使用、内存/垃圾收集、TPS和JVM参数
![](https://upyun.oneone.life/upyun-img/1637224976595-936ab9da-68a6-44b7-b07a-b74de01c45d4.png)
![](https://upyun.oneone.life/upyun-img/1637225035796-a4d76b2b-07f9-40f3-8b7f-b6614e686944.png)
![](https://upyun.oneone.life/upyun-img/1637225079828-e98e4181-a9d1-4f5c-a71d-7cd8912d660e.png)
