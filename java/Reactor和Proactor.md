# Reactor和Proactor

两种I/O多路复用模式：Reactor和Proactor



一般地,I/O多路复用机制都依赖于一个事件**多路分离器(Event Demultiplexer)**。分离器对象可将来自事件源的I/O事件分离出来，并分发到对应的**read/write事件处理器(Event Handler)**。开发人员预先注册需要处理的事件及其事件处理器（或回调函数）；事件分离器负责将请求事件传递给事件处理器。



**两个与事件分离器有关的模式是Reactor和Proactor。Reactor模式采用同步IO，而Proactor采用异步IO。**



### Reactor模式

![](https://pic4.zhimg.com/80/v2-0f72e05db326c4d1f4e416178cc8c658_720w.jpg)

> Reactor 是这样一种模式，它要求主线程（I/O处理单元）只负责监听文件描述上是否有事件发生，有的话就立即将该事件通知工作线程（逻辑单元）。除此之外，主线程不做任何其他实质性的工作。读写数据，接受新的连接，以及处理客户请求均在工作线程中完成。

使用同步I/O模型（以epoll_wait为例）实现的Reactor模式的工作流程是：

1. 主线程往epoll内核时间表中注册socket上的读就绪事件。
2. 主线程调用epoll_wait等待socket上有数据可读。
3. 当socket上有数据可读时，epoll_wait通知主线程。主线程则将socket可读事件放入请求队列。
4. 睡眠在请求队列上的某个工作线程被唤醒，它从socket读取数据，并处理客户请求，然后往epoll内核事件表中注册该socket上的写就绪事件。
5. 主线程调用epoll_wait等待socket可写。
6. 当socket可写时，epoll_wait通知
7. 睡眠在请求队列上的某一个工作线程被唤醒，它往socket上写入服务器处理客户请求的结果。



![img](https:////upload-images.jianshu.io/upload_images/2100026-a2e15f5fd17e96d8.png?imageMogr2/auto-orient/strip|imageView2/2/w/884/format/webp)

### Proactor模式

> 与Reactor模式不同，Proactor模式将所有I/O操作都交给主线程和内核来处理，工作线程仅仅负责业务逻辑。

使用异步I/O模型（以aio_read和aio_write为例）实现的Proactor模式的工作流程是：

1. 主线程调用aio_read函数向内核注册socket上的读完事件，并告诉内核用户读缓冲区的位置，以及读操作完成时如何通知应用程序（可以通过信号）
2. 主线程继续处理其他逻辑。
3. 当socket上的数据被读入用户缓冲区后，内核将向应用程序发送一个信号，已通知应用程序数据已经可以用了。
4. 应用程序预先定义好的信号处理函数选择一个工作线程来处理客户请求。工作线程处理完客户请求之后，调用aio_write函数向内核注册socket上的写完事件，并告诉内核用户写缓冲区的位置，以及写操作完成时如何通知应用程序
5. 主线程继续处理其他逻辑。
6. 当用户缓冲区的数据被写入socket之后，内核将向应用程序发送一个信号，已通知应用程序数据已经发送完毕。
7. 应用程序预先定义好的信号处理函数选择一个工作线程来做善后处理，比如决定是否关闭socket。



![img](https:////upload-images.jianshu.io/upload_images/2100026-273b3206fddb27fb.png?imageMogr2/auto-orient/strip|imageView2/2/w/920/format/webp)

2、通俗理解

使用Proactor框架和Reactor框架都可以极大的简化网络应用的开发，但它们的重点却不同。

Reactor框架中用户定义的操作是在实际操作之前调用的。比如你定义了操作是要向一个SOCKET写数据，那么当该SOCKET可以接收数据的时候，你的操作就会被调用；**而Proactor框架中用户定义的操作是在实际操作之后调用的。比如你定义了一个操作要显示从SOCKET中读入的数据，那么当读操作完成以后，你的操作才会被调用。**

**Proactor和Reactor都是并发编程中的设计模式。**在我看来，他们都是用于派发/分离IO操作事件的。这里所谓的IO事件也就是诸如read/write的IO操作。"派发/分离"就是将单独的IO事件通知到上层模块。两个模式不同的地方在于，**Proactor用于异步IO，而Reactor用于同步IO。**

部分参考自

http://www.cnblogs.com/dawen/archive/2011/05/18/2050358.html