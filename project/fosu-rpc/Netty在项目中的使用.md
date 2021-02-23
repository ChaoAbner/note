# Netty在项目中的使用

## Netty介绍

1. Netty是基于NIO的网络通信框架，可以快速开发网络应用程序
2. 极大简化了TCP和UDP套接字的网络编程，性能和安全性都更好
3. 支持多种协议比如FTP，HTTP，SMTP，还有多种二进制和基于文本的协议

## Netty特点

- 统一的 API，支持多种传输类型，阻塞和非阻塞的。
- 简单而强大的线程模型。 
- **自带编解码器解决 TCP 粘包/拆包问题**。 自带各种协议栈。 
- 真正的无连接数据包套接字支持。 
- 比直接使用 Java 核心 API 有更高的吞吐量、更低的延迟、更低的资源消耗和更少的内存复 制。 
- 安全性不错，有完整的 SSL/TLS 以及 StartTLS 支持。 
- 社区活跃 成熟稳定，经历了大型项目的使用和考验，而且很多开源项目都使用到了 Netty 比如我们经 常接触的 Dubbo、RocketMQ 等等。 
- .....

## Netty实现RPC实战

### 传输实体

我们定义了RPC请求和RPC响应的传输实体：

**RpcRequest**

```java
public class RpcRequest implements Serializable {

    private static final long serialVersionUID = 567705521652488183L;

    /**
     * request ID
     */
    private String requestId;

    /**
     * 调用的接口名
     */
    private String interfaceName;

    /**
     * 方法名
     */
    private String methodName;

    /**
     * 参数
     */
    private Object[] parameters;

    /**
     * 参数类型
     */
    private Class<?>[] paramTypes;

    /**
     * 接口版本
     */
    private String version;

    /**
     * 组名
     */
    private String group;

}
```

**RpcResponse**

```java
public class RpcResponse<T> implements Serializable {

    private static final long serialVersionUID = -7914756332224655031L;

    /**
     * request ID
     */
    private String requestId;

    /**
     * 响应码
     */
    private Integer code;

    /**
     * 响应消息
     */
    private String message;

    /**
     * 返回数据
     */
    private T data;

}

```

### 客户端

客户端有一个向服务端发送消息的`sendMessage()`方法，通过这个方法可以将消息就是`RpcRequest`对象发送到服务端，并且可以同步的获取到五负端返回的结果也就是`RpcResponse`对象。

```java
public class NettyRpcClient implements RpcRequestTransport {

    private final Bootstrap bootstrap;
    private final EventLoopGroup eventLoopGroup;
    private final UnprocessedRequests unprocessedRequests;
    private final ChannelProvider channelProvider;
    private final ServiceDiscovery serviceDiscovery;

    public NettyRpcClient() {
        eventLoopGroup = new NioEventLoopGroup();
        bootstrap = new Bootstrap();
        bootstrap.group(eventLoopGroup)
                .handler(new LoggingHandler(LogLevel.INFO))
                .channel(NioSocketChannel.class)
                // 5s超时时间
                .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 5000)
                .handler(new ChannelInitializer<SocketChannel>() {

                    @Override
                    protected void initChannel(SocketChannel ch) throws Exception {
                        ChannelPipeline p = ch.pipeline();
                        p.addLast(new IdleStateHandler(0, 5, 0, TimeUnit.SECONDS));
                        p.addLast(new RpcMessageDecoder());
                        p.addLast(new RpcMessageEncoder());
                        p.addLast(new NettyRpcClientHandler());
                    }
                });
        this.serviceDiscovery = ExtensionLoader.getExtensionLoader(ZkServiceDiscovery.class).getExtension("zk");
        this.channelProvider = SingletonFactory.getInstance(ChannelProvider.class);
        this.unprocessedRequests = SingletonFactory.getInstance(UnprocessedRequests.class);
    }

    /**
     * 连接Server，并且返回Channel，用于发rpc message给Server
     * @param inetSocketAddress     Server地址
     * @return                      Channel
     */
    @SneakyThrows
    public Channel doConnect(InetSocketAddress inetSocketAddress) {
        CompletableFuture<Channel> completableFuture = new CompletableFuture<>();
        bootstrap.connect(inetSocketAddress).addListener((ChannelFutureListener) future -> {
            if (future.isSuccess()) {
                log.info("client连接：[{}] 成功",inetSocketAddress.toString());
            } else {
                throw new IllegalArgumentException();
            }
        });
        return completableFuture.get();
    }

    @Override
    public Object sendRpcRequest(RpcRequest rpcRequest) {
        // 异步响应
        CompletableFuture<RpcResponse<Object>> resultFuture = new CompletableFuture<>();
        String rpcServiceName = rpcRequest.toRpcProperties().toRpcServiceName();
        // 服务发现，选择请求的服务器
        InetSocketAddress inetSocketAddress = serviceDiscovery.lookupService(rpcServiceName);
        Channel channel = getChannel(inetSocketAddress);
        if (channel.isActive()) {
            unprocessedRequests.put(rpcRequest.getRequestId(), resultFuture);
            RpcMessage rpcMessage = new RpcMessage();
            rpcMessage.setData(rpcMessage);
            rpcMessage.setCodec(SerializationTypeEnum.PROTOSTUFF.getCode());
            rpcMessage.setCompressType(CompressTypeEnum.GZIP.getCode());
            rpcMessage.setMessageType(RpcConstant.REQUEST_TYPE);
            channel.writeAndFlush(rpcMessage).addListener((ChannelFutureListener) future -> {
                if (future.isSuccess()) {
                    log.info("客户端发送消息：[{}]", rpcMessage);
                } else {
                    future.channel().close();
                    resultFuture.completeExceptionally(future.cause());
                    log.error("客户端发送消息失败：", future.cause());
                }
            });
        } else {
            throw new IllegalArgumentException();
        }
        return resultFuture;
    }

    public Channel getChannel(InetSocketAddress inetSocketAddress) {
        Channel channel = channelProvider.get(inetSocketAddress);
        if (channel == null) {
            channel = doConnect(inetSocketAddress);
            channelProvider.set(inetSocketAddress, channel);
        }
        return channel;
    }

    public void close() {
        eventLoopGroup.shutdownGracefully();
    }
}

```

sendMessage() 方法分析：

1. 首先初始化了一个 Bootstrap 
2. 通过 Bootstrap 对象连接服务端 
3. 通过 Channel 向服务端发送消息 RpcRequest 
4. 发送成功后，阻塞等待 ，直到 Channel 关闭 
5. 拿到服务端返回的结果 RpcResponse

#### 自定义 ChannelHandler 处理服务端消息

```java
public class NettyRpcClientHandler extends ChannelInboundHandlerAdapter {
    private final UnprocessedRequests unprocessedRequests;
    private final NettyRpcClient nettyRpcClient;

    public NettyRpcClientHandler() {
        this.unprocessedRequests = SingletonFactory.getInstance(UnprocessedRequests.class);
        this.nettyRpcClient = SingletonFactory.getInstance(NettyRpcClient.class);
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {

        try {
            log.info("客户端收到消息：[{}]", msg);
            if (msg instanceof RpcMessage) {
                RpcMessage rpcMessage = (RpcMessage) msg;
                byte messageType = rpcMessage.getMessageType();
                if (messageType == RpcConstant.HEARTBEAT_RESPONSE_TYPE) {
                    log.info("heart [{}]", rpcMessage.getData());
                } else {
                    RpcResponse<Object> rpcResponse = (RpcResponse<Object>) rpcMessage.getData();
                    unprocessedRequests.complete(rpcResponse);
                }
            }
        } finally {
            ReferenceCountUtil.release(msg);
        }

        super.channelRead(ctx, msg);
    }

    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
        if (evt instanceof IdleStateEvent) {
            IdleState state = ((IdleStateEvent) evt).state();
            if (state == IdleState.WRITER_IDLE) {
                log.info("write idle happen [{}]", ctx.channel().remoteAddress());
                Channel channel = nettyRpcClient.getChannel((InetSocketAddress) ctx.channel().remoteAddress());
                RpcMessage rpcMessage = new RpcMessage();
                rpcMessage.setCodec(SerializationTypeEnum.PROTOSTUFF.getCode());
                rpcMessage.setCompressType(CompressTypeEnum.GZIP.getCode());
                rpcMessage.setMessageType(RpcConstant.HEARTBEAT_REQUEST_TYPE);
                rpcMessage.setData(RpcConstant.PING);
                channel.writeAndFlush(rpcMessage).addListener(ChannelFutureListener.CLOSE_ON_FAILURE);
            }
        } else {
            super.userEventTriggered(ctx, evt);
        }
    }

    /**
     * Called when an exception occurs in processing a client message
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        log.error("client catch exception：", cause);
        cause.printStackTrace();
        ctx.close();
    }
}

```

NettyClientHandler 用于读取服务端发送过来的 RpcResponse 消息对象，并将 RpcResponse 消息对象保存到 AttributeMap 上 ， AttributeMap 可以看作是一个 Channel 的共享数据源。 

这样的话，我们就能通过 channel 和 key 将数据读取出来。

### 服务端

**初始化服务端**

`NettyServer`的作用就是开启一个服务器接受客户端的请求并处理。

```java
public class NettyRpcServer {

    public final static int PORT = 9999;

    private final ServiceProvider serviceProvider = SingletonFactory.getInstance(ServiceProviderImpl.class);

    public void registerService(Object service, RpcServiceProperties rpcServiceProperties) {
        serviceProvider.publishService(service, rpcServiceProperties);
    }

    @SneakyThrows
    public void start() {
        CustomShutdownHook.getCustomShutdownHook().clearAll();
        String host = InetAddress.getLocalHost().getHostAddress();
        NioEventLoopGroup bossGroup = new NioEventLoopGroup(1);
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();
        DefaultEventExecutorGroup serviceHandlerGroup = new DefaultEventExecutorGroup(RuntimeUtil.getCpus() * 2,
                ThreadPoolFactoryUtil.createThreadFactory("service-handler-group", false));

        try {
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    // TCP默认开启了 Nagle 算法，该算法的作用是尽可能的发送大数据快，减少网络传输
                    // TCP_NODELAY 参数的作用就是控制是否启用 Nagle 算法
                    .childOption(ChannelOption.TCP_NODELAY, true)
                    // 开启TCP心跳机制
                    .childOption(ChannelOption.SO_KEEPALIVE, true)
                    //表示系统用于临时存放已完成三次握手的请求的队列的最大长度
                    // 如果连接建立频繁，服务器处理创建新连接较慢，可以适当调大这个参数
                    .option(ChannelOption.SO_BACKLOG, 128)
                    .handler(new LoggingHandler(LogLevel.INFO))
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) {
                            ChannelPipeline p = ch.pipeline();
                            // 30 秒之内没有收到客户端请求的话就关闭连接
                            p.addLast(new IdleStateHandler(30, 0, 0, TimeUnit.SECONDS));
                            p.addLast(new RpcMessageDecoder());
                            p.addLast(new RpcMessageEncoder());
                            p.addLast(serviceHandlerGroup, new NettyRpcServerHandler());
                        }
                    });
            ChannelFuture f = serverBootstrap.bind(host, PORT).sync();
            f.channel().closeFuture().sync();
        } catch (InterruptedException e) {
            log.error("netty服务器启动失败");
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
            serviceHandlerGroup.shutdownGracefully();
        }
    }
}
```

#### 自定义ChannelHandler处理客户端消息

`NettyServerHandler` 用于介绍客户端发送过来的消息并返回结果给客户端。

```java
public class NettyRpcServerHandler extends ChannelInboundHandlerAdapter {

    private final RpcRequestHandler rpcRequestHandler;

    public NettyRpcServerHandler() {
        this.rpcRequestHandler = SingletonFactory.getInstance(RpcRequestHandler.class);
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        if (msg instanceof RpcMessage) {
            log.info("server接收到了msg：[{}]", msg);
            byte messageType = ((RpcMessage) msg).getMessageType();
            RpcMessage rpcMessage = new RpcMessage();
            rpcMessage.setCodec(SerializationTypeEnum.PROTOSTUFF.getCode());
            rpcMessage.setCompressType(CompressTypeEnum.GZIP.getCode());
            if (messageType == RpcConstant.HEARTBEAT_REQUEST_TYPE) {
                rpcMessage.setMessageType(RpcConstant.HEARTBEAT_RESPONSE_TYPE);
                rpcMessage.setData(RpcConstant.PONG);
            } else {
                rpcMessage.setMessageType(RpcConstant.RESPONSE_TYPE);
                RpcRequest rpcRequest = (RpcRequest) ((RpcMessage) msg).getData();
                // 调用方法
                Object result = rpcRequestHandler.handle(rpcRequest);
                log.info(String.format("server 执行结果: %s", result.toString()));
                if (ctx.channel().isActive() && ctx.channel().isWritable()) {
                    RpcResponse<Object> rpcResponse = RpcResponse.success(result, rpcRequest.getRequestId());
                    rpcMessage.setData(rpcResponse);
                } else {
                    RpcResponse<Object> rpcResponse = RpcResponse.fail(RpcResponseCodeEnum.FAIL);
                    rpcMessage.setData(rpcResponse);
                }
            }
            ctx.writeAndFlush(rpcMessage);
        }
    }
}
```

## 编码器

#### RpcMessage定义

![image-20210220202856000](http://img.fosuchao.com/image-20210220202856000.png)

#### 自定义编码器

`RpcMessageEncoder`是我们自定义的编码器。它负责处理"**出站**消息，将消息格式转换为字节数组然后写入到字节数据的容器 ByteBuf 对象中。

```java
public class RpcMessageEncoder extends MessageToByteEncoder<RpcMessage> {

    private static final AtomicInteger ATOMIC_INTEGER = new AtomicInteger(0);

    @Override
    protected void encode(ChannelHandlerContext ctx, RpcMessage msg, ByteBuf out) throws Exception {
        try {
            // 写魔数
            out.writeBytes(RpcConstant.MAGIC_NUMBER);
            // 版本
            out.writeByte(RpcConstant.VERSION);
            // 跳过消息长度的四个字节
            out.writerIndex(out.writerIndex() + 4);
            // 消息类型
            byte messageType = msg.getMessageType();
            out.writeByte(messageType);
            // 编码
            out.writeByte(msg.getCodec());
            // 压缩类型
            out.writeByte(CompressTypeEnum.GZIP.getCode());
            out.writeInt(ATOMIC_INTEGER.getAndIncrement());
            // 消息长度计算
            byte[] bodyBytes = null;
            int fullLength = RpcConstant.HEADER_LENGTH;
            if (messageType != RpcConstant.HEARTBEAT_REQUEST_TYPE && messageType != RpcConstant.HEARTBEAT_RESPONSE_TYPE) {
                // 如果消息类型不是心跳，那么fullLength = 头长度 + body长度
                String codecName = SerializationTypeEnum.getName(msg.getCodec());
                log.info("codec name: [{}]", codecName);
                // 序列化
                Serializer serializer = ExtensionLoader.getExtensionLoader(Serializer.class).getExtension(codecName);
                bodyBytes = serializer.serialize(msg);
                // 压缩
                String compressName = CompressTypeEnum.getName(msg.getCompressType());
                Compress compressor = ExtensionLoader.getExtensionLoader(Compress.class).getExtension(compressName);
                bodyBytes = compressor.compress(bodyBytes);
                fullLength += bodyBytes.length;
            }
            if (bodyBytes != null) {
                out.writeBytes(bodyBytes);
            }
            int writeIndex = out.writerIndex();
            // 写入消息长度
            out.writerIndex(writeIndex - fullLength + RpcConstant.MAGIC_NUMBER.length + 1);
            out.writeInt(fullLength);
            // 恢复之前的writeIndex
            out.writerIndex(writeIndex);
        } catch (Exception e) {
            log.error("编码失败", e);
        }
    }
}
```

#### 自定义解码器

`RpcMessageDecoder`是自定义的解码器，用于处理**入站**的消息，

```java
public class RpcMessageDecoder extends LengthFieldBasedFrameDecoder {

    public RpcMessageDecoder() {
        // lengthFieldOffset: magic code is 4B, and version is 1B, and then full length. so value is 5
        // lengthFieldLength: full length is 4B. so value is 4
        // lengthAdjustment: full length include all data and read 9 bytes before, so the left length is (fullLength-9). so values is -9
        // initialBytesToStrip: we will check magic code and version manually, so do not strip any bytes. so values is 0
        this(RpcConstant.MAX_FRAME_LENGTH, 5, 4, -9, 0);
    }

    public RpcMessageDecoder(int maxFrameLength, int lengthFieldOffset, int lengthFieldLength,
                             int lengthAdjustment, int initialBytesToStrip) {
        super(maxFrameLength, lengthFieldOffset, lengthFieldLength, lengthAdjustment, initialBytesToStrip);
    }

    @Override
    protected Object decode(ChannelHandlerContext ctx, ByteBuf in) throws Exception {
        Object decoded = super.decode(ctx, in);
        if (decoded instanceof ByteBuf) {
            ByteBuf buf = (ByteBuf) decoded;
            if (buf.readableBytes() >= RpcConstant.MAX_FRAME_LENGTH) {
                try {
                    return decodeFrame(buf);
                } catch (Exception e) {
                    log.error("解码错误", e);
                } finally {
                    buf.release();
                }
            }
        }
        return decoded;
    }

    private Object decodeFrame(ByteBuf in) {
        int magicLen = RpcConstant.MAGIC_NUMBER.length;
        byte[] tmp = new byte[magicLen];
        in.readBytes(tmp);
        // 校验魔数
        for (int i = 0; i < magicLen; i++) {
            if (tmp[i] != RpcConstant.MAGIC_NUMBER[i]) {
                throw new IllegalArgumentException("错误的魔数");
            }
        }
        // 读取版本
        byte version = in.readByte();
        if (version != RpcConstant.VERSION) {
            throw new IllegalArgumentException("错误的版本");
        }
        // 读取消息长度
        int fullLength = in.readInt();
        // 读取其他属性
        byte messageType = in.readByte();
        byte codecType = in.readByte();
        byte compressType = in.readByte();
        int requestId = in.readInt();
        // build RpcMessage
        RpcMessage rpcMessage = RpcMessage.builder()
                .codec(codecType)
                .compressType(compressType)
                .messageType(messageType)
                .requestId(requestId).build();
        if (messageType == RpcConstant.HEARTBEAT_REQUEST_TYPE) {
            rpcMessage.setData(RpcConstant.PING);
        } else if (messageType == RpcConstant.HEARTBEAT_RESPONSE_TYPE) {
            rpcMessage.setData(RpcConstant.PONG);
        } else {
            int bodyLength = fullLength - RpcConstant.HEADER_LENGTH;
            if (bodyLength > 0) {
                byte[] bs = new byte[bodyLength];
                in.readBytes(bs);
                // 解压缩
                String compressName = CompressTypeEnum.getName(compressType);
                Compress compressor = ExtensionLoader.getExtensionLoader(Compress.class).getExtension(compressName);
                bs = compressor.decompress(bs);
                // 反序列化
                String codecName = SerializationTypeEnum.getName(codecType);
                Serializer serializer = ExtensionLoader.getExtensionLoader(Serializer.class).getExtension(codecName);
                if (messageType == RpcConstant.REQUEST_TYPE) {
                    RpcRequest tmpValue = serializer.deserialize(bs, RpcRequest.class);
                    rpcMessage.setData(tmpValue);
                } else if (messageType == RpcConstant.RESPONSE_TYPE) {
                    RpcResponse tmpValue = serializer.deserialize(bs, RpcResponse.class);
                    rpcMessage.setData(tmpValue);
                }
            }
        }
        return rpcMessage;
    }

}
```