## IdleStateHandler心跳机制

在服务端pipeline中添加`IdleStateHandler`处理器，可以对客户端进行心跳监测， 当客户端连接长时间不活跃的时候，我们可以选择关闭连接，释放资源。

### 使用

[代码连接](https://github.com/ChaoAbner/netty-example/tree/master/src/main/java/com/fosuchao/heartbeat)

`IdleStateHandler`的构造函数有多个，下图的四个参数分别是**读空闲监测时间，写空闲监测时间，读写空闲监测时间**，**时间单位**。如果不传入，默认为0，传入0的话默认不进行空闲检测。

![1581666600116](F:\typoraImg\1581666600116.png)

![1581666622270](F:\typoraImg\1581666622270.png)

​		在我们的自定义处理器中，继承`ChanneInboundHandlerAdapter`类，重写`userEventTriggered`方法，当连接监测为空闲状态的时候，这个方法会被触发，所以我们可以在这个方法中添加自己的逻辑。来处理这种空闲的情况。

### 原理

​		在`IdleStateHandler`类中的channelActive方法中，添加了自己的`initialize`方法。

![1581666908209](F:\typoraImg\1581666908209.png)

​		在`initialize`方法中，先判断有没有传入超时时间，如果大于0，定时任务添加到对应线程`EventLoopExecutor`对应的任务队列`taskQueue`中，在对应线程的run()方法中循环执行。

![1581667008765](F:\typoraImg\1581667008765.png)

`ReaderIdleTimeoutTask`作为一个`Runnable`传入线程定时执行。

![1581667311452](F:\typoraImg\1581667311452.png)

### 总结

`IdleStateHandler`心跳检测主要是通过向线程任务队列中添加定时任务，判断`channelRead`()方法或`write`()方法是否调用空闲超时，如果超时则触发超时事件执行自定义`userEventTrigger`()方法；

Netty通过`IdleStateHandler`实现最常见的心跳机制不是一种双向心跳的PING-PONG模式，而是客户端发送心跳数据包，服务端接收心跳但不回复，因为如果服务端同时有上千个连接，心跳的回复需要消耗大量网络资源；如果服务端一段时间内内有收到客户端的心跳数据包则认为客户端已经下线，将通道关闭避免资源的浪费；在这种心跳模式下服务端可以感知客户端的存活情况，无论是宕机的正常下线还是网络问题的非正常下线，服务端都能感知到，而客户端不能感知到服务端的非正常下线；

要想实现客户端感知服务端的存活情况，需要进行双向的心跳；Netty中的`channelInactive`()方法是通过Socket连接关闭时挥手数据包触发的，因此可以通过`channelInactive`()方法感知正常的下线情况，但是因为网络异常等非正常下线则无法感知；

