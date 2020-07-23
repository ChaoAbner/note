# TCP中的TIME_WAIT

## 为什么要有TIME_WAIT?

![](https://upload.wikimedia.org/wikipedia/commons/thumb/5/55/TCP_CLOSE.svg/2000px-TCP_CLOSE.svg.png)

TIME_WAIT是TCP主动关闭连接一方的一个状态，TCP断开连接的时序图如下：

当主动断开连接的一方（Initiator）发送FIN包给对方，且对方回复了ACK+FIN，然后Initiator回复了ACK后就进入TIME_WAIT状态，一直将持续2MSL后进入CLOSED状态。

那么，我们来看如果Initiator不进入TIME_WAIT状态而是直接进入CLOSED状态会有什么问题？

考虑这种情况，服务器运行在80端口，客户端使用的连接端口是12306，数据传输完毕后服务端主动关闭连接，但是没有进入TIME_WAIT，而是直接计入CLOSED了。这时，客户端又通过同样的端口12306与服务端建立了一个新的连接。假如上一个连接过程中网络出现了异常，导致了某个包重传并延时到达了服务端，这时服务端就无法区分这个包是上一个连接的还是这个连接的。所以，主动关闭连接一方要等待**2MSL**，然后才能CLOSE，**保证连接中的IP包都要么传输完成**，要么被丢弃了。

## MSL

MSL全称是maximum segment lifetime，最长分节生命期。MSL是任何IP数据报能够在因特网存活的最长时间。我们知道，这个时间是有限的，因为每个数据报都含有一个限跳（hop limit）的8位字段，它的最大值是255（简单的讲就是不同经过超过255个路由器）。尽管这个跳数限制而不是真正的时间限制，我们仍然假设最大限跳的分组在网络中存在的时间不可能超过MSL秒。

MSL的具体值通常为30秒或者是2分钟。


## TIME_WAIT会带来什么问题

系统中TIME_WAIT的连接数很多，会导致什么问题呢？这要分别针对客户端和服务器端来看的。

首先，如果是客户端发起了连接，传输完数据然后主动关闭了连接，这时这个连接在客户端就会处于TIMEWAIT状态，同时占用了一个本地端口。如果客户端使用短连接请求服务端的资源或者服务，客户端上将有大量的连接处于TIMEWAIT状态，占用大量的本地端口。最坏的情况就是，本地端口都被用光了，这时将无法再建立新的连接。

**针对这种情况，对应的解决办法有2个：**

1. 使用长连接，如果是http，可以使用keepalive
2. 增加本地端口可用的范围，比如Linux中调整内核参数：net.ipv4.ip_local_port_range

对于服务器而已，由于服务器是被动等待客户端建立连接的，因此即使服务器端有很多TIME_WAIT状态的连接，也不存在本地端口耗尽的问题。大量的TIME_WAIT的连接会导致如下问题：
1. 内存占用：因为每一个TCP连接都会有占用一些内存。
2. 在某些Linux版本上可能导致性能问题，因为数据包到达服务器的时候，内核需要知道数据包是属于哪个TCP连接的，在某些Linux版本上可能会遍历所有的TCP连接，所以大量TIME_WAIT的连接将导致性能问题。不过，现在的内核都对此进行了优化(待确认)。

那系统中处于TIME_WAIT状态的TCP连接数有上限吗？有的，这是通过net.ipv4.tcp_max_tw_buckets参数来控制的，默认值为180000。当超过了以后，系统就开始关闭这些连接，同时会在系统日志中打印日志。此时，可以将这个值调大一些，从这个参数的默认值就可以看出，对服务器而已，处于TIME_WAIT状态的TCP连接多点也没有什么关系，只是多占用些内存而已。

## 常见的TIMEWAIT错误参数

如果用TIME_WAIT作为关键字到网络上搜索，会得到很多关于如何减少TIME_WAIT数量的建议，其中有些建议是有错误或者有风险的，列举如下：

net.ipv4.tcp_syncookies = 1，这个参数表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击。这个和TIME_WAIT没有什么关系。
net.ipv4.tcp_tw_reuse = 1，这个参数表示重用TIME_WAIT的连接，重用的条件是TCP的4元组（源地址、源端口、目标地址、目标端口）要完全一致，而且开启了net.ipv4.tcp_timestamps，且新建立连接的使用的timestamp要大于当前连接的timestamp。所以，开启了这个参数对减少TIME_WAIT的TCP连接有点用，但条件太苛刻，所以实际用处不大。
net.ipv4.tcp_tw_recycle = 1，这个参数表示开启TIME_WAIT回收功能，开启了这个参数后，将大大减小TIME_WAIT进入CLOSED状态的时间。但是开启了这个功能了风险很大，可能会导致处于NAT后面的某些客户端无法建立连接。因为，开启这个功能后，它要求来自同一个IP的TCP新连接的timestamp要大于之前连接的timestamp。

## TIMEWAIT的“正确”处理方法

1. TIME_WAIT状态的设计初衷是为了保护我们的。服务端不必担心系统中有几w个处于TIME_WAIT状态的TCP连接。可以调大net.ipv4.tcp_max_tw_buckets这个参数。
2. 使用短连接的客户端，需要关注TIME_WAIT状态的TCP连接，建议是采用长连接，同时调节参数net.ipv4.ip_local_port_range，增加本地可用端口的范围。
3. 可以开启net.ipv4.tcp_tw_reuse这个参数，但是实际用处有限。
4. 不要开启net.ipv4.tcp_tw_recycle这个参数，它带来的问题比用处大。
5. 某些Linux的发行版可以调节TIME_WAIT到CLOSED的等待时间(比如Ali的Linux内核提供了参数net.ipv4.tcp_tw_timeout
)，可以稍微调小一点这个参数。
