## 常见的RPC框架介绍

我们这里说的 RPC 框架指的是可以让客户端直接调用服务端方法，就像调用本地方法一样简单的，比较出名的RPC框架大概有Dubbo、Motan、gRPC这些。

如果需要和 HTTP 协议打交道，解析和封装 HTTP 请求和响应。这类框架并不能算是“RPC 框架”，比如Feign。

## Dubbo

Apache Dubbo 是一款高性能、轻量级的开源 Java RPC 框架，它提供了三 大核心能力： 

1. 面向接口的远程方法调用 

2. 智能容错和负载均衡 

3. 服务自动注册和发现。 

   > 简单来说 Dubbo 是一个分布式服务框架，致力于提供高性能和透明化的 RPC 远程服务调用方案，以及 SOA 服务治理方案。

参考文档： 

1. Github ：https://github.com/apache/incubator-dubbo 
2. 官网：https://dubbo.apache.org/zh-cn/  

## GRPC

gRPC是Google 开源的一个高性能、通用的开源 RPC 框架。其由主要面向移动应用开发并**基于 HTTP/2 协议标准而设计**，基于**ProtoBuf序列化协议**开发，并且支持众多开发语言。通过 ProtoBuf 定义接口和数据类型还挺繁琐的，虽然 gRPC 确实很多亮点的地方，但是我还是选 择 Dubbo。 

如果要进一步学习的学习的话，这里有参考文档： 

1. Github：https://github.com/grpc/grpc  
2. 官网：https://grpc.io/  

## Thrift 

Thrift是Facebook开源的跨语言的 RPC 通信框架，目前已经捐献给Apache基金会管理，由于其跨语言特性和出色的性能，在很多互联网公司得到应用，有能力的公司甚至会基于 thrift 研发一套分布式服务框架，增加诸如服务注册、服务发现等功能。 Thrift 支持多种不同的编程语言，包括 C++ 、Java 、Python 、PHP 、Ruby等（相比于 gRPC 支持的语言更多 ）。 

1. 官网：https://thrift.apache.org/  
2. Thrift 简单介绍：https://www.jianshu.com/p/8f25d057a5a9 



gRPC 和 Thrift 虽然支持跨语言的 RPC 调用，但是因为它们只提供了最基本的 RPC 框架功能，缺 乏一系列配套的服务化组件和服务治理功能的支撑。 Dubbo 不论是从功能完善程度、生态系统还是社区活跃度来说都是最优秀的。最重要的是其在国 内有很多成功的案例比如当当网、滴滴等等。下图展示了 Dubbo 的生态系统。 T

