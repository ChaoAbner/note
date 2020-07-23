# 了解使用Netty

## NIO入门

​		网络编程的基本模型时Client/Server模型。也就是两个进程进行通信。服务器监听客户端的连接。通过TCP三次握手连接，若连接成功，则进行Socket通信。

### 传统BIO编程![1574767140472](F:\typoraImg\1574767140472.png)

#### **缺点**

​		缺乏弹性伸缩能力，服务器线程数和客户并发访问数1：1，导致服务器线程资源消耗严重，可能导致线程堆栈溢出，创建新线程失败等问题。

#### **示例代码**

**服务端：**

```java
package com.atguigu.javaGuide.io.bio;

import java.io.IOException;
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * @Description: BIO服务端
 * @Auther: Joker Ye
 * @Date: 2019/11/14 20:42
 */
public class BIOServer {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(3333);

        new Thread(() -> {
            while (true) {
                try {
                    // 用阻塞方法获取新的连接
                    Socket socket = serverSocket.accept();

                    // 为每个连接创建一个线程 用于读取数据
                    new Thread(() -> {
                        try {
                            int len;
                            byte[] data = new byte[1024];
                            InputStream inputStream = socket.getInputStream();
                            // 按字节流的形式读取
                            while ((len = inputStream.read(data)) != -1){
                                System.out.println(new String(data, 0 ,len));
                            }
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }).start();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

}

```

**客户端：**

```java
package com.atguigu.javaGuide.io.bio;

import java.io.IOException;
import java.net.Socket;
import java.util.Date;

/**
 * @Description: BIO客户端
 * @Auther: Joker Ye
 * @Date: 2019/11/14 20:37
 */
public class BIOClient {

    public static void main(String[] args) {
        new Thread(() -> {
            try {
                Socket socket = new Socket("127.0.0.1", 3333);
                while (true) {
                    try {
                        socket.getOutputStream().write((new Date() + ":hello world").getBytes());
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }).start();
    }
}

```

### 伪异步IO编程

​		为解决同步I/O的一对一处理的问题。通过后端用一个线程池来处理多个客户端的请求，形成客户端M : 服务端N的关系，M可以大于N。

​		由于线程池的线程数量是可控的，这样就防止了线程资源的枯竭。

![1574767552061](F:\typoraImg\1574767552061.png)

#### **核心代码**

![1574767719276](F:\typoraImg\1574767719276.png)

#### **缺点**

​		对Socket的输入流进行读取操作的时候，会一直阻塞下去，直到遇到下面三种事件：

- 有数据可读
- 可用数据读取完毕
- 发生空指针或者I/O异常

![1574767927191](F:\typoraImg\1574767927191.png)

​		无法从根本上解决同步IO导致的通信线程阻塞问题。还可能会**造成级联故障**。

![1574768036414](F:\typoraImg\1574768036414.png)

![1574768049828](F:\typoraImg\1574768049828.png)

### NIO编程

​		当开发高负载，高并发的**网络应用**时，应该采用非阻塞IO来实现。体现NIO的特点。

#### 相关概念

- 缓冲区Buffer	

  在NIO编程中，所有读写操作都是通过缓冲区进行的。**缓冲区实际上是一个数组。**（在面向流的IO中，可用将数据写入或读到**Stream**对象中）

- 通道Channel

  ![1574768688353](F:\typoraImg\1574768688353.png)

- 多路复用器Selector

  ![1574769142318](F:\typoraImg\1574769142318.png)

#### NIO服务端时序图

![1574769164151](F:\typoraImg\1574769164151.png)

#### 示例代码

**客户端：**

```java
package com.atguigu.javaGuide.io.nio;

import java.io.IOException;
import java.net.Socket;
import java.util.Date;

/**
 * @Description: BIO客户端
 * @Auther: Joker Ye
 * @Date: 2019/11/14 20:37
 */
public class NIOClient {

    public static void main(String[] args) {
        new Thread(() -> {
            try {
                Socket socket = new Socket("127.0.0.1", 3333);
                while (true) {
                    try {
                        socket.getOutputStream().write((new Date() + ":hello world").getBytes());
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }).start();
    }
}

```

**服务端：**

```java
package com.atguigu.javaGuide.io.nio;

import java.io.IOException;
import java.io.InputStream;
import java.net.InetSocketAddress;
import java.net.ServerSocket;
import java.net.Socket;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.nio.charset.Charset;
import java.util.Iterator;
import java.util.Set;

/**
 * @Description: BIO服务端
 * @Auther: Joker Ye
 * @Date: 2019/11/14 20:42
 */
public class NIOServer {
    public static void main(String[] args) throws IOException {
        // 1. serverSelector负责轮询是否有新的连接，服务端监测到新的连接之后，不再创建一个新的线程，
        // 而是直接将新连接绑定到clientSelector上，这样就不用 IO 模型中 1w 个 while 循环在死等

        Selector serverSelector = Selector.open();
        Selector clientSelector = Selector.open();

        // 是否有新的连接
        new Thread(() -> {
            try {
                // 服务端IO配置注册启动
                ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
                serverSocketChannel.bind(new InetSocketAddress(3333));
                serverSocketChannel.configureBlocking(false);
                serverSocketChannel.register(serverSelector, SelectionKey.OP_ACCEPT);

                while (true) {
                    // 监测是否有新连接，1表示超时时间为1ms
                    if(serverSelector.select(1) > 0){
                        Set<SelectionKey> selectionKeys = serverSelector.selectedKeys();
                        Iterator<SelectionKey> keyIterator = selectionKeys.iterator();

                        while (keyIterator.hasNext()){
                            SelectionKey selectionKey = keyIterator.next();

                            if(selectionKey.isAcceptable()){
                                try {
                                    // (1) 每来一个新连接，不需要创建一个线程，而是直接注册到clientSelector
                                    SocketChannel clientChannel = ((ServerSocketChannel) selectionKey.channel()).accept();
                                    clientChannel.configureBlocking(false);
                                    clientChannel.register(clientSelector, SelectionKey.OP_READ);
                                } finally {
                                    keyIterator.remove();
                                }
                            }
                        }
                    }
                }
            } catch (IOException e) {
                e.printStackTrace();
            }

        }).start();

        // 读取消息
        new Thread(() -> {
            while (true) {
                // 轮询判断是否有数据可读
                try {
                    if (clientSelector.select(1) > 0) {
                        Set<SelectionKey> clientSelectKey = clientSelector.selectedKeys();
                        Iterator<SelectionKey> clientKeyIterator = clientSelectKey.iterator();

                        while (clientKeyIterator.hasNext()){
                            SelectionKey selectKey = clientKeyIterator.next();

                            if (selectKey.isReadable()) {
                                try {
                                    // 每来一个连接，不需要创建一个线程，而是之策到clientSelector
                                    SocketChannel clientChannel = (SocketChannel) selectKey.channel();
                                    ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
                                    // 面向buffer
                                    clientChannel.read(byteBuffer);
                                    byteBuffer.flip();
                                    System.out.println(Charset.defaultCharset().newDecoder().decode(byteBuffer).toString());

                                } finally {
                                    clientKeyIterator.remove();
                                    selectKey.interestOps(SelectionKey.OP_READ);
                                }
                            }
                        }
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }


}

```

#### 优缺点

缺点：

- 编程复杂
- epoll bug，会导致Selector空轮询，最终导致CPU使用率100%。（JDK1.7仍可能发生）

优点：

![1574773297564](F:\typoraImg\1574773297564.png)

### AIO编程

​		NIO2.0引入新的异步通道的概念。并提供了异步文件通道和异步套接字通道的实现。

异步通道有两种方式获取操作结果

![1574773397328](F:\typoraImg\1574773397328.png)

### 不同模型对比

![1574774444437](F:\typoraImg\1574774444437.png)

![1574774457611](F:\typoraImg\1574774457611.png)

