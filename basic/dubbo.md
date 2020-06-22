# Table of Contents

* [概述](#概述)
  * [架构演变](#架构演变)
  * [RPC](#rpc)
  * [需求](#需求)
  * [特性](#特性)
    * [连通性](#连通性)
    * [健壮性](#健壮性)
    * [伸缩性](#伸缩性)
    * [升级性](#升级性)
  * [XML 配置](#xml-配置)
    * [provider.xml 示例](#providerxml-示例)
    * [consumer.xml示例](#consumerxml示例)
    * [配置之间的关系](#配置之间的关系)
    * [配置含义](#配置含义)
* [框架设计](#框架设计)
  * [核心交互角色](#核心交互角色)
  * [内部分层架构](#内部分层架构)
  * [远程调用细节](#远程调用细节)
    * [服务提供者暴露一个服务的详细过程](#服务提供者暴露一个服务的详细过程)
    * [服务消费者消费一个服务的详细过程](#服务消费者消费一个服务的详细过程)
    * [满眼都是 Invoker](#满眼都是-invoker)
  * [领域模型](#领域模型)
  * [线程模型](#线程模型)
* [协议](#协议)
  * [协议比较](#协议比较)
  * [协议的配置](#协议的配置)
* [集群](#集群)
  * [集群容错](#集群容错)
    * [集群容错模式](#集群容错模式)
    * [集群模式配置](#集群模式配置)
  * [负载均衡](#负载均衡)
* [参考](#参考)


# 概述

## 架构演变


![img](https://img-blog.csdnimg.cn/20190427171306960.png)


（1）、单一应用架构 当网站流量很小时，只需一个应用，将所有功能都部署在一起，以减少部署节点和成本。此时，用于简化增删改查工作量的数据访问框架 (ORM) 是关键。

![img](https://img-blog.csdnimg.cn/20190427171321515.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpcWlhbmcxMjE3,size_16,color_FFFFFF,t_70)


 适用于小型网站，小型管理系统，将所有功能都部署到一个功能里，简单易用。 缺点：  	1、性能扩展比较难   2、协同开发问题   3、不利于升级维护（2）、垂直应用架构 当访问量逐渐增大，单一应用增加机器带来的加速度越来越小，将应用拆成互不相干的几个应用，以提升效率。此时，用于加速前端页面开发的 Web 框架 (MVC) 是关键。

![img](https://img-blog.csdnimg.cn/20190427171332825.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpcWlhbmcxMjE3,size_16,color_FFFFFF,t_70)


 通过切分业务来实现各个模块独立部署，降低了维护和部署的难度，团队各司其职更易管理，性能扩展也更方便，更有针对性。 缺点： 公用模块无法重复利用，开发性的浪费（3）、分布式服务架构 当垂直应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。此时，用于提高业务复用及整合的分布式服务框架 (RPC) 是关键。

![img](https://img-blog.csdnimg.cn/20190427171345969.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpcWlhbmcxMjE3,size_16,color_FFFFFF,t_70)


（4）、流动计算架构 当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个调度中心基于访问压力实时管理集群容量，提高集群利用率。此时，用于提高机器利用率的资源调度和治理中心 (SOA)[ Service Oriented Architecture] 是关键。

![img](https://img-blog.csdnimg.cn/20190427171354391.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpcWlhbmcxMjE3,size_16,color_FFFFFF,t_70)

## RPC

RPC【Remote Procedure Call】是指远程过程调用，是一种进程间通信方式，他是一种技术的思想，而不是规范。它允许程序调用另一个地址空间（通常是共享网络的另一台机器上）的过程或函数，而不用程序员显式编码这个远程调用的细节。即程序员无论是调用本地的还是远程的函数，本质上编写的调用代码基本相同。

![img](https://img-blog.csdnimg.cn/20190429202450144.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpcWlhbmcxMjE3,size_16,color_FFFFFF,t_70)


一次完整的 RPC 调用流程如下：1）服务消费方（client）调用以本地调用方式调用服务；2）client stub 接收到调用后负责将方法、参数等组装成能够进行网络传输的消息体；3）client stub 找到服务地址，并将消息发送到服务端；4）server stub 收到消息后进行解码；5）server stub 根据解码结果调用本地的服务；6）本地服务执行并将结果返回给 server stub；7）server stub 将返回结果打包成消息并发送至消费方；8）client stub 接收到消息，并进行解码；9）服务消费方得到最终结果。

RPC 框架的目标就是要将 2 到 8 这些步骤都封装起来，这些细节对用户来说是透明的，不可见的。

![img](https://img-blog.csdnimg.cn/20190427171703697.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpcWlhbmcxMjE3,size_16,color_FFFFFF,t_70)

## 需求

![image](http://dubbo.apache.org/docs/zh-cn/user/sources/images/dubbo-service-governance.jpg)

在大规模服务化之前，应用可能只是通过 RMI 或 Hessian 等工具，简单的暴露和引用远程服务，通过配置服务的URL地址进行调用，通过 F5 等硬件进行负载均衡。

**当服务越来越多时，服务 URL 配置管理变得非常困难，F5 硬件负载均衡器的单点压力也越来越大。** 此时需要一个服务注册中心，动态地注册和发现服务，使服务的位置透明。并通过在消费方获取服务提供方地址列表，实现软负载均衡和 Failover，降低对 F5 硬件负载均衡器的依赖，也能减少部分成本。

**当进一步发展，服务间依赖关系变得错踪复杂，甚至分不清哪个应用要在哪个应用之前启动，架构师都不能完整的描述应用的架构关系。** 这时，需要自动画出应用间的依赖关系图，以帮助架构师理清理关系。

**接着，服务的调用量越来越大，服务的容量问题就暴露出来，这个服务需要多少机器支撑？什么时候该加机器？** 为了解决这些问题，第一步，要将服务现在每天的调用量，响应时间，都统计出来，作为容量规划的参考指标。其次，要可以动态调整权重，在线上，将某台机器的权重一直加大，并在加大的过程中记录响应时间的变化，直到响应时间到达阈值，记录此时的访问量，再以此访问量乘以机器数反推总容量。

以上是 Dubbo 最基本的几个需求。

## 特性

Dubbo 架构具有以下几个特点，分别是连通性、健壮性、伸缩性、以及向未来架构的升级性。

### 连通性

- 注册中心负责服务地址的注册与查找，相当于目录服务，服务提供者和消费者只在启动时与注册中心交互，注册中心不转发请求，压力较小
- 监控中心负责统计各服务调用次数，调用时间等，统计先在内存汇总后每分钟一次发送到监控中心服务器，并以报表展示
- 服务提供者向注册中心注册其提供的服务，并汇报调用时间到监控中心，此时间不包含网络开销
- 服务消费者向注册中心获取服务提供者地址列表，并根据负载算法直接调用提供者，同时汇报调用时间到监控中心，此时间包含网络开销
- 注册中心，服务提供者，服务消费者三者之间均为长连接，监控中心除外
- 注册中心通过长连接感知服务提供者的存在，服务提供者宕机，注册中心将立即推送事件通知消费者
- 注册中心和监控中心全部宕机，不影响已运行的提供者和消费者，消费者在本地缓存了提供者列表
- 注册中心和监控中心都是可选的，服务消费者可以直连服务提供者

### 健壮性

- 监控中心宕掉不影响使用，只是丢失部分采样数据
- 数据库宕掉后，注册中心仍能通过缓存提供服务列表查询，但不能注册新服务
- 注册中心对等集群，任意一台宕掉后，将自动切换到另一台
- 注册中心全部宕掉后，服务提供者和服务消费者仍能通过本地缓存通讯
- 服务提供者无状态，任意一台宕掉后，不影响使用
- 服务提供者全部宕掉后，服务消费者应用将无法使用，并无限次重连等待服务提供者恢复

### 伸缩性

- 注册中心为对等集群，可动态增加机器部署实例，所有客户端将自动发现新的注册中心
- 服务提供者无状态，可动态增加机器部署实例，注册中心将推送新的服务提供者信息给消费者

### 升级性

当服务集群规模进一步扩大，带动IT治理结构进一步升级，需要实现动态部署，进行流动计算，现有分布式服务架构不会带来阻力。下图是未来可能的一种架构：

![dubbo-architucture-futures](http://dubbo.apache.org/docs/zh-cn/user/sources/images/dubbo-architecture-future.jpg)

|      |      |
| ---- | ---- |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |

## XML 配置

### provider.xml 示例

```xml
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    <dubbo:application name="demo-provider"/>
    <dubbo:registry address="zookeeper://127.0.0.1:2181"/>
    <dubbo:protocol name="dubbo" port="20890"/>
    <bean id="demoService" class="org.apache.dubbo.samples.basic.impl.DemoServiceImpl"/>
    <dubbo:service interface="org.apache.dubbo.samples.basic.api.DemoService" ref="demoService"/>
</beans>
```

### consumer.xml示例

```xml
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    <dubbo:application name="demo-consumer"/>
    <dubbo:registry group="aaa" address="zookeeper://127.0.0.1:2181"/>
    <dubbo:reference id="demoService" check="false" interface="org.apache.dubbo.samples.basic.api.DemoService"/>
</beans>
```

### 配置之间的关系

![dubbo-config](http://dubbo.apache.org/docs/zh-cn/user/sources/images/dubbo-config.jpg)

### 配置含义

| 标签                   | 用途         | 解释                                                         |
| ---------------------- | ------------ | ------------------------------------------------------------ |
| `<dubbo:service/>`     | 服务配置     | 用于暴露一个服务，定义服务的元信息，一个服务可以用多个协议暴露，一个服务也可以注册到多个注册中心 |
| `<dubbo:reference/>`   | 引用配置     | 用于创建一个远程服务代理，一个引用可以指向多个注册中心       |
| `<dubbo:protocol/>`    | 协议配置     | 用于配置提供服务的协议信息，协议由提供方指定，消费方被动接受 |
| `<dubbo:application/>` | 应用配置     | 用于配置当前应用信息，不管该应用是提供者还是消费者           |
| `<dubbo:module/>`      | 模块配置     | 用于配置当前模块信息，可选                                   |
| `<dubbo:registry/>`    | 注册中心配置 | 用于配置连接注册中心相关信息                                 |
| `<dubbo:monitor/>`     | 监控中心配置 | 用于配置连接监控中心相关信息，可选                           |
| `<dubbo:provider/>`    | 提供方配置   | 当 ProtocolConfig 和 ServiceConfig 某属性没有配置时，采用此缺省值，可选 |
| `<dubbo:consumer/>`    | 消费方配置   | 当 ReferenceConfig 某属性没有配置时，采用此缺省值，可选      |
| `<dubbo:method/>`      | 方法配置     | 用于 ServiceConfig 和 ReferenceConfig 指定方法级的配置信息   |
| `<dubbo:argument/>`    | 参数配置     | 用于指定方法参数配置                                         |

# 框架设计

## 核心交互角色

![dubbo-architucture](http://dubbo.apache.org/docs/zh-cn/user/sources/images/dubbo-architecture.jpg)

节点角色说明

| 节点        | 角色说明                               |
| ----------- | -------------------------------------- |
| `Provider`  | 暴露服务的服务提供方                   |
| `Consumer`  | 调用远程服务的服务消费方               |
| `Registry`  | 服务注册与发现的注册中心               |
| `Monitor`   | 统计服务的调用次数和调用时间的监控中心 |
| `Container` | 服务运行容器                           |

调用关系说明

1. 服务容器负责启动，加载，运行服务提供者。
2. 服务提供者在启动时，向注册中心注册自己提供的服务。
3. 服务消费者在启动时，向注册中心订阅自己所需的服务。
4. 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
5. 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
6. 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

## 内部分层架构

![/dev-guide/images/dubbo-framework.jpg](http://dubbo.apache.org/docs/zh-cn/dev/sources/images/dubbo-framework.jpg)

图例说明：

- 图中左边淡蓝背景的为服务消费方使用的接口，右边淡绿色背景的为服务提供方使用的接口，位于中轴线上的为双方都用到的接口。
- 图中从下至上分为十层，各层均为单向依赖，右边的黑色箭头代表层之间的依赖关系，每一层都可以剥离上层被复用，其中，Service 和 Config 层为 API，其它各层均为 SPI。
- 图中绿色小块的为扩展接口，蓝色小块为实现类，图中只显示用于关联各层的实现类。
- 图中蓝色虚线为初始化过程，即启动时组装链，红色实线为方法调用过程，即运行时调时链，紫色三角箭头为继承，可以把子类看作父类的同一个节点，线上的文字为调用的方法。

**各层说明**

- **config 配置层**：对外配置接口，以 `ServiceConfig`, `ReferenceConfig` 为中心，可以直接初始化配置类，也可以通过 spring 解析配置生成配置类
- **proxy 服务代理层**：服务接口透明代理，生成服务的客户端 Stub 和服务器端 Skeleton, 以 `ServiceProxy`为中心，扩展接口为 `ProxyFactory`
- **registry 注册中心层**：封装服务地址的注册与发现，以服务 URL 为中心，扩展接口为 `RegistryFactory`, `Registry`, `RegistryService`
- **cluster 路由层**：封装多个提供者的路由及负载均衡，并桥接注册中心，以 `Invoker` 为中心，扩展接口为 `Cluster`, `Directory`, `Router`, `LoadBalance`
- **monitor 监控层**：RPC 调用次数和调用时间监控，以 `Statistics` 为中心，扩展接口为 `MonitorFactory`, `Monitor`, `MonitorService`
- **protocol 远程调用层**：封装 RPC 调用，以 `Invocation`, `Result` 为中心，扩展接口为 `Protocol`, `Invoker`, `Exporter`
- **exchange 信息交换层**：封装请求响应模式，同步转异步，以 `Request`, `Response` 为中心，扩展接口为 `Exchanger`, `ExchangeChannel`, `ExchangeClient`, `ExchangeServer`
- **transport 网络传输层**：抽象 mina 和 netty 为统一接口，以 `Message` 为中心，扩展接口为 `Channel`, `Transporter`, `Client`, `Server`, `Codec`
- **serialize 数据序列化层**：可复用的一些工具，扩展接口为 `Serialization`, `ObjectInput`, `ObjectOutput`, `ThreadPool`

**关系说明**

- 在 RPC 中，Protocol 是核心层，也就是只要有 Protocol + Invoker + Exporter 就可以完成非透明的 RPC 调用，然后在 Invoker 的主过程上 Filter 拦截点。
- 图中的 Consumer 和 Provider 是抽象概念，只是想让看图者更直观的了解哪些类分属于客户端与服务器端，不用 Client 和 Server 的原因是 Dubbo 在很多场景下都使用 Provider, Consumer, Registry, Monitor 划分逻辑拓普节点，保持统一概念。
- 而 Cluster 是外围概念，所以 Cluster 的目的是将多个 Invoker 伪装成一个 Invoker，这样其它人只要关注 Protocol 层 Invoker 即可，加上 Cluster 或者去掉 Cluster 对其它层都不会造成影响，因为只有一个提供者时，是不需要 Cluster 的。
- Proxy 层封装了所有接口的透明化代理，而在其它层都以 Invoker 为中心，只有到了暴露给用户使用时，才用 Proxy 将 Invoker 转成接口，或将接口实现转成 Invoker，也就是去掉 Proxy 层 RPC 是可以 Run 的，只是不那么透明，不那么看起来像调本地服务一样调远程服务。
- 而 Remoting 实现是 Dubbo 协议的实现，如果你选择 RMI 协议，整个 Remoting 都不会用上，Remoting 内部再划为 Transport 传输层和 Exchange 信息交换层，Transport 层只负责单向消息传输，是对 Mina, Netty, Grizzly 的抽象，它也可以扩展 UDP 传输，而 Exchange 层是在传输层之上封装了 Request-Response 语义。
- Registry 和 Monitor 实际上不算一层，而是一个独立的节点，只是为了全局概览，用层的方式画在一起。

**模块分包**

![/dev-guide/images/dubbo-modules.jpg](http://dubbo.apache.org/docs/zh-cn/dev/sources/images/dubbo-modules.jpg)

模块说明：

- **dubbo-common 公共逻辑模块**：包括 Util 类和通用模型。
- **dubbo-remoting 远程通讯模块**：相当于 Dubbo 协议的实现，如果 RPC 用 RMI协议则不需要使用此包。
- **dubbo-rpc 远程调用模块**：抽象各种协议，以及动态代理，只包含一对一的调用，不关心集群的管理。
- **dubbo-cluster 集群模块**：将多个服务提供方伪装为一个提供方，包括：负载均衡, 容错，路由等，集群的地址列表可以是静态配置的，也可以是由注册中心下发。
- **dubbo-registry 注册中心模块**：基于注册中心下发地址的集群方式，以及对各种注册中心的抽象。
- **dubbo-monitor 监控模块**：统计服务调用次数，调用时间的，调用链跟踪的服务。
- **dubbo-config 配置模块**：是 Dubbo 对外的 API，用户通过 Config 使用Dubbo，隐藏 Dubbo 所有细节。
- **dubbo-container 容器模块**：是一个 Standlone 的容器，以简单的 Main 加载 Spring 启动，因为服务通常不需要 Tomcat/JBoss 等 Web 容器的特性，没必要用 Web 容器去加载服务。

整体上按照分层结构进行分包，与分层的不同点在于：

- container 为服务容器，用于部署运行服务，没有在层中画出。
- protocol 层和 proxy 层都放在 rpc 模块中，这两层是 rpc 的核心，在不需要集群也就是只有一个提供者时，可以只使用这两层完成 rpc 调用。
- transport 层和 exchange 层都放在 remoting 模块中，为 rpc 调用的通讯基础。
- serialize 层放在 common 模块中，以便更大程度复用。

## 远程调用细节

### 服务提供者暴露一个服务的详细过程

![/dev-guide/images/dubbo_rpc_export.jpg](http://dubbo.apache.org/docs/zh-cn/dev/sources/images/dubbo_rpc_export.jpg)

上图是服务提供者暴露服务的主过程：

首先 `ServiceConfig` 类拿到对外提供服务的实际类 ref(如：HelloWorldImpl),然后通过 `ProxyFactory` 类的 `getInvoker` 方法使用 ref 生成一个 `AbstractProxyInvoker` 实例，到这一步就完成具体服务到 `Invoker` 的转化。接下来就是 `Invoker` 转换到 `Exporter` 的过程。

Dubbo 处理服务暴露的关键就在 `Invoker` 转换到 `Exporter` 的过程，上图中的红色部分。下面我们以 Dubbo 和 RMI 这两种典型协议的实现来进行说明：

**Dubbo 的实现**

Dubbo 协议的 `Invoker` 转为 `Exporter` 发生在 `DubboProtocol` 类的 `export` 方法，它主要是打开 socket 侦听服务，并接收客户端发来的各种请求，通讯细节由 Dubbo 自己实现。

**RMI 的实现**

RMI 协议的 `Invoker` 转为 `Exporter` 发生在 `RmiProtocol`类的 `export` 方法，它通过 Spring 或 Dubbo 或 JDK 来实现 RMI 服务，通讯细节这一块由 JDK 底层来实现，这就省了不少工作量。

### 服务消费者消费一个服务的详细过程

![/dev-guide/images/dubbo_rpc_refer.jpg](http://dubbo.apache.org/docs/zh-cn/dev/sources/images/dubbo_rpc_refer.jpg)

上图是服务消费的主过程：

首先 `ReferenceConfig` 类的 `init` 方法调用 `Protocol` 的 `refer` 方法生成 `Invoker` 实例(如上图中的红色部分)，这是服务消费的关键。接下来把 `Invoker` 转换为客户端需要的接口(如：HelloWorld)。

关于每种协议如 RMI/Dubbo/Web service 等它们在调用 `refer` 方法生成 `Invoker` 实例的细节和上一章节所描述的类似。

### 满眼都是 Invoker

由于 `Invoker` 是 Dubbo 领域模型中非常重要的一个概念，很多设计思路都是向它靠拢。这就使得 `Invoker`渗透在整个实现代码里，对于刚开始接触 Dubbo 的人，确实容易给搞混了。 下面我们用一个精简的图来说明最重要的两种 `Invoker`：服务提供 `Invoker` 和服务消费 `Invoker`：

![/dev-guide/images/dubbo_rpc_invoke.jpg](http://dubbo.apache.org/docs/zh-cn/dev/sources/images/dubbo_rpc_invoke.jpg)

为了更好的解释上面这张图，我们结合服务消费和提供者的代码示例来进行说明：

服务消费者代码：

```java
public class DemoClientAction {
 
    private DemoService demoService;
 
    public void setDemoService(DemoService demoService) {
        this.demoService = demoService;
    }
 
    public void start() {
        String hello = demoService.sayHello("world" + i);
    }
}
```

上面代码中的 `DemoService` 就是上图中服务消费端的 proxy，用户代码通过这个 proxy 调用其对应的 `Invoker` [[5\]](http://dubbo.apache.org/zh-cn/docs/dev/implementation.html#fn5)，而该 `Invoker` 实现了真正的远程服务调用。

服务提供者代码：

```java
public class DemoServiceImpl implements DemoService {
 
    public String sayHello(String name) throws RemoteException {
        return "Hello " + name;
    }
}
```

上面这个类会被封装成为一个 `AbstractProxyInvoker` 实例，并新生成一个 `Exporter` 实例。这样当网络通讯层收到一个请求后，会找到对应的 `Exporter` 实例，并调用它所对应的 `AbstractProxyInvoker` 实例，从而真正调用了服务提供者的代码。Dubbo 里还有一些其他的 `Invoker` 类，但上面两种是最重要的。

## 领域模型

**服务域/实体域/会话域分离**

任何框架或组件，总会有核心领域模型，比如：Spring 的 Bean，Struts 的 Action，Dubbo 的 Service，Napoli 的 Queue 等等。这个核心领域模型及其组成部分称为实体域，它代表着我们要操作的目标本身。实体域通常是线程安全的，不管是通过不变类，同步状态，或复制的方式。

服务域也就是行为域，它是组件的功能集，同时也负责实体域和会话域的生命周期管理， 比如 Spring 的 ApplicationContext，Dubbo 的 ServiceManager 等。服务域的对象通常会比较重，而且是线程安全的，并以单一实例服务于所有调用。

什么是会话？就是一次交互过程。会话中重要的概念是上下文，什么是上下文？比如我们说：“老地方见”，这里的“老地方”就是上下文信息。为什么说“老地方”对方会知道，因为我们前面定义了“老地方”的具体内容。所以说，上下文通常持有交互过程中的状态变量等。会话对象通常较轻，每次请求都重新创建实例，请求结束后销毁。简而言之：把元信息交由实体域持有，把一次请求中的临时状态由会话域持有，由服务域贯穿整个过程。

![ddd](http://dubbo.apache.org/docs/zh-cn/dev/sources/images/ddd.jpg)

- Protocol 是服务域，它是 Invoker 暴露和引用的主功能入口，它负责 Invoker 的生命周期管理。
- Invoker 是实体域，它是 Dubbo 的核心模型，其它模型都向它靠扰，或转换成它，它代表一个可执行体，可向它发起 invoke 调用，它有可能是一个本地的实现，也可能是一个远程的实现，也可能一个集群实现。
- Invocation 是会话域，它持有调用过程中的变量，比如方法名，参数等。

## 线程模型

如果事件处理的逻辑能迅速完成，并且不会发起新的 IO 请求，比如只是在内存中记个标识，则直接在 IO 线程上处理更快，因为减少了线程池调度。

但如果事件处理逻辑较慢，或者需要发起新的 IO 请求，比如需要查询数据库，则必须派发到线程池，否则 IO 线程阻塞，将导致不能接收其它请求。

如果用 IO 线程处理事件，又在事件处理过程中发起新的 IO 请求，比如在连接事件中发起登录请求，会报“可能引发死锁”异常，但不会真死锁。

![dubbo-protocol](https://dubbo.gitbooks.io/dubbo-user-book/sources/images/dubbo-protocol.jpg)

因此，需要通过不同的派发策略和不同的线程池配置的组合来应对不同的场景:

```xml
<dubbo:protocol name="dubbo" dispatcher="all" threadpool="fixed" threads="100" />
```

Dispatcher

- `all` 所有消息都派发到线程池，包括请求，响应，连接事件，断开事件，心跳等。
- `direct` 所有消息都不派发到线程池，全部在 IO 线程上直接执行。
- `message` 只有请求响应消息派发到线程池，其它连接断开事件，心跳等消息，直接在 IO 线程上执行。
- `execution` 只请求消息派发到线程池，不含响应，响应和其它连接断开事件，心跳等消息，直接在 IO 线程上执行。
- `connection` 在 IO 线程上，将连接断开事件放入队列，有序逐个执行，其它消息派发到线程池。

ThreadPool

- `fixed` 固定大小线程池，启动时建立线程，不关闭，一直持有。(缺省)
- `cached` 缓存线程池，空闲一分钟自动删除，需要时重建。
- `limited` 可伸缩线程池，但池中的线程数只会增长不会收缩。只增长不收缩的目的是为了避免收缩时突然来了大流量引起的性能问题。



# 协议

## 协议比较

| 协议名称   | 实现描述                                                     | 连接                                                         | 使用场景                                                     |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| dubbo      | 传输：mina、netty、grizzy  序列化：dubbo、hessian2、java、json | dubbo缺省采用单一长连接和NIO异步通讯                         | 1.传入传出参数数据包较小  2.消费者 比提供者多  3.常规远程服务方法调用  4.不适合传送大数据量的服务，比如文件、传视频 |
| rmi        | 传输：java  rmi  序列化：java 标准序列化                     | 连接个数：多连接  连接方式：短连接  传输协议：TCP/IP  传输方式：BIO | 1.常规RPC调用  2.与原RMI客户端互操作  3.可传文件  4.不支持防火墙穿透 |
| hessian    | 传输：Serverlet容器  序列化：hessian二进制序列化             | 连接个数：多连接     连接方式：短连接     传输协议：HTTP     传输方式：同步传输 | 1.提供者比消费者多  2.可传文件  3.跨语言传输                 |
| http       | 传输：servlet容器  序列化：表单序列化                        | 连接个数：多连接     连接方式：短连接     传输协议：HTTP     传输方式：同步传输 | 1.提供者多余消费者  2.数据包混合                             |
| webservice | 传输：HTTP  序列化：SOAP文件序列化                           | 连接个数：多连接     连接方式：短连接     传输协议：HTTP     传输方式：同步传输 | 1.系统集成  2.跨语言调用                                     |
| thrift     | 与thrift RPC实现集成，并在基础上修改了报文头                 | 长连接、NIO异步传输                                          |                                                              |

 

## 协议的配置

dubbo:protocal>（只需在服务提供方配置即可）

| 属性          | 类型   | 是否必填 | 缺省值                                                       | 描述                                                         |
| ------------- | ------ | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| name          | string | **必填** | dubbo                                                        | 协议名称                                                     |
| port          | int    | 可选     | dubbo协议缺省端口为20880,rmi协议缺省端口为1099，http和hessian协议缺省端口为80；如果配置为-1或者没有配置port,则会分配一个没有被占用的端口。 | 服务端口                                                     |
| threadpool    | string | 可选     | fixed                                                        | 线程池类型，可选：fixed/cached                               |
| threads       | int    | 可选     | 100                                                          | 服务h定大小)                                                 |
| iothreads     | int    | 可选     | CPU个数+1                                                    | io线程池大小（固定）                                         |
| accepts       | int    | 可选     | 0                                                            | 服务提供方最大可接受连接数                                   |
| serialization | string | 可选     | dubbo协议缺省为hessian2,rmi缺省协议为java,http协议缺省为json | 可填序列化：dubbo\|hessian2\|java\|compactedjava\|fastjson   |
| dispatcher    | string | 可选     | dubbo协议缺省为all                                           | 协议的消息派发方式，用于指定线程模型，比如：dubbo协议的all, direct, message,execution, connection等参考：[https://blog.csdn.net/fd2025/article/](https://blog.csdn.net/fd2025/article/details/79985542)[details/79985542](https://blog.csdn.net/fd2025/article/details/79985542) |
| queues        | int    | 可选     | 0                                                            | 线程池队列大小，当线程池满时，排队等待执行的队列大小，建议不要设置，当线程程池时应立即失败，重试其它服务提供机器，而不是排队，除非有特殊需求。 |
| charset       | string | 可选     | UTF-8                                                        | 序列化编码                                                   |
| server        | server | string   | dubbo协议缺省为netty，http协议缺省为servlethessian协议缺省为jetty | 协议的服务器端实现类型，比如：dubbo协议的mina,netty等，http协议的jetty,servlet等 |

```
1    <dubbo:protocol name="hessian" server="jetty"
2                     client="httpclient"
3                     port="20880"
4                     threadpool="fixed"
5                     threads="50"
6                      iothreads="5"
7                     accepts="1000"/>
```

**dubbo多连接配置：**

Dubbo 协议缺省每服务每提供者每消费者使用单一长连接，如果数据量较大，可以使用多个连接。

```
1 <dubbo:service connections="1"/>
2 <dubbo:reference connections="1"/>
```

- `<dubbo:service connections="0">` 或 `<dubbo:reference connections="0">` 表示该服务使用 JVM 共享长连接。**缺省**
- `<dubbo:service connections="1">` 或 `<dubbo:reference connections="1">` 表示该服务使用独立长连接。
- `<dubbo:service connections="2">` 或`<dubbo:reference connections="2">` 表示该服务使用独立两条长连接。

为防止被大量连接撑挂，可在服务提供方限制大接收连接数，以实现服务提供方自我保护。

```
<dubbo:protocol name="dubbo" accepts="1000" />
```

`dubbo.properties` 配置：

```
1 dubbo.service.protocol=dubbo
```

 

# 集群

## 集群容错

在集群调用失败时，Dubbo 提供了多种容错方案，缺省为 failover 重试。



![img](https://dubbo.gitbooks.io/dubbo-user-book/content/sources/images/cluster.jpg)



各节点关系：

- 这里的 `Invoker` 是 `Provider` 的一个可调用 `Service` 的抽象，`Invoker` 封装了 `Provider` 地址及 `Service` 接口信息
- `Directory` 代表多个 `Invoker`，可以把它看成 `List<Invoker>` ，但与 `List` 不同的是，它的值可能是动态变化的，比如注册中心推送变更
- `Cluster` 将 `Directory` 中的多个 `Invoker` 伪装成一个 `Invoker`，对上层透明，伪装过程包含了容错逻辑，调用失败后，重试另一个
- `Router` 负责从多个 `Invoker` 中按路由规则选出子集，比如读写分离，应用隔离等
- `LoadBalance` 负责从多个 `Invoker` 中选出具体的一个用于本次调用，选的过程包含了负载均衡算法，调用失败后，需要重选

### 集群容错模式

可以自行扩展集群容错策略，参见：[集群扩展](https://dubbo.gitbooks.io/dubbo-dev-book/impls/cluster.html)

**Failover Cluster**

失败自动切换，当出现失败，重试其它服务器 [1](https://dubbo.gitbooks.io/dubbo-user-book/content/demos/fault-tolerent-strategy.html#fn_1)。通常用于读操作，但重试会带来更长延迟。可通过 `retries="2"` 来设置重试次数 (不含第一次)。

重试次数配置如下：

```
<dubbo:service retries="2" />
```

或

```
<dubbo:reference retries="2" />
```

或

```
<dubbo:reference>
    <dubbo:method name="findFoo" retries="2" />
</dubbo:reference>
```

**Failfast Cluster**

快速失败，只发起一次调用，失败立即报错。通常用于非幂等性的写操作，比如新增记录。

**Failsafe Cluster**

失败安全，出现异常时，直接忽略。通常用于写入审计日志等操作。

**Failback Cluster**

失败自动恢复，后台记录失败请求，定时重发。通常用于消息通知操作。

**Forking Cluster**

并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，但需要浪费更多服务资源。可通过 `forks="2"` 来设置最大并行数。

**Broadcast Cluster**

广播调用所有提供者，逐个调用，任意一台报错则报错 [2](https://dubbo.gitbooks.io/dubbo-user-book/content/demos/fault-tolerent-strategy.html#fn_2)。通常用于通知所有提供者更新缓存或日志等本地资源信息。

### 集群模式配置

按照以下示例在服务提供方和消费方配置集群模式

```
<dubbo:service cluster="failsafe" />
```

或

```
<dubbo:reference cluster="failsafe" />
```

\1. 该配置为缺省配置[ ↩](https://dubbo.gitbooks.io/dubbo-user-book/content/demos/fault-tolerent-strategy.html#reffn_1)

\2. `2.1.0` 开始支持[ ↩](https://dubbo.gitbooks.io/dubbo-user-book/content/demos/fault-tolerent-strategy.html#reffn_2)

## 负载均衡

在集群负载均衡时，Dubbo 提供了多种均衡策略，默认为 random 随机调用。
负载均衡策略:
1、Random LoadBalance
 随机，按权重设置随机概率。在一个截面上碰撞的概率高，但调用量越大分布越均匀，而且按概率使用权重后也比较均匀，有利于动态调整提供者权重。可以在dubbo管理控制台调整权重。

![å¨è¿éæå¥å¾çæè¿°](https://img-blog.csdnimg.cn/20190429194857668.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpcWlhbmcxMjE3,size_16,color_FFFFFF,t_70)

2、RoundRobin LoadBalance
 轮循，按公约后的权重设置轮循比率。存在慢的提供者累积请求的问题，比如：第二台机器很慢，但没挂，当请求调到第二台时就卡在那，久而久之，所有请求都卡在调到第二台上。

![å¨è¿éæå¥å¾çæè¿°](https://img-blog.csdnimg.cn/20190429194732657.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpcWlhbmcxMjE3,size_16,color_FFFFFF,t_70)

3、LeastActive LoadBalance
 最少活跃调用数，相同活跃数的随机，活跃数指调用前后计数差。使慢的提供者收到更少请求，因为越慢的提供者的调用前后计数差会越大。

![å¨è¿éæå¥å¾çæè¿°](https://img-blog.csdnimg.cn/20190429194925289.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpcWlhbmcxMjE3,size_16,color_FFFFFF,t_70)

4、ConsistentHash LoadBalance
 一致性 Hash，相同参数的请求总是发到同一提供者。当某一台提供者挂时，原本发往该提供者的请求，基于虚拟节点，平摊到其它提供者，不会引起剧烈变动。算法详细参考：http://en.wikipedia.org/wiki/Consistent_hashing 默认只对第一个参数Hash，如果要修改，请配置 <dubbo:parameter key="hash.arguments" value="0,1" />。默认用 160 份虚拟节点，如果要修改，请配置 <dubbo:parameter key="hash.nodes" value="320" />。

![å¨è¿éæå¥å¾çæè¿°](https://img-blog.csdnimg.cn/20190429195001720.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpcWlhbmcxMjE3,size_16,color_FFFFFF,t_70)

---------------------
作者：苍穹尘 
来源：CSDN 
原文：https://blog.csdn.net/lizhiqiang1217/article/details/89682045 
版权声明：本文为博主原创文章，转载请附上博文链接！

# 参考

https://dubbo.gitbooks.io/dubbo-user-book/content/

http://dubbo.apache.org/zh-cn/docs/user/quick-start.html

[史上最强Dubbo面试28题答案详解：核心功能+服务治理+架构设计等](http://youzhixueyuan.com/dubbo-interview-question-answers.html)

[史上最全 40 道 Dubbo 面试题及答案，看完碾压面试官！](https://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247487261&idx=1&sn=d3dd9825515f2f79f5e44def251a8e5c&chksm=eb538a2bdc24033dd8c1ceba703688647fb2d7cdcda9b100fbaf60b65a4114ede7c444d087d6&scene=0&subscene=131&ascene=7&devicetype=android-27&version=26070333&nettype=WIFI&abtest_cookie=AwABAAoACwATAAMAI5ceAFmZHgBhmR4AAAA%3D&lang=zh_CN&pass_ticket=5pKok9efg6CdY8TTkcZ66WPITUrocUC9NfjgorqdXP2OPkCIkGF4eN7TX8UjBOab&wx_header=1)

[Dubbo面试题汇总](https://www.cnblogs.com/yang-lq/p/9168216.html)
