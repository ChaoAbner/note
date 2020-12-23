LinkedHashMap是一个根据插入或访问顺序实现的有序的HashMap

底层是使用双向链表实现的，继承了hashmap，复写了一些方法实现有序

默认是根据插入顺序排序，比如put(1, "a"), put(2, "b"), put(3, "c") 

查询的values顺序就是 a b c。

## 类图

![image-20201203120306947](http://img.fosuchao.com/image-20201203120306947.png)



## 成员变量

![image-20201203120718526](http://img.fosuchao.com/image-20201203120718526.png)



## 实现

### 定义

![image-20201203142958835](http://img.fosuchao.com/image-20201203142958835.png)

### 构造器

过构造函数传入accessOrder值控制插入顺序或访问顺序存储

![image-20201203143059925](http://img.fosuchao.com/image-20201203143059925.png)

### 实现顺序存储

#### 插入顺序

![image-20201203143420307](http://img.fosuchao.com/image-20201203143420307.png)

#### 访问顺序

当传入的accessOrder为true时，在get方法中会调用 afterNodeAccess，将被访问节点移动到链表最后

![image-20201203143653511](http://img.fosuchao.com/image-20201203143653511.png)

**afterNodeAccess**

![image-20201203143727852](http://img.fosuchao.com/image-20201203143727852.png)

### 实现LRU

我们可以通过继承LinkedHashMap并且重写removeEldestEntry方法，返回值为true来实现LRU

![image-20201203143508808](http://img.fosuchao.com/image-20201203143508808.png)

### 删除节点

LinkedHashMap 重写了afterNodeRemoval方法，删除节点后重新整理链表

![image-20201203143928856](http://img.fosuchao.com/image-20201203143928856.png)