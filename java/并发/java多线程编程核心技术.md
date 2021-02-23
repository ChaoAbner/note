## 对象和变量的并发访问

### volatile关键字

`synchronized`和`volatile`的不同点：

- volatile是线程同步的轻量级实现，性能要比synchronized高。volatile只能修饰变量，而synchronized能修饰方法和代码块。
- 多线程访问volatile不会发生阻塞，而synchronized会出现阻塞。
- volatile能保证数据的可见性，但是不能保证原子性（即不具备同步性）；而synchronized能保证原子性，也可以简介保证可见性，因为它会讲四有内存和公共内润中的数据做同步。
- volatile解决是是变量在多个线程之间的可见性；而synchronized解决的是多个线程之间访问资源的同步性。

#### volatile出现非线程安全的原因：

- read和load阶段：从主存复制变量到当前线程工作内存；

- use和assign阶段：执行代码，改变共享变量值；
- store和write阶段：用工作内存数据刷新主存对应变量的值。

变量在内存中工作的过程如图：

![1571830379275](http://img.fosuchao.com/1571830379275.png)

#### 使用原子类进行i++操作

​		除了使用synchronized实现同步外，还可以使用AtomicInteget原子类进行实现。

```java
public class AddCount extends Thread {
    private AtomicInteger count = new AtomicInteger(0);

    @Override
    public void run() {
        for(int i = 0; i < 1000; i++) {
            System.out.println(count.incrementAndGet());
        }
    }
}
// ---- 
public class Run {
    public static void main(String[] args) {
        AddCount addCount = new AddCount();

        for(int i = 0; i < 5; i ++) {
            new Thread(addCount).start();
        }
    }
}
// 输出结果成功从1 到 5000
```

​		原子类也并不完全安全

```java
public static AtomicLong count = new AtomicLong();

public void addCount() {
	System.out.println("加100之后的值是" count.addAndGet(100));
	count.addAndGet(1);
}
```

​		此时，如果用多个线程来调用这个方法的时候，就会出现打印顺序出错的问题，因为`addAndGet`方法是原子的，但是方法和方法之间的调用却不是安全的。

​		解决这样的问题需要使用同步。即在`addCount`方法加上`synchronized`关键字。

## 线程之间的通信

- 使用wait/notify实现线程间的通信
- 生产者/消费者模式的实现
- join方法的使用
- ThreadLocal类的使用

### 等待/通知机制

#### wait/notify机制的实现

​		wait()将当前线程置入“预执行队列”中，知道街道通知或终端时位置。调用wait之前，线程必须获得该对象的对象级别锁，即只能在同步方法或同步块中调用wait方法。在执行wait方法后，当前线程释放锁。在wait返回前，线程与其它线程竞争重新获得锁。

​		notify()也要在同步方法或同步块中调用，即在调用前，线程也必须获得该对象的对象级别锁。若有多个线程（优先级相同）等待，则随机对一个线程发出通知notify（或者对优先级高的发出通知），并使它等待获取该对象的对象锁。在执行notify方法后，当前线程不会马上释放该对象锁，wait状态的线程也不能马上获得对象锁，需要等到执行notify方法的线程将程序执行完，也就是退出的synchronized代码块后，当前线程才会释放锁。



####  线程在wait状态中使用interrupt会报错

- 执行完同步代码块就会释放对象的锁
- 在执行同步代码块的过程中，遇到异常会导致线程终止，锁被释放
- 在执行同步代码块的过程中，执行了锁的所属对象的wait方法，这个线程会释放对象锁，而此线程对象会进入线程等待池，等待被唤醒。



#### wait（long）方法的使用

```java
// 等待1秒钟后，线程自动唤醒
lock.wait(1000);
```

### 方法join的使用

​		通常情况下，主线程创建并启动子线程，如果子线程中要进行大量的耗时运算，主线程往往早于子线程结束前结束。

​		如果主线程想要等待子线程结束后再结束，就要用到join()方法了，它的作用是等待线程对象销毁。

#### join()和异常

​		当一个线程在join()方法中，线程未销毁的情况下执行interrupt()方法就会抛出异常。

#### join(long)的使用

​		long类型的参数就是设定等待的时间。

#### join(long)和sleep(long)的区别

​		join(long)功能的内部是使用wait(long)方法来实现的，所以join(long)方法具有释放锁的特点。

### ThreadLocal的使用

 		变量值的共享可以使用Public static的形式，而每个线程要有自己的共享变量要使用到ThreadLocal了。

## Lock的使用

### 使用ReentrantLock类

​		synchronized关键字可以实现线程之间同步互斥，ReentrantLock也可以达到同样的效果，扩展功能上也更加强大，比如：嗅探锁定、多路分支通知等功能。使用也更灵活。

​		简单使用同步

```java
public class MyService {
    private Lock lock = new ReentrantLock();

    public void testMethod() {
        // 获取锁
        lock.lock();
        for(int i = 0; i < 5; i ++) {
            System.out.println("THreadName = " + Thread.currentThread().getName() + " " + (i + 1));
        }
        // 释放锁
        lock.unlock();
    }
}
// -----
public class MyThread extends Thread {
    private MyService myService;

    MyThread(MyService myService){
        this.myService = myService;
    }

    @Override
    public void run() {
        myService.testMethod();
    }
}

```

​		调用lock.lock()代码的线程就持有了“对象监视器”，其他线程只有等待锁被释放时再次争抢。

#### 使用Condition实现等待/通知

​		ReentrantLock需要实现与synchronized与wait()和notify()等方法实现等待/通知模式，需要借助Condition对象。

​		在调用Condition对象的await()方法之前，必须获取到监视器对象（即lock.lock()），不然就会报错。

```java
public class MyService {
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    public void await() {
        lock.lock();
        System.out.println("await时间：" + System.currentTimeMillis());
        try {
            condition.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void signal() {
        lock.lock();
        System.out.println("signal时间：" + System.currentTimeMillis());
        condition.signal();
        lock.unlock();
    }

}
```

```java
public class ThreadA extends Thread {
    private MyService myService;

    ThreadA(MyService myService) {
        this.myService = myService;
    }

    @Override
    public void run() {
        myService.await();
    }
}

```

```java
public class Run {
    public static void main(String[] args) throws InterruptedException {
        MyService myService = new MyService();
        ThreadA threadA = new ThreadA(myService);
        threadA.start();
        Thread.sleep(3000);
        myService.signal();
    }
}

```

Object中的wait方法相当于Condition类中的await方法；

Object中的nofity方法相当于Condition类中的signal方法；

Object中的nofityAll方法相当于Condition类中的signalAll方法；

#### lock对象的其它方法

- int getHoldCount() 

  查询当前线程保持此锁定的个数，也就是调用lock()方法的次数。

- int getQueueLength()  

  返回正在等待获取此锁定的线程估计数。比如有5个线程，1个线程在执行await方法，那么调用getQueueLength的返回值是4。说明有4个线程同时在等待lock的释放。

- int getWaitQueueLength(Condition condition)  

  返回等待与此锁定相关的给定条件Condition的线程估计数，比如有5个线程，每个线程都执行了**同一个condition对象**的await方法，则调用getWaitQueueLength(Condition condition)的返回值是5。

### 使用ReentrantReadWriteLock类

​		ReentrantLock具有完全互斥排他的效果，虽然保证了线程的安全性，但是效率非常低下。而使用ReentrantReadWriteLock可以提升代码运行速度。

​		读写锁有两种锁，一个是读操作相关的锁，也称为共享锁，另一个是写操作相关的锁，也叫排他锁。

​		多个读锁锁不互斥，读锁和写锁互斥，写锁和写锁互斥。

​		