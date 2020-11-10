# 概述

**堆内内存**

说到堆外内存，我们不免联想到堆内内存，也是平时我们接触最多的。我们应用程序中的对象实例都是分配在堆内存上面，堆内存的结构如下：

![image-20201109112311618](http://img.fosuchao.com/image-20201109112311618.png)

关于年轻代等的具体结构，这里不再赘述，想要了解可参考[链接](https://www.lagou.com/lgeduarticle/59550.html)。

堆内存分配和回收是由JVM进行管理的，因为它们遵循JVM的内存管理机制，JVM会有多种垃圾回收算法对无用的对象进行回收，对于系统资源的利用是基本可控的。

**堆外内存**

说回堆外内存，顾名思义，堆外内存就是把内存对象分配在Java虚拟机的堆外的内存，这些内存不由JVM管理，而是由操作系统来直接管理。

Java对于堆外内存的操作，主要依赖于**Unsafe**类中的操作堆外内存的方法。



# Java中的实现与分析

### DirectByteBuffer

Java中是通过**DirectByteBuffer**作为堆外内存的一个实现，在Netty等NIO框架中常被使用，**DirectByteBuffer**对于堆外内存的创建，使用和销毁，都是通过**Unsafe**中的native方法来实现。

**Unsafe中的内存操作api**

```java
//分配内存, 相当于C++的malloc函数
public native long allocateMemory(long bytes);
//扩充内存
public native long reallocateMemory(long address, long bytes);
//释放内存
public native void freeMemory(long address);
//在给定的内存块中设置值
public native void setMemory(Object o, long offset, long bytes, byte value);
//内存拷贝
public native void copyMemory(Object srcBase, long srcOffset, Object destBase, long destOffset, long bytes);
//获取给定地址值，忽略修饰限定符的访问限制。与此类似操作还有: getInt，getDouble，getLong，getChar等
public native Object getObject(Object o, long offset);
//为给定地址设置值，忽略修饰限定符的访问限制，与此类似操作还有: putInt,putDouble，putLong，putChar等
public native void putObject(Object o, long offset, Object x);
//获取给定地址的byte类型的值（当且仅当该内存地址为allocateMemory分配时，此方法结果为确定的）
public native byte getByte(long address);
//为给定地址设置byte类型的值（当且仅当该内存地址为allocateMemory分配时，此方法结果才是确定的）
public native void putByte(long address, byte x);
```

下面来看看DirectByteBuffer的构造器实现代码

```java
DirectByteBuffer(int cap) {                   
    super(-1, 0, cap, cap);
    boolean pa = VM.isDirectMemoryPageAligned();
    int ps = Bits.pageSize();
    long size = Math.max(1L, (long)cap + (pa ? ps : 0));
    Bits.reserveMemory(size, cap);

    long base = 0;
    try {
      // 通过Unsafe分配堆外内存，返回地址
      base = unsafe.allocateMemory(size);
    } catch (OutOfMemoryError x) {
      Bits.unreserveMemory(size, cap);
      throw x;
    }
    // 内存初始化
    unsafe.setMemory(base, size, (byte) 0);
    if (pa && (base % ps != 0)) {
      address = base + ps - (base & (ps - 1));
    } else {
      address = base;
    }
    // 通过cleaner对象来管理内存，实现堆外内存的回收
    cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
    att = null;
}
```

可以看到在DirectByteBuffer中有一个cleaner变量，这个变量就是用于构建垃圾回收追踪对象并用于堆外内存释放的，那么它具体是怎么做的呢？

先看Cleaner这个类，Cleaner**继承自**Java四大引用类型之一的虚引用——PhantomReference，通常**PhantomReference与引用队列ReferenceQueue结合使用**，可以实现虚引用关联对象被垃圾回收时能够进行系统通知、资源清理等功能。

> **虚引用**主要**用**来跟踪对象被垃圾回收器回收的活动。
>
> 众所周知，无法通过虚引用获取与之关联的对象实例，且当对象仅被虚引用引用时，在任何发生GC的时候，其均可被回收。
>
> 如果程序发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动。

**回收具体流程**

当某个被Cleaner引用的对象将要被回收时，JVM垃圾收集器会将此对象的引用放入到对象引用中的pending链表中，等待Reference-Handler进行相关处理。其中Reference-Handler是一个拥有最高优先级的守护线程，会循环不断的处理pendding链表中的对象引用，执行Cleaner的clean方法进行相关的清理工作。

即：

1. 判断不可达对象，放入pending链表
2. Reference-Handler守护线程对链表进行处理
3. 移除链表中的Cleaner对象，并执行他相应的方法，并且执行Cleaner.clean方法

![image-20201109142844814](http://img.fosuchao.com/image-20201109142844814.png)

所以当DirectByteBuffer仅被Cleaner引用（即为虚引用）时，其可以在任意GC时段被回收。当DirectByteBuffer实例对象被回收时，在Reference-Handler线程操作中，会调用Cleaner的clean方法根据创建Cleaner时传入的Deallocator来进行堆外内存的释放。

**Deallocator**

![image-20201109144509959](http://img.fosuchao.com/image-20201109144509959.png)

由上面的代码可以看出，Deallocator是DirectByteBuffer中的一个内部类，它实现了Runnable接口，其中在run方法中调用了释放内存的逻辑。

**并且相应的Deallocator已经在执行DirectByteBuffer构造器方法的时候与Cleaner关联起来了。**

![image-20201109144831234](http://img.fosuchao.com/image-20201109144831234.png)

**Cleaner的clean方法**

![image-20201109145115135](http://img.fosuchao.com/image-20201109145115135.png)

**Reference部分代码**

![image-20201109144217397](http://img.fosuchao.com/image-20201109144217397.png)



# 注意点

### 堆外内存的大小

堆外内存默认大小是64M，但是我们可以通过**-XX:MaxDirectMemorySize**这个JVM参数来修改它的大小，因为堆外内存并不受JVM的控制，所以它的内存分配可以分配更大的空间，这个取决于机器内存的大小，堆外内存可以让我们随意分配，随着的风险也是非常大的，如果没有对内存进行及时的回收，即使允许分配更多的空间，机器的内存也不够分配，从而也一样会发生**OutOfMemory**异常。



### 堆外内存的优点

1. 不需要垃圾回收

   > 堆外内存由操作系统管理而不是JVM，从而在使用堆外内存的时候可以减少GC产生的回收停顿对于应用的影响。

2. 减少了数据拷贝操作

   > 在I/O通信的操作的时候，堆内的数据需要flush到远程，需要将数据从堆内拷贝到堆外内存，而堆外内存则不需要这个操作。



### 堆外内存的缺点

1. 内存泄漏

   > 不合理或者非法使用api，没有对堆外内存进行回收或者一直触发不了full gc导致内存泄漏，难以排查。

2. 不适合存储复杂和长生命周期的对象

   > 对于需要频繁操作的内存，并且仅仅是临时存在一会的，都建议使用堆外内存，并且做成缓冲池，不断循环利用这块内存。



# 堆外内存的回收

1. Full GC（调用system.gc()，前提jvm参数没有**-XX:+DisableExplicitGC**）
2. 调用Unsafe中的freeMemory方法



**参考**

[Java魔法类：Unsafe应用解析](https://tech.meituan.com/2019/02/14/talk-about-java-magic-class-unsafe.html)

[JVM源码分析之堆外内存完全解读](http://lovestblog.cn/blog/2015/05/12/direct-buffer/)

[理解Java内存区域与Java内存模型](https://blog.csdn.net/javazejian/article/details/72772461#java%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B%E6%A6%82%E8%BF%B0)

