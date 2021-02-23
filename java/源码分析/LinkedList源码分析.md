先来看看LinkedList的类图

![image-20201130114103770](http://img.fosuchao.com/image-20201130114103770.png)



## 成员变量

![image-20201130115242019](http://img.fosuchao.com/image-20201130115242019.png)



## 存储结构

LinkedList中使用内部类Node来存储元素。

![image-20201130115406158](http://img.fosuchao.com/image-20201130115406158.png)



## 方法

LinkedList自己实现的方法主要有以下几个

![image-20201130115647051](http://img.fosuchao.com/image-20201130115647051.png)

- linkFirst：插入到头节点
- linkLast：插入到尾节点
- linkBefore：插入一个节点的前一个节点
- unlinkFirst：删除头节点
- unlinkLast：删除尾节点
- unLink：删除某个节点

### Deque

另外还可以使用Deque中定义的一些方法

![image-20201130120824480](http://img.fosuchao.com/image-20201130120824480.png)

![image-20201130120839712](http://img.fosuchao.com/image-20201130120839712.png)

