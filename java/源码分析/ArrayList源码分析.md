ArrayList是一个可以动态扩容的数组

先来看看它的类图

![image-20201130112808908](http://img.fosuchao.com/image-20201130112808908.png)

接下来分析它的源码。

## 成员变量

先来看看ArrayList中存在的主要的成员变量

![image-20201130100720073](http://img.fosuchao.com/image-20201130100720073.png)

## 存储结构

在ArrayList底层默认是使用Object数组来存储元素的。

## 构造器

ArrayList有三个构造器，分别是：

![image-20201130095528058](http://img.fosuchao.com/image-20201130095528058.png)

- 无参构造器
- 参数为继承Collection的集合对象
- 参数为整型值

### ArrayList()

![image-20201130104515396](http://img.fosuchao.com/image-20201130104515396.png)

无参构造器默认将`elementData`赋值为一个空数组`DEFAULTCAPACITY_EMPTY_ELEMENTDATA`。

### ArrayList(Collection<? extends E> c)

判断参入的参数数组长度是否等于0

- 不等于0则判断参数的类型是不是ArrayList
  - 相等则直接赋值给elementData
  - 不相等则通过Arrays.copyOf的方法复制给elementData
- 等于0则将elementData初始化为空数组

![image-20201130104710829](http://img.fosuchao.com/image-20201130104710829.png)

### ArrayList(int initialCapacity)

传入的参数作为初始容量。

> 注意这里会对负数的参数进行非法性判断

![image-20201130104743559](http://img.fosuchao.com/image-20201130104743559.png)



## 动态扩容

分析一下ArrayList动态扩容的流程

### add

动态扩容，首先需要加入新的元素，所以我们先来看看add方法

![image-20201130102132310](http://img.fosuchao.com/image-20201130102132310.png)

可以看到，在add方法中，先通过`ensureCapacityInternal`方法对内部的容量进行保证。

确保内部的数组可以添加下一个元素后，才进行赋值，并且size++。

**作用**

保证内部容量能插入元素的前提下，插入新元素

### ensureCapacityInternal

目前方法中只有一个`ensureExplicitCapacity`方法，并且参数还调用了`calculateCapacity`方法，我们分别来看看这两个函数做了什么。

![image-20201130102626796](http://img.fosuchao.com/image-20201130102626796.png)

### calculateCapacity

在`calculateCapacity`方法中，传入了当前的elementData和添加元素后的size

判断`elementData`是否等于`DEFAULTCAPACITY_EMPTY_ELEMENTDATA`，即调用无参构造器且没有元素的情况，如果等于的话，则返回默认长度，否则返回`minCapacity`。

**作用**

判断空元素的情况，返回添加元素后的容量

![image-20201130102754845](http://img.fosuchao.com/image-20201130102754845.png)

### ensureExplicitCapacity

**作用**

判断当前`elementData`是否能容下新的元素，如果不能，则调用`grow`进行扩容。

![image-20201130103328498](http://img.fosuchao.com/image-20201130103328498.png)

### grow

可以看出新容量`newCapacity`是老容量（elementData长度）的1.5倍

这里会判断新容量溢出和是否超出最大限制容量的情况

通过`Arrays.copyOf()`对`elementData`进行扩容。

![image-20201130103743088](http://img.fosuchao.com/image-20201130103743088.png)

### ensureCapacity

在添加第一个元素的时候，在ensureCapacity方法中会判断`elementData`是否等于`DEFAULTCAPACITY_EMPTY_ELEMENTDATA`，如果相等，则会将数组的初始容量赋值为默认值10。

![image-20201130104632825](http://img.fosuchao.com/image-20201130104632825.png)

`ensureCapacity`是一个公有方法，它在ArrayList里面并没有被调用，明显它是提供给开发者调用的

它用来确保数组有足够的容量能够存下需要添加的元素，当我们需要**add大量元素之前使用`ensureCapacity`方法，可以减少增量重新分配的次数**。

我们通过下面的代码实际测试以下这个方法的效果：

![image-20201203145247368](http://img.fosuchao.com/image-20201203145247368.png)

运行结果：

```
使用ensureCapacity方法前：2158
```

![image-20201203145318225](http://img.fosuchao.com/image-20201203145318225.png)

运行结果：

```
使用ensureCapacity方法前：1773
```

通过运行结果，我们可以看出向 ArrayList 添加大量元素之前最好先使用`ensureCapacity` 方法，以减少增量重新分配的次数。

## 注意点

### 线程不安全

ArrayList是线程不安全的，举个例子

当`elementData`的长度为10，元素总数为8的时候，有两个线程并发添加元素，并且同时进入到了`ensureExplicitCapacity`方法中，此时`minCapatity`都是9，但是`elementData[size++] = e`，会被调用两次。

当size为10的时候，就会报数组越界的异常了。

![image-20201130105933499](http://img.fosuchao.com/image-20201130105933499.png)

### Arraylist 和 Vector 的区别

1. `ArrayList` 是 `List` 的主要实现类，底层使用 `Object[ ]`存储，适用于频繁的查找工作，线程不安全 ；
2. `Vector` 是 `List` 的古老实现类，底层使用 `Object[ ]`存储，线程安全的。

### Arraylist 与 LinkedList 区别

1. **是否保证线程安全：** `ArrayList` 和 `LinkedList` 都是不同步的，也就是不保证线程安全；

2. **底层数据结构：** `Arraylist` 底层使用的是 **`Object` 数组**；`LinkedList` 底层使用的是 **双向链表** 数据结构（JDK1.6 之前为循环链表，JDK1.7 取消了循环。注意双向链表和双向循环链表的区别，下面有介绍到！）

3. **插入和删除是否受元素位置的影响：**

    ① **`ArrayList` 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。** 比如：执行`add(E e)`方法的时候， `ArrayList` 会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是 O(1)。但是如果要在指定位置 i 插入和删除元素的话（`add(int index, E element)`）时间复杂度就为 O(n-i)。因为在进行上述操作的时候集合中第 i 和第 i 个元素之后的(n-i)个元素都要执行向后位/向前移一位的操作。 

   ② **`LinkedList` 采用链表存储，所以对于`add(E e)`方法的插入，删除元素时间复杂度不受元素位置的影响，近似 O(1)，如果是要在指定位置`i`插入和删除元素的话（`(add(int index, E element)`） 时间复杂度近似为`o(n))`因为需要先移动到指定位置再插入。**

4. **是否支持快速随机访问：** `LinkedList` 不支持高效的随机元素访问，而 `ArrayList` 支持。快速随机访问就是通过元素的序号快速获取元素对象(对应于`get(int index)`方法)。

5. **内存空间占用：** `ArrayList` 的空 间浪费主要体现在在 list 列表的结尾会预留一定的容量空间，而 `LinkedList` 的空间花费则体现在它的每一个元素都需要消耗比 `ArrayList` 更多的空间（因为要存放直接后继和直接前驱以及数据）。