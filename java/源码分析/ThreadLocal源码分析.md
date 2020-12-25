在我们多线程去写一个变量的时候，可能会出现线程安全性问题，ThreadLocal就是让每个线程都有拥有一个私有的变量，读写都是对自己的私有变量进行操作，不会对其他的线程产生影响。



## 简单使用

下面的代码演示一下ThreadLocal的简单使用

![image-20201225164040781](http://img.fosuchao.com/image-20201225164040781.png)

![image-20201225164133145](http://img.fosuchao.com/image-20201225164133145.png)

在main函数中，每个线程都在userThreadLocal中设置了userInfo，然后休眠一段随机的时间，再去获取之前设置的值。

运行结果：

![image-20201225164206855](http://img.fosuchao.com/image-20201225164206855.png)

可以看出，每个线程获取到的值还是之前设置的值，并没有被其他线程修改。



## get

先来看看get()方法

![image-20201225164625066](http://img.fosuchao.com/image-20201225164625066.png)

1. 先获取当前的线程
2. 根据当前的线程获取对应的ThreadLocalMap
3. 根据获取到的ThreadLocalMap获取对应的值

### getMap()方法

![image-20201225164701551](http://img.fosuchao.com/image-20201225164701551.png)

可以看出getMap()方法其实就是拿到当前线程中的threadLocals变量。

### Thread类中的threadLocals变量

![image-20201225165018123](http://img.fosuchao.com/image-20201225165018123.png)

每个线程内部都会有一个ThreadLocalMap变量（threadLocals），这个threadLocals是属于这个线程的ThreadLocal值，但是**这个映射由ThreadLocal类维护的**。

下面看看ThreadLocal中的set()方法就可以明白了。

## set

![image-20201225165307328](http://img.fosuchao.com/image-20201225165307328.png)

1. 获取当前线程
2. 获取当前线程的ThreadLocalMap实例（threadLocals变量）
   1. 如果不为空，则以当前ThreadLocal实例为key设置当前的value
   2. 如果为空，则通过createMap方法去创建Map，并且设置value

### createMap()方法

![image-20201225165536113](http://img.fosuchao.com/image-20201225165536113.png)

## ThreadLocalMap

ThreadLocalMap其实就是一个哈希表，他有set和get方法，key是ThreadLocal，值就是线程设置的值

ThreadLocalMap内部有一个Entry变量，用来保存key和value，可以看到这个Entry是继承于WeakReference的，这个跟垃圾回收相关，下面会提到

![image-20201225170645249](http://img.fosuchao.com/image-20201225170645249.png)

理所当然，ThreadLocalMap中也有一个Entry数组，用来保存Entry，get的时候可以通过计算key的哈希值，获取到对应的Entry，并获取到value。

![image-20201225170859422](http://img.fosuchao.com/image-20201225170859422.png)

## 垃圾回收

ThreadLocal作为Entry的key是弱引用，我们先看看java四种引用类型

- **强引用**：我们常常new出来的对象就是强引用类型，只要强引用存在，垃圾回收器将永远不会回收被引用的对象，哪怕内存不足的时候
- **软引用**：使用SoftReference修饰的对象被称为软引用，软引用指向的对象在内存要溢出的时候被回收
- **弱引用**：使用WeakReference修饰的对象被称为弱引用，只要发生垃圾回收，若这个对象只被弱引用指向，那么就会被回收
- **虚引用**：虚引用是最弱的引用，在 Java 中使用 PhantomReference 进行定义。虚引用中唯一的作用就是用队列接收对象即将死亡的通知

可以看出只要发生垃圾回收，被WeakReference修饰的ThreadLocal就会被GC回收，从而导致，**key为空，但是value还存在的情况，这样就导致了内存泄露**。

所以我们要即使的调用remove方法，去及时清除掉相应的entry。



[javaGuide链接](https://snailclimb.gitee.io/javaguide/#/docs/java/multi-thread/%E4%B8%87%E5%AD%97%E8%AF%A6%E8%A7%A3ThreadLocal%E5%85%B3%E9%94%AE%E5%AD%97)