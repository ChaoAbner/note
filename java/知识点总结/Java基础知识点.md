# Java基础知识

## Java中的8种基本数据类型及其取值范围

Java种的8种基本数据类型分别是：**byte，short，int，long，float，double，char以及boolean**。boolean类型的取值为true和false两种，其余每一种基本类型都占有一定的字节，并且拥有着最大值和最小值。比如int的取值范围为 Integer.MIN_VALUE 到 Integer.MAX_VALUE。这个题目的答案，我希望大家可以自己动手输入代码来一一查看以加深记忆。这里给出**每种基本类型所占用的字节数：**

- byte：1字节
- short：2字节
- int：4个字节
- long：8字节
- float：4字节
- double：8字节
- char：2字节
- boolean：Java规范中并没有规定boolean类型所占字节数

## Java8新特性

**Lambda 表达式** − Lambda允许把函数作为一个方法的参数（函数作为参数传递进方法中。

**方法引用**− 方法引用提供了非常有用的语法，可以直接引用已有Java类或对象（实例）的方法或构造器。与lambda联合使用，方法引用可以使语言的构造更紧凑简洁，减少冗余代码。

**默认方法**− 默认方法就是一个在接口里面有了一个实现的方法。

**新工具**− 新的编译工具，如：Nashorn引擎 jjs、 类依赖分析器jdeps。

**Stream API** −新添加的Stream API（java.util.stream） 把真正的**函数式编程风格引入到Java**中。

**Date Time API** − 加强对日期与时间的处理。

**Optional 类** − Optional 类已经成为 Java 8 类库的一部分，用**来解决空指针异常**。

**Nashorn, JavaScript 引擎** − Java 8提供了一个新的Nashorn javascript引擎，它允许我们在JVM上运行特定的javascript应用。

## ==与equals

“==”对比**两个对象**基于**内存引用**，如果两个对象的引用完全相同（指向同一个对象）时，“==”操作将返回true，否则返回false。“==”如果两边是**基本类型，就是比较数值是否相等**。

## 为什么重写equals还要重写hashcode？

HashMap中，如果要比较key是否相等，要同时使用这两个函数！

因为自定义的类的hashcode()方法继承于Object类，其hashcode码为默认的内存地址，这样即便有相同含义的两个对象，比较也是不相等的。**HashMap中的比较key是这样的，先求出key的hashcode(),比较其值是否相等，若相等再比较equals(),若相等则认为他们是相等的**。若equals()不相等则认为他们不相等。如果只重写hashcode()不重写equals()方法，当比较equals()时只是看他们是否为同一对象（即进行内存地址的比较）,所以必定要两个方法一起重写。HashMap用来判断key是否相等的方法，其实是调用了HashSet判断加入元素 是否相等。重载hashCode()是为了对同一个key，能得到相同的Hash Code，这样HashMap就可以定位到我们指定的key上。重载equals()是为了向HashMap表明当前对象和key上所保存的对象是相等的，这样我们才真正地获得了这个key所对应的这个键值对。

**两个对象hashcode相等，但是不一定equals()返回true**

https://blog.csdn.net/kangkanglou/article/details/78954894

## JDK

### JDK和JRE的区别是什么？

Java运行时环境(JRE)是将要执行Java程序的Java虚拟机。它同时也包含了执行applet需要的浏览器插件。Java开发工具包**(JDK)是完整的Java软件开发包**，包含了JRE，编译器和其他的工具(比如：JavaDoc，Java调试器)，可以让开发者开发、编译、执行Java应用程序。

### Java中的LongAdder和AtomicLong有什么区别？

**JDK1.8引入了LongAdder类**。

**AtomicLong**：就像我们所知道的那样,AtomicLong的原理是依靠底层的cas来保障原子性的更新数据，在要添加或者减少的时候，会使用死循环不断地cas到特定的值，从而达到更新数据的目的。

**LongAdder：Cell**类的内部是一个volatile的变量value，然后更改这个变量唯一的方式通过cas。我们可以猜测到LongAdder的高明之处可能在于将之前单个节点的并发分散到各个节点的，这样从而**提高在高并发时候的效率**。

## 多态

### 方法覆盖(Overriding)和方法重载(Overloading)

方法的重载和重写都是实现多态的方式，区别在于前者实现的是编译时的多态性，而后者实现的是运行时的多态性。

**重载**发生在**一个类中**，同名的方法如果有**不同的参数列表（参数类型不同、参数个数不同或者二者都不同）**则视为重载；

**重写**发生在**子类与父类之间**，重写要求**子类被重写方法与父类被重写方法有相同的返回类型**，比父类被重写方法更好访问，不能比父类被重写方法声明更多的异常（里氏代换原则）。重载对返回类型没有特殊的要求。

## 异常

### throws,throw,try,catch,finally

Java 通过面向对象的方法进行异常处理，把各种不同的异常进行分类，并提供了良好的接口。

在Java中，**每个异常都是一个对象，它是Throwable类或其它子类的实例**。当一个方法出现异常后便抛出一个异常对象，该对象中包含有异常信息，调用这个对象的方法可以捕获到这个异常并进行处理。

Java的异常处理是通过5个关键词来实现的：**try、catch、throw、throws和finally**。一般情况下是用try来执行一段程序，如果出现异常，系统会抛出（throws）一个异常，这时候你可以通过它的类型来捕捉（catch）它，或最后（finally）由缺省处理器来处理。用try来指定一块预防所有”异常”的程序。紧跟在try程序后面，应包含一个catch子句来指定你想要捕捉的”异常”的类型。**throw语句用来明确地抛出一个”异常”**。**throws用来标明一个`成员函数`可能抛出的各种”异常”**。Finally为确保一段代码不管发生什么”异常”都被执行一段代码。可以在一个成员函数调用的外面写一个try语句，在这个成员函数内部写另一个try语句保护其他代码。每当遇到一个try语句，”异常“的框架就放到堆栈上面，直到所有的try语句都完成。如果下一级的try语句没有对某种”异常”进行处理，堆栈就会展开，直到遇到有处理这种”异常”的try语句。

## 抽象，接口

### Java的接口和C++的虚类的相同和不同处。

由于Java不支持多继承，而有可能某个类或对象要使用分别在几个类或对象里面的方法或属性，现有的单继承机制就不能满足要求。
与继承相比，接口有更高的灵活性，因为接口中没有任何实现代码。当一个类实现了接口以后，该类要实现接口里面所有的方法和属性，并且接口里面的属性在默认状态下面都是public static,所有方法默认情况下是public.一个类可以实现多个接口。

### abstract class和interface有什么区别

声明方法的存在而不去实现它的类被叫做抽象类（abstract class），它用于要创建一个体现某些基本行为的类，并为该类声明方法，但不能在该类中实现该类的情况。**不能创建abstract 类的实例**。然而**可以创建一个变量，其类型是一个抽象类，并让它指向具体子类的一个实例**。不能有抽象构造函数或抽象静态方法。Abstract 类的子类为它们父类中的所有抽象方法提供实现，否则它们也是抽象类为。取而代之，在子类中实现该方法。知道其行为的其它类可以在类中实现这些方法。

接口（interface）是抽象类的变体。在接口中，所有方法都是抽象的。多继承性可通过实现这样的接口而获得。接口中的所有方法都是抽象的，没有一个有程序体。接口只可以定义static final成员变量。接口的实现与子类相似，除了该实现类不能从接口定义中继承行为。当类实现特殊接口时，它定义（即将程序体给予）所有这种接口的方法。然后，它可以在实现了该接口的类的任何对象上调用接口的方法。由于有抽象类，它允许使用接口名作为引用变量的类型。通常的动态联编将生效。引用可以转换到接口类型或从接口类型转换，instanceof 运算符可以用来决定某对象的类是否实现了接口。

### Comparable和Comparator接口的作用以及它们的区别

Java提供了只包含一个compareTo()方法的Comparable接口。这个方法可以个给两个对象排序。具体来说，它返回负数，0，正数来表明输入对象小于，等于，大于已经存在的对象。
Java提供了包含compare()和equals()两个方法的Comparator接口。compare()方法用来给两个输入参数排序，返回负数，0，正数表明第一个参数是小于，等于，大于第二个参数。equals()方法需要一个对象作为参数，它用来决定输入参数是否和comparator相等。只有当输入参数也是一个comparator并且输入参数和当前comparator的排序结果是相同的时候，这个方法才返回true。

## 浅拷贝和深拷贝

https://blog.csdn.net/bingbeichen/article/details/83615903

## 零拷贝

**netty零拷贝**

​		从WIKI的定义中，我们看到**“零拷贝”是指计算机操作的过程中，CPU不需要为数据在内存之间的拷贝消耗资源。**而它通常是指计算机在网络上发送文件时，不需要将文件内容拷贝到用户空间（User Space）而直接在内核空间（Kernel Space）中传输到网络的方式。

![1582268598944](F:\typoraImg\1582268598944.png)

​		在零拷贝中，**避免了数据在用户空间和内存空间实际的拷贝**。从而提高性能。**Linux**中的**sendfile**()以及Java **NIO**中的**FileChannel.transferTo()**方法都实现了零拷贝的功能，而在Netty中也通过在FileRegion中包装了NIO的FileChannel.transferTo()方法实现了零拷贝。

详情看https://www.javazhiyin.com/59585.html

## 反射

### 定义

JAVA语言编译之后会生成一个.class文件，反射就是通过字节码文件找到某一个类、类中的方法以及属性等。反射的实现主要借助以下四个类：Class：类的对象，Constructor：类的构造方法，Field：类中的属性对象，Method：类中的方法对象。

作用：反射机制指的是程序在运行时能够获取自身的信息。在JAVA中，只要给定类的名字，那么就可以通过反射机制来获取类的所有信息。

### 反射创建对象的方法

- 方法1：通过**类对象调用newInstance()方法**，例如：String.class.newInstance()
- 方法2：通过**类对象的getConstructor()或getDeclaredConstructor()方法获得构造器（Constructor）对象并调用其newInstance()方法创建对象**，例如：String.class.getConstructor(String.class).newInstance("Hello");

## 泛型

### extends 和super 泛型限定符

详情：https://www.zhihu.com/question/20400700

（1）泛型中上界和下界的定义

上界<? extend Fruit>

下界<? super Apple>

（2）上界和下界的特点

上界的list只能get，不能add（确切地说不能add出除null之外的对象，包括Object）

下界的list只能add，只能get的时候只能用Object类型

（3）上界<? extend Fruit> ，表示所有继承Fruit的子类，但是具体是哪个子类，无法确定，所以调用add的时候，要add什么类型，谁也不知道。但是get的时候，不管是什么子类，不管追溯多少辈，肯定有个父类是Fruit，所以，我都可以用最大的父类Fruit接着，也就是把所有的子类向上转型为Fruit。

下界<? super Apple>，表示Apple的所有父类，包括Fruit，一直可以追溯到老祖宗Object 。那么当我add的时候，我不能add Apple的父类，因为不能确定List里面存放的到底是哪个父类。但是我可以add Apple及其子类。因为不管我的子类是什么类型，它都可以向上转型为Apple及其所有的父类甚至转型为Object 。但是当我get的时候，Apple的父类这么多，我用什么接着呢，除了Object，其他的都接不住。

![1582530565127](F:\typoraImg\1582530565127.png)

> PECS原则

PECS原则（producer extends consumer super）

1. **频繁往外读取内容的，适合用上界Extends。**
2. **经常往里插入的，适合用下界Super。**



## I/O

### 如何实现java序列化

原型模式会使用到java序列化的知识

序列化就是一种用来处理对象流的机制，所谓对象流也就是将对象的内容进行流化。可以对流化后的对象进行读写操作，也可将流化后的对象传输于网络之间。序列化是为了解决在对对象流进行读写操作时所引发的问题。
序列化的实现：将需要被序列化的类实现Serializable接口，该接口没有需要实现的方法，implements Serializable只是为了标注该对象是可被序列化的，然后使用一个输出流(如：FileOutputStream)来构造一个 ObjectOutputStream(对象流)对象，接着，使用ObjectOutputStream对象的writeObject(Object obj)方法就可以将参数为obj的对象写出(即保存其状态)，要恢复的话则用输入流。

## Java的内存模型

Java内存模型即Java Memory Model，简称**JMM**。JMM定义了Java 虚拟机(JVM)在计算机内存(RAM)中的工作方式。JVM是整个计算机虚拟模型，所以**JMM是隶属于JVM**的。

如果我们要想深入了解Java并发编程，就要先理解好Java内存模型。**Java内存模型定义了多线程之间共享变量的可见性以及如何在需要的时候对共享变量进行同步**。原始的Java内存模型效率并不是很理想，因此Java1.5版本对其进行了重构，现在的Java8仍沿用了Java1.5的版本。

*** ** **java使用共享内存的方式来实线程之间的通信**，这种共享内存的模型就是java内存模型。共享内存这里指的是主内存（物理内存），每个线程都独有一个本地内存，**本地内存中存储了该线程以读/写共享变量的副本**。本地内存是JMM的一个抽象概念，并不真实存在。它涵盖了缓存，写缓冲区，寄存器以及其他的硬件和编译器优化。当线程要修改变量的时候，从主存读取，修改后再刷新到主存，整个流程即是修改变量的过程。
![1581681715124](F:\typoraImg\1581681715124.png)

此知识点涉及到**可见性**和**线程安全**的问题（volatile，synchronized	）

## String、StringBuffer和StringBuilder

**String**：String是不可变对象final，底层用了一个**char数组**来保存他的所有字符。平时使用的a + b的方法其实都是重新创建了一个新的String对象，再赋值给新对象，再指向新对象的引用的，而不是原来的对象。

String真的不可变吗？不是的，可以通过反射的方式拿到char数组，对里面的字符进行修改。

![1582255228638](F:\typoraImg\1582255228638.png)

StringBuffer线程安全，StringBuilder线程不安全

底层实现上的话，StringBuffer其实就是比StringBuilder多了Synchronized修饰符。

## Java中是否可以覆盖(override)一个private或者是static的方法？

“static”关键字表明一个成员变量或者是成员方法可以在没有所属的类的实例变量的情况下被访问。
Java中**static方法不能被覆盖**，因为**方法覆盖是基于运行时动态绑定**的，而**static方法是编译时静态绑定的**。static方法跟类的任何实例都不相关，所以概念上不适用。

## Object类的方法并简要说明

**Object**()默认构造方法。

**clone**() 创建并返回此对象的一个副本。

**equals**(Object obj) 指示某个其他对象是否与此对象“相等”。

**finalize**()当垃圾回收器确定不存在对该对象的更多引用时，由对象的垃圾回收器调用此方法。

**getClass**()返回一个对象的运行时类。

**hashCode**()返回该对象的哈希码值。

 **notify**()唤醒在此对象监视器上等待的单个线程。 

**notifyAll**()唤醒在此对象监视器上等待的所有线程。

**toString**()返回该对象的字符串表示。

**wait**()导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法。

**wait**(long timeout)导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者超过指定的时间量。

**wait**(long timeout, int nanos) 导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者其他某个线程中断当前线程，或者已超过某个实际时间量。

## 集合类

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

