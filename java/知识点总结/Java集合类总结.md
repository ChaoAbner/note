Java集合总结

### 常用集合类

最常用的集合类是List 和 Map。

List 的具体实现包括 ArrayList 和 Vector，它们是可变大小的列表，比较适合构建、存储和操作任何类型对象的元素列表。List 适用于按数值索引访问元素的情形。

Map 提供了一个更通用的元素存储方法。 Map 集合类用于存储元素对（称作"键"和"值"），其中每个键映射到一个值。

### List、Map、Set三个接口存取元素时，各有什么特点？

List以特定**索引**来存取元素，**可以有重复元素**。

Set不能存放重复元素（用对象的equals()方法来区分元素是否重复）。

Map保存键值对（key-value pair）映射，映射关系可以是一对一或多对一。

**Set和Map容器都有基于哈希存储和排序树的两种实现版本**，基于哈希存储的版本理论存取时间复杂度为O(1)，而基于排序树版本的实现在插入或删除元素时会按照元素或元素的键（key）构成排序树从而达到排序和去重的效果。

### ArrayList、Vector、LinkedList的存储性能和特性

ArrayList 和Vector都是使用**数组**方式存储数据，此数组元素数大于实际存储的数据以便增加和插入元素，它们都允许直接按序号索引元素，但是插入元素要涉及数组元素移动等内存操作，所以索引数据快而插入数据慢，Vector中的方法由于添加了synchronized修饰，因此Vector是线程安全的容器，但性能上较ArrayList差，因此已经是Java中的遗留容器。

LinkedList使用**双向链表**实现存储（将内存中零散的内存单元通过附加的引用关联起来，形成一个可以按序号索引的线性结构，这种链式存储方式与数组的连续存储方式相比，内存的利用率更高），按序号索引数据需要进行前向或后向遍历，但是插入数据时只需要记录本项的前后项即可，所以插入速度较快。由于**ArrayList和LinkedList都是非线程安全的**，如果遇到多个线程操作同一个容器的场景，则可以通过工具类Collections中的synchronizedList方法将其转换成线程安全的容器后再使用（这是对装潢模式的应用，将已有对象传入另一个类的构造器中创建新的对象来增强实现）。

### Collection 和 Collections的区别

Collection是集合类的上级接口，继承与他的接口主要有Set 和List.
Collections是针对集合类的一个帮助类，他提供一系列静态方法实现对**各种集合的搜索、排序、线程安全化**等操作。	

### 什么是迭代器

Iterator提供了统一遍历操作集合元素的统一接口, Collection接口实现Iterable接口,
每个集合都通过实现Iterable接口中iterator()方法返回Iterator接口的实例, 然后对集合的元素进行迭代操作.
有一点需要注意的是：

**在迭代元素的时候不能通过集合的方法删除元素, 否则会抛出ConcurrentModificationException 异常. 但是可以通过Iterator接口中的remove()方法进行删除**.

### Java中的Map

java为数据结构中的映射定义了一个接口java.util.Map;它有四个实现类,分别是HashM	ap HashTable LinkedHashMap 和TreeMap.

Map主要用于存储健值对，根据键得到值，因此**不允许键重复(重复了覆盖了),但允许值重复和键为null值**。

Hashmap 是一个最常用的Map,它根据键的HashCode值存储数据,根据键可以直接获取它的值，具有很快的访问速度，遍历时，取得数据的顺序是完全随机的。 HashMap最多只允许一条记录的键为Null;允许多条记录的值为 Null;**HashMap不支持线程的同步**，即任一时刻可以有多个线程同时写HashMap;可能会导致数据的不一致。

如果需要同步的两种方法：

**可以用 Collections的synchronizedMap方法使HashMap具有同步的能力，或者使用ConcurrentHashMap。**

Hashtable与 HashMap类似,它继承自Dictionary类，不同的是:它不允许记录的键或者值为空;它支持线程的同步，即任一时刻只有一个线程能写Hashtable,因此也导致了 Hashtable在写入时会比较慢。

LinkedHashMap 是HashMap的一个子类，保存了记录的插入顺序，在用Iterator遍历LinkedHashMap时，先得到的记录肯定是先插入的.也可以在构造时用带参数，按照应用次数排序。在遍历的时候会比HashMap慢，不过有种情况例外，当HashMap容量很大，实际数据较少时，遍历起来可能会比 LinkedHashMap慢，因为LinkedHashMap的遍历速度只和实际数据有关，和容量无关，而HashMap的遍历速度和他的容量有关。

**TreeMap**实现**SortMap接口，能够把它保存的记录根据键排序,**默认是**按键值的升序排序**，也可以指定排序的比较器，当用**Iterator 遍历TreeMap时，得到的记录是排过序**的。

一般情况下，我们用的最多的是HashMap,在Map 中插入、删除和定位元素，HashMap 是最好的选择。但如果您要按自然顺序或自定义顺序遍历键，那么TreeMap会更好。如果需要输出的顺序和输入的相同,那么用LinkedHashMap 可以实现,它还可以按读取顺序来排列.

HashMap是一个最常用的Map，它根据键的hashCode值存储数据，根据键可以直接获取它的值，具有很快的访问速度。HashMap最多只允许一条记录的键为NULL，允许多条记录的值为NULL。

HashMap不支持线程同步，即任一时刻可以有多个线程同时写HashMap，可能会导致数据的不一致性。如果需要同步，**可以用Collections的synchronizedMap方法使HashMap具有同步的能力**。

**Hashtable**与HashMap类似，不同的是：**它不允许记录的键或者值为空；它支持线程的同步**，**即任一时刻只有一个线程能写Hashtable，因此也导致了Hashtable在写入时会比较慢**。

**LinkedHashMap**保存了记录的插入顺序**，在**用Iterator遍历LinkedHashMap时，先得到的记录肯定是先插入**的。

在遍历的时候会比HashMap慢TreeMap能够把它保存的记录根据键排序，默认是按升序排序，也可以指定排序的比较器。当用Iterator遍历TreeMap时，得到的记录是排过序的。

#### 讲一下ConcurrentHashMap

##### 结构属性

(1) From 1.7 to 1.8, since HashEntry has changed from a linked **list to a red-black tree**, the time complexity of concurrentHashMap ranges from **O(n) to O(log(n))**.

(2) HashEntry 最小数量是 2

(3)Segment 数组初始容量是 16;

(4) HashEntry 在1.8中称为Node，链表在容量超过8的时候会转为红黑树。

**锁粒度**

JDK1.7版本锁的粒度是基于段（segment）的，包含多个HashEntry, JDK1.8锁的粒度是HashEntry(第一个节点)

**结构**

ConcurrentHashMap只是从JDK1.7版本中添加了同步操作来控制并发性。**ReentrantLock+segment+HashEntry**

ConcurrentHashMap的JDK1.8版本的数据结构接近于HashMap。JDK1.8 **synchronized+CAS+Node+红黑树**

更细粒度下synchronized不必reentrantLock差。

**1.7**

![](http://img.fosuchao.com/20200303110611.png)

`get`方法：先用hash定位相应的segment,然后散列到指定的HashEntry，遍历HashEntry下的链表进行比较，返回成功，如果不成功则返回null

`put`方法：先用hash定位相应的segment,如果当前segment为空，则通过CAS操作进行初始化赋值。若不为空则再散列到指定的HashEntry进行赋值。

如果已经有一个线程来获取这个段的锁，当前线程将以**自旋方式继续调用tryLock()**方法来获取锁，挂起的次数超过指定的次数，等待唤醒

`size`方法：**循环三次**，比较当前的size结果是否更上一次相同，相同说明当前的size就是真正的size，即返回。如果三次对比都不相同，则用锁锁住所有segment，在获取实际的size返回。

**1.8**

![](http://img.fosuchao.com/20200303110636.png)

JDK1.8的实现**抛弃了segment**的概念，直接采用了**Node数组+链表+红黑树**的数据结构。**并发控制同步和CAS操作**，整个看起来像一个优化的线程安全的HashMap，尽管你也可以在JDK1.8中看到段的数据结构。但是为了与旧版本兼容，属性被简化了。

红黑树旋转的时候，使用synchronized锁住头节点，更小粒度的锁使得效率更高了。

1.8的主要结构：

- Node：存储数据的条目
- TreeNode：红黑树结构
- TreeBin：字面上理解为存储树结构的容器，树结构指的是TreeNode, TreeBin是封装TreeNode的容器，它为**黑红树的转换和锁的控制**提供了一些条件。

`put`方法

1、如果没有`initializedFirstCall`， `initTable()`方法来执行初始化过程
2、直接通过CAS插入如果不存在哈希冲突，进入第三步，如果容器正在扩展，它将调用`helpTransfer`()方法来帮助扩展容量。

3、若需要扩容，先扩容
4、如果存在散列冲突，并且为确保线程安全而将其锁定。有两种情况，一种是链表形式被直接遍历，另一种是按红黑树结构插入红黑树，

5、最后，如果链表的数目大于阈值8，则有必要先将其转换为黑红树林结构，再打破进入循环

6、如果添加成功，则调用addCount()方法来计算大小。

`get`方法

1、计算哈希值，定位到表索引位置，如果是第一个节点，它将返回
2、如果在调用时遇到扩展，则标志扩展了节点的FindingNode's find方法，查找节点，匹配返回。

3、如果以上都不匹配，则向下遍历该节点，再匹配回来，否则最后将返回null。



#### HashMap和ConcurrentHashMap的区别



#### HashSet原理

​		HashSet内部使用了HashMap作为元素的存储结构。HashSet将要添加的元素作为HashMap的key来存储。并且所有key的值都是PRESENT。

​		因为Map是无序的，因此HashSet也无法保证顺序。HashSet的方法也是借助HashMap的方法来实现的。

```java
private transient HashMap<E,Object> map;  //map集合，HashSet存放元素的容器
private static final Object PRESENT = new Object(); //map，中键对应的value值
```

Add()

```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```

​		PRESENT为HashSet类中定义的一个使用static final 修饰的常量，并无实际的意义，HashSet的add方法调用HashMap的put()方法实现，**如果键已经存在，HashMap.put()放回的是旧值，添加失败，**如果添加成功map.put()方法返回的是null ,HashSet.add()方法返回true,要添加的元素作为可map中的key 。

remove()

```java
public boolean remove(Object o) {
    return map.remove(o)==PRESENT;
}
```

​		删除方法，调用map.remove()方法实现，map.remove()能**找到指定的key**,则返回key对应的value,对于Hashset而言，它所有的key对应的值都是PRESENT。

#### HashMap原理

HashMap是一种键值对存储的数据结构，底层使用数组+链表/红黑树

**线程不安全**

1、当多个线程同时put的时候，如果得到的索引位置或者节点位置相同，而且目标位置刚好位null的时候，会导致**数据插入丢失**。

2、可能**造成死循环**，两个线程都要resize，但是线程A还没有resize，线程B被调度执行，完成了resize操作后，开始迁移数据，因为**迁移是由链表尾向头**进行的。等迁移完成后，时间片还给线程A执行的时候，这是就发生了死循环。

![img](https:////upload-images.jianshu.io/upload_images/7853175-ab75cd3738471507.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

**多线程HashMap的resize**

我们假设有两个线程同时需要执行resize操作，我们原来的桶数量为2，记录数为3，需要resize桶到4，原来的记录分别为：[3,A],[7,B],[5,C]，在原来的map里面，我们发现这三个entry都落到了第二个桶里面。
 假设线程thread1执行到了transfer方法的Entry next = e.next这一句，然后时间片用完了，此时的e = [3,A], next = [7,B]。线程thread2被调度执行并且顺利完成了resize操作，需要注意的是，此时的[7,B]的next为[3,A]。此时线程thread1重新被调度运行，此时的thread1持有的引用是已经被thread2 resize之后的结果。线程thread1首先将[3,A]迁移到新的数组上，然后再处理[7,B]，而[7,B]被链接到了[3,A]的后面，处理完[7,B]之后，就需要处理[7,B]的next了啊，而通过thread2的resize之后，[7,B]的next变为了[3,A]，此时，**[3,A]和[7,B]形成了环形链表**，在get的时候，如果get的key的桶索引和[3,A]和[7,B]一样，那么就会陷入死循环。

**如果在取链表的时候从头开始取（现在是从尾部开始取）的话，则可以保证节点之间的顺序，那样就不存在这样的问题了。**

##### **链表转红黑树**

当链表长度超过8的时候，并且满足hashMap容量大于64的时候，会将链表转化为红黑树。如果hashMap 的容量没有超过64的话，优先进行扩容(resize())。

![1581925782849](F:\typoraImg\1581925782849.png)

![1581925879564](F:\typoraImg\1581925879564.png)

**默认容量**

默认容量是16，必须是**2的指数次幂**，可以指定容量，当不满足2的指数次幂的时候，会取到最近距离的2的指数次幂的数来作为初始容量。

![1581925969001](F:\typoraImg\1581925969001.png)

**为什么是2的指数次幂**

​		因为hashmap求hashcode的方法是移位的方法，而不是我们平时用的求%方法，因为用移位来运算的话，当数据量大的时候，**效率会高**很多。

详情：https://blog.nowcoder.net/n/53e3036fdbed4eab97c0841393397d6f

```java
求hash值的原理：
将对象的hashcode转化成整形hash值：
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }

putVal(int hash, ...)中的 (n - 1) & hash
```

- 位运算效率高

- 长度为奇数的话，导致位运算过后的结果都是偶数，使得key的分布很集中。

  也就是说**2的N次幂有助于减少碰撞的几率**，空间利用率比较大。这样你就明白为什么第一次扩容会从16 ->32了吧？总不会再说32+1=33或者其余答案了吧？至于加载因子，如果设置太小不利于空间利用，设置太大则会导致碰撞增多，降低了查询效率，所以设置了0.75。