# Spring知识点

## Spring

### Spring的七大模块

1. Spring **Core**： Core封装包是框架的最基础部分，提供IOC和依赖注入特性。这里的基础概念是BeanFactory，它提供对Factory模式的经典实现来消除对程序性单例模式的需要，并真正地允许你从程序逻辑中分离出依赖关系和配置。

2. Spring **Context**: 构建于[Core](http://www.mianwww.com/html/2014/03/19750.html#beans-introduction)封装包基础上的 [Context](http://blog.chinaunix.net/u/9295/ch03s08.html)封装包，提供了一种框架式的对象访问方法，有些象JNDI注册器。Context封装包的特性得自于Beans封装包，并添加了对国际化（I18N）的支持（例如资源绑定），事件传播，资源装载的方式和Context的透明创建，比如说通过Servlet容器。

3. Spring **DAO**:  [DAO](http://www.mianwww.com/html/2014/03/19750.html#dao-introduction) (Data Access Object)提供了JDBC的抽象层，它可消除冗长的JDBC编码和解析数据库厂商特有的错误代码。 并且，JDBC封装包还提供了一种比编程性更好的声明性事务管理方法，不仅仅是实现了特定接口，而且对所有的POJOs（plain old Java objects）都适用。

4. Spring **ORM**: [ORM](http://www.mianwww.com/html/2014/03/19750.html#orm-introduction) 封装包提供了常用的“对象/关系”映射APIs的集成层。 其中包括[JPA](http://blog.chinaunix.net/u/9295/ch12s07.html)、[JDO](http://blog.chinaunix.net/u/9295/ch12s03.html)、[Hibernate](http://blog.chinaunix.net/u/9295/ch12s02.html) 和 [iBatis](http://blog.chinaunix.net/u/9295/ch12s06.html) 。利用ORM封装包，可以混合使用所有Spring提供的特性进行“对象/关系”映射，如前边提到的简单声明性事务管理。

5. Spring **AOP**: Spring的 [AOP](http://www.mianwww.com/html/2014/03/19750.html#aop-introduction) 封装包提供了符合AOP Alliance规范的面向方面的编程实现，让你可以定义，例如方法拦截器（method-interceptors）和切点（pointcuts），从逻辑上讲，从而减弱代码的功能耦合，清晰的被分离开。而且，利用source-level的元数据功能，还可以将各种行为信息合并到你的代码中。

6. Spring **Web**: Spring中的 Web 包提供了基础的针对Web开发的集成特性，例如多方文件上传，利用Servlet listeners进行IOC容器初始化和针对Web的ApplicationContext。当与WebWork或Struts一起使用Spring时，这个包使Spring可与其他框架结合。

7. Spring Web **MVC**: Spring中的[MVC](http://www.mianwww.com/html/2014/03/19750.html#mvc-introduction)封装包提供了Web应用的Model-View-Controller（MVC）实现。Spring的MVC框架并不是仅仅提供一种传统的实现，它提供了一种清晰的分离模型，在领域模型代码和Web Form之间。并且，还可以借助Spring框架的其他特性。

### 获取连接池的方式

- DBCP数据源

- C3P0数据源
- Spring的数据源实现类(DriverManagerDataSource)
- 获取JNDI数据源

### 面向对象设计原则

面向对象共有六大原则：**开闭原则、单一职责原则、里式替换原则、依赖倒置原则、接口隔离原则、迪米特法则**。

1. 开闭原则：开闭原则的定义是软件中的对象(类，模块，函数等)应该**对于扩展是开放的，但是对于修改是关闭的。**

2. 单一职责原则：就一个类而言，应该仅有一个引起他变化的原因。也就是说**一个类应该只负责一件事情**。

3. 里式替换原则：“子类能够替换基类，否则不应当设计为其子类。”也就是说，**子类只能去扩展基类，而不是隐藏或覆盖基类。**

4. 依赖倒置原则：**模块间的依赖是通过抽象来发生的，实现类之间不发生直接的依赖关系**，其依赖关系是通过接口是来实现的。

5. 接口隔离原则：客户端不应该依赖他不需要的接口。**类间的依赖关系应该建立在最小的接口上**

6. 迪米特法则：**一个对象应该对其他对象保持最小的了解**。（低耦合）

### bean的生命周期

![1581520565041](F:\typoraImg\1581520565041.png)

#### 1. 实例化Bean

对于BeanFactory容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用createBean进行实例化。 
对于ApplicationContext容器，当容器启动结束后，便实例化所有的bean。 
容器通过获取BeanDefinition对象中的信息进行实例化。并且这一步仅仅是简单的实例化，并未进行依赖注入。 
实例化对象被包装在BeanWrapper对象中，BeanWrapper提供了设置对象属性的接口，从而避免了使用反射机制设置属性。

#### 2. 设置对象属性（依赖注入）

实例化后的对象被封装在BeanWrapper对象中，并且此时对象仍然是一个原生的状态，并没有进行依赖注入。 
紧接着，Spring根据BeanDefinition中的信息进行依赖注入。 
并且通过BeanWrapper提供的设置属性的接口完成依赖注入。

#### 3. 注入Aware接口

紧接着，Spring会检测该对象是否实现了xxxAware接口，并将相关的xxxAware实例注入给bean。

#### 4. BeanPostProcessor

当经过上述几个步骤后，bean对象已经被正确构造，但如果你想要对象被使用前再进行一些自定义的处理，就可以通过BeanPostProcessor接口实现。 
该接口提供了两个函数：

- postProcessBeforeInitialzation( Object bean, String beanName ) 
  当前正在初始化的bean对象会被传递进来，我们就可以对这个bean作任何处理。 
  这个函数会先于InitialzationBean执行，因此称为前置处理。 
  所有Aware接口的注入就是在这一步完成的。
- postProcessAfterInitialzation( Object bean, String beanName ) 
  当前正在初始化的bean对象会被传递进来，我们就可以对这个bean作任何处理。 
  这个函数会在InitialzationBean完成后执行，因此称为后置处理。

#### 5. InitializingBean与init-method

当BeanPostProcessor的前置处理完成后就会进入本阶段。 
InitializingBean接口只有一个函数：

- afterPropertiesSet()

这一阶段也可以在bean正式构造完成前增加我们自定义的逻辑，但它与前置处理不同，由于该函数并不会把当前bean对象传进来，因此在这一步没办法处理对象本身，只能增加一些额外的逻辑。 
若要使用它，我们需要让bean实现该接口，并把要增加的逻辑写在该函数中。然后Spring会在前置处理完成后检测当前bean是否实现了该接口，并执行afterPropertiesSet函数。

当然，Spring为了降低对客户代码的侵入性，给bean的配置提供了init-method属性，该属性指定了在这一阶段需要执行的函数名。Spring便会在初始化阶段执行我们设置的函数。init-method本质上仍然使用了InitializingBean接口。

#### 6. DisposableBean和destroy-method

和init-method一样，通过给destroy-method指定函数，就可以在bean销毁前执行指定的逻辑。**如果Bean类有实现org.springframework.beans.factory.DisposableBean接口，则执行他的destroy()**方法



注解**Scope**描述的是Spring容器如何新建Bean实例的。Spring的**Scope的作用域**有以下几种，通过@Scope注解来实现。

（1）**Singleton**：一个Spring容器中只有一个Bean的实例，此为Spring的默认配置，全容器共享一个实例。

（2）**Prototype**：每次调用新建一个Bean实例。

（3）**Request**：Web项目中，给每一个 http request 新建一个Bean实例。

（4）**Session**：Web项目中，给每一个 http session 新建一个Bean实例。

（5）**GlobalSession**：这个只在portal应用中有用，给每一个 global http session 新建一个Bean实例。

### 关于Servlet

![1581520928549](F:\typoraImg\1581520928549.png)

### Spring中自动装配的方式

- **no**：不进行自动装配，手动设置Bean的依赖关系。
-  **byName**：根据Bean的名字进行自动装配。
-  **byType**：根据Bean的类型进行自动装配。
-  **constructor**：类似于byType，不过是应用于构造器的参数，如果正好有一个Bean与构造器的参数类型相同则可以自动装配，否则会导致错误。
-  **autodetect**：如果有默认的构造器，则通过constructor的方式进行自动装配，否则使用byType的方式进行自动装配。

自动装配没有自定义装配方式那么精确，而且不能自动装配简单属性（基本类型、字符串等），在使用时应注意。

### IoC和DI？并且简要说明一下DI是如何实现的

IOC（inversion of control）即控制反转，DI(dependency injection)即依赖注入，是对IoC更简单的诠释。Spring将传统上程序直接操控对象的方式转变成由容器控制，更利于应用之间的关系解耦。当需要的时候，直接注入IOC容器中的对象。

配置对象的工作应该由容器负责，查找资源的逻辑应该从应用组件的代码中抽取出来，交给容器来完成。DI是对IoC更准确的描述，即**组件之间的依赖关系由容器在运行期决定**，形象的来说，即由**容器动态的将某种依赖关系注入到组件之中**。

**传统方法**：

一个类A需要用到接口B中的方法，那么就需要为类A和接口B建立关联或依赖关系，最原始的方法是在类A中创建一个接口B的实现类C的实例，但这种方法需要开发人员自行维护二者的依赖关系，也就是说当依赖关系发生变动的时候需要修改代码并重新构建整个系统。

**使用IOC容器：**

如果通过一个容器来管理这些对象以及对象的依赖关系，则只需要在类A中定义好用于关联接口B的方法（构造器或setter方法），将类A和接口B的实现类C放入容器中，通过对容器的配置来实现二者的关联。
**依赖注入可以通过setter方法注入（设值注入）、构造器注入和接口注入三种方式来实现**，Spring支持setter注入和构造器注入，通常使用构造器注入来注入必须的依赖关系，对于可选的依赖关系，则setter注入是更好的选择，setter注入需要类提供无参构造器或者无参的静态工厂方法来创建对象。

**实现：**

IoC的一个重点是**在系统运行中，动态的向某个对象提供它所需要的其他对象**。这一点是通过DI（Dependency Injection，依赖注入）来实现的。比如对象A需要操作数据库，以前我们总是要在A中自己编写代码来获得一个Connection对象，有了 spring我们就只需要告诉spring，A中需要一个Connection，至于这个Connection怎么构造，何时构造，A不需要知道。在系统运行时，spring会在适当的时候制造一个Connection，然后像打针一样，注射到A当中，这样就完成了对各个对象之间关系的控制。A需要依赖 Connection才能正常运行，而这个Connection是由spring注入到A中的，依赖注入的名字就这么来的。那么DI是如何实现的呢？ Java 1.3之后一个重要特征是**反射（reflection），它允许程序在运行的时候动态的生成对象、执行对象的方法、改变对象的属性，spring就是通过反射来实现注入的**。（原理：反射）

举个简单的例子，我们找女朋友常见的情况是，我们到处去看哪里有长得漂亮身材又好的女孩子，然后打听她们的兴趣爱好、qq号、电话号、ip号、iq号………，想办法认识她们，投其所好送其所要，这个过程是复杂深奥的，我们必须自己设计和面对每个环节。传统的程序开发也是如此，在一个对象中，如果要使用另外的对象，就必须得到它（自己new一个，或者从JNDI中查询一个），使用完之后还要将对象销毁（比如Connection等），对象始终会和其他的接口或类藕合起来。

②实现IOC的步骤

定义用来描述bean的配置的Java类

解析bean的配置，將bean的配置信息转换为上面的BeanDefinition对象保存在内存中，spring中采用HashMap进行对象存储，其中会用到一些xml解析技术（或者注解）

遍历存放BeanDefinition的HashMap对象，逐条取出BeanDefinition对象，获取bean的配置信息，利用Java的反射机制实例化对象，將实例化后的对象保存在另外一个Map中即可。

### AOP

#### 一些概念

-  连接点（Joinpoint）：**程序执行的某个特定位置**（如：某个方法调用前、调用后，方法抛出异常后）。一个类或一段程序代码拥有一些具有边界性质的特定点，这些代码中的特定点就是连接点。**Spring仅支持方法的连接点**。

- 切点（Pointcut）：如果连接点相当于数据中的记录，那么切点相当于查询条件，一个切点可以匹配多个连接点。Spring AOP的规则解析引擎负责解析切点所设定的查询条件，找到对应的连接点。
- 增强（Advice）：增强是织入到目标类连接点上的一段程序代码。Spring提供的增强接口都是带方位名的，如：BeforeAdvice、AfterReturningAdvice、ThrowsAdvice等。
- 引介（Introduction）：引介是一种特殊的增强，它为类添加一些属性和方法。这样，即使一个业务类原本没有实现某个接口，通过引介功能，可以动态的未该业务类添加接口的实现逻辑，让业务类成为这个接口的实现类。
- 织入（Weaving）：织入是将增强添加到目标类具体连接点上的过程，AOP有三种织入方式：①编译期织入：需要特殊的Java编译期（例如AspectJ的ajc）；②装载期织入：要求使用特殊的类加载器，在装载类的时候对类进行增强；③运行时织入：在运行时为目标类生成代理实现增强。**Spring采用了动态代理的方式实现了运行时织入，而AspectJ采用了编译期织入和装载期织入的方式**。
-  切面（Aspect）：切面是由切点和增强（引介）组成的，它包括了对横切关注功能的定义，也包括了对连接点的定义。

#### 原理

AOP（Aspect Orient Programming），指面向方面（切面）编程，作为面向对象的一种补充，用于处理系统中分布于各个模块的横切关注点，比如**事务管理、日志、缓存**等等。

AOP实现的关键在于AOP框架自动创建的AOP代理，AOP代理主要分为**静态代理和动态代理**，**静态代理的代表为AspectJ**；而**动态**代理则以**Spring AOP**为代表。通常使用AspectJ的编译时增强实现AOP，AspectJ是静态代理的增强，所谓的静态代理就是AOP框架会在编译阶段生成AOP代理类，因此也称为编译时增强。

Spring AOP中的**动态**代理主要有两种方式，**JDK动态代理和CGLIB动态代理**。JDK动态代理通过**反射**来接收被代理的类，并且要求**被代理的类必须实现一个接口**。JDK动态代理的核心是**InvocationHandler接口和Proxy类**。

如果目标类**没有实现接口，那么Spring AOP会选择使用CGLIB来动态代理目标类**。CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成某个类的子类，注意，**CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的**。

### 注解autowire和resource

1、共同点

两者都可以写在字段和setter方法上。两者如果都写在字段上，那么就不需要再写setter方法。

2、不同点

（1）@Autowired

@Autowired为Spring提供的注解，需要导入包org.springframework.beans.factory.annotation.Autowired;只按照**byType**注入。

@Autowired注解是按照类型（byType）装配依赖对象，默认情况下它**要求依赖对象必须存在，如果允许null值，可以设置它的required属性为false**。如果我们想使用按照名称（byName）来装配，可以结合@Qualifier注解一起使用。

（2）@Resource

@Resource默认按照**ByName**自动注入，由J2EE提供，需要导入包javax.annotation.Resource。@Resource有两个重要的属性：name和type，而Spring将@Resource注解的name属性解析为bean的名字，而type属性则解析为bean的类型。所以，如果使用name属性，则使用byName的自动注入策略，而使用type属性时则使用byType自动注入策略。如果既不制定name也不制定type属性，这时将通过反射机制使用byName自动注入策略。

### BeanFactory和ApplicationContext

**概念**：
BeanFactory是spring中比较原始，比较古老的Factory。因为比较古老，所以BeanFactory无法支持spring插件，例如：AOP、Web应用等功能。

**ApplicationContext是BeanFactory的子类**，因为古老的BeanFactory无法满足不断更新的spring的需求，于是ApplicationContext就基本上代替了BeanFactory的工作，以**一种更面向框架的工作方式以及对上下文进行分层和实现继承，并在这个基础上对功能进行扩展**：
<1>MessageSource, 提供国际化的消息访问
<2>资源访问（如URL和文件）
<3>事件传递
<4>Bean的自动装配
<5>各种不同应用层的Context实现

**区别**：

<1>如果使用ApplicationContext，如果配置的bean是singleton，**那么不管你有没有或想不想用它，它都会被实例化。好处是可以预先加载**，坏处是浪费内存。
<2>BeanFactory，当使用BeanFactory实例化对象时，**配置的bean不会马上被实例化，而是等到你使用该bean的时候（getBean）才会被实例化。好处是节约内存，坏处是速度比较慢**。多用于移动设备的开发。
<3>没有特殊要求的情况下，应该使用ApplicationContext完成。因为BeanFactory能完成的事情，ApplicationContext都能完成，并且提供了更多接近现在开发的功能。



## Spring Boot

http://www.ityouknow.com/springboot/2019/07/24/springboot-interview.html

## Spring MVC

### MVC处理请求的流程，工作原理

   1、  首先用户发送请求——>DispatcherServlet，前端控制器收到请求后自己不进行处理，而是委托给其他的解析器进行处理，作为统一访问点，进行全局的流程控制；

   2、  DispatcherServlet——>HandlerMapping，HandlerMapping 将会把请求映射为 HandlerExecutionChain 对象（包含一个 Handler 处理器（页面控制器）对象、多个 HandlerInterceptor 拦截器）对象，通过这种策略模式，很容易添加新的映射策略；

   3、  DispatcherServlet——>HandlerAdapter，HandlerAdapter 将会把处理器包装为适配器，从而支持多种类型的处理器，即适配器设计模式的应用，从而很容易支持很多类型的处理器；

   4、  HandlerAdapter——>处理器功能处理方法的调用，HandlerAdapter 将会根据适配的结果调用真正的处理器的功能处理方法，完成功能处理；并返回一个 ModelAndView 对象（包含模型数据、逻辑视图名）；

   5、  ModelAndView 的逻辑视图名——> ViewResolver， ViewResolver 将把逻辑视图名解析为具体的 View，通过这种策略模式，很容易更换其他视图技术；

   6、  View——>渲染，View 会根据传进来的 Model 模型数据进行渲染，此处的 Model 实际是一个 Map 数据结构，因此很容易支持其他视图技术；

   7、  返回控制权给 DispatcherServlet，由 DispatcherServlet 返回响应给用户，到此一个流程结束。

8、ModelAndView的视图是逻辑视图，DispatcherServlet还要借助ViewResolver完成从逻辑视图到真实视图对象的解析工作。

9、视图渲染，就是将ModelAndView对象中的数据放到request域中，用来让页面加载数据的。

10、客户端得到响应。

![img](F:\typoraImg\1586802-20190526203737947-944715206.png)

