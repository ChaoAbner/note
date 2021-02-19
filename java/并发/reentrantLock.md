## ReentrantLock

ReentrantLock默认创建的锁是非公平锁

ReentrantLock中的Sync内部类继承了AQS

### Sync

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = -5179523762034025860L;
	
    // lock方法由公平锁和非公平锁各自实现
    abstract void lock();

    // 默认实现非公平加锁
    final boolean nonfairTryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            // 尝试CAS尝试获取，获取成功则直接返回
            if (compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        // 可重入锁
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}
```

### 非公平锁 NonfairSync

```java
static final class NonfairSync extends Sync {

        final void lock() {
            // 先通过CAS尝试获取锁
            if (compareAndSetState(0, 1))
 				setExclusiveOwnerThread(Thread.currentThread());
            else
                // 否则执行AQS中的acquire方法
                acquire(1);
        }

        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }
```

### 公平锁 FairSync

```java
static final class FairSync extends Sync {
    
    final void lock() {
        // 直接的调用acquire方法
        acquire(1);
    }

    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            // hasQueuedPredecessors方法用来判断队列中是否有其它线程等待
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}
```

公平锁与非公平锁的区别：

1. 非公平锁会在`lock`方法中使用**CAS**尝试获取锁，而公平锁不会
2. 公平锁在`tryAcquire`方法中会先用`hasQueuedPredecessors`方法判断同步队列中是否有其它线程等待

## AQS

### acquire

`acquire`是获取锁的方法

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

### tryAcquire

`tryAcquire`方法会让子类去重写，这是一种模板方法模式，是基于继承实现的。

### addWaiter

`addWaiter`方法根据给定的模式去创建一个排队节点，即将线程作为节点放入队列尾部中等待。

返回新创建的节点node

```java
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node;
}
```



### acquireQueued

`acquireQueued`方法接受新创建好的节点。

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        // 自旋
        for (;;) {
            // 获取上一个节点
            final Node p = node.predecessor();
            // 如果上一个节点是head，那么尝试去获取锁
            if (p == head && tryAcquire(arg)) {
                // 获取锁成功，设置当前节点为head
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            // 加锁失败，通过park方法进入等待
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

#### waitStatus

每个node有五种状态：

```java
static final int CANCELLED =  1;

static final int SIGNAL    = -1;

static final int CONDITION = -2;

static final int PROPAGATE = -3;

// 还有一种状态值为0，表示以上状态都不是
```

- SIGNAL：当一个节点的状态是SIGNAL的时候，必须要park它的后继节点
- CANCELLED：由于超时或中断，该节点被取消。节点不会离开此状态。特别是取消节点的线程不会再次阻塞

#### shouldParkAfterFailedAcquire

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;
    if (ws == Node.SIGNAL)
        // SIGNAL说明需要去park后继节点，返回true
        return true;
    if (ws > 0) {
        // 大于0，即该节点被取消，则跳过当前节点
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        // 通过CAS去设置状态
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

#### parkAndCheckInterrupt

如果`shouldParkAfterFailedAcquire`方法判断了当前线程需要被挂起，那么就会执行`parkAndCheckInterrupt`方法。它通过`park`方法去把当前的线程挂起等待。

```java
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    return Thread.interrupted();
}
```

### selfInterrupt

获取锁失败，中断当前线程。

### release

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        // 头节点合法，那么去唤醒他的后继节点，unparkSuccessor
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

#### unparkSuccessor

`unparkSuccessor`方法是`release`方法中

```java
    private void unparkSuccessor(Node node) {
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);

        Node s = node.next;
        // 如果后继节点是被取消状态无法执行
        if (s == null || s.waitStatus > 0) {
            s = null;
            // 从尾节点开始遍历到第一个可以执行的节点
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            // 释放第一个可以执行的线程
            LockSupport.unpark(s.thread);
    }
```

