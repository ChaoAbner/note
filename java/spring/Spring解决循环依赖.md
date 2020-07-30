# Spring解决循环依赖

## 什么是循环依赖？

循环依赖其实就是循环引用，也就是两个或则两个以上的bean互相持有对方，**最终形成闭环**。比如A依赖于B，B依赖于C，C又依赖于A。

注意，这里不是函数的循环调用，是对象的相互依赖关系。循环调用其实就是一个死循环，最终的结果就是导致程序内存溢出。

![image-20200725113301622](http://img.fosuchao.com/image-20200725113301622.png)

## 简单的程序模拟

创建一个A类：

```java
public class A {

    public A() {
        new B();
    }
}
```

创建一个B类：

```java
public class B {

    public B() {
        new A();
    }
}
```

在A类和B类的构造函数中分别实例化对方。

就会产生循环依赖的错误，最终造成栈溢出。

![image-20200725112720851](http://img.fosuchao.com/image-20200725112720851.png)

## Spring的循环依赖

Spring中循环依赖场景有：

1. 构造器的循环依赖

   ```java
   @Service
   public class A {
       
       public A(B b) {
       }
   }
   
   @Service
   public class B {
       
       public B(A a) {
       }
   }
   ```

   > 构造器造成的循环依赖，Spring是无法解决的。只会抛出一个`BeanCurrentlyInCreationException`异常，表示发生了循环依赖。这也是构造器注入的劣势。

2. field属性（setter方法）的循环依赖。

   ```java
   @Service
   public class AServiceImpl implement AService {
       
       @Autowired
       BService bservice;
   }
   
   @Service
   public class BServiceImpl implement BService {
       
       @Autowired
       AService aservice;
   }
   ```

   这是一个我们平时很常用的注入服务代码，两个服务之间相互调用是非常常见的。但是这种情况也构成了循环依赖。

   但是我们在使用的时候并没有报错，正是因为Spring帮我们解决了这个问题。

## 怎么检测是否存在循环依赖

检测循环依赖相对比较容易，Bean在创建的时候可以给该Bean打标，如果递归调用回来发现正在创建中的话，即说明了循环依赖了。

## Spring怎么解决循环依赖

Spring的循环依赖的理论依据其实是基于Java的引用传递，当我们获取到对象的引用时，对象的field或则属性是可以延后设置的(但是构造器必须是在获取引用之前)。

**Spring的单例对象的初始化主要分为三步：**

（1）`createBeanInstance`：实例化，其实也就是调用对象的构造方法实例化对象

（2）`populateBean`：填充属性，这一步主要是多bean的依赖属性进行填充

（3）`initializeBean`：调用spring xml中的init 方法。

从上面讲述的单例bean初始化步骤我们可以知道，**循环依赖主要发生在第一、第二部。也就是构造器循环依赖和field循环依赖**。

那么我们要解决循环引用也应该从初始化过程着手，对于单例来说，在Spring容器整个生命周期内，有且只有一个对象，所以很容易想到这个对象应该存在Cache中，Spring为了解决单例的循环依赖问题，使用了三级缓存。

首先我们看源码：

```java
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {
	...
	// 从上至下 分表代表这“三级缓存”
	private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256); //一级缓存
	private final Map<String, Object> earlySingletonObjects = new HashMap<>(16); // 二级缓存
	private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16); // 三级缓存
	...
	
	/** Names of beans that are currently in creation. */
	// 这个缓存也十分重要：它表示bean创建过程中都会在里面呆着~
	// 它在Bean开始创建时放值，创建完成时会将其移出~
	private final Set<String> singletonsCurrentlyInCreation = Collections.newSetFromMap(new ConcurrentHashMap<>(16));

	/** Names of beans that have already been created at least once. */
	// 当这个Bean被创建完成后，会标记为这个 注意：这里是set集合 不会重复
	// 至少被创建了一次的,都会放进这里
	private final Set<String> alreadyCreated = Collections.newSetFromMap(new ConcurrentHashMap<>(256));
}
```

>  `AbstractBeanFactory`继承自`DefaultSingletonBeanRegistry`

这三级缓存分别指：

1. `singletonObjects`：用于存放完全初始化好的 bean，**从该缓存中取出的 bean 可以直接使用**
2. `earlySingletonObjects`：提前曝光的单例对象的cache，存放原始的 bean 对象（尚未填充属性），用于解决循环依赖
3. `singletonFactories`：单例对象工厂的cache，存放 bean 工厂对象，用于解决循环依赖

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    Object singletonObject = this.singletonObjects.get(beanName);
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
            singletonObject = this.earlySingletonObjects.get(beanName);
            if (singletonObject == null && allowEarlyReference) {
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    singletonObject = singletonFactory.getObject();
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return (singletonObject != NULL_OBJECT ? singletonObject : null);
}

```


上面的代码需要解释两个参数：

`isSingletonCurrentlyInCreation()`判断当前单例bean是否正在创建中，也就是没有初始化完成(比如A的构造器依赖了B对象所以得先去创建B对象， 或则在A的`populateBean`过程中依赖了B对象，得先去创建B对象，这时的A就是处于创建中的状态。)
`allowEarlyReference` 是否允许从`singletonFactories`中通过`getObject`拿到对象。

分析`getSingleton()`的整个过程，Spring首先从一级缓存`singletonObjects`中获取。如果获取不到，并且对象正在创建中，就再从二级缓存`earlySingletonObjects`中获取。如果还是获取不到且允许`singletonFactories`通过getObject()获取，就从三级缓存`singletonFactory.getObject()`(三级缓存)获取，如果获取到了则：

```java
this.earlySingletonObjects.put(beanName, singletonObject);
this.singletonFactories.remove(beanName);
```


从`singletonFactories`中移除，并放入`earlySingletonObjects`中。其实也就是**从三级缓存移动到了二级缓存**。

从上面三级缓存的分析，我们可以知道，Spring解决循环依赖的诀窍就在于singletonFactories这个三级cache。这个cache的类型是ObjectFactory，定义如下：

```java
public interface ObjectFactory<T> {
    T getObject() throws BeansException;
}
```


这个接口在下面被引用

```java
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(singletonFactory, "Singleton factory must not be null");
    synchronized (this.singletonObjects) {
        if (!this.singletonObjects.containsKey(beanName)) {
            this.singletonFactories.put(beanName, singletonFactory);
            this.earlySingletonObjects.remove(beanName);
            this.registeredSingletons.add(beanName);
        }
    }
}
```


​		这里就是解决循环依赖的关键，这段代码发生在`createBeanInstance`之后，也就是说单例对象此时已经被创建出来(调用了构造器)。这个对象已经被生产出来了，虽然还不完美（还没有进行初始化的第二步和第三步），但是已经能被人认出来了（根据对象引用能定位到堆中的对象），所以Spring此时将这个对象提前曝光出来让大家认识，让大家使用。

这样做有什么好处呢？让我们来分析一下“A的某个field或者setter依赖了B的实例对象，同时B的某个field或者setter依赖了A的实例对象”这种循环依赖的情况。

A首先完成了初始化的第一步，并且将自己提前曝光到`singletonFactories`中，此时进行初始化的第二步，发现自己依赖对象B，此时就尝试去get(B)，发现B还没有被create，所以走create流程，B在初始化第一步的时候发现自己依赖了对象A，于是尝试get(A)，尝试一级缓存`singletonObjects`(肯定没有，因为A还没初始化完全)，尝试二级缓存`earlySingletonObjects`（也没有），尝试三级缓存`singletonFactories`，由于A通过ObjectFactory将自己提前曝光了，所以B能够通过ObjectFactory.getObject拿到A对象(虽然A还没有初始化完全，但是总比没有好呀)，B拿到A对象后顺利完成了初始化阶段1、2、3，**完全初始化之后将自己放入到一级缓存singletonObjects中**。此时返回A中，A此时能拿到B的对象顺利完成自己的初始化阶段2、3，最终A也完成了初始化，进去了一级缓存`singletonObjects`中，而且更加幸运的是，由于B拿到了A的对象引用，所以B现在hold住的A对象完成了初始化。

知道了这个原理时候，肯定就知道为啥Spring不能解决“A的构造方法中依赖了B的实例对象，同时B的构造方法中依赖了A的实例对象”这类问题了！因为加入`singletonFactories`三级缓存的前提是执行了构造器，所以构造器的循环依赖没法解决。

