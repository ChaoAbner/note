# fork()作用和原理

Unix中的fork（）没有使用COW技术，所以执行较慢。

![1584601183988](F:\typoraImg\1584601183988.png)

而linux对fork()进行了一些改进，增加了COW技术。

实际上，linux执行fork()的时候，不会为子进程完全开辟出新的空间来复制父进程中的数据或地址空间。而是跟父进程共享一块物理内存。

当子进程需要修改里面的内容的时候，这时才开辟新的内存空间，并将新的数据写入。

这样减小了内存的消耗，并且加快了fork（）执行的速度。

在linux中fork的速度还是比较客观的。



**1、fork函数原型**

```c
#include<unistd.h>
pid_t fork(void)
```


返回值：
pid_t 是进程描述符，实质就是一个int，如果fork函数调用失败，返回一个负数，调用成功则返回两个值：0和子进程ID。
函数功能：
以当前进程作为父进程创建出一个新的子进程，并且将父进程的所有资源拷贝给子进程，这样子进程作为父进程的一个副本存在。父子进程几乎时完全相同的，但也有不同的如父子进程ID不同。
需要注意的是：
当fork系统调用成功时，它会返回两个值：一个是0，另一个是所创建的新的子进程的ID（>0）。当fork成功调用后此时有两个数据相同的父子进程，我们可以通过fork的返回值来判断接下来程序是在执行父进程还是子进程。

返回值：
pid_t 是进程描述符，实质就是一个int，如果fork函数调用失败，返回一个负数，调用成功则返回两个值：0和子进程ID。
函数功能：
以当前进程作为父进程创建出一个新的子进程，并且将父进程的所有资源拷贝给子进程，这样子进程作为父进程的一个副本存在。父子进程几乎时完全相同的，但也有不同的如父子进程ID不同。
需要注意的是：
当fork系统调用成功时，它会返回两个值：一个是0，另一个是所创建的新的子进程的ID（>0）。当fork成功调用后此时有两个数据相同的父子进程，我们可以通过fork的返回值来判断接下来程序是在执行父进程还是子进程。

```c
id==0：执行子进程
id>0：在父进程中执行
id<0：fork函数调用失败
```

**2、fork函数的底层实现原理**
fork()系统调用通过复制一个现有进程来创建一个全新的进程。进程被存放在一个叫做任务队列的双向循环链表当中，链表当中的每一项都是类型为task_struct称为进程描述符的结构，也就是我们写过的进程PCB.

Tips：内核通过一个位置的进程标识值或PID来标识每一个进程。//最大值默认为32768，short int短整型的最大值.，他就是系统中允许同时存在的进程最大的数目。
可以到目录 /proc/sys/kernel中查看pid_max：

当进程调用fork后，当控制转移到内核中的fork代码后，内核会做4件事情:
1、分配新的内存块和内核数据结构给子进程

2、将父进程部分数据结构内容拷贝至子进程

3、添加子进程到系统进程列表当中

4、fork返回，开始调度器调度
在这里有一个疑问，那么fork函数在底层到底做了什么呢？
Linux平台通过clone()系统调用实现fork()。 fork()，vfork()和clone()库函数都根据各自需要的参数标志去调用clone()，然后由clone()去调用do_fork()， 再然后do_fork()完成了创建中的大部分工作，该函数调用copy_process().做最后的那部分工作。具体的图解是借用一位写的非常好的大神的博客https://blog.csdn.net/Dawn_sf/article/details/78709839

那么fork函数为什么是一次调用，却返回了两次呢？

当程序执行到下面的语句： pid=fork();
由于在复制时复制了父进程的堆栈段，所以两个进程都停留在fork函数中，等待返回。因此fork函数会返回两次，一次是在父进程中返回，另一次是在子进程中返回，这两次的返回值是不一样的。
fork调用的一个奇妙之处就是它仅仅被调用一次，却能够返回两次，它可能有三种不同的返回值：

1）在父进程中，fork返回新创建子进程的进程ID；
2）在子进程中，fork返回0；
3）如果出现错误，fork返回一个负值。

我们可以通过fork返回的值来判断当前进程是子进程还是父进程。通俗的解释，可以这样看待：“其实就相当于链表，进程形成了链表，父进程的fork函数返回的值指向子进程的进程id, 因为子进程没有子进程，所以其fork函数返回的值为0.

调用fork之后，数据、堆、栈有两份，代码仍然为一份但是这个代码段成为两个进程的共享代码段都从fork函数中返回。当父子进程有一个想要修改数据或者堆栈时，两个进程真正分裂。

而且我们还需要注意的是：子进程的代码是从fork处执行的，fork底层实现采用了COW（copy_on_write）技术—-写入时拷贝
写时拷贝思想：父进程和子进程共享页帧而不是复制页帧。然而，只要页帧被共享，它们就不能被修改，即页帧被保护。无论父进程还是子进程何时试图写一个共享的页帧，就产生一个异常，这时内核就把这个页复制到一个新的页帧中并标记为可写。原来的页帧仍然是写保护的：当其他进程试图写入时，内核检查写进程是否是这个页帧的唯一属主，如果是，就把这个页帧标记为对这个进程是可写的。
**3、vfork函数**
既然讲到了fork函数，那么我们很容易想到另一个创建进程的函数vfork：
vfork最初是因为fork没有实现COW机制，而很多情况下fork之后会紧接着exec，而exec的执行相当于之前fork复制的空间全部变成了无用功，所以设计了vfork。
函数原型

```c
#include <sys/types.h>
#include <unistd.h>

pid_t vfork(void);
```


函数解释：
（1）vfork函数用于创建一个子进程，而子进程和父进程共享地址空间，fork的子进程具有独立的地址空间
（2）vfork保证子进程先运行，在它调用exec或（exit）之后父进程才可能被调度运行
注意的是：
vfork创建的子进程结束时需要调用exit（）函数退出，如果没有调用该函数时会出现短错误的；这是因为在函数栈上面，子进程运行结束了，main的函数栈被子进程释放了，然后父进程在使用的时候，就访问不到了，所以一旦vfork出子进程，退出的时候需要使用exit来结束。

函数解释：
（1）vfork函数用于创建一个子进程，而子进程和父进程共享地址空间，fork的子进程具有独立的地址空间
（2）vfork保证子进程先运行，在它调用exec或（exit）之后父进程才可能被调度运行
注意的是：
vfork创建的子进程结束时需要调用exit（）函数退出，如果没有调用该函数时会出现短错误的；这是因为在函数栈上面，子进程运行结束了，main的函数栈被子进程释放了，然后父进程在使用的时候，就访问不到了，所以一旦vfork出子进程，退出的时候需要使用exit来结束。

**4、fork和vfork的区别**
（1）fork：子进程拷贝父进程的代码段和数据段
vfork：子进程和父进程共享代码段和数据段
（2）fork中父子进程的先后运行次序不定
vfork：保证子进程先运行，子进程exit后父进程才开始被调度运行
（3） vfork （）保证子进程先运行，在她调用exec 或exit 之后父进程才可能被调度运行。如果在 调用这两个函数之前子进程依赖于父进程的进一步动作，则会导致死锁。
（4）就算fork实现了写时拷贝，但其效率仍然没有vfork高，但是vfork在一般平台上都存在问题，所以一般不推荐使用

