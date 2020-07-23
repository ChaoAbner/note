![1570351959498](F:\typoraImg\1570351959498.png)



![1570353529864](F:\typoraImg\1570353529864.png)

### springboot使用Filter：

有两种方式：

1. 使用spring boot 提供的FilterRegistrationBean注册Filter
2. 使用原生servlet注解定义Filter

两种方式的本质都一样，都是去FilterRegistrationBean注册Filter，详情见连接：

https://blog.csdn.net/heweimingming/article/details/79993591



### springboot使用拦截器

https://blog.csdn.net/heweimingming/article/details/79993591



### 怎么合适使用

- 如果是非spring项目，不能使用拦截器，只能使用过滤器。
- 如果是处理controller前后，既可以使用拦截器也可以使用过滤器。
- 如果是处理dispaterServlet前后，只能使用过滤器。



### 理解

**拦截器** ：是在面向切面编程的就是在你的service或者一个方法，前调用一个方法，或者在方法后调用一个方法比如动态代理就是拦截器的简单实现，在你调用方法前打印出字符串（或者做其它业务逻辑的操作），也可以在你调用方法后打印出字符串，甚至在你抛出异常的时候做业务逻辑的操作。

**过滤器**：是在javaweb中，你传入的request、response提前过滤掉一些信息，或者提前设置一些参数，然后再传入servlet或者struts的action进行业务逻辑，比如过滤掉非法url（不是login.do的地址请求，如果用户没有登陆都过滤掉），或者在传入servlet或者 struts的action前统一设置字符集，或者去除掉一些非法字符.。



### 常见用途

1、对请求进行验证，比如API的使用需要验证用户身份等。

2、日志记录：记录请求信息的日志，以便进行信息监控、信息统计、计算PV（Page View）等。

3、权限检查：如登录检测，进入处理器检测检测是否登录，如果没有直接返回到登录页面；

4、性能监控：有时候系统在某段时间莫名其妙的慢，可以通过拦截器在进入处理器之前记录开始时间，在处理完后记录结束时间，从而得到该请求的处理时间（如果有反向代理，如apache可以自动记录）；

5、通用行为：读取cookie得到用户信息并将用户对象放入请求，从而方便后续流程使用，还有如提取Locale、Theme信息等，只要是多个处理器都需要的即可使用拦截器实现。

6、OpenSessionInView：如[hibernate](http://lib.csdn.net/base/javaee)，在进入处理器打开Session，在完成后关闭Session。