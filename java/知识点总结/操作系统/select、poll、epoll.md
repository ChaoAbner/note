# select、poll、epoll

- 阻塞 I/O（blocking IO）
- 非阻塞 I/O（nonblocking IO） 返回error
- I/O 多路复用（ IO multiplexing）select、poll、epoll
- 信号驱动 I/O（ signal driven IO）
- 异步 I/O（asynchronous IO）

linux中万物皆文件，不管是设备还是其它的应用程序进程，都有一个文件描述符（fd）与之对应。

文件描述符用来表示打开的文件。



​		epoll，poll和select都是linux下I/O多路复用的实现，可以实现单线程管理多个连接，select是基于轮询的，轮询连接的状态，返回I/O状态，poll和select的原理基本相同，只是poll没有最大连接数的限制，因为它是基于链表的，而select是基于数组的，有最大连接数的限制。epoll和那两者的区别是，epoll不是基于轮询的检查，而是为每个fd注册回调，I/O准备好时，会执行回调，效率比select和poll高很多。


## select

![1584171047848](F:\typoraImg\1584171047848.png)

关键字：bitmap（存储文件描述符，用一位来表示一个fd）、select（事件监听）	

​		rset指的是一个bitmap，用来表示被占用的文件描述符。就是说bitmap中已经包含了fds的所有信息。

​		判断所有的fds是否有数据操作（read/write...）等等，需要将bitmap从用户态拷贝到内核态。然后从内核态来判断的。

​		如果判断到有数据来，会将FD置位（bitmap中），然后select返回。在进行读写操作。

## poll

![1584171764283](F:\typoraImg\1584171764283.png)

关键字：pollfd（fd，events，revents）、pollfds（pollfd集合）、poll（事件监听）

​		poll解决了select的第一和第二个缺点。但是用户态和内核态依然不共享数据。

​		声明一个结构体pollfd，包含**文件描述符fd**，**关注事件events**，**标志位revents**（用于标志是否有数据），用于代替select中的bitmap。

​		revents是可以重用的。每次进行了数据读取处理的时候，会把revents标识位置为0。一个循环下来，revents仍然是最初的样子。

## epoll

关键字：epoll_event（存储fd，event）、epoll_create（）、epoll_wait（监听事件）

![1584171380547](F:\typoraImg\1584171380547.png)

用户态和内核态共享内存。不会进行数据拷贝。

### **epoll的LT和ET的区别**

#### LT

​		LT：**水平触发**，效率会低于ET触发，尤其在大并发，大流量的情况下。但是LT对代码编写要求比较低，不容易出现问题。

​		LT模式服务编写上的表现是：**只要有数据没有被获取，内核就不断通知你**，**因此不用担心事件丢失的情况**。

#### ET



​		ET：**边缘触发**，效率非常高，在并发，大流量的情况下，会比LT少很多epoll的系统调用，因此效率高。但是对编程要求高，需要细致的处理每个请求，否则容易发生丢失事件的情况。
​		下面举一个列子来说明LT和ET的区别（都是非阻塞模式，阻塞就不说了，效率太低）：
采用LT模式下， 如果accept调用有返回就可以马上建立当前这个连接了，再epoll_wait等待下次通知，和select一样。
​		但是对于ET而言，如果accpet调用有返回，除了建立当前这个连接外，不能马上就epoll_wait还需要继续循环accpet，直到返回-1，且errno==EAGAIN，TAF里面的示例代码：

## **select/epoll的区别**

​		**select**的特点：select 选择句柄的时候，是遍历所有句柄，**也就是说句柄有事件响应时，select需要遍历所有句柄才能获取到哪些句柄有事件通知**，**效率非常低**。但是如果连接很少的情况下， select和epoll的LT触发模式相比， 性能上差别不大。
这里要多说一句，**select支持的句柄数是有限制的， 同时只支持1024个**，这个是句柄集合限制的，如果超过这个限制，很可能导致溢出，而且非常不容易发现问题， TAF就出现过这个问题， 调试了n天，才发现：）当然可以通过修改linux的socket内核调整这个参数。
​		**epoll**的特点：**epoll对于句柄事件的选择不是遍历的，是事件响应的**，就是句柄上事件来就马上选择出来，不需要遍历整个句柄链表，因此效率非常高，内核将句柄用红黑树保存的。
​		对于epoll而言还有ET和LT的区别，**LT表示水平触发**，**ET表示边缘触发**，两者在性能以及代码实现上差别也是非常大的。