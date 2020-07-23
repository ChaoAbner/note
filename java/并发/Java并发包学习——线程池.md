Java并发包学习——线程池

## Executors

### diagram

![](http://img.fosuchao.com/20200227095619.png)

### newFixedThreadpool

​		创建一个固定核心线程数的线程池，最大线程数max和core数相等。所以不管有多少任务提交，最终都是由核心线程中的线程去执行。如果核心线程一直阻塞，则其它任务也一直在队列中等待。任务全部结束后进程不会自动结束。需要手动结束。

![](http://img.fosuchao.com/20200227113122.png)

### newSingleThreadpool

​		创建一个只有一个线程的线程池，所有任务都有这一个线程去执行，其它任务或者新提交的任务再队列中等待，直到线程空闲。任务全部结束后进程不会自动结束。需要手动结束。

![](http://img.fosuchao.com/20200227113407.png)

### newCachedThreadpool

​		创建一个核心线程数为0，最大线程数为Integer.MAX_VALUE的线程池，一起运行的线程数量不能超过最大值，只要不超过最大值，新提交的任务就会直接被运行。运行完成的线程会被直接回收。常用来跑一些短周期的异步任务（short-lived asynchronous tasks）.任务全部结束后进程不会自动结束。需要手动结束。

![](http://img.fosuchao.com/20200227113726.png)



### workStealingPool

​		与上面创建线程池方式不同的是，workStealingPool是用ForkJoinPool实现的。属于精灵线程（后台线程，守护线程）。当所有活跃的线程都结束后，也随之结束。**通常用于管理和监控操作**。

​		workStealingPool可以传入Callable接口，返回Future获取返回结果。

```java
public class WorkStealingPoolExample2 {
    static ExecutorService executor = Executors.newWorkStealingPool();

    public static void main(String[] args) throws InterruptedException, ExecutionException {
        List<Callable<String>> callableList = IntStream.range(0, 20).boxed().map(i ->
                (Callable<String>) () -> {
                    System.out.println(Thread.currentThread().getName() + "working..");
                    sleep(2);
                    System.out.println(Thread.currentThread().getName() + "finish!!");
                    return "Task-" + i;
                }).collect(toList());
        
        
        List<Future<String>> futures = executor.invokeAll(callableList);
        for (Future<String> future : futures) {
            // future.get()方法会阻塞，直到结果返回
            System.out.println(future.get());
        }

    }

    private static void sleep(int time) {
        try {
            TimeUnit.SECONDS.sleep(time);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```

### newScheduleThreadPool

​		使用ScheduleThreadPool实现定时任务, 如果任务的执行时间大于了定时时间，那么任务将会被推迟。跟JDK中的Timer很像。不能避免任务被推迟的问题。

​		可以指定运行的核心线程数量。

```java
public class ScheduleThreadPoolExample {
    static ScheduledExecutorService executor = Executors.newScheduledThreadPool(4);

    public static void main(String[] args) {
//        for (int i = 0; i < 5; i++) {
            executor.scheduleAtFixedRate(new TimerTask(), 1L, 1, TimeUnit.SECONDS);
//        }
    }
}

class TimerTask implements Runnable {
    String name = Thread.currentThread().getName();
    @Override
    public void run() {
        try {
            System.out.println(name + "task working...");
            TimeUnit.SECONDS.sleep(3);
            System.out.println(name + "task working done!");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

#### Timer

使用Timer实现定时任务。Timer中需要传入一个TimerTask任务，而TimerTask实际上继承了Runnable接口。

```java
public class SimpleTimer {
    public static void main(String[] args) {
        Timer timer = new Timer();
        TimerTask timerTask = new TimerTask(){
            @Override
            public void run() {
                try {
                    System.out.println("working...");
                    TimeUnit.SECONDS.sleep(5);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
        timer.schedule(timerTask, 0, 1000);
    }
}
```

#### Quartz

[相关链接](https://blog.csdn.net/w405722907/article/details/72458059)

使用Quartz实现定时任务。Quartz类似于linux中的`crontab`。它的使用需要三个要素。

1、做什么（JobDetail）2、什么时候做（Trigger） 3、任务调度器（schedule）

 **自定义Job**

```java
public class MyJob implements Job {
    @Override
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        System.out.println(new Date() + " working...");
        try {
            TimeUnit.SECONDS.sleep(50);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

**开启定时任务**

```java
public class QuartzScheduleExample {
    public static void main(String[] args) throws ParseException, SchedulerException {
        // 传入任务
        JobDetail jobDetail = new JobDetailImpl("myJob-1",
                "group", MyJob.class);
		// 定义触发器
        CronTrigger trigger = new CronTriggerImpl(
                "myJob-1", "group", "0/3 * * * * ?");
		// 调度工厂
        StdSchedulerFactory factory = new StdSchedulerFactory();
        Scheduler scheduler = factory.getScheduler();
        // 配置任务并执行
        scheduler.scheduleJob(jobDetail, trigger);
        scheduler.start();
    }
}

```

需要注意的是：

​		Quartz默认开启10个线程去处理定时任务，目的为了解决定时任务的执行时间大于定时时间的问题。

​		比如说，一个任务要执行50s，定时3秒执行一次，那么30秒后，10个线程都会处于执行状态。此时任务才会被推迟，直到第一个线程执行完后才会继续开始定时任务。

![](http://img.fosuchao.com/20200227154248.png)

## ThreadPoolExecutor

### 参数含义

![](http://img.fosuchao.com/20200227095709.png)

### 工作流程

![](http://img.fosuchao.com/20200227095734.png)

​		比如设置核心池线程数是5，任务队列数是5，最大线程数是10，那么当线程超过5个的时候，新的线程会被放到任务队列中，知道任务队列也被放满的时候，那么接着增加核心池中的线程数，知道最大线程数10的时候（核心池10个，队列5个），如果再有增加，则通过响应的饱和策略来处理。如果没有设置策略，则会抛出拒绝异常。

### 线程池的关闭

​		当我们的所有任务执行完的时候，发现进程不会停止，因为线程池是可以复用的，线程工作完回收变成空闲状态了。如果要停止，我们需要手动调用一些方法。

假如线程池中有二十个线程

#### shutdown

`流畅的关闭ThreadPoolExecutor，`停止接收外部submit 的工作，10个线程正在工作，10个线程空闲。空闲的线程会被interrupted，工作的线程会在工作完成（finish）后被interrupted。再退出（exit）

- 停止接收外部submit的任务
- 内部正在跑的任务和队列里等待的任务，会执行完
- 等到第二步完成后，才真正停止

#### shutdownNow

将线程池状态置为`STOP`。**企图**立即停止，事实上不一定，方法有返回值（List）.

- 停止接收外部submit任务
- 忽略队列中正在等待的任务（未执行）
- 尝试中断正在执行的任务
- 返回未执行的任务列表

#### awaitTermination

​		接收`timeout`和`TimeUnit`两个参数，用于**设定超时时间及单位**。当等待超过设定时间时，会监测ExecutorService是否已经关闭，若关闭则返回true，否则返回false。一般情况下**会和shutdown方法组合使用**。

```java
ExecutorService es = new Executors.newFixedThreadPool(10);
es.execute(task);
try{
	es.shutdown();
	if (!es.awaitTermination(10, TimeUnit.SECONDS)){
		// 20秒没有结束，强制关闭
		es.shutdownNow();
	}
} catch(Exception e) {}
```

