# 分类

Dubbo提供了多种注册中心，分别如下：

- Multicast 注册中心
- Zookeeper 注册中心
- Redis 注册中心
- Simple 注册中心



# 注册中心

在当前微服务架构中，注册中心是不可缺少的基础服务之一。

随着应用程序的不断庞大，越来越多的业务加入到我们的应用中，不同的应用之间相互依赖，逻辑混乱，从而为了解决当前系统的复杂性和服务解耦的问题，我们需要将一些常用的服务或者公有服务抽离出来，放在注册中心，即成为一个服务提供者，将自己的服务供给其他需要调用的消费者去使用。



## 结构

注册中心主要涉及三个角色：

1. 服务提供者
2. 服务消费者
3. 注册中心

他们之间的关系如下图

![image-20201106151538555](http://img.fosuchao.com/image-20201106151538555.png)



## 工作流程

1. 在各个微服务启动时，会将自己的网络地址等信息注册到注册中心，包括调用的接口等信息，都保存在注册中心。

2. 服务消费者订阅注册中心，即可以从注册中心获取所有服务的地址，并且通过这些服务的地址通过RPC去调用相应的接口服务。

   > 注册中心会通过负载均衡算法合理分配调用请求，即服务治理。

3. 注册中心会定时的想服务提供者发送心跳包，确保这些服务接口的可靠性，如果某个服务长时间无法通信，那么则会被抛弃。

4. 当服务的地址发生变化的时候，会重新注册到注册中心，然后服务消费者通过查询注册中心直接更新，而不用人工修改。



介绍了一下注册中心的作用和工作流程，下面我们来看看Dubbo的注册中心吧。

# Multicast注册中心

Multicast 注册中心不需要启动任何中心节点，只要广播地址一样，就可以互相发现。

![/user-guide/images/multicast.jpg](http://img.fosuchao.com/multicast.jpg)

1. 提供方启动时广播自己的地址
2. 消费方启动时广播订阅请求
3. 提供方收到订阅请求时，单播自己的地址给订阅者，如果设置了 `unicast=false`，则广播给订阅者
4. 消费方收到提供方地址时，连接该地址进行 RPC 调用。

组播受网络结构限制，只适合小规模应用或开发阶段使用。组播地址段: 224.0.0.0 - 239.255.255.255

## 配置

```xml
<dubbo:registry address="multicast://224.5.6.7:1234" />
```

或

```xml
<dubbo:registry protocol="multicast" address="224.5.6.7:1234" />
```

为了减少广播量，Dubbo 缺省使用单播发送提供者地址信息给消费者，如果一个机器上同时启了多个消费者进程，消费者需声明 `unicast=false`，否则只会有一个消费者能收到消息;当服务者和消费者运行在同一台机器上，消费者同样需要声明`unicast=false`，否则消费者无法收到消息，导致No provider available for the service异常：

```xml
<dubbo:registry address="multicast://224.5.6.7:1234?unicast=false" />
```

或

```xml
<dubbo:registry protocol="multicast" address="224.5.6.7:1234">
    <dubbo:parameter key="unicast" value="false" />
</dubbo:registry>
```



# Zookeeper注册中心

[Zookeeper](http://zookeeper.apache.org/) 是 Apache Hadoop 的子项目，是一个树型的目录服务，支持变更推送，适合作为 Dubbo 服务的注册中心，工业强度较高，可用于生产环境，并推荐使用 [1](https://dubbo.apache.org/zh-cn/docs/2.7/user/references/registry/zookeeper/#fn:1)。

![/user-guide/images/zookeeper.jpg](http://img.fosuchao.com/zookeeper.jpg)

流程说明：

- 服务提供者启动时: 向 `/dubbo/com.foo.BarService/providers` 目录下写入自己的 URL 地址
- 服务消费者启动时: 订阅 `/dubbo/com.foo.BarService/providers` 目录下的提供者 URL 地址。并向 `/dubbo/com.foo.BarService/consumers` 目录下写入自己的 URL 地址
- 监控中心启动时: 订阅 `/dubbo/com.foo.BarService` 目录下的所有提供者和消费者 URL 地址。

支持以下功能：

- 当提供者出现断电等异常停机时，注册中心能自动删除提供者信息
- 当注册中心重启时，能自动恢复注册数据，以及订阅请求
- 当会话过期时，能自动恢复注册数据，以及订阅请求
- 当设置 `<dubbo:registry check="false" />` 时，记录失败注册和订阅请求，后台定时重试
- 可通过 `<dubbo:registry username="admin" password="1234" />` 设置 zookeeper 登录信息
- 可通过 `<dubbo:registry group="dubbo" />` 设置 zookeeper 的根节点，不配置将使用默认的根节点。
- 支持 `*` 号通配符 `<dubbo:reference group="*" version="*" />`，可订阅服务的所有分组和所有版本的提供者



## 使用

在 provider 和 consumer 中增加 zookeeper 客户端 jar 包依赖：

```xml
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.3.3</version>
</dependency>
```

或直接[下载](http://repo1.maven.org/maven2/org/apache/zookeeper/zookeeper)。

Dubbo 支持 zkclient 和 curator 两种 Zookeeper 客户端实现：

**注意:在2.7.x的版本中已经移除了zkclient的实现,如果要使用zkclient客户端,需要自行拓展**

### 使用 zkclient 客户端

从 `2.2.0` 版本开始缺省为 zkclient 实现，以提升 zookeeper 客户端的健壮性。[zkclient](https://github.com/sgroschupf/zkclient) 是 Datameer 开源的一个 Zookeeper 客户端实现。

缺省配置：

```xml
<dubbo:registry ... client="zkclient" />
```

或：

```properties
dubbo.registry.client=zkclient
```

或：

```
zookeeper://10.20.153.10:2181?client=zkclient
```

需依赖或直接[下载](http://repo1.maven.org/maven2/com/github/sgroschupf/zkclient)：

```xml
<dependency>
    <groupId>com.github.sgroschupf</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.1</version>
</dependency>
```



### 使用 curator 客户端

从 `2.3.0` 版本开始支持可选 curator 实现。[Curator](https://github.com/Netflix/curator) 是 Netflix 开源的一个 Zookeeper 客户端实现。

如果需要改为 curator 实现，请配置：

```xml
<dubbo:registry ... client="curator" />
```

或：

```properties
dubbo.registry.client=curator
```

或：

```xml
zookeeper://10.20.153.10:2181?client=curator
```

需依赖或直接[下载](http://repo1.maven.org/maven2/com/netflix/curator/curator-framework)：

```xml
<dependency>
    <groupId>com.netflix.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>1.1.10</version>
</dependency>
```

Zookeeper 单机配置:

```xml
<dubbo:registry address="zookeeper://10.20.153.10:2181" />
```

或：

```xml
<dubbo:registry protocol="zookeeper" address="10.20.153.10:2181" />
```

Zookeeper 集群配置：

```xml
<dubbo:registry address="zookeeper://10.20.153.10:2181?backup=10.20.153.11:2181,10.20.153.12:2181" />
```

或：

```xml
<dubbo:registry protocol="zookeeper" address="10.20.153.10:2181,10.20.153.11:2181,10.20.153.12:2181" />
```

同一 Zookeeper，分成多组注册中心:

```xml
<dubbo:registry id="chinaRegistry" protocol="zookeeper" address="10.20.153.10:2181" group="china" />
<dubbo:registry id="intlRegistry" protocol="zookeeper" address="10.20.153.10:2181" group="intl" />
```



## 安装

安装方式参见: [Zookeeper安装手册](https://dubbo.apache.org/zh-cn/docs/2.7/admin/install/zookeeper/)，只需搭一个原生的 Zookeeper 服务器，并将 [Quick Start](https://dubbo.apache.org/zh-cn/docs/2.7/user/preface/usage/) 中 Provider 和 Consumer 里的 `conf/dubbo.properties` 中的 `dubbo.registry.address` 的值改为 `zookeeper://127.0.0.1:2181` 即可使用。



# Redis注册中心

基于 Redis 实现的注册中心。

> Redis 过期数据通过心跳的方式检测脏数据，服务器时间必须同步，并且对服务器有一定压力，否则过期检测会不准确

![/user-guide/images/dubbo-redis-registry.jpg](http://img.fosuchao.com/dubbo-redis-registry.jpg)

使用 Redis 的 Key/Map 结构存储数据结构：

- 主 Key 为服务名和类型
- Map 中的 Key 为 URL 地址
- Map 中的 Value 为过期时间，用于判断脏数据，脏数据由监控中心删除

使用 Redis 的 Publish/Subscribe 事件通知数据变更：

- 通过事件的值区分事件类型：`register`, `unregister`, `subscribe`, `unsubscribe`
- 普通消费者直接订阅指定服务提供者的 Key，只会收到指定服务的 `register`, `unregister` 事件
- 监控中心通过 `psubscribe` 功能订阅 `/dubbo/*`，会收到所有服务的所有变更事件

调用过程：

1. 服务提供方启动时，向 `Key:/dubbo/com.foo.BarService/providers` 下，添加当前提供者的地址
2. 并向 `Channel:/dubbo/com.foo.BarService/providers` 发送 `register` 事件
3. 服务消费方启动时，从 `Channel:/dubbo/com.foo.BarService/providers` 订阅 `register` 和 `unregister` 事件
4. 并向 `Key:/dubbo/com.foo.BarService/consumers` 下，添加当前消费者的地址
5. 服务消费方收到 `register` 和 `unregister` 事件后，从 `Key:/dubbo/com.foo.BarService/providers` 下获取提供者地址列表
6. 服务监控中心启动时，从 `Channel:/dubbo/*` 订阅 `register` 和 `unregister`，以及 `subscribe` 和`unsubsribe`事件
7. 服务监控中心收到 `register` 和 `unregister` 事件后，从 `Key:/dubbo/com.foo.BarService/providers` 下获取提供者地址列表
8. 服务监控中心收到 `subscribe` 和 `unsubsribe` 事件后，从 `Key:/dubbo/com.foo.BarService/consumers` 下获取消费者地址列表



## 配置

```xml
<dubbo:registry address="redis://10.20.153.10:6379" />
```

或

```xml
<dubbo:registry address="redis://10.20.153.10:6379?backup=10.20.153.11:6379,10.20.153.12:6379" />
```

或

```xml
<dubbo:registry protocol="redis" address="10.20.153.10:6379" />
```

或

```xml
<dubbo:registry protocol="redis" address="10.20.153.10:6379,10.20.153.11:6379,10.20.153.12:6379" />
```



## 选项

- 可通过 `<dubbo:registry group="dubbo" />` 设置 redis 中 key 的前缀，缺省为 `dubbo`。
- 可通过`<dubbo:registry cluster="replicate" />` 设置 redis 集群策略，缺省为`failover`：
  - `failover`: 只写入和读取任意一台，失败时重试另一台，需要服务器端自行配置数据同步
  - `replicate`: 在客户端同时写入所有服务器，只读取单台，服务器端不需要同步，注册中心集群增大，性能压力也会更大



## 安装

安装方式参见: [Redis安装手册](https://dubbo.apache.org/zh-cn/docs/2.7/admin/install/redis/)，只需搭一个原生的 Redis 服务器，并将 [Quick Start](https://dubbo.apache.org/zh-cn/docs/2.7/user/preface/usage/) 中 Provider 和 Consumer 里的 `conf/dubbo.properties` 中的 `dubbo.registry.address` 的值改为 `redis://127.0.0.1:6379` 即可使用



# Simple注册中心

Simple 注册中心本身就是一个普通的 Dubbo 服务，可以减少第三方依赖，使整体通讯方式一致。



## 配置

将 Simple 注册中心暴露成 Dubbo 服务：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    <!-- 当前应用信息配置 -->
    <dubbo:application name="simple-registry" />
    <!-- 暴露服务协议配置 -->
    <dubbo:protocol port="9090" />
    <!-- 暴露服务配置 -->
    <dubbo:service interface="org.apache.dubbo.registry.RegistryService" ref="registryService" registry="N/A" ondisconnect="disconnect" callbacks="1000">
        <dubbo:method name="subscribe"><dubbo:argument index="1" callback="true" /></dubbo:method>
        <dubbo:method name="unsubscribe"><dubbo:argument index="1" callback="false" /></dubbo:method>
    </dubbo:service>
    <!-- 简单注册中心实现，可自行扩展实现集群和状态同步 -->
    <bean id="registryService" class="org.apache.dubbo.registry.simple.SimpleRegistryService" />
</beans>
```

引用 Simple Registry 服务：

```xml
<dubbo:registry address="127.0.0.1:9090" />
```

或者：

```xml
<dubbo:service interface="org.apache.dubbo.registry.RegistryService" group="simple" version="1.0.0" ... >
```

或者：

```xml
<dubbo:registry address="127.0.0.1:9090" group="simple" version="1.0.0" />
```



## 适用性说明

此 `SimpleRegistryService` 只是简单实现，不支持集群，可作为自定义注册中心的参考，但不适合直接用于生产环境。