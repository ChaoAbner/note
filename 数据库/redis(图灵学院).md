## 什么是Redis

​		Redis是c语言开发的一个高性能键值对（key-value）数据库。提供各种键值对基本类型来适应不同场景下的数据存储。Redis支持的数据类型如下：

- 字符串类型
- 散列类型（Hash）
- 字符串列表类型（list）
- 字符串集合类型（set）
- 有序字符串集合类型（sorted set）

## Redis应用场景

- 缓存（数据查询，短链接，新闻内容，商品内容等等） 最常用
- 聊天室的在线好友列表
- 任务队列（秒杀，抢购，12306等）
- 应用排行榜
- 网站访问统计
- 数据过期处理
- 分布式集群架构中的sesstion分离。

## Redis安装

### ubuntu 16.04安装redis
#### 方式一 ：apt安装

在 Ubuntu 系统安装 Redi 可以使用以下命令:

`sudo apt-get update`
`sudo apt-get install redis-server`
启动 Redis
` redis-server`
查看 redis 是否启动？
` redis-cli`
以上命令将打开以下终端：

`redis 127.0.0.1:6379>`
127.0.0.1 是本机 IP ，6379 是 redis 服务端口。现在我们输入 PING 命令。

`redis 127.0.0.1:6379> ping`
`PONG`
以上说明我们已经成功安装了redis。

#### 方式二：编译安装
　　保证网络畅通，选定好下载工作路径，执行以下命令下载redis-5.0.7：

`sudo wget http://download.redis.io/releases/redis-5.0.7.tar.gz`

　　解压该文件：

`sudo tar -zxvf redis-5.0.7.tar.gz`

　　会在当前目录下生成文件夹redis-5.0.7，我把它移动到了/usr/redis目录下： 
　　
　　如果没有安装gcc，需要先安装：

`sudo apt-get install gcc`

　　进入到redis-5.0.7目录下，依次执行下面两条命令进行编译：

`sudo make`
`sudo make install`

​		会安装到目录/usr/local/bin下： 　
　
　　现在进入先前解压后得到的文件夹（我的在/usr/redis），复制配置文件redis.conf到/etc/redis/下，并用vi命令编辑该文件，**将“daemonize no”修改为“daemonize yes”**，即设置成作为后台进程运行，修改完成后保存退出。 
　　 
　　进入到/usr/local/bin目录下，执行命令：（默认端口6379）

`redis-server /etc/redis/redis.conf`
`redis-cli -p 6379`

　　然后执行命令ping，若输出为pong，则证明服务成功启动。 
　　 
执行quit命令退出，现在可以通过下面的命令查看到该进程：

`ps -ef|grep redis`

**如何停止服务器**


在客户端里面输入shutdown命令即可，退出客户端用exit

**如何卸载redis服务**

卸载redis服务，只需把/usr/local/目录下的redis删除即可

为了卸载干净，可以把解压和编译的redis包也给删除了

## Redis操作

redis-cli连接上数据库后，既可以在命令行发送命令

- `ping` 数据库连接成功，则返回pong

### 存储String

- `set` 设置指定key的内容
- `get` 获取指定key的内容
- `del` 删除指定key的内容
- `keys *` 查看当前库中所有的key值

### 存储Hash

#### 赋值

- `hset field key value`	在某个field中为指定的key存储value
- `hmset filed key value [key1 value1..]` 在某个field中为指定的多个key存储value

#### 取值

- `hget field key`  在某个field中获取指定的key对应的value
- `hmget field key [key1...]`  在某个field中获取指定的多个key对应的value
- `hgetall field`  在某个field中所有的key-value

#### 删除

- `del field` 删除整个field
- `hdel field key [key1...]`  删除某个field中指定的一个或多个字段

#### 增加数字

- `hincriby field key increment`  设置某个field中的key对应的值增加increment，如age增加5

#### 自学命令

- `hexists field key` 判断某个field中是否存在指定的key
- `hlen field`  获取某个field中key的数量
- `hkeys field`  获取某个field中所有的key
- `hvals field`  谋取某个field中所有的value

### 存储list

#### 添加操作

`lpush list value1 value2...`：在指定的key所关联的list的头部插入所有的values，如果该key不存在，该命令在插入的之前创建一个与该key关联的空链	表，之后再向该链表的头部插入数据。插入成功，返回元素的个数。

`rpush list value1、value2…`：在该list的尾部添加元素

#### 查看操作

`lrange list start end`：获取链表中从start到end的元素的值，start、end可	为负数，若为-1则表示链表尾部的元素，-2则表示倒数第二个，依次类推… 

`lpushx list value`：仅当参数中指定的key存在时（如果与key管理的list中没	有值时，则该key是不存在的）在指定的key所关联的list的头部插入value。

**`rpushx list value`**：在该list的尾部添加元素

#### 两端弹出

**`lpop list `**：从头部弹出元素。

**`rpop list `**：从尾部弹出元素。

**`rpoplpush resource destination`**： 将链表尾部弹出并添加到另一个链表的头部

#### 获取元素个数

`llen list `：返回指定的key关联的链表中的元素的数量。

**`lset list index value`**：设置链表中的index的脚标的元素值，0代表链表的头元	素，-1代表链表的尾元素。

`lrem list count value`：删除count个值为value的元素，如果count大于0，从头向尾遍历并删除count个值为value的元素，如果count小于0，则从尾向头遍历并删除。如果count等于0，则删除链表中所有等于value的元素

**`linsert list before|after pivot value`**：在pivot元素前或者后插入value这个元素。

 #### 应用场景

![](http://img.fosuchao.com/20200228170827.png)

### set存储

​		添加、删除和判断某一元素是否存在等操作的时间复杂度为O(1)，元素不能重复。

#### 添加/删除

- `sadd set values[val1, val2...]` 向set中添加一个或多个元素

- `srem set menbers[member1, menber2... ]` 删除set中指定一个或多个成员。

#### 获取元素

- `smembers set` 获取set中所有元素

- `sismember set member` 判断指定member是否存在于该set中，1表示存在，0表示不存在或者set不存在

#### 差集运算

- `sdiff set1 set2`  返回set1与set2中相差的成员。

![](http://img.fosuchao.com/20200228170841.png)

#### 交集运算

- `sinter set1 set2 set3 ...`  返回指定集合中都有的成员

#### 并集运算

- `sunion set1 set2 set3...` 返回指定集合的并集

#### 扩展命令

- `scard set` 获取集合中成员的数量
- `srandmember set` 随机获取集合中的某个成员
- `sdiffstore destination set1 set2`  将set1与set2的差集存储到destination上
- `sinterstore destination  set1 set2 ...` 	将set1与set2等集合的交集存储到destination上
- `sunionstore destination  set1 set2 ...`  将set1与set2等集合的并集存储到destination上

### sorted set存储

不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。

有序集合的成员是唯一的,但分数(score)却可以重复。

```
redis 127.0.0.1:6379> ZADD database 1 redis
(integer) 1
redis 127.0.0.1:6379> ZADD database 2 mongodb
(integer) 1
redis 127.0.0.1:6379> ZADD database 3 mysql
(integer) 1
redis 127.0.0.1:6379> ZADD database 3 mysql
(integer) 0
redis 127.0.0.1:6379> ZADD database 4 mysql
(integer) 0
redis 127.0.0.1:6379> ZRANGE database 0 10 WITHSCORES
1) "redis"
2) "1"
3) "mongodb"
4) "2"
5) "mysql"
6) "4"
```

[常见命令](https://www.runoob.com/redis/redis-sorted-sets.html)

## 高级

​		进入[官网下载](https://redis.io/download)进行下载稳定版本，通过源码编译安装。通常把软件装到**linux下面的/usr/local**下面。

### 安装

Download, extract and compile Redis with:

```
$ wget http://download.redis.io/releases/redis-5.0.7.tar.gz
$ tar xzf redis-5.0.7.tar.gz
$ cd redis-5.0.7
$ make
```

The binaries that are now compiled are available in the `src` directory. Run Redis with:

```
# 使用配置文件启动
$ src/redis-server redis.conf
```

You can interact with Redis using the built-in client:

```
# 启动客户端
$ src/redis-cli
redis> set foo bar
OK
redis> get foo
"bar"
```

### 配置

redis安装目录下有一个redis.conf文件，里面有所有的配置信息，包括持久化RDB，AOF配置，端口，访问IP，密码，集群等配置。

#### 后台运行

修改配置文件中的`daemonize`,开启后台运行。

```
daemonize yes
```

#### AOF

##### 开启AOF

![](http://img.fosuchao.com/20200228103101.png)

##### AOF配置

![](http://img.fosuchao.com/20200228103152.png)

AOF持久化默认是每秒持久化一次**修改的命令**（set，del...）

持久化方式：像aof文件中追加修改的命令。

当redis重启恢复的时候，执行aof中的所有命令即可恢复到上一次的状态。

`appendfsync everysec`

##### AOF重写

为什么要AOF重写？因为当我们频繁对一个key进行修改的时候，会写入很多这个key 的命令，但是当重启载入数据的时候，我们只要最后一次修改的数据就可以了。不然非常消耗时间却又做了无用的事情。

**如下执行命令**：

![](http://img.fosuchao.com/20200228103452.png)

**没有重写AOF**

![](http://img.fosuchao.com/20200228103700.png)

**重写AOF**

手动将aof文件进行减肥操作。

```
# 使用aof重写命令
bgrewriteaof
```

![](http://img.fosuchao.com/20200228104804.png)

通过上面的命令即可，执行完可以看到aof文件的改变如下：

**将命令变成最后一次执行的命令**

![](http://img.fosuchao.com/20200228105036.png)

**默认配置**

aof重写的默认配置如下，我们自己也可以进行修改

- aof文件大小翻倍的时候进行重写

- 只有在aof文件大小超过64mb的情况下才会进行重写

**![](http://img.fosuchao.com/20200228105158.png)**

#### 混合持久化

结合AOF和RDB的优缺点，我们可以使用混合持久化的方式，提高了重启恢复的速度。又提升了持久化的速度。

混合持久化也是一种aof持久化。因为数据都是保存再aof文件中

**开启混合持久化**

redis.conf中的 `aof-use-rdb-preamble`设置为`yes`

混合持久化中aof文件的结构：

![](http://img.fosuchao.com/20200228104405.png)

aof文件上面保存的式RDB的二进制格式的数据快照。如果客户端继续有修改的命令，则通过追加写入到aof文件中。

### 集群模式

#### 哨兵模式

![](http://img.fosuchao.com/20200228151326.png)

Redis3.0以前的版本要实现集群一般式借助哨兵`sentinel`工具来监控master节点的状态，如果节点异常，则会做主从切换，将某一台slave作为master。

哨兵的配置略微复杂，并且**性能和高可用性等各方面表现一般**，特别是在主从切换的瞬间存在**访问瞬断**的情况，而且哨兵模式**只有一个主节点对外提供服务**，没法支持很高的并发，且单个主节点内存也不宜设置得过大，否则会导致持久化文件过大，影响数据恢复或主从同步的效率。

哨兵的缺点/不足

- 配置复杂
- 性能，高可用表现一般
- 存在访问瞬断的情况
- 只有一个主节点对外提供服务，无法支持很高的并发
- 持久化文件大可能会变得很大，影响数据恢复或主从同步效率

##### 作用

- Master 状态监测
- 如果Master 异常，则会进行Master-slave 转换，将其中一个Slave作为Master，将之前的Master作为Slave 

- Master-Slave切换后，master_redis.conf、slave_redis.conf和sentinel.conf的内容都会发生改变，即master_redis.conf中会多一行slaveof的配置，sentinel.conf的监控目标会随之调换 

##### 工作方式

1. 每个Sentinel以每秒钟一次的频率向它所知的**Master**，**Slave**以及其他 **Sentinel** 实例发送一个 PING 命令。
2. 如果一个实例（instance）距离最后一次有效回复 PING 命令的**时间超过** `down-after-milliseconds` 选项所指定的值， 则这个实例会被 Sentinel 标记为**主观下线**。 
3. 如果一个Master被标记为主观下线，则正在监视这个Master的**所有 Sentinel** 要以每秒一次的频率确认Master的确进入了**主观下线状态**。 
4. 当有足够数量的 Sentinel（大于等于配置文件指定的值）在指定的时间范围内确认Master的确进入了主观下线状态， 则**Master会被标记为客观下线** 。
5. 在一般情况下， 每个 Sentinel 会以每 10 秒一次的频率向它已知的所有Master，Slave发送 INFO 命令 。
6. 当Master被 Sentinel 标记为客观下线时，Sentinel 向下线的 Master 的所有 Slave 发送 INFO 命令的频率会从 10 秒一次改为每秒一次 。
7. 若没有足够数量的 Sentinel 同意 Master 已经下线， Master 的客观下线状态就会被移除。 
8. 若 Master 重新向 Sentinel 的 PING 命令返回有效回复， Master 的主观下线状态就会被移除。

##### 选举方式

选举一个哨兵中run id最小的。选举完成在轮流去看选的是谁。

##### slave选举为master

在选举的时候，写数据进不来，被阻塞

- slave的优先级（配置中可配置）
- 同步数据最多（偏移量最大的）
- run id中最小的

#### 高可用集群

![](http://img.fosuchao.com/20200228151358.png)

redis集群是一个由**多个主从节点群**组成的分布式服务器群，它具有复制、高可用和分片特性。Redis集群**不需要sentinel哨兵也能完成节点移除和故障转移的功能**。需要将每个节点设置成集群模式，这种集群模式没有中心节点，可水平扩展，据官方文档称可以线性扩展到上万个节点(官方推荐不超过1000个节点)。redis集群的性能和高可用性均优于之前版本的哨兵模式，且集群配置非常简单 

#### 搭建

##### 高可用集群的搭建

 **redis安装**

下载地址：http://redis.io/download 安装步骤：

安装gcc` yum install gcc`

\# 把下载好的redis-5.0.2.tar.gz放在/usr/local文件夹下，并解压

`wget http://download.redis.io/releases/redis-5.0.2.tar.gz tar xzf redis-5.0.2.tar.gz`

`cd redis-5.0.2`

**进入到解压好的redis-5.0.2目录下，进行编译与安装**

`make & make install`

启动并指定配置文件

`src/redis-server redis.conf`（注意要使用后台启动，所以修改redis.conf里的daemonize改为yes)

验证启动是否成功 

`ps -ef | grep redis` 

进入redis客户端 

`/usr/local/redis/bin/redis-cli` 

退出客户端

`quit`

退出redis服务： 

`（1）pkill redis-server` 
`（2）kill 进程号 
（3）src/redis-cli shutdown`

 **redis集群搭建** 

redis集群需要**至少要三个master节点**，我们这里搭建三个master节点，并且给每个master再搭建一个slave节点，总共6个redis节点，这里用三台机器部署6个redis实例，每台机器一主一从，搭建集群的步骤如下：

第一步：在第一台机器的/usr/local下创建文件夹redis-cluster，然后在其下面分别创建2个文件夾如下 

（1）`mkdir -p /usr/local/redis-cluster` 

（2）`mkdir 8001、 mkdir 8004`  第一步：把之前的redis.conf配置文件copy到8001下，修改如下内容： 

（3）`daemonize yes` 

（4）`port 8001`（分别对每个机器的端口号进行设置）

 （5）`dir /usr/local/redis-cluster/8001/`（指定数据文件存放位置，必须要指定不同的目录位置，不然会丢失数据） 

（6）`cluster-enabled yes`（启动集群模式） 

（7）`cluster-config-file nodes-8001.conf`（集群节点信息文件，这里800x最好和port对应上）

 （8）`cluster-node-timeout 5000`

   (9)  `# bind 127.0.0.1`（去掉bind绑定访问ip信息）

   (10)  `protected-mode  no`   （关闭保护模式）  

（11）`appendonly yes`

 如果要设置密码需要增加如下配置：

 （12）`requirepass 123456`     (设置redis访问密码)

 （13）`masterauth 123456`      (设置集群节点间访问密码，跟上面一致)  第三步：把修改后的配置文件，copy到8002，修改第2、3、5项里的端口号，可以用批量替换： %s/源字符串/目的字符串/g 

第二步：另外两台机器也需要做上面几步操作，第二台机器用8002和8005，第三台机器用8003和8006

第三步：分别启动6个redis实例，然后检查是否启动成功 

（1）`/usr/local/redis-5.0.2/src/redis-server /usr/local/redis-cluster/800*/redis.conf` 

（2）`ps -ef | grep redis` 查看是否启动成功

 第四步：用redis-cli创建整个redis集群(redis5以前的版本集群是依靠ruby脚本redis-trib.rb实现) 

`/usr/local/redis-5.0.2/src/redis-cli -a 123456 --cluster create --cluster-replicas 1 192.168.0.61:8001 192.168.0.62:8002 192.168.0.63:8003 192.168.0.61:8004 192.168.0.62:8005 192.168.0.63:8006` 

代表为每个创建的主服务器节点创建一个从服务器节点  

第七步：验证集群： 

（1）连接任意一个客户端即可：`./redis-cli -c -h -p` (-a访问服务端密码，-c表示集群模式，指定ip地址和端口号）如：`/usr/local/redis-5.0.2/src/redis-cli -a 123456 -c -h 192.168.0.61 -p 800*` 

（2）进行验证： `cluster info`（查看集群信息）、`cluster nodes`（查看节点列表） （3）进行数据操作验证 

（4）关闭集群则需要逐个进行关闭，使用命令： `/usr/local/redis/bin/redis-cli -a 123456 -c -h 192.168.0.60 -p 800* shutdown`

#### 集群原理分析

Redis Cluster将所有数据划分为16384个槽（2^14），每个节点负责一部分的槽位，默认是平均分配槽位。

客户端连接时，会将集群的槽位信息缓存在客户端本地。这样当客户端查找一个key的时候，就根据定位算法算出key对应的相应槽位信息，在根据槽位信息存储到相应的服务器上。

同时因为槽位的信息可能会存在客户端与服务器不一致的情况，还需要纠正机制来实现槽位信息的校验调整。

**槽位定位算法**

Cluster 默认会对 key 值使用 crc16 算法进行 hash 得到一个整数值，然后用这个整数值对 16384 进行取模来得到具体槽位。

`HASH_SLOT = CRC16(key) % 16384`

**跳转重定位**

当客户端向一个错误的节点发出了指令，该节点会发现指令的 key 所在的槽位并不归自己管理，这时它会向客户端发送一个特殊的跳转指令携带目标操作的节点地址，告诉客户端去连这个节点去获取数据。客户端收到指令后除了**跳转到正确的节点**上去操作，还会同步**更新纠正本地的槽位映射表缓存**，后续所有 key 将使用新的槽位映射表。

![](http://img.fosuchao.com/20200228165314.png)

**网络抖动**

真实世界的机房网络往往并不是风平浪静的，它们经常会发生各种各样的小问题。比如网络抖动就是非常常见的一种现象，突然之间部分连接变得不可访问，然后很快又恢复正常。

为解决这种问题，Redis Cluster 提供了一种选项**cluster-node-timeout**，表示当某个**节点持续 timeout 的时间失联**时，才可以认定该节点出现故障，需要进行**主从切换**。如果没有这个选项，网络抖动会导致主从频繁切换 (数据的重新复制)。 

#### 集群的水平拓展

查看redis集群的命令帮助

`redis-cli --cluster help`

![](http://img.fosuchao.com/20200228165640.png)

1. create：创建一个集群环境host1:port1 ... hostN:portN
2. call：可以执行redis命令
3. add-node：将一个节点添加到集群里，第一个参数为新节点的ip:port，第二个参数为集群中任意一个已经存在的节点的ip:port 
4. del-node：移除一个节点
5. reshard：重新分片
6. check：检查集群状态 

#### 集群的选举原理

当slave发现自己的master变为FAIL状态时（心跳超过timeout时就会变成fail状态），便尝试进行Failover，以期成为新的master。由于挂掉的master可能会有多个slave，从而存在多个slave竞争成为master节点的过程， 其过程如下：

1. slave发现自己的master变为FAIL
2. 将自己记录的集群currentEpoch加1，并广播`FAILOVER_AUTH_REQUEST` 信息
3. 其他节点收到该信息，只有master响应，判断请求者的合法性，并发送`FAILOVER_AUTH_ACK`，对每一个epoch只发送一次ack
4. 尝试failover的slave收集`FAILOVER_AUTH_ACK`
5. **超过半数后**变成新Master
6. 广播**Pong通知**其他集群节点。

`currentEpoch`：选举轮数，集群中没进行一次选举，这个就会+1