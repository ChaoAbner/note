AbstractQueuedSynchronizer（AQS），队列同步器，许多Java并发包中的Lock工具都是有继承这个抽象类来实现。

## AQS的内部实现

同步器依赖内部的同步队列（一个FIFO双向队列）来完成同步状态的管理，当前线程获取同步状态失败时，同步器会将当前线程以及等待状态等信息构造成一个节点（Node）并将其加入同步队列，同时会阻塞当前线程，当同步状态释放时，会把守节点中的线程唤醒，使其再次尝试获取同步状态。

Node的主要属性如下：

```java
static final class Node {
    int waitStatus; // 表示节点的状态
    Node prev;	// 前继节点
    Node next; 	// 后继节点
    Node nextWaiter; 	// 存储在condition队列中的后继节点
    Thread thread; 		// 当前线程
}
```

接下来我们通过观察ReentrantLock来逐一分析AQS的实现。

## ReentrantLock

ReentrantLock的类图如下

![](http://img.fosuchao.com/20200306175720.png)

ReentrantLock有一个抽象内部类Sync，而有两个内部类NonfairSync和FairSync继承并实现了Sync类，分别实现了非公平锁和公平锁。

**Sync本身就是实现的非公平锁**。

## 源码分析

**ReentrantLock.lock()**

```java
 public void lock() {
     sync.acquire(1);
 }
```

sync.acquire实际调用了AQS中的acquire

![](http://img.fosuchao.com/20200306191418.png)

​		tryAcquire可参考本文下面将非公平锁和公平锁的部分，这里简单地讲就是获取锁的函数，如果获取锁成功，tryAcquire返回true，则acquire不会继续往下执行，如果获取锁失败，则acquire会继续执行下面的`addWaiter`、`acquireQueued`和`selfInterrupt`方法。接着往下看。

**addWaiter()**

```java
private Node addWaiter(Node mode) {
    // 创建一个独占的Node节点，模式为排他模式Node.EXCLUSIVE
    Node node = new Node(mode);
    // 采用自旋的方式将新的node插入到等待队列中
    for (;;) {
        Node oldTail = tail;
        if (oldTail != null) {
            // 将node的prev指向tail
            node.setPrevRelaxed(oldTail);
            if (compareAndSetTail(oldTail, node)) {
                // 将tail的next指向node，形成双向链表
                oldTail.next = node;
                return node;
            }
        } else {
            // 如果tail为null，进行初始化。
            // 将tail始化为new Node, head被设置为null
            initializeSyncQueue();
        }
    }
}
```

假如我们有三个线程，A、B和C，线程A已经成功获取了锁，然后线程B尝试获取锁，此时aqs中的队列结构就是这样

![](http://img.fosuchao.com/20200306193726.png)

**acquireQueued**

addWaiter返回了插入的节点Thread-B Node

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean interrupted = false;
    try {
        for (;;) {
            // 获取node的prev节点，为null则抛出异常
            final Node p = node.predecessor();
            // 只有前驱节点为head才有资格争抢锁
            if (p == head && tryAcquire(arg)) {
                // 再次尝试获取锁成功
                // 将node设为新的head，并将thread置为null
                setHead(node);
                p.next = null; // help GC
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node))
                // 不成功，继续阻塞等待唤醒。
                interrupted |= parkAndCheckInterrupt();
        }
    } catch (Throwable t) {
        cancelAcquire(node);
        if (interrupted)
            selfInterrupt();
        throw t;
    }
}
```

从这里可以知道的是，在AQS中的等待队列即双向链表的**head始终为null**。

此时队列中的状态如下：

![](http://img.fosuchao.com/20200306194748.png)

**ReentrantLock.unlock()**

```java
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    // 当state=0时，将当前持有的thread置为null
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```

**AQS的release**

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        // 成功释放锁
        Node h = head;
        if (h != null && h.waitStatus != 0)
            // 唤醒head.next节点。
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

**非公平锁中tryAcquire()**

![](http://img.fosuchao.com/20200306174332.png)

实现步骤：

1. getState()先获取锁的状态
2. 如果state等于0，则通过`CAS`将state设置为1，并将`AbstractOwnableSynchronizer`中的`exclusiveOwnerThread`设置当前线程，返回true
3. 判断是否为重入锁，如果当前的currentThread等于之前设置的thread的话，将state+1并更新state值。
4. 不满足上面条件则获取锁失败，返回false。

**公平锁**

公平锁的实现与非公平锁的差别就在这个`hasQueueedPredecessors`方法，只有当这个方法的条件满足时，才继续进行后续的获取锁操作。

![](http://img.fosuchao.com/20200306175117.png)

**hasQueueedPredecessors分析**

`hasQueuedPredecessors`是AQS中的方法，先来看一下它做了什么

![](http://img.fosuchao.com/20200306175059.png)

`hasQueuedPredecessors`实际上就是让head的next为null的时候，才允许其它线程去争抢锁。这样来实现公平锁。