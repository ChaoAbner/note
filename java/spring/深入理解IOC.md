深入理解IOC容器

此篇博客是读Spring官方文档后的一些记录和理解。

## IOC是什么

Inversion of Control (IoC)，控制反转，通过容器将所有配置的Bean统一存储管理起来，其它类或对象直接从容器中获取锁需要的对象即可。这种方式将类对象之间更好的解耦。实现依赖倒转。

通常不使用IOC时类之间的关系：

![](http://img.fosuchao.com/20200302171112.png)

使用了IOC：

![](http://img.fosuchao.com/20200302171229.png)

## IOC示例

### 配置信息

可以通过配置xml文件来设置bean信息，当然xml配置只是其中一种方式，还可以使用注解配置，甚至使用我们自己自定义的配置方法。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">

  <bean id="..." class="...">
    <!-- collaborators and configuration for this bean go here -->
  </bean>

  <bean id="..." class="...">
    <!-- collaborators and configuration for this bean go here -->
  </bean>

  <!-- more bean definitions go here -->

</beans>
```

### 初始化容器

可以创建多个xml配置文件，通过数组导入。

```java
ApplicationContext context = new ClassPathXmlApplicationContext(
        new String[] {"services.xml", "daos.xml"});

// ApplicationContext也是一个BeanFactory
BeanFactory factory = context;
```

## Bean定义

在容器中Bean被定义成`BeanDefinition`，BeanDefinition包括一下的信息。

| Feature                                 | Explained in...                                              |
| --------------------------------------- | ------------------------------------------------------------ |
| class（类名）                           | [Instantiating beans](https://docs.spring.io/spring/docs/2.5.x/reference/beans.html#beans-factory-class) |
| name（bean的名称或id，唯一）            | [Naming beans](https://docs.spring.io/spring/docs/2.5.x/reference/beans.html#beans-beanname) |
| scope（bean作用域）                     | [Bean scopes](https://docs.spring.io/spring/docs/2.5.x/reference/beans.html#beans-factory-scopes) |
| constructor arguments（构造函数参数）   | [Injecting dependencies](https://docs.spring.io/spring/docs/2.5.x/reference/beans.html#beans-factory-collaborators) |
| properties（构造函数参数值）            | [Injecting dependencies](https://docs.spring.io/spring/docs/2.5.x/reference/beans.html#beans-factory-collaborators) |
| autowiring mode                         | [Autowiring collaborators](https://docs.spring.io/spring/docs/2.5.x/reference/beans.html#beans-factory-autowire) |
| dependency checking mode                | [Checking for dependencies](https://docs.spring.io/spring/docs/2.5.x/reference/beans.html#beans-factory-dependencies) |
| lazy-initialization mode（懒加载模式）  | [Lazily-instantiated beans](https://docs.spring.io/spring/docs/2.5.x/reference/beans.html#beans-factory-lazy-init) |
| initialization method（bean初始化方法） | [Initialization callbacks](https://docs.spring.io/spring/docs/2.5.x/reference/beans.html#beans-factory-lifecycle-initializingbean) |
| destruction method（bean销毁方法）      | [Destruction callbacks](https://docs.spring.io/spring/docs/2.5.x/reference/beans.html#beans-factory-lifecycle-disposablebean) |

除了使用配置来定义bean的信息，某些BeanFactory实现还允许**注册工厂**外创建的现有对象(通过用户代码)

## 依赖注入 —DI

Dependencies Injection, 依赖注入(DI)背后的基本原理是,对象定义它们的依赖项(也就是说他们使用的其他对象)只能通过**构造函数参数,参数工厂方法或属性的设置在对象实例后构造或从一个工厂方法返回**。

容器的工作是在创建bean时实际注入这些依赖项。从根本上说，这与bean本身负责实例化或定位其依赖项的名称控制反转(IoC)是相反的

DI存在于两个主要的变体中，即**构造函数**注入和**Setter**注入。Spring推荐使用setter注入，因为构造函数参数过多可能导致混乱。

### Constructor注入

```java
public class Foo {

    public Foo(Bar bar, Baz baz) {
        // ...
    }
}
```

**xml配置**

`constructor-arg`：构造函数参数，可以指定参数的**类型type**，**值value**

```xml
<beans>
    <bean name="foo" class="x.y.Foo">
        <constructor-arg>
            <bean class="x.y.Bar"/>
        </constructor-arg>
        <constructor-arg>
            <bean class="x.y.Baz"/>
        </constructor-arg>
    </bean>
</beans>
```

### Setter注入

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can 'inject' a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually 'uses' the injected MovieFinder is omitted...
}
```

### 循环依赖问题

如果主要使用构造函数注入，那么可以编写和配置类和bean，从而创建一个不可解析的循环依赖场景。
考虑这样的场景: 类A需要通过构造函数注入提供类B的实例，而类B需要通过构造函数注入提供类A的实例。如果您将bean配置为类A和B相互注入，Spring IoC容器将在运行时检测这个循环引用，并抛出一个`BeanCurrentlyInCreationException`异常。

## 自动装配—Autowire

自动装配功能有五种模式。每个bean都指定了自动装配，因此可以为某些bean启用自动装配，而不需要自动装配其他bean。使用自动装配，可以减少或消除指定属性或构造函数参数的需要，从而节省大量的输入

### 自动装配模式

| Mode        | Explanation                                                  |
| ----------- | ------------------------------------------------------------ |
| no          | 完全没有自动装配。Bean引用必须通过' ref '元素定义。这是缺省设置，对于较大的部署，不建议更改此设置，因为显式地指定协作者可以提供更好的控制和清晰度。在某种程度上，它是关于系统结构的一种文档形式。 |
| byName      | 通过**属性名**自动装配。该选项将检查容器并查找与需要自动装配的属性名称完全相同的bean。例如，如果您有一个bean定义，它通过名称设置为autowire，并且它包含一个*master*属性(也就是说，它有一个*setMaster(..)*方法)，Spring将查找一个名为' master '的bean定义，并使用它来设置属性。 |
| byType      | 如果容器中恰好有一个属性类型的bean，则允许自动装配该属性。如果有一个以上的异常，就会抛出一个致命的异常，这表明您不能对该bean使用*byType* autowiring。 |
| constructor | 这类似于*byType*，但适用于构造函数参数。如果容器中没有一个构造函数参数类型的bean，则会引发致命错误。 |
| autodetect  | 通过bean类的自行选择*构造函数*或*byType*。如果找到默认构造函数，将应用*byType*模式。 |

#### **优点**

- 自动装配可以显著减少所需的配置量。
- 自动装配会导致配置随着对象的发展而不断更新。例如，如果您需要向类添加一个附加依赖项，则可以自动满足该依赖项，而不需要修改配置。

#### **缺点**

- Spring管理对象之间的关系不能明确地记录下来
- 连接信息可能对从Spring容器生成文档的工具不可用。

## 方法注入

Method Injection

​		对于大多数应用程序场景，容器中的大多数bean都是单例的。当一个单例bean需要与另一个单例bean协作时，或者一个非单例bean需要与另一个非单例bean协作时，通过将一个bean定义为另一个bean的属性来处理这种依赖关系的典型而常见的方法已经足够了。当bean的生命周期不同时，就会出现问题。考虑一个单例bean a，它需要使用一个非单例(原型)bean B，可能在a上的每个方法调用上都是这样。容器将只创建单例bean a。

## Bean的作用域

Bean scopres

​		容器中的bean基本都是单例的。

​		当我们创建一个bean定义时，实际上创建的是一个用于创建由该bean定义定义的类的实际实例的方法。bean定义是菜谱的概念很重要，因为它意味着，就像类一样，您可以从**单个菜谱创建多个对象实例**。

​		不仅可以控制要插入到由特定bean定义创建的对象中的各种依赖项和配置值，还可以**控制由特定bean定义创建的对象的范围**。这种方法非常强大，允许您灵活地选择通过配置创建的对象的范围，而不必在Java类级别“加入”对象的范围。可以将bean定义为部署在多种作用域中的一种:开箱即用，Spring框架正好支持五种作用域(其中三种作用域)

| Scope                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [singleton](https://docs.spring.io/spring/docs/2.5.x/reference/beans.html#beans-factory-scopes-singleton) | 针对每个Spring IoC容器将单个bean定义作用于单个对象实例。     |
| [prototype](https://docs.spring.io/spring/docs/2.5.x/reference/beans.html#beans-factory-scopes-prototype) | 将单个bean定义作用于任意数量的对象实例。                     |
| [request](https://docs.spring.io/spring/docs/2.5.x/reference/beans.html#beans-factory-scopes-request) | 将单个bean定义的范围限定为单个HTTP请求的生命周期;也就是说，每个HTTP请求都将在单个bean定义的基础上创建自己的bean实例。仅在web感知的Spring“ApplicationContext”上下文中有效。 |
| [session](https://docs.spring.io/spring/docs/2.5.x/reference/beans.html#beans-factory-scopes-global-session) | 将单个bean定义定义为HTTP“会话”的生命周期。仅在web感知的Spring“ApplicationContext”上下文中有效。 |
| [global session](https://docs.spring.io/spring/docs/2.5.x/reference/beans.html#beans-factory-scopes-global-session) | 将单个bean定义的范围限定为全局HTTP“会话”的生命周期。通常只在portlet上下文中使用时有效。仅在web感知的Spring“ApplicationContext”上下文中有效。 |

下面介绍singleton和prototype两种作用域，并借用Spring官方的图解

**singleton scope**

![](http://img.fosuchao.com/20200302192025.png)

**prototype scope**

![](http://img.fosuchao.com/20200302192215.png)

## 自定义Bean

### initialization callbacks

​		实现`org.springframework.beans.factory.InitializingBean`接口允许bean在容器设置了bean**的所有必要属性之后执行初始化工作**

```xml
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
```

在容器初始化ExampleBean类的bean的时候，执行init方法

```java
public class ExampleBean {
    public void init() {
        // do some initialization work
    }
}
```

### Destruction callbacks

实施`org.springframework.beans.factory`。当包含它的容器被销毁时，可处置bean接口允许bean获得回调。可处置bean接口指定了一个方法:

```xml
<bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
```

当exampleInitBean的容器被销毁时，回调cleanup方法。（通常是释放资源，help GC）

```java
public class ExampleBean {
    public void cleanup() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

## 容器扩展点

### BeanPostProcessors

这个接口定义了许多回调方法，作为应用程序开发人员，您可以实现这些方法来提供您自己的(或覆盖容器默认值)实例化逻辑、依赖解析逻辑等等。如果您希望在Spring容器完成实例化、配置和以其他方式初始化bean之后执行一些自定义逻辑，您可以插入一个或多个BeanPostProcessor实现。

### beanfactorypostprocessor

​		这个接口的语义类似于BeanPostProcessor，但有一个主要区别: Spring IOC 允许beanfactorypostprocessor读取配置元数据，并可能在容器实际实例化任何其他bean之前更改它。

​		同样，BeanFactoryPostProcessors是每个容器的作用域。这仅在使用容器层次结构时才相关。如果您在一个容器中定义一个BeanFactoryPostProcessor，它将只在该容器中的bean定义上执行其任务。另一个容器中的Bean定义不会被另一个容器中的BeanFactoryPostProcessors后处理，即使这两个容器都是相同层次结构的一部分。

## ApplicationContext

​		ApplicationContext提供了BeanFactory的所有功能。为了以更面向框架的方式工作，使用分层和层次上下文，ApplicationContext还提供以下功能:

- MessageSource 以i18n风格提供对消息的访问。
- *访问资源*，例如url和文件。
- 事件传播到实现“ApplicationListener”接口的bean。
- 加载多个(层次化的)上下文，允许每个上下文都集中在一个特定的层上，例如应用程序的web层。

**ApplicationContext和BeanFactory对比。**

| Feature                                           | `BeanFactory` | `ApplicationContext` |
| ------------------------------------------------- | ------------- | -------------------- |
| Bean instantiation/wiring                         | Yes           | Yes                  |
| Automatic `BeanPostProcessor` registration        | No            | Yes                  |
| Automatic `BeanFactoryPostProcessor` registration | No            | Yes                  |
| Convenient `MessageSource` access (for i18n)      | No            | Yes                  |
| `ApplicationEvent` publication                    | No            | Yes                  |

### 事件—Events

​		ApplicationContext中的事件处理是通过ApplicationEvent类和ApplicationListener接口提供的。

​		如果**实现ApplicationListener接口的bean被部署到上下文中，那么每当将ApplicationEvent发布到ApplicationContext时，该bean都会得到通知**。本质上，这是标准的观察者设计模式。Spring提供了以下标准事件:

- ContextRefreshedEvent

  应用**程序上下文初始化或刷新时**发布，例如，在ConfigurableApplicationContext接口上使用refresh()方法。这里的“初始化”意味着加载所有bean，检测并激活后处理器bean，预实例化单例对象，并且ApplicationContext对象已准备好使用。

- ContextStartedEvent

  在**启动ApplicationContext时**发布，使用ConfigurableApplicationContext接口上的start()方法。这里的“已启动”意味着所有生命周期bean都将收到一个显式的启动信号。这通常用于在显式停止后重新启动，但也可以用于启动尚未配置为自动启动的组件(例如，尚未在初始化时启动)。

- ContextStoppedEvent

  当**ApplicationContext停止**时，使用ConfigurableApplicationContext接口上的stop()方法发布。这里的“停止”意味着所有生命周期bean都将收到一个显式的停止信号。可以通过start()调用重新启动已停止的上下文。

- ContextClosedEvent

  在**ApplicationContext关闭时**发布，在ConfigurableApplicationContext接口上使用close()方法。这里的“关闭”意味着**所有的单例bean都被销毁**了。封闭的环境已经到了生命的尽头;无法刷新或重新启动。

- RequestHandledEvent

  一个特定于web的事件，通知所有bean HTTP请求已得到服务(这将在请求完成后发布)。注意，此事件仅适用于使用Spring的DispatcherServlet的web应用程序。

