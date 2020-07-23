[Spring AOP文档](https://docs.spring.io/spring/docs/2.5.x/reference/aop.html#aop-introduction)

# 手写AOP

## AOP是什么

AOP是一种面向切面的思想。

AOP思想的一些名词：

`Aspect`: 跨越多个类的关注的模块化。事务管理是J2EE应用程序中横切关注点的一个很好的例子。在Spring AOP中，方面是使用常规类(基于模式的方法)或使用@Aspect注释的常规类(@AspectJ样式)实现的。



`Join point`: 程序执行过程中的一个点，如方法的执行或异常的处理。在Spring AOP中，连接点总是表示方法执行。



`Advice`: 方面在特定连接点上采取的操作。不同类型的Advice包括**"around," "before" 和"after"** advice.。(通知类型将在下面讨论。)许多AOP框架，包括Spring，将一个Advice建模为一个**拦截器**，维护一个围绕连接点的拦截器链。



`Pointcut`: 匹配连接点的谓词。通知与切入点表达式相关联，并在与切入点匹配的任何连接点上运行(例如，执行具有特定名称的方法)。连接点与切入点表达式匹配的概念是AOP的核心，Spring默认使用AspectJ切入点表达式语言。



`Introducion`: 代表类型声明其他方法或字段。Spring AOP允许您向任何被建议的对象引入新的接口(和相应的实现)。例如，您可以使用一个介绍来让一个bean实现一个IsModified接口，以简化缓存。(介绍在AspectJ社区中称为类型间声明。)



`Target`:被一个或多个方面通知的对象。也称为被通知对象。因为Spring AOP是使用运行时代理实现的，所以这个对象将始终是代理对象。



`AOP proxy`: AOP框架为了实现方面契约(通知方法执行等等)而创建的对象。在Spring框架中，AOP代理将是JDK动态代理或CGLIB代理。



`Weaving`:将方面与其他应用程序类型或对象链接，以创建建议的对象。这可以在编译时(例如，使用AspectJ编译器)、加载时或运行时完成。与其他纯Java AOP框架一样，Spring AOP在运行时执行编织。



### 关于AspectJ

spring支持AspectJ的语法，和@AspectJ注解，AOP运行时仍然是纯Spring AOP，并且不依赖于AspectJ编译器或编织器。

```xml
<aop:aspectj-autoproxy/>
```

## 配置

AOP配置常用有**Java注解**配置和**xml**配置。

## 相关示例

### Pointcut示例

```java
// 定义一个切面，里面有多个切点
@Aspect
public class SystemArchitecture {

  /**
   * A join point is in the web layer if the method is defined
   * in a type in the com.xyz.someapp.web package or any sub-package
   * under that.
   */
  @Pointcut("within(com.xyz.someapp.web..*)")
  public void inWebLayer() {}

  /**
   * A join point is in the service layer if the method is defined
   * in a type in the com.xyz.someapp.service package or any sub-package
   * under that.
   */
  @Pointcut("within(com.xyz.someapp.service..*)")
  public void inServiceLayer() {}

  /**
   * A join point is in the data access layer if the method is defined
   * in a type in the com.xyz.someapp.dao package or any sub-package
   * under that.
   */
  @Pointcut("within(com.xyz.someapp.dao..*)")
  public void inDataAccessLayer() {}

  /**
   * A business service is the execution of any method defined on a service
   * interface. This definition assumes that interfaces are placed in the
   * "service" package, and that implementation types are in sub-packages.
   * 
   * If you group service interfaces by functional area (for example, 
   * in packages com.xyz.someapp.abc.service and com.xyz.def.service) then
   * the pointcut expression "execution(* com.xyz.someapp..service.*.*(..))"
   * could be used instead.
   *
   * Alternatively, you can write the expression using the 'bean'
   * PCD, like so "bean(*Service)". (This assumes that you have
   * named your Spring service beans in a consistent fashion.)
   */
  @Pointcut("execution(* com.xyz.someapp.service.*.*(..))")
  public void businessService() {}
  
  /**
   * A data access operation is the execution of any method defined on a 
   * dao interface. This definition assumes that interfaces are placed in the
   * "dao" package, and that implementation types are in sub-packages.
   */
  @Pointcut("execution(* com.xyz.someapp.dao.*.*(..))")
  public void dataAccessOperation() {}

}
```

### Advice示例

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class BeforeExample {
  //在dataAccessOperation方法执行之前执行doAccessCheck方法
  @Before("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
  public void doAccessCheck() {
    // ...
  }

}
```

## 步骤

