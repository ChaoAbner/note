# Future

## 含义

Future，顾名思义，Future代表一个未来的凭证，用于异步的获取任务的结果。

通常，我们提交线程任务的时候都是等到线程处理完成任务然后再同步获取结果，而当遇到一些场景的时候，这种同步的方法就变得不再适用了。

举个例子：小明早上准备去上班，但是刚好今天是老婆的生日，需要先去楼下蛋糕店订个蛋糕，等到下班了再回来取。正常的流程应该是下图

![](http://img.fosuchao.com/20200305192624.png)

这种正常的流程，我们就称之为Future，属于异步获取结果的范畴。

如果是使用同步的话，小明等到蛋糕做好的时候才去上班，可能小明的工资已经被扣光了噢。

![](http://img.fosuchao.com/20200305202028.png)

同步方法显然在这种场景是很不合理的，当前有更重要的任务去做，但是又要等待不太重要的结果，于是造成了一些不必要的消耗和浪费。

而使用Future就可以很好的处理这样的异步任务。

## 方法

Future中共有5个方法：

![](http://img.fosuchao.com/20200305193143.png)

- cancel(boolean)：试图取消一个任务，如果任务已经完成或者已经被取消了还有其它一些原因导致无法取消，这就将导致取消失败。否则，**此任务尚未启动，这个任务就会取消运行**。如果任务已经开始，然后会根据mayInterruptIfRunning参数确定执行该任务的线程是否应该被中断。

- get()

  如果任务完成，直接返回结果，如果还未完成，则**阻塞**获取任务的结果。

- get(long, TimeUnit)

  get的超时版本，如果超过了时间，则不再阻塞等待结果。

- isCancelled()

  判断任务是否被取消。

- isDone()

  判断任务是否已经完成。

## 使用

通常我们会使用Future的实现类`FutureTask`来实现Future的异步调用过程。

首先实现Callable接口，Callable不同于Runnable，Callable是由返回值的任务。

```java
class Demo implements Callable<Integer> {

    @Override
    public Integer call() throws Exception {
        // 线程中执行的逻辑
        System.out.println("计算结果中。。。");
        Thread.sleep(2000);
        return 1;
    }
}
```

使用一个FutureTask来接受一个Callable任务，返回一个Future。

```java
public static void main(String[] args) throws Exception{
    Callable demo = new Demo();
    FutureTask<Integer> integerFutureTask = new FutureTask<Integer>(demo);
    new Thread(integerFutureTask).start();
    doOtherTask();
    long start = System.currentTimeMillis();
    System.out.println("线程执行完成，返回结果为：" + integerFutureTask.get());
    long end = System.currentTimeMillis();
    System.out.println("取回结果的时间为：" + (end - start) + "ms");
}

public static void doOtherTask() throws InterruptedException {
    System.out.println("正在做干别的事情");
    Thread.sleep(1000);
}
```

代码的执行结果如下：

```
正在做干别的事情
计算结果中。。。
线程执行完成，返回结果为：1
取回结果的时间为：1001
```

我们在doOtherTask()函数中花费了1秒左右的时间，最后取结果离doOtherTask()执行完成间隔了1001ms，说明doOtherTask()并没有阻塞我们的异步任务，而是并发的取做这个任务，直到我们调用get的时候，阻塞将结果返回。总共其实还是花费了我们任务需要的2秒左右。

## 原理

我们通过`FutureTask`的代码来分析Future的实现原理。

`FutureTask`使用一些整形数字来代表任务运行时的状态`state`

```java
private volatile int state;
private static final int NEW          = 0;
private static final int COMPLETING   = 1;
private static final int NORMAL       = 2;
private static final int EXCEPTIONAL  = 3;
private static final int CANCELLED    = 4;
private static final int INTERRUPTING = 5;
private static final int INTERRUPTED  = 6;
```

可能的状态变化过程：

* NEW -> COMPLETING -> NORMAL
* NEW -> COMPLETING -> EXCEPTIONAL
* NEW -> CANCELLED
* NEW -> INTERRUPTING -> INTERRUPTED

FutureTask中实现Future的get方法的核心代码是awaitDone(),等待任务结束的函数。

![](http://img.fosuchao.com/20200305195748.png)

**awaitDone方法**

![](http://img.fosuchao.com/20200305200813.png)

采用**自旋**的方法不断的进行状态的判断和尝试根据情况更新状态，这就解释了为什么get方法为什么是阻塞的。

如果没有异常或者其它影响。会将当前线程park住，LockSupport实际使用了`Unsafe`类中的`part`方法

![](http://img.fosuchao.com/20200305201047.png)

**run方法**

被Thread真正执行的run方法。

```java
public void run() {
    if (state != NEW ||
        !RUNNER.compareAndSet(this, null, Thread.currentThread()))
        return;
    try {
        Callable<V> c = callable;
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
                result = c.call();	// 调用传入的callable中的call方法，获取返回结果。
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                setException(ex);
            }
            if (ran)
                set(result);	// 设置任务完成返回的结果，set方法中会将state设置为NORMAL
        }
    } finally {
        runner = null;
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
```

当run方法执行成功后，即结果设置成功。state会被设置为NORMAL，此时awaitDone方法会返回当前的state，get会获取到最新的state交给report进行处理返回。

![](http://img.fosuchao.com/20200305201505.png)

如果state为NORMAL则返回结果，否则会抛出异常。

## 总结

以上介绍了Future的使用方法和工作原理。实现了异步调用返回的一个接口，其实现方式很值得我们去学习和思考。