# IO模型

## 了解Unix网络编程的五种I/O模型

### 阻塞I/O

从系统调用一直到内核中的数据复制完成，才返回结果。

![](http://img.fosuchao.com/20200307144255.png)

### 非阻塞I/O

系统调用后不阻塞，期间不断轮询内核进程是否准备好数据，数据复制后即返回。

![](http://img.fosuchao.com/20200307144311.png)

### I/O复用

![](http://img.fosuchao.com/20200307144341.png)

### 信号驱动的I/O

系统调用后不阻塞，内核将数据准备好主动通知工作进程/线程，在阻塞获取数据。

![](http://img.fosuchao.com/20200307144407.png)

### 异步I/O

系统调用后不阻塞，内核将数据准备好主动通知工作进程/线程，直接可以获取数据。

![](http://img.fosuchao.com/20200307144424.png)

**同步IO与异步IO**

同步IO和异步IO的区别就在于：**数据访问**的时候进程是否阻塞！

**阻塞IO和非阻塞IO**

阻塞IO和非阻塞IO的区别就在于：**应用程序的调用**是否立即返回！

Java支持三种网络编程模型/IO模型：**BIO，NIO，AIO**，其中**Netty基于NIO**

## BIO

BIO即同步阻塞IO，每有一个客户端连接，服务端便开启一个线程去处理客户端的连接。

#### **1：1的同步阻塞IO**

![](http://img.fosuchao.com/20200307143628.png)

#### **问题**

- 并发数大时，不断创建新线程，造成CPU占用满。
- 建立连接后，当前线程没有数据可读，就阻塞在read操作上，造成不必要的线程开销。
- 不便于维护

#### M:N的同步阻塞IO

![](http://img.fosuchao.com/20200307143407.png)

比起1：1的阻塞IO，性能有所提升，不过线程池线程毕竟有限，仍然是阻塞IO。

## NIO

- NIO即同步非阻塞IO（No-block I/O & New I/O），其中有个重要的组件：**Selector**(选择器 & 多路复用器)，**Channel**（管道）,**Buffer**（缓冲区），服务器实现一个线程处理多个连接。客户端发送的请求（管道维护连接）都会注册到多路复用器上（Selector），多路复用器轮询到连接的I/O请求则去处理。

- NIO是**面向缓冲区的，面向块编程**，**通过缓冲区读写数据**。
- 当线程下的所有连接没有读写操作时，线程可以去做其它的事情，而不是阻塞到那里。
- HTTP2.0使用了**多路复用**的技术，做到**同一个连接并发处理多个请求**，而且并发请求的书比HTTP1.1大好几个量级。

#### **架构**

![](http://img.fosuchao.com/20200307161734.png)

#### **Buffer**

Buffer（缓冲区）本质上是一个可以读写数据的内存块，可以理解成一个容器对象（包含数组）。

#### **Channel**

Channel（管道），类似于Stream（流），区别是**Channel可以同时进行读写**，**流只能读或者写，不能同时进行**。

Channel**可以从Buffer中读取写入**数据。

可以实现**异步读写**数据。

常用Channel：

- FileChannel：文件读写
- DatagramChannel：基于UDP的数据读写
- ServerSocketChannel，SocketChannel：基于TCP的数据读写

#### **Selector**

可以监测已注册的Channel是否有事件发生，有的话便进行处理。这样就让一个线程管理多个连接。

避免了多线程之间的上下文切换导致的开销。

#### **原理分析**

![](http://img.fosuchao.com/20200307161720.png)

每当selector注册一个channel，服务端的**serverSocketChannel就会得到一个SocketChannel和SelectionKey**，并使用SelectionKey来和改Selector关联，通过SelectionKey可以反向获取SocketChannel和Channel方法，也就是说用**SelectionKey来管理相应的Channel**（SocketChannel）。

#### **问题**

NIO有一个epoll bug，就是Selector空轮询，最终导致CPU 100%。JDK1.7这个问题都还存在。

## **AIO**

异步I/O，现在用的不多，这里不做详细讨论。

### 总结

BIO可以适用于一些服务请求连接数目小的场景，对服务器资源要求高。

NIO适用于一些连接请求时间短的场景（轻操作），比如弹幕系统，聊天服务器，服务器间通讯等。编程复杂。

AIO适用于连接数目多而且连接时间长的场景（重操作），比如相册服务器，充分调用OS参与并发操作。编程复杂。

## Netty介绍

​		Netty是一个异步，基于事件驱动的网络应用框架，可用于开发高性能、高可靠的网络IO程序。

​		Netty基于NIO，使用主从**Reactor**多线程模式。主要在**针对在TCP协议**下，面向Clients端的**高并发**应用，或者p2p场景下的大量数据传输的应用。

### 特点		

- 高并发

  使用NIOEventLoopGroups创建分发组（主）和工作组（从），一个组是一个线程池，可以指定创建线程的多少。

- Reactor模式

  在Netty中，bossGroup指的就是Reactor中的acceptor,workerGroup就是Reactor中的reactor。

  bossGroup的工作就是接受客户端的连接事件，建立对应client的Handler，并向Reactor注册此Handler。并把相应的I/O事件分发给workerGroup去完成。

### Reactor模式

#### NIO+单线程Reactor模式

![](http://img.fosuchao.com/20200307143813.png)

#### NIO+多线程Reactor模式

![](http://img.fosuchao.com/20200307143903.png)

#### NIO+主从多线程Reactor模式

![](http://img.fosuchao.com/20200307143921.png)

### Netty中的Reactor模型

#### **单Reactor单线程**

单线程处理连接和事件处理

![](http://img.fosuchao.com/20200307154518.png)

#### 单Reactor多线程

将分发线程和工作线程分开。

![](http://img.fosuchao.com/20200307152856.png)

#### 主从Reactor多线程

将建立连接的Reactor和处理I/O的请求分开。

![](http://img.fosuchao.com/单reactor主从模型.png)

#### netty对主从Reactor的应用

`BossGroup`负责维护的selector只负责关心accept事件，并将连接的Accept事件对应的SocketChannel封装成NIOSocketChannel注册到worker线程组，`WorkerGroup`的selector则负责维护通道中的I/O事件（read/write），然后通过**pipeline中的handler进行处理**。

![](http://img.fosuchao.com/20200307154919.png)

#### netty的Reactor工作架构

![](http://img.fosuchao.com/20200307155257.png)

上图说明

1. Netty抽象出两个线程组，其实BossGroup负责接受客户端的连接。WorkerGroup专门负责网络I/O处理。
2. NioEventLoopGroup相当于一个事件循环组，组中有很多个事件循环NioEventLoop。
3. NioEventLoop是一个无限循环的执行处理任务的线程，每个NioEventLoop中都有一个Selector，用于绑定它所管理的所有channel的网络通信。
4. 每个BossNioEventLoop的执行步骤
   - 轮询accept事件
   - 处理accept事件，与client建立连接，生成NioSocketChannel,并将其注册到某个workerNioEvent中的Selector上。
   - 处理任务队列中的任务，即runAllTasks。
5. 每个WorkNioEventLoop的执行步骤
   - 轮询read，write事件
   - 处理I/O事件，即read，write，在对应的NioSocketChannel上处理
   - 处理任务队列中的任务。即runAllTasks。
6. 处理业务时，会使用到pipeline（管道），pipeline中包含了一个channel，可以获取相应的channel，channel中维护了很多handler（处理器）。



### 应用场景

![](http://img.fosuchao.com/20200307145250.png)