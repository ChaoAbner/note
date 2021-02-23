# ZooKeeper+Curator的使用

`fosu-rpc`使用了 `Zookeeper` 来存储服务的相关信息 ，并且使用的是 ZooKeeper 的Java客户端 `Curator` 来对 `ZooKeeper` 进行增删改查等操作。 所以，本文就简单介绍一下 ~常用命令 以及 `Curator` 的基本使用。

## Curator 

`Curator` 是Netflix公司开源的一套 `ZooKeeper` 的Java客户端框架，相比于 `Zookeeper` 自带的客户端 `zookeeper` 来说，`Curator` 的封装更加完善，各种API都可以比较方便地使用。

下面我们就来简单地演示一下 Curator 的使用吧！ `Curator4.0+`版本对`ZooKeeper 3.5.x`支持比较好。开始之前，请先将下面的依赖添加进你的项目。

```xml
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>4.2.0</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>4.2.0</version>
</dependency>
```

### 连接Zookeeper

通过 `CuratorFrameworkFactory` 创建 `CuratorFramework` 对象，然后再调用 `CuratorFramework` 对象的 `start()` 方法即可！

```java
private static final int BASE_SLEEP_TIME = 1000;
private static final int MAX_RETRIES = 3;

// Retry strategy. Retry 3 times, and will increase the sleep time between retries.
RetryPolicy retryPolicy = new ExponentialBackoffRetry(BASE_SLEEP_TIME, MAX_RETRIES);
CuratorFramework zkClient = CuratorFrameworkFactory.builder()
    // the server to connect to (can be a server list)
    .connectString("127.0.0.1:2181")
    .retryPolicy(retryPolicy)
    .build();
zkClient.start();
```

**一些参数的说明**

- **baseSleepTimeMs** ：重试之间等待的初始时间 
- **maxRetries** ：最大重试次数 
- **connectString** ：要连接的服务器列表 
- **retryPolicy** ：重试策略

### 数据节点的增删改查

ZooKeeper将znode分为四大类：

- 持久节点（PERSISTENT）：一旦创建一直存在即使ZooKeeper集群宕机，直到将其删除
- 临时节点（EPHEMERAL）：**临时节点的生命周期与客户端会话绑定，会话消失则节点消失**，临时节点只能做为叶子节点，不能创建子节点
- 持久顺序节点：：除了具有持久（PERSISTENT）节点的特性之 外， 子节点的名称还具有顺序性。比如 /node1/app0000000001 、 /node1/app0000000002 
- 临时顺序（EPHEMERAL_SEQUENTIAL）节点 ：除了具备临时（EPHEMERAL）节点的特性 之外，子节点的名称还具有顺序性

你在使用的`ZooKeeper`的时候，会发现 `CreateMode` 类中实际有 7种`znode`类型 ，但是用的最多的还是上面介绍的4种。

#### 创建持久化节点

有两种方式创建：

```java
//注意:下面的代码会报错，下文说了具体原因
zkClient.create().forPath("/node1/00001");
zkClient.create().withMode(CreateMode.PERSISTENT).forPath("/node1/00002");
```

上面的代码会报错，因为父节点`node1`还没有创建

可以先创建父节点`node1`，再执行上面的代码就不回报错了

更推荐下面的方法，可以保证父节点不存在也能自动创建父节点。

```java
zkClient.create().creatingParentsIfNeeded().withMode(CreateMode.PERSISTENT).forPath("/node1/00001");
```

#### 创建临时节点

```java
zkClient.create().creatingParentsIfNeeded().withMode(CreateMode.EPHEMERAL).forPath("/node1/00001");
```

#### 创建节点并指定数据内容

```java
zkClient.create().creatingParentsIfNeeded().withMode(CreateMode.EPHEMERAL).forPath("/node1/00001","java".getBytes());
zkClient.getData().forPath("/node1/00001");//获取节点的数据内容，获取到的是 byte数组
```

