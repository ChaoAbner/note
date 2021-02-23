线程池实现类ThreadPoolExecutor是Executor框架最核心的类

## 类图

![image-20201226104938985](http://img.fosuchao.com/image-20201226104938985.png)



## 构造函数

我们先来看看他的构造函数，最重要的就是他构造函数的参数

![image-20201226105155721](http://img.fosuchao.com/image-20201226105155721.png)

ThreadPoolExecutor的几个重要参数

- corePoolSize：核心线程数定义了最小可以同时运行的线程数量
- maximumPoolSize：最大线程数，当队列中存放的任务满了，当前可以同时运行的线程可以达到最大线程数
- workQueue：当线程池的线程数达到了核心线程数，当新任务来的时候并且队列容量未满，此时会将新任务存放在队列中
- keepAliveTime：线程池中的线程数大于**corePoolSize**的时候，如果没有新的任务，这些临时线程不会被立刻销毁，而是等待时间超过了**keepAliveTime**之后才会被回收销毁
- threadFactor：executor创建新线程的时候会用到
- Handler：拒绝策略，下面会单独介绍

下面这张图可以加深你对线程池中各个参数的相互关系的理解

![image-20201226111744747](http://img.fosuchao.com/image-20201226111744747.png)

**ThreadPoolExecutor**饱和策略的定义

如果当前同时运行的**线程数量达到最大线程数量**并且**队列也已经被放满了任务**时，`ThreadPoolTaskExecutor` 定义一些策略:

- **`ThreadPoolExecutor.AbortPolicy`**：抛出 `RejectedExecutionException`来拒绝新任务的处理。
- **`ThreadPoolExecutor.CallerRunsPolicy`**：调用执行自己的线程运行任务，也就是直接在调用`execute`方法的线程中运行(`run`)被拒绝的任务，如果执行程序已关闭，则会丢弃该任务。因此这种策略会降低对于新任务提交速度，影响程序的整体性能。如果您的应用程序可以承受此延迟并且你要求任何一个任务请求都要被执行的话，你可以选择这个策略。
- **`ThreadPoolExecutor.DiscardPolicy`：** 不处理新任务，直接丢弃掉。
- **`ThreadPoolExecutor.DiscardOldestPolicy`：** 此策略将丢弃最早的未处理的任务请求。

## Executors

我们可以通过Executors类返回线程池对象，Executors内部也是通过ThreadPoolExecutor去构建返回线程池的，不过通过构造函数参数的不同，去创建了不同功能的线程池。

> 另外《阿里巴巴 Java 开发手册》中强制线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 构造函数的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险

![image-20201226112020814](http://img.fosuchao.com/image-20201226112020814.png)

Executors 返回线程池对象的弊端如下：

- **`FixedThreadPool` 和 `SingleThreadExecutor`** ： 允许请求的队列长度为 Integer.MAX_VALUE,可能堆积大量的请求，从而导致 OOM。
- **`CachedThreadPool` 和 `ScheduledThreadPool`** ： 允许创建的线程数量为 Integer.MAX_VALUE ，可能会创建大量线程，从而导致 OOM。



## 方法

### execute()

![image-20201226113650463](http://img.fosuchao.com/image-20201226113650463.png)

### addWorker()

这个方法主要用来创建新的工作线程，如果返回true说明创建和启动工作线程成功，否则的话返回的就是false

![image-20201226113935263](http://img.fosuchao.com/image-20201226113935263.png)

[ThreadPoolExecutor的JavaGuide链接](https://snailclimb.gitee.io/javaguide/#/./docs/java/multi-thread/java%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93?id=%e4%b8%89-%e9%87%8d%e8%a6%81threadpoolexecutor-%e7%b1%bb%e7%ae%80%e5%8d%95%e4%bb%8b%e7%bb%8d)

