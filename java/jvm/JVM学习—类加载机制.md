# JVM类加载机制

JVM由三个主要的子系统构成

- 类加载器系统

- 运行时数据区（内存结构）

- 执行引擎

## JVM的整体结构

​		我们写好的.java文件通过java编译器编译后，生成.class字节码文件，类加载器将.class文件加载到运行时数据区，执行引擎执行.class文件。

![](http://img.fosuchao.com/20200304093020.png)

## 类加载过程

​		类加载过程分成多个部分。

![](http://img.fosuchao.com/20200304093326.png)

类加载：类加载器将class文件加载到虚拟机的内存

- 加载：在硬盘上查找并通过IO读入字节码文件

  1. 通过一个类的全限定名来获取定义此类的二进制字节流
  2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
  3. 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口。

- 连接：执行校验、准备、解析（可选）步骤
  - 验证：校验字节码文件的正确性

    1. 文件格式的验证（魔术Magic，主次版本号的范围）

    2. 元数据验证

       ![image-20200724160546629](http://img.fosuchao.com/image-20200724160546629.png)

    3. 字节码验证

    4. 符号引用验证

       ![image-20200724160624930](http://img.fosuchao.com/image-20200724160624930.png)

  - 准备：给类的**静态变量**分配内存，并**赋予默认值**

    ![image-20200724160736553](http://img.fosuchao.com/image-20200724160736553.png)

  - 解析：类装载器装入类所引用的其他所有类

    1. 类或接口的解析
    2. 字段解析
    3. 方法解析
    4. 接口方法解析

- 初始化：对类的静态变量**初始化为指定的值**，**执行静态代码块**



**VM加载jar包是否会将包里的所有类全部加载进内存？**

答：JVM对class文件是**按需加载**(**运行期间动态加载**)，非一次性加载，见示例(启动需要加上参数：`-verbose:class`，可查看启动时类加载过程。)

## 类加载器种类

- 启动类加载器（Bootstrap ClassLoader）：

  负责加载JRE的核心类库，如`jre`目标下的`rt.jar, charsets.jar`等

- 扩展类加载器（Extension ClassLoader）：

  负责加载JRE扩展目录`ext`中JAR类包

- 系统类加载器（Application ClassLoader）

  负责加载`ClassPath`路径下的类包，就是我们写的代码

- 自定义类加载器（Custom ClassLoader）：

  负责加载用户自定义路径下的类包

### 双亲委派机制

![](http://img.fosuchao.com/20200304093800.png)

- 全盘负责委托机制：

  当一个ClassLoader加载一个类时，除非显示的使用另一个ClassLoader，该类所依赖和引用的类也由这个ClassLoader载入

- 双亲委派机制：

  一个类加载器收到了类加载请求，首先不会自己去加载，二十先委托上级加载器寻找目标类，直到启动类加载器，如果还找不到，则在交给子加载器处理。如果找到了，则结束。

### 双亲委派模式优势

- 沙箱安全机制：防止JDK的核心类被恶意篡改
- 避免类的重复加载：当上级类加载器加载成功，就没有必要由子加载器再次加载。



启动类加载器无法被Java程序直接引用，用户在编写自定义类加载器时，如果需要把加载请求委派给引导类加载器去处理，那直接使用null代替即可，下面的代码展示的就是`java.lang.ClassLoader.getClassLoader()`方法去的代码片段，其中的注释和代码实现都明确地说明了以null值来代表引导类加载器的约定规则。

ClassLoader.getClassLoader()方法的代码片段：

![image-20200724162327883](http://img.fosuchao.com/image-20200724162327883.png)

### 双亲委派的实现

![image-20200724162945716](http://img.fosuchao.com/image-20200724162945716.png)

### 破坏双亲委派机制

**打破双亲委派模型的三种情况**

1. 双亲委派模型与JDK1.2时引入的，为了向前兼容，新增一个 protected 的 findClass() 方法。在此之前用户继承 ClassLoader 类就是为了重写 loadClass() 方法。而之后则不推荐覆盖 loadClass() 方法，而是**把类加载逻辑写到 findClass() 方法中**。

2. 双亲委派很好的解决了各个类加载器的基础类的统一问题。但是如果基础类需要回调用户代码则无法实现。*比如 JNDI（Java 命名和目录接口），由启动类加载器加载，实现对资源进行集中管理和查找，它需要调用由独立厂商实现并部署在应用程序的 ClassPath 下的 JNDI 接口提供者（SPI），但是启动类加载器不可能“认识”这些代码。JDBC、JCE、JAXB、JBI也是如此。*

   解决方案为：**线程上下文类加载器**。该类加载器通过 Thread 类的 setContextClassLoader() 方法进行设置。 如果创建线程时没有还未设置，则从父线程中继承一个，如果应用程序的全局范围内都没有设置过的话，那么该类加载器就是应用程序类启动器。

   然后上下文类加载器去加载所需要的SPI代码，也就是父类加载器请求子类加载器去完成类加载的动作。

3. 由于用户对程序动态性的追求而导致。比如**代码热替换、模块热部署**等。

   OSGI（Open Service Gateway Initiative）开放服务网关协议为 Java 动态化模块化系统的一系列规范。而 **OSGI 实现模块化热部署的关键则是它自定义的类加载器机制的实现**。每一个程序模块（OSGI中称为 Bundle）都有一个自己的类加载器，当需要更换一个 Bundle 时，就把 Bundle 连同类加载器一起换掉以实现代码的热替换。

   在OSGI环境下，类加载器不再是双亲委派模型的树状结构，而是进一步发展为更加复杂的网状结构。加载流程为：

   > 1. 将以 java.* 开头的类委派给父类加载器加载
   > 2. 否则，将委派列表名单内的类委派给父类加载器加载
   > 3. 否则，将 Import 列表中的类委派给 Export 这个类的 Bundle 的类加载器加载
   > 4. 否则，查找当前 Bundle 的 ClassPath ，使用自己的类加载器加载
   > 5. 否则，查找类是否在自己的 Fragment Bundle 中，如果在，则委派给 Fragment Bundle 的类加载器加载
   > 6. 否则，查找 Dynamic Import 列表的 Bundle，委派给对应的 Bundle 的类加载器加载
   > 7. 否则，类查找失败。