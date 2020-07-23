本文将要提到的线程及其相关内容，均是指 `Windows` 操作系统中的线程，不涉及其它操作系统。

## 核心概念

　　**优先级调度算法**

　　处理器是一个操作系统执行任务的工具，线程是一个操作系统执行任务的基本单位，处理器的数量决定了不可能所有线程都能同时得到执行。这就需要通过某种算法来进行任务高度。而 Windows 是一个抢占式的多任务操作系统，我们来看下维基百科对于抢占式的定义：

> In computing, preemption is the act of temporarily interrupting a task being carried out by a computer system, without requiring its cooperation, and with the intention of resuming the task at a later time. Such a change is known as a context switch. It is normally carried out by a privileged task or part of the system known as a preemptive scheduler, which has the power to preempt, or interrupt, and later resume, other tasks in the system.
>
> --- [Preemption (computing)](http://en.wikipedia.org/wiki/Preemption_(computing))

　　上面这段英文意味着在抢占式的操作系统中，执行任务的多个线程之间会因为某种因素互相争抢在处理器上运行的机会。而这种因素就是标题所说的 “优先级”。

 

> 线程是根据其优先级而调度执行的。 即使线程正在运行时中执行，所有线程都是由操作系统分配处理器时间片的。 用于确定线程执行顺序的调度算法的详细情况随每个操作系统的不同而不同。
>
> 在某些操作系统下，具有最高优先级（相对于可执行线程而言）的线程经过调度后总是首先运行。 如果具有相同优先级的多个线程都可用，则计划程序将遍历处于该优先级的线程，并为每个线程提供一个固定的时间片(段)来执行。 只要具有较高优先级的线程可以运行，具有较低优先级的线程就不会执行。 如果在给定的优先级上不再有可运行的线程，则计划程序将移到下一个较低的优先级并在该优先级上调度线程以执行。 如果此时具有较高优先级的线程可以运行，则具有较低优先级的线程将被抢先，并允许具有较高优先级的线程再次执行。 除此之外，当应用程序的用户界面在前台和后台之间移动时，操作系统还可以动态调整线程优先级。 其他操作系统可以选择使用不同的调度算法。
>
> --- [调度线程](http://msdn.microsoft.com/zh-cn/office/2k34xtf3(v=vs.95).aspx)

　　上面文字中提到优先级可以由操作系统动态调整。Windows 除了会在前后台切换的时候调整优先级还会为 I/O 操作动态提升优先级，或者使用 “饥渴” 的时间片分配策略来动态调整，如果有线程一直渴望得到时间片但是很长时间都没有获得时间片，Windows 就会临时将这个线程的优先级提高，并一次分配给2倍的时间片来执行，当用完2倍的时间片后，优先级又会恢复到之前的水平。

　　**线程运行状态**

![img](https://images0.cnblogs.com/i/30991/201407/201947106629585.jpg)

 

　　一个线程从开始到终止可能会有上述几种状态，这几种状态可以互相转换。（上图中的 “就绪” 指的是 runnable 状态，又称为 “ready to run” 状态，个人感觉翻译成 “就绪” 比 “可运行” 要来得明白）。

 

## Thread.Yeild

　　对上述概念有所了解后，我将正式介绍这三个方法的区别。

 

　　该方法是在 .Net 4.0 中推出的新方法，它对应的底层方法是 SwitchToThread。

　　Yield 的中文翻译为 “放弃”，这里意思是主动放弃当前线程的时间片，并让操作系统调度其它就绪态的线程使用**一个**时间片。但是如果调用 Yield，只是把当前线程放入到就绪队列中，而不是阻塞队列。如果没有找到其它就绪态的线程，则当前线程继续运行。

> Yielding is limited to the processor that is executing the calling thread. The operating system will not switch execution to another processor, even if that processor is idle or is running a thread of lower priority. If there are no other threads that are ready to execute on the current processor, the operating system does not yield execution, and this method returns false. 
>
> This method is equivalent to using platform invoke to call the native Win32 SwitchToThread function. You should call the Yield method instead of using platform invoke, because platform invoke bypasses any custom threading behavior the host has requested.
>
> --- [Thread.Yield Method](http://msdn.microsoft.com/en-us/library/system.threading.thread.yield(v=vs.110).aspx)

 

　　**优势**：比 Thread.Sleep(0) 速度要快，可以让低于当前优先级的线程得以运行。可以通过返回值判断是否成功调度了其它线程。

　　**劣势**：只能调度同一个处理器的线程，不能调度其它处理器的线程。当没有其它就绪的线程，会一直占用 CPU 时间片，造成 CPU 100%占用率。

 

 

## Thread.Sleep(0)

　　Sleep 的意思是告诉操作系统自己要休息 n 毫秒，这段时间就让给另一个就绪的线程吧。当 n=0 的时候，意思是要放弃自己剩下的时间片，但是仍然是就绪状态，其实意思和 Yield 有点类似。但是 Sleep(0) 只允许那些优先级相等或更高的线程使用当前的CPU，其它线程只能等着挨饿了。如果没有合适的线程，那当前线程会重新使用 CPU 时间片。

 

> If you specify 0 milliseconds, the thread will relinquish the remainder of its time slice but remain ready.
>
> --- [Sleep Function](http://msdn.microsoft.com/en-us/library/windows/desktop/ms686298(v=vs.85).aspx)

 

　　**优势**：相比 Yield，可以调度任何处理器的线程使用时间片。

　　**劣势**：只能调度优先级相等或更高的线程，意味着优先级低的线程很难获得时间片，很可能永远都调用不到。当没有符合条件的线程，会一直占用 CPU 时间片，造成 CPU 100%占用率。

 

## Thread.Sleep(1)

 　　该方法使用 1 作为参数，这会强制当前线程放弃剩下的时间片，并休息 1 毫秒（因为不是实时操作系统，时间无法保证精确，一般可能会滞后几毫秒或一个时间片）。但因此的好处是，所有其它就绪状态的线程都有机会竞争时间片，而不用在乎优先级。

　　**优势**：可以调度任何处理器的线程使用时间片。无论有没有符合的线程，都会放弃 CPU 时间，因此 CPU 占用率较低。

　　**劣势**：相比 Thread.Sleep(0)，因为至少会休息一定时间，所以速度要更慢。 

 

## 实验告诉你：单一线程

　　**测试环境**：Windows 7 32位、VMWare workstation、**单处理器单核芯**

　　**开发环境**：Visual Studio 2012、控制台项目、**Release 编译后将 exe 拷贝到虚拟机后运行**

 

　　**Thread.Yeild**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
static void Main()    
{
        string s = ""; 
        while (true)
        {      
              s = DateTime.Now.ToString(); //模拟执行某个操作
              Thread.Yeild(0);
        }    
}        
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　执行效果 

![img](https://images0.cnblogs.com/i/30991/201407/201950121789864.gif)

 

　　**Thread.Sleep(0)**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
static void Main()    
{
        string s = "";
        while (true)
        {
            s = DateTime.Now.ToString(); 
            Thread.Sleep(0);
        }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　执行效果 

![img](https://images0.cnblogs.com/i/30991/201407/201951266931267.gif)

 

　　**Thread.Sleep(1)**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
static void Main()    
{
        string s = "";
        while (true)
        {
             s = DateTime.Now.ToString();
             Thread.Sleep(1);
        }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　执行效果 

![img](https://images0.cnblogs.com/i/30991/201407/201952528035959.gif)

 

　　通过上述三个实验，很明显说明 Thread.Sleep(1) 对于解决资源 100% 占用是有明显效果的。

 

## 实验告诉你：多线程（同优先级）

　　我的实验方法很简单，就是通过 while 让该线程不断的执行，为了让大家一目了然两个线程的交替，通过向控制台输出不同的字符串来验证。

　　while 语句执行速度相当快，所以必须所以加上一些代码来浪费CPU时间，以免大家只能看到不断的刷屏。

 

　　辅助代码

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private static void WasteTime()    
{
        // 耗时约 200ms
        DateTime dt = DateTime.Now;
        string s = "";
        while (DateTime.Now.Subtract(dt).Milliseconds <= 200)
        {
            s = DateTime.Now.ToString(); //加上这句,防止被编译器优化
        }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　**不使用任何方法** 

　　让线程们自己争用 CPU 时间。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
static void Main(string[] args)    
{
        Thread t = new Thread(() =>
        {
            while (true)
            {
                 WasteTime();
                 Console.WriteLine("Thread 1 ==========");
            }
        });
        t.IsBackground = true;
        Thread t2 = new Thread(() =>
        {
            while (true)
            {
                WasteTime();
                Console.WriteLine("Thread 2");
            }
        });

        t2.IsBackground = true;
        t2.Start();
        t.Start();
        Console.ReadKey();

        t.Abort();
        t2.Abort();
        Console.ReadKey();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 


　　执行效果 

![img](https://images0.cnblogs.com/i/30991/201407/201957208189046.gif)

 

　　**Thread.Yeild**

　　修改第一个线程的方法体，加入Thread.Yield。其余代码不变。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
Thread t = new Thread(() =>
{
        while (true)
        {
            WasteTime();
            Thread.Yield();
            Console.WriteLine("Thread 1 ==========");
        }
});
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　执行效果 

![img](https://images0.cnblogs.com/i/30991/201407/201958418659086.gif)

 

　　**Thread.Sleep(0)**

　　仿照 Thread.Yield，只不过用 Thread.Sleep(0）替换。

 

　　执行效果

![img](https://images0.cnblogs.com/i/30991/201407/201959197091357.gif)

 

　　**Thread.Sleep(1)**

　　仿照 Thread.Yield，用 Thread.Sleep(1）替换。

 

　　执行效果

![img](https://images0.cnblogs.com/i/30991/201407/202000062242580.gif)

 

 

　　从上面的示例看出，未使用 Thread 的这几个方法(Yield，Sleep)的例子，两个同等优先级的线程可以获得差不多完全一样的CPU时间。

　　而使用了 Thread 方法的那几个例子或多或少都让另一个线程多获取了些时间片，但是不同的方法执行效果差得并不多。这是因为它们所让出的时间片往往都只是几十毫秒的事情，这么短的时间，对于我的测试代码来说很难顺利扑捉每个瞬间。

![img](https://images0.cnblogs.com/i/30991/201407/202001255066192.jpg)

Thread2 要得到更多的时间片

 

 

## 实验告诉你：多线程（不同优先级）

　　先来调整下代码，修改 Thread 1，让它的优先级变成 “AboveNormal”。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
Thread t = new Thread(() =>
{
        while (true)
        {
            WasteTime();
             Console.WriteLine("Thread 1 ==========");
        }
});

t.Priority = ThreadPriority.AboveNormal; // 加入这句话
t.IsBackground = true;
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　**不使用任何方法** 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
Thread t = new Thread(() =>
{
        while (true)
        {
            WasteTime();
            Console.WriteLine("Thread 1 ==========");
        }    
});    

t.Priority = ThreadPriority.AboveNormal;   
t.IsBackground = true;      

Thread t2 = new Thread(() =>    
{
        while (true)
        {
            WasteTime();
            Console.WriteLine("Thread 2");
        }    

});   

t2.IsBackground = true;
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　执行效果 

![img](https://images0.cnblogs.com/i/30991/201407/202003395685336.gif)

　　从实验中可以表明，低优先级的几乎很少有使用时间片的时候。

 

　　**Thread.Yeild**

　　修改 Thread 的方法体

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
Thread t = new Thread(() =>    
{
        while (true)
        {
            WasteTime();
            Console.WriteLine("Thread 1 ========== {0}",Thread.Yield());
        }    
});
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　执行效果 

![img](https://images0.cnblogs.com/i/30991/201407/202004362562850.gif)

 

　　使用了 Yeild 之后，很明显低优先级的线程现在也能够获得CPU时间了。

 

　　**Thread.Sleep(0）**

　　修改方法体

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
Thread t = new Thread(() =>    
{
        while (true)
        {
            WasteTime();
            Console.WriteLine("Thread 1 ==========");
            Thread.Sleep(0);
        }    
});
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　执行效果 

![img](https://images0.cnblogs.com/i/30991/201407/202005400534360.gif)

 

　　有没有发现低优先级的线程又一次被无情的抛弃了？

 

　　**Thread.Sleep(1)**

　　仿照 Sleep(0)，用 Sleep(1）替换。

 

　　执行效果

![img](https://images0.cnblogs.com/i/30991/201407/202006105845865.gif)

 

　　通过上面的实验，我想你应该已经比较清楚了在不同优先级的情况下，哪个方法更适用于去切换线程。

 

## 本人观点

　　有鉴于 Thread.Sleep(0) 的表现，本人认为应该无情的把它取缔掉。至于是用 Thread.Yeild，还是 Thread.Sleep(n) (n>0)，那就根据实际情况吧。欢迎大家补充~