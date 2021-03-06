## 网络OSI模型

![1564299438415](C:\Users\叶超\AppData\Roaming\Typora\typora-user-images\1564299438415.png)

### 第七层 应用层

应用层（Application Layer）提供为应用软件的而设的接口，以设置与另一个软件之间的通信。如：HTTP, SMTP, FTP, SHH, POP3, HTML等。

### 第六层 表达层

表达层（Presentation Layer）把数据转换为能与接收者的系统格式兼容并适合传输的格式。

### 第五层 会话层

会话层（Session Layer）负责在数据传输中设置和维护计算机网络中两台计算机之间的通信连接。

### 第四层 传输层

传输层（Transport Layer）把传输表头加至数据以形成数据包。传输表头包含了所使用的协议等发送信息。如：传输控制协议（TCP）等。

### 第三层 网络层

网络层（Network Layer）决定数据的路径选择和转寄，将网络表头加至数据包，以形成分组。网络表头包含网络数据。例如：互联网协议（IP）等。

### 第二层 数据链路层

数据链路层（Data Link Layer）负责网络寻址、错误侦测和改错。当表头和表尾被加至数据包时，会形成帧。数据链表头是包含了物理地址（MAC）和错误侦测及改错的方法。数据链表尾是一串指示数据包末端的字符串。例如以太网、无线局域网和通用分组无线服务等。

### 第一层 物理层

物理层（Physical Layer）在局部局域网上传送数据帧，它负责管理计算机通信设备和网络媒体之间的互通。包括了针脚、电压、线缆规范、集线器、中继器、网卡、主机接口卡等。



## 传输类型的种类

1、面向有连接型
2、面向无连接型

TCP/IP协议一般会被认为是两个协议，其实他是一个协议群。包含了很多协议包括
应用协议：http smtp ftp等
传输协议： TCP UDP
路由控制协议： RIP OSPF BGP
网际协议: IP ICMP ARP



路由控制器：记录着网络地址与下一步应该将数据包转给发送至路由器的地址



## 网络拓扑结构

​		拓扑结构是指网络上设备和连接介质是如何连接的。所有的网络设计都来源于3种基本的拓扑结构：**总线型、星型、环型**。

|      |                                                              |
| ---- | ------------------------------------------------------------ |
|      | ![img](file:///C:\Users\叶超\AppData\Local\Temp\ksohtml\wps199.tmp.png) |

|      |                                                              |
| ---- | ------------------------------------------------------------ |
|      | ![img](file:///C:\Users\叶超\AppData\Local\Temp\ksohtml\wps63A0.tmp.png) |

​		星型（树型）网络具有分层的结构化特点。如果星型网络上与集线器连接的一台计算机或电缆发生故障，那么只是此台计算机不能够发送或接收网络数据，网络的其余部分仍然能够正常工作。另外，每个节点跟集线器都有单独的传输线路，因此采用星型结构时，这些节点具有同时跟集线器通信的条件，传输速度可以很高。可以这就是为什么它现在被大量采用的主要原因。

​		实际工作的网络，经常采用多级星型结构，以便提供足够的端口连接所有的设备。

## DNS

### 定义

可以有效管理IP地址与主机名之间的对应关系的系统。

### DNS的查询机制

![1564300539625](C:\Users\叶超\AppData\Roaming\Typora\typora-user-images\1564300539625.png)

​		解析器为了调查IP地址，向域名服务器进行查询处理。接受这个查询请求的域名服务器首先会在自己的数据库进行查找。如果有该域名所对应的IP地址就返回。如果没有，则域名服务器再向上一层根域名服务器进行查询处理。因此，如上图所示，从根开始对这棵树按照顺序进行遍历，知道找到制定的域名服务器，并由这个域名服务器返回对应的IP地址等数据。

​		解析器和域名服务器将最新了解到的信息保存在缓存里。以便减少之后每次查询时的性能消耗。





ARP：一种解决地址问题的协议。一台主机通过目标IP地址为线索，通过广播发送ARP请求，目标主机收到请求后，将自己的mac地址通过ARP相应发送回请求主机。即完成数据链路层的通信。

![1564300945905](C:\Users\叶超\AppData\Roaming\Typora\typora-user-images\1564300945905.png)

​		某主机通过广播发送一个APR请求包，可以被同一个链路上的所有主机和路由器收到并进行解析，如果ARP请求包中的目标与自己的IP地址一致，便将自己的MAC地址打包为ARP相应返回给该主机。

### IP地址和MAC地址缺一不可吗？

答案是肯定的。如果两个不在同一链路的主机想要知道对方的MAC地址。只是通过广播的方式并不能找到对方的位置。因为中间的路由器隔断了两个网络。此时，主机A就必须先将数据发送给路由器C的MAC地址C1，再由路由器转发到相应的主机。

![1564301500783](C:\Users\叶超\AppData\Roaming\Typora\typora-user-images\1564301500783.png)

## RARP

​		RARP是将ARP反过来，由MAC地址定位IP地址的一种协议。例如将打印机服务器等小型嵌入式设备接入到网络时就经常用到。

​		平时我们通过个人电脑设置IP地址，也可以通过DHCP自动分配获取IP地址。然而，对于使用嵌入式设备时候，会遇到没有任何输入接口或无法通过DHCP动态获取IP地址的情况。这种情况下，就要使用RARP。

![1564301668345](C:\Users\叶超\AppData\Roaming\Typora\typora-user-images\1564301668345.png)

## ICMP

​		ICMP主要功能包括：确认IP包是否成功送达目标地址，通知在发送过程当中IP包被废弃的具体原因，改善网络设置等。有这些功能，即可获得网络是否正常、设置是否有误以及设备有何异常等信息，从而便于进行网络上的问题诊断。对IP起到辅助的作用。

​		ICMP的消息大致可以分为两类：一类是通过出错原因的错误消息，另一类是用于诊断的查询消息。

![1564302047812](C:\Users\叶超\AppData\Roaming\Typora\typora-user-images\1564302047812.png)

## DHCP

### DHCP的作用

​		如果逐一为主机设置IP地址会非常的繁琐，特别是现在各种移动设备，每移动到一个新的地方，都要重新设置IP地址。

​		DHCP则实现了IP地址的自动设置、统一管理IP地址分配。有了DHCP，计算机只要连接到了网络，就能进行TCP/IP通信。

![1564302453837](C:\Users\叶超\AppData\Roaming\Typora\typora-user-images\1564302453837.png)

### DHCP的工作机制

​		架设一台DHCP服务器。然后将DHCP所要分配的IP地址设置到服务器上。此外，还需将相应的子网掩码、路由控制信息以及DNS服务器的地址等设置到服务器上。从DHCP中获取IP的流程如下图。

![1564302678099](C:\Users\叶超\AppData\Roaming\Typora\typora-user-images\1564302678099.png)

​		若DHCP服务器发生故障，将无法自动分配IP地址，从而导致网段内所有主机之间无法进行TCP/IP通信。所以通常会架设两台或两台以上的DHCP服务器。

​		为了检查所要分配的IP地址以及已经分配了的IP地址是否可用，DHCP服务器或DHCP客户端必须具备以下功能：

- DHCP服务器

  在分配IP地址前发送ICMP会送请求包，确认没有返回应答。

- DHCP客户端

  针对从DHCP哪里获得的IP地址发送ARP请求包，确认没有应答。

## NAT

### NAT定义

​		NAT是用于在本地网络中使用私有地址，在连接互联网时转而使用全局IP地址的技术。除转换IP地址外，还出现了可以转换TCP、UDP端口号的NAPT技术（通常说NAT即指的是NAPT），由此可以实现用一个全局IP地址与多个主机的通信。

​		NAT实际是为正在面临地址枯竭的IPv4所开发的技术，在IPv6中为了提高网络安全也在使用NAT，在IPv4和IPv6的相互通信中常常使用NAT-PT。

## NAT的工作机制

![1564303299591](C:\Users\叶超\AppData\Roaming\Typora\typora-user-images\1564303299591.png)

![1564303351671](C:\Users\叶超\AppData\Roaming\Typora\typora-user-images\1564303351671.png)

## 两种传输协议TCP和UDP

### TCP

​		TCP是面向连接的、可靠的流协议。流就是值不间断的数据结构。可以想象为水管中的水流。当应用程序采用TCP发送消息时，能保持发送数据的顺序，但还是发送犹如水流一样无间隔的数据流给接收端。

​		TCP为提供可靠性传输，实行`顺序控制`或`重发控制`机制。此外还具备`流控制（流量控制）	`、`拥塞控制`、提高网络利用率等功能。

### UDP

​		UDP是不具有可靠性的数据报协议，它是面向无连接的，可以随时发送数据。细微的处理它会交给上面的应用层去完成。在UDP的情况下，虽然可以保证数据的大小，却不能保证数据能否一定到达。因此，应用有时会根据自己的需要选择是否进行重发消息。UDP经常用于以下方面：

- 包总量较小的通信（DNS、SNMP等）
- 视频、音频等多媒体通信（即时通信）
- 限定于LAN等特定网络的应用通信
- 广播通信（广播、多播）

### TCP和UDP的区别

​		TCP和UDP的优缺点不能绝对地、简单地的进行比较，TCP的特点可以为应用提供可靠稳定的数据传输。而UDP则用于高速传输和实时性要求较高的应用中。举个例子，在打电话的时候，声音的消息就是通过UDP协议进行发送的。如果某一段声音数据丢失，采用TCP协议的话会进行重发该段消息，那么就可能会对接收者产生误导。而采用UDP发送的话则只是影响一小段的通话。

​		多播和广播通信的协议也是依赖于UDP的。

## 端口号

### 定义

​		MAC地址用来识别同一数据链路上的不同计算机，IP地址用来识别TCP/IP网络中互连的主机和路由器。在传输层也有一个类似于地址的表示，就是端口号。端口号用来区分同一台主机中进行通信的不同应用程序。

### 根据端口号识别应用

​		同一台计算机能够运行多个应用程序进行通信，正式因为传输层协议通过端口号准确的识别应用程序，使得消息数据能够准确的传输。

![1564473404364](C:\Users\叶超\AppData\Roaming\Typora\typora-user-images\1564473404364.png)

### 通过端口号、IP地址、协议号进行通信

​		仅凭端口号来识别某个通信是远远不够的。

​		在`TCP/IP`和`UDP/IP`协议的通信中，通常采用五个信息来识别一个通信。分别是：

- 源IP地址

- 目标IP地址

- 协议号

- 源端口号

- 目标端口号

  只要有一项不同，则被认为是其它通信。

  ![1564473680547](C:\Users\叶超\AppData\Roaming\Typora\typora-user-images\1564473680547.png)

## 路由算法

​		路由控制有各种各样的算法，比较常用的算法有两个，分别是：`距离向量算法（Distance-Vector）`和`链路状态算法(Link-State).`

### 距离向量算法

​		距离向量算法是路由根据距离（代价）来决定目标网络或目标主机位置的方法。

​		当网络变得复杂的时候，每次获取准确距离可能会消耗一定的时间，而且可能造成路由循环等问题。

![1564475274755](C:\Users\叶超\AppData\Roaming\Typora\typora-user-images\1564475274755.png)

### 链路状态算法

​		链路状态算法是路由器在了解了网络的整体连接状态的基础上生成路由控制表的一种方法。

​		在复杂的网络中，路由管理和处理代理的信息需要高速CPU的处理能力和大量内存。

![1564475543857](C:\Users\叶超\AppData\Roaming\Typora\typora-user-images\1564475543857.png)

