# RPC框架其他重要模块分析

## 动态代理屏蔽网络传输细节

```java
@Slf4j
public class RpcClientProxy implements InvocationHandler {
    public <T> T getProxy(Class<T> clazz) {
        return (T) Proxy.newProxyInstance(clazz.getClassLoader(), new Class<?>[]{clazz}, this);
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) {
    }
}
```

当我们去调用一个远程方法的时候，实际上是通过代理对象去调用的

获取代理对象的方法如下：

```java
    public <T> T getProxy(Class<T> clazz) {
        return (T) Proxy.newProxyInstance(clazz.getClassLoader(), new Class<?>[]{clazz}, this);
    }
```

网络传输的细节写在了`invoke`方法中

```java
public Object invoke(Object proxy, Method method, Object[] args) {
    log.info("invoked method: [{}]", method.getName());
    RpcRequest rpcRequest = RpcRequest.builder().methodName(method.getName())
        .parameters(args)
        .interfaceName(method.getDeclaringClass().getName())
        .paramTypes(method.getParameterTypes())
        .requestId(UUID.randomUUID().toString())
        .group(rpcServiceProperties.getGroup())
        .version(rpcServiceProperties.getVersion())
        .build();
    RpcResponse<Object> rpcResponse = null;
    if (rpcRequestTransport instanceof NettyRpcClient) {
        CompletableFuture<RpcResponse<Object>> completableFuture = (CompletableFuture<RpcResponse<Object>>) rpcRequestTransport.sendRpcRequest(rpcRequest);
        rpcResponse = completableFuture.get();
    }
    if (rpcRequestTransport instanceof SocketRpcClient) {
        rpcResponse = (RpcResponse<Object>) rpcRequestTransport.sendRpcRequest(rpcRequest);
    }
    this.check(rpcResponse, rpcRequest);
    return rpcResponse.getData();
}
```

## 通过注解注册/消费服务

fosu-rpc接用了Spring容器相关的功能，实现了通过注解注入先注入相关服务。

相关实现位于：`rpc-core`模块下的`src/main/java/com/fosuchao/spring`包下面

我们定义了两个注解：

- `RpcService`：注册Rpc服务
- `RpcReference`：消费服务（调用Rpc服务）

### RpcService

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Inherited
public @interface RpcService {

    /**
     * Service version, default value is empty string
     */
    String version() default "";

    /**
     * Service group, default value is empty string
     */
    String group() default "";

}
```

### RpcReference

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD})
@Inherited
public @interface RpcReference {

    /**
     * Service version, default value is empty string
     */
    String version() default "";

    /**
     * Service group, default value is empty string
     */
    String group() default "";

}
```

### 实现原理

我们需要实现`BeanPostProcessor`接口并重写`postProcessBeforeInitialization()`方法和`postProcessAfterInitalization()`方法。

Spring bean在实例化之前会调用`postProcessBeforeInitialization()`方法，在实例化之后会调用`postProcessAfterInitalization()`方法。

```java
public class SpringBeanPostProcessor implements BeanPostProcessor {

    private final ServiceProvider serviceProvider;
    private final RpcRequestTransport rpcClient;

    public SpringBeanPostProcessor() {
        this.serviceProvider = SingletonFactory.getInstance(ServiceProviderImpl.class);
        this.rpcClient = ExtensionLoader.getExtensionLoader(RpcRequestTransport.class).getExtension("netty");
    }

    @Override
    @SneakyThrows
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if (bean.getClass().isAnnotationPresent(RpcService.class)) {
            log.info("[{}] is annotated with [{}]", bean.getClass().getName(), RpcService.class.getCanonicalName());
            RpcService rpcService = bean.getClass().getAnnotation(RpcService.class);
            // build RpcServiceProperties
            RpcServiceProperties rpcServiceProperties =
                    RpcServiceProperties.builder().version(rpcService.version()).group(rpcService.group()).build();
            serviceProvider.publishService(bean, rpcServiceProperties);
        }
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        Class<?> targetClass = bean.getClass();
        Field[] declaredFields = targetClass.getDeclaredFields();
        for (Field declaredField : declaredFields) {
            RpcReference rpcReference = declaredField.getAnnotation(RpcReference.class);
            if (rpcReference != null) {
                RpcServiceProperties rpcServiceProperties =
                        RpcServiceProperties.builder().version(rpcReference.version()).group(rpcReference.group()).build();
                RpcClientProxy rpcClientProxy = new RpcClientProxy(rpcClient, rpcServiceProperties);
                Object proxy = rpcClientProxy.getProxy(declaredField.getType());
                declaredField.setAccessible(true);
                try {
                    declaredField.set(bean, proxy);
                } catch (IllegalAccessException e) {
                    e.printStackTrace();
                }
            }
        }
        return bean;
    }
}

```

我们规定使用了`RpcService`和`RpcReference`注解的都算是Srping Bean。

- 所以我们可以在`postProcessBeforeInitialization`方法中判断类上是否有`RpcService`注解，如果有的话就取去`group`和`version`的值在调用`ServiceProvider`的`publishService`方法去发布服务即可
- 我们可以在`postProcessAfterInitialization`方法中遍历类的属性（这里是成员变量），判断是否带有`RpcReference`注解，如果有那么就通过反射将设个属性赋值即可。