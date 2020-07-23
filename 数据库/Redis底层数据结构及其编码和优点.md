# Redis底层数据结构及其编码和优点

## Redis中的数据结构

### 简单动态字符串

#### **SDS的结构定义**

![](http://img.fosuchao.com/20200503153452.png)

**特性**

![](http://img.fosuchao.com/20200503153947.png)

### 双端链表

#### **链表的结构定义**

![](http://img.fosuchao.com/20200503153801.png)

![](http://img.fosuchao.com/20200503153824.png)

#### **特性**

![](http://img.fosuchao.com/20200503153921.png)

### 字典

哈希表的结构定义

![](http://img.fosuchao.com/20200503154104.png)

`table`是一个类型为`dictEntry`的数组，下图是一个大小为4的空哈希表

![](http://img.fosuchao.com/20200503154228.png)

#### **dictEntry结构**

![](http://img.fosuchao.com/20200503154352.png)

next用来解决哈希冲突的问题

![](http://img.fosuchao.com/20200503154403.png)

**字典的结构定义**

![](http://img.fosuchao.com/20200503154606.png)

字典总共定义了两个哈希表dictht，使用的时候只使用一个即可，另外一个是用来rehash的时候进行扩容和复制元素的。

![](http://img.fosuchao.com/20200503154633.png)

### 跳跃表

​		跳表是一种**有序**数据结构，它在每个节点中维护了多个指向其它节点的指针，从而达到快速访问节点的目的。

​		跳表支持评论O(logn)最差O(N)的复杂度进行节点查找，这个效率可以跟平衡树相媲美。并且实现简单，很多程序都使用了跳表来代替平衡树。

#### **跳表的实现**

跳表由`redis.h/zskiplistNode`和`redis/zskiplist`两个结构定义。

下图为一个跳跃表

![](http://img.fosuchao.com/20200503161035.png)

#### **zskiplistNode**![](http://img.fosuchao.com/20200503161006.png)

#### **zskiplist**

![](http://img.fosuchao.com/20200503161129.png)

![](http://img.fosuchao.com/20200503161145.png)

#### **字段的含义**

![](http://img.fosuchao.com/20200503161243.png)

![](http://img.fosuchao.com/20200503161253.png)

### 整数集合

#### **整数集合结构定义**

![](http://img.fosuchao.com/20200503161331.png)

#### **升级**

​		当我们要将一个新元素添加到整数集合里面，并且新元素的类型比整数集合现有所有元素的类型都要长时，整数集合需要先进行升级，然后才能将新元素添加到整数集合中。

​		假如整数集合中保存了三个元素：1，2，3。分别使用int16_t类型来保存，当我们要添加65535到整数集合中时，65535的类型int32_t比当前集合中所有元素的类型都要长，这时就要将底层数组的类型进行升级。

#### **重点**

![](http://img.fosuchao.com/20200503161944.png)

### 压缩列表

​		压缩列表是由一系列特殊编码的连续内存块组成的**顺序型**数据结构。一个压缩列表可以包含任意多个节点（entry），每个节点可以保存一个字节数组或者一个整数值。

#### **组成部分**

![](http://img.fosuchao.com/20200503162249.png)

#### **重点**

![](http://img.fosuchao.com/20200503162330.png)

## Redis中的对象

​		Redis使用对象来表示数据库中的键和值，当新建一个数据时，Redis至少会创建两个对象，分别表示键和值。

​		Redis中的每个对象都由一个redisObject的结构来表示，它有三个属性：**type、encoding、ptr**

![](http://img.fosuchao.com/20200502163745.png)

### 类型

| 类型常量     | 对象的名称   |
| ------------ | ------------ |
| REDIS_STRING | 字符串对象   |
| REDIS_LIST   | 列表对象     |
| REDIS_HASH   | 哈希对象     |
| REDIS_SET    | 集合对象     |
| REDIS_ZSET   | 有序集合对象 |

我们可以使用`type`命令对不同的对象进行输出，即查看相应对象的`type`属性

![](http://img.fosuchao.com/20200502164119.png)

### 编码和底层实现

​		对象的`ptr`属性即指针指向对象的底层实现的数据结构，而这些数据结构由对象的`encoding`属性来决定。

**对象的编码**

![](http://img.fosuchao.com/20200502164339.png)

​	**每种类型的对象**都至少使用了两种不同的编码。即**多种底层数据结构来实现**。

**不同类型和编码的对象**

![](http://img.fosuchao.com/20200502164500.png)

**OBJECT ENCODING对不同编码的输出**

![](http://img.fosuchao.com/20200502164607.png)

![](http://img.fosuchao.com/20200502164617.png)		

Redis会根据不同的场景选择不同的编码来优化当前场景的对象使用效率。

![](http://img.fosuchao.com/20200502164754.png)

### 字符串对象

​		字符串对象的编码可以是`int`、`raw`、`embstr`

​		如果保存的是**整数**，则可以用long型代替，这时对象的编码就是int。

​		如果保存的是**字符串**，并且这个字符串长度**大于**32个字节。那么就会用SDS来保存，编码为raw，如下图

![](http://img.fosuchao.com/20200502165623.png)

​		如果保存的是**字符串**，字符串长度**小于等于**32个字节，那么就会使用embstr编码。

![](http://img.fosuchao.com/20200502165654.png)

使用embstr编码保存短字符串的好处：

![](http://img.fosuchao.com/20200502165844.png)

### 列表对象

列表对象的编码可以是`ziplist`或者`linkedlist`

**编码的转换**

![](http://img.fosuchao.com/20200502170457.png)

**ziplist编码的列表对象**

![](http://img.fosuchao.com/20200502170039.png)

**linkedlist编码的列表对象**

linkedlist编码的列表对象使用双端链表作为底层实现，每个双端链表的节点（node）都保存了一个字符串对象，而每个字符串对象保存了一个列表元素。

```
// node结构
{
	StringObject: 列表元素
}
```

![](http://img.fosuchao.com/20200502170103.png)

### 哈希对象

哈希对象的编码可以是`ziplist`或者`hashtable`

**ziplist编码**

![](http://img.fosuchao.com/20200502171536.png)

![](http://img.fosuchao.com/20200503170421.png)

![](http://img.fosuchao.com/20200502171813.png)

**hashtable编码**

![](http://img.fosuchao.com/20200502171827.png)

![](http://img.fosuchao.com/20200502181307.png)

**编码转换**

![](http://img.fosuchao.com/20200502181338.png)

### 集合对象

集合对象的编码可以是`intset`和`hashtable`

**intset编码**

![](http://img.fosuchao.com/20200502181833.png)

**hashtable编码**

![](http://img.fosuchao.com/20200503170524.png)

**编码转换**

![](http://img.fosuchao.com/20200502181928.png)

### 有序集合对象

有序集合对象编码可以是`ziplist`或者`skiplist`

**ziplist编码**

![](http://img.fosuchao.com/20200502183012.png)

**skiplist编码**

​		skiplist编码的有序集合对象使用zset结构作为底层实现，zset的结构如下：

```c
typedef struct zset {
	zskiplist *zsl;
	dict *dict;
} zset;
```

![](http://img.fosuchao.com/20200502183509.png)

​		zset还保存了一个dict字典，用来为有序集合保存成员到分值的映射。通过这个字典，redis可以用**O(1)的复杂度查找给定的成员的分值**，ZSCORE命令就是根据这个特性实现的。

**为什么有序集合要同时使用skiplist和dict来实现呢？**

​		其实有序集合可以只用字典或者跳跃表其中的一种数据结构实现，但无论是哪种，都会有性能上的损失。

​		只用**字典**，还能通过O(1)的复杂度查找分值，但是ZRANGE和ZRANK的命令，程序都要对所有元素进行排序，这样要至少花费O(NlogN)的时间复杂度。以及额外的O(N)内存空间。

​		只用**跳跃表**，查找分数的复杂度又会变成O(logN)了。

![](http://img.fosuchao.com/20200502193001.png)

**编码转换**

![](http://img.fosuchao.com/20200502193034.png)