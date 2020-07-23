# Java.util.concurrent 下的常用工具类

## CountDownLatch

### 介绍

CountDownLatch是一种同步帮助，允许一个或多个线程等待，直到在其他线程中执行的一组操作完成。 

实现了一个计数器的功能，可以指定一个计数值，只有任意线程调用了CountDown方法才能使这个计数值-1，知道这个计数值为零的时候，返回await()方法，类似于Thread类中的join方法，不过CountDownLatch的使用更加灵活。

他的await()方法可以使用在多个线程中，直到count=0的时候才返回。

### 示例

举一个例子，多线程爬虫获取结果，然后再汇总处理。

```java
/**
 * @Description: 使用countdownlatch模拟一个爬虫应用（类似join的功能）
 * @Auther: Joker Ye
 * @Date: 2020/2/6 10:46
 */
public class CountDownLatchSimple {
    private static CountDownLatch latch = new CountDownLatch(5);

    public static void main(String[] args) throws InterruptedException {
        // 主任务，开启爬虫线程
        for (int i = 0; i < 5; i++) {
            String bound = i * 100 + "-" + (i + 1) * 100;
            new Thread(new Spider("爬取数据范围：" + bound, latch)).start();
        }

        // 等待所有爬虫线程完成
        latch.await();
        System.out.println("all work done!");
        System.out.println("进行数据分析");
    }
}

class Spider implements Runnable {
    private String work;
    private CountDownLatch latch;
    final Random random = new Random(System.currentTimeMillis());

    public Spider(String work, CountDownLatch latch) {
        this.work = work;
        this.latch = latch;
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + "正在执行爬虫任务");
        try {
            Thread.sleep(random.nextInt(3000));
            System.out.println(Thread.currentThread().getName() + "任务完成");
            latch.countDown();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```

### 适用场景

前面的任务可以使用并发处理提高效率，然后又使用串行方式的场景。

处理一种有层次关系的事情时，用join较难处理，这时可以使用CountDownLatch

![](http://img.fosuchao.com/20200226150757.png)

## CyclicBarrier

### 介绍

​		CyclicBarrier允许一组线程相互等待，每次有一个线程进入await()方法，会唤醒其它正在await中的线程，当这时的count=0，其它线程会被返回，不再等待。如果传递Runnable参数，则会回调传入的Runnable的run方法。并开始下一代的循环（完成当前周期的所有任务）

![](http://img.fosuchao.com/20200226161348.png)

![](http://img.fosuchao.com/20200226161410.png)

### 示例

```java
/**
 * @Description: 各个线程相互等待，直到count=0
 * @Auther: Joker Ye
 * @Date: 2020/2/6 10:47
 */
public class CyclicBarrierTest {
    static CyclicBarrier cyclicBarrier = new CyclicBarrier(3, new Runnable() {
        @Override
        public void run() {
            System.out.println("all task done!");
        }
    });

    public static void main(String[] args) {

        IntStream.of(1, 2, 3)
                .forEach(item -> {
                    new Thread(new Worker(cyclicBarrier),  "Thread-" + item).start();
                });
    }
}

class Worker implements Runnable {
    CyclicBarrier cyclicBarrier;

    public Worker(CyclicBarrier cyclicBarrier) {
        this.cyclicBarrier = cyclicBarrier;
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + "执行任务中。。。");
        try {
            TimeUnit.SECONDS.sleep(3);
            System.out.println(Thread.currentThread().getName() + "任务执行完毕！");
            cyclicBarrier.await();
        } catch (InterruptedException | BrokenBarrierException e) {
            e.printStackTrace();
        }
    }
}

```

### vs CountDownLatch

#### 线程之间

CyclicBarrier线程之间相互等待，新的线程会尝试唤醒其它线程，如果count=0，则成功唤醒并回调，count不等于0则所有线程继续await。

CountDownLatch线程之间互不相关。只关注count是否=0。

#### 使用

CyclicBarrier可以循环使用，可以主动调用reset方法，或者任务全部结束后会重新开启下一代。正如其名。

CountDownLatch不可以循环使用。

## Exchanger

### 介绍

​		线程可以对的同步点和对中的元素交换元素的同步点。每一个线程进入 `exchange`方法提出了一些目标，与合作伙伴的线匹配，并接收其伴侣的对象返回。交换机可以被看作是一个双向的 `SynchronousQueue`形式。交换器等应用遗传算法和管道的设计是有用的。  

### 示例

**简单交换**

```java
/**
 * @Description:
 * @Auther: Joker Ye
 * @Date: 2020/2/6 10:47
 */
public class ExchangerSimple {
    static Exchanger<Integer> exchanger = new Exchanger<>();

    public static void main(String[] args) {
        new Thread(new WorkerA(exchanger)).start();
        new Thread(new WorkerB(exchanger)).start();
    }
}

class WorkerA implements Runnable {
    Exchanger<Integer> exchanger;

    public WorkerA(Exchanger<Integer> exchanger) {
        this.exchanger = exchanger;
    }

    @Override
    public void run() {
        System.out.println("wokerA工作中。。");
        try {
            Thread.sleep(1000);
            Integer exchange = exchanger.exchange(1);
            System.out.println("A收到B的报告，B的工作量为：" + exchange);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class WorkerB implements Runnable {
    Exchanger<Integer> exchanger;

    public WorkerB(Exchanger<Integer> exchanger) {
        this.exchanger = exchanger;
    }

    @Override
    public void run() {
        System.out.println("wokerB工作中。。");
        try {
            Thread.sleep(2000);
            Integer exchange = exchanger.exchange(2);
            System.out.println("B收到A的报告，A的工作量为：" + exchange);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

**循环交换信息**

```java
/**
 * @description: Exchanger循环交换
 * @author: Joker Ye
 * @create: 2020/2/26 17:04
 */
public class ExchangerCycle {
    static Exchanger<AtomicReference> exchanger = new Exchanger<>();

    public static void main(String[] args) {
        new Thread(new PlayerA(exchanger)).start();
        new Thread(new PlayerB(exchanger)).start();
    }
}

class PlayerA implements Runnable {
    Exchanger<AtomicReference> exchanger;

    public PlayerA(Exchanger<AtomicReference> exchanger) {
        this.exchanger = exchanger;
    }

    @Override
    public void run() {
        AtomicReference reference = new AtomicReference<>("A的玩具");

        try {
            while (true) {
                reference = exchanger.exchange(reference);
                System.out.println("A收到了：" + reference.get());
                Thread.sleep(1000);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class PlayerB implements Runnable {
    Exchanger<AtomicReference> exchanger;

    public PlayerB(Exchanger<AtomicReference> exchanger) {
        this.exchanger = exchanger;
    }

    @Override
    public void run() {
        AtomicReference reference = new AtomicReference<>("B的玩具");

        try {
            while (true) {
                reference = exchanger.exchange(reference);
                System.out.println("B收到了：" + reference.get());
                Thread.sleep(1000);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

### 注意事项

1. 一般只适用于两个线程之间交换数据，如果有多个线程，会导致数据交换的不确定性，**至少要求线程成对出现。**
2. 要注意超时的问题，一旦一个线程超时没有交换数据，会导致另一个线程一直等待。

## Semaphore

### 介绍

计数信号量。从概念上讲，一个信号量维护一组允许。每个线程可以通过acquire获取一定数量的信号量，当剩余信号量为空时，其它线程等待。

信号量通常是用来限制线程的数量比可以访问一些（物理或逻辑）资源

### 示例

#### 锁的作用

```java
/**
 * @Description: 使用Semaphore实现一个锁
 * @Auther: Joker Ye
 * @Date: 2020/2/6 10:47
 */
public class SemaphoreLock {
    // 构造函数参数为指定的信号量/许可证，每次线程进入可以获取
    static Semaphore semaphore = new Semaphore(1);

    public static void main(String[] args) {
        new Thread(new Work(semaphore, 1)).start();
        new Thread(new Work(semaphore, 1)).start();
    }
}

class Work implements Runnable {
    Semaphore semaphore;
    Integer permits;

    public Work(Semaphore semaphore, Integer permits) {
        this.semaphore = semaphore;
        this.permits = permits;
    }

    @Override
    public void run() {
        String currThread = Thread.currentThread().getName();
        try {
            semaphore.acquire(permits);
            System.out.println(currThread + "获取了锁");
            System.out.println(currThread + "正在工作");
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            semaphore.release();
            System.out.println(currThread + "释放了锁");
        }
    }
}
```

#### 简单使用

```java
public class SemaphoreSimple {
    static Semaphore semaphore = new Semaphore(2);

    public static void main(String[] args) {
//        new Thread(new Work(semaphore, 2)).start();
        new Thread(new Work(semaphore, 1)).start();
        new Thread(new Work(semaphore, 1)).start();
    }
}


class Work1 implements Runnable {
    Semaphore semaphore;
    Integer permits;

    public Work1(Semaphore semaphore, Integer permits) {
        this.semaphore = semaphore;
        this.permits = permits;
    }

    @Override
    public void run() {
        String currThread = Thread.currentThread().getName();
        try {
            semaphore.acquire(permits);
            System.out.println(currThread + "正在工作");
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            semaphore.release();
        }
    }
}
```

