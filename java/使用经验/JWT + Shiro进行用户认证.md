# Shiro + JWT进行用户登录验证



## 特性

- 完全使用了 Shiro 的注解配置，保持高度的灵活性。
- 放弃 Cookie ，Session ，使用JWT进行鉴权，完全实现无状态鉴权。
- JWT 密钥支持过期时间。
- 对跨域提供支持。



## Maven依赖

```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.3.2</version>
</dependency>
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>3.2.0</version>
</dependency>
```



## 验证方法

- 客户端进行用户登录后，若登录成功，则通过`JWT`颁发`token`
- 之后客户端每次请求服务器都必须头部带上token信息
- `token`通过JWT解析验证后通过Shiro进行身份认证



## Shiro

### Shiro组成

​	`shiro`一般有config，realm两个主要部分组成

- config作为shiro的配置类
- realm作为shiro进行身份认证和授权的类

### Shiro内置过滤器，可以实现权限相关的拦截器

常用的过滤器：

- `anon`: 无需认证（登录）可以访问
- `authc`: 必须认证才可以访问
- `user`: 如果使用rememberMe的功能可以直接访问
- `perms`： 该资源必须得到资源权限才可以访问
- `role`: 该资源必须得到角色权限才可以访问

### Shiro基本操作

#### 获取当前用户

```java
public static User getCurrentUser() {
	return (User) SecurityUtils.getSubject().getPrincipal();
}
```

#### Controller中设置注解要求请求认证

`role:update`为请求的URI

设置了`RequiresPermissions`注解，则该请求会通过Shiro的身份认证成功后才会进行后续操作。

```java
@PostMapping("update")
@RequiresPermissions("role:update")
@ControllerEndpoint(operation = "修改角色", exceptionMessage = "修改角色失败")
public FebsResponse updateRole(Role role) {
    this.roleService.updateRole(role);
    return new FebsResponse().success();
}
```

#### ShiroFilterFactoryBean设置

在Shrio的配置文件中进行设置

- 设置免认证url(过滤器)

```java
// 创建一个shiroFilterFactoryBean对象
ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();

// 声明一个Map
LinkedHashMap<String, String> filterChainDefinitionMap = new LinkedHashMap<>();

// 添加免认证的url
filterChainDefinitionMap.put(url, "anon");

// 配置退出过滤器，Shiro内部已经实现了
filterChainDefinitionMap.put(febsProperties.getShiro().getLogoutUrl(), "logout");

// 除了上面的以外，其它url都需要认证,未认证则自动访问LoginURL
filterChainDefinitionMap.put("/**", "user");

// 将Map添加到ShiroFilterFactoryBean
shiroFilterFactoryBean.setFilter(filterChainDefinitionMap);
```

### Shiro配置类ShiroConfig

```java
@Configuration
public class ShiroConfig{
	    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();

        // 设置 securityManager
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        // 登录的 url
        shiroFilterFactoryBean.setLoginUrl(febsProperties.getShiro().getLoginUrl());
        // 登录成功后跳转的 url
        shiroFilterFactoryBean.setSuccessUrl(febsProperties.getShiro().getSuccessUrl());
        // 未授权 url
        shiroFilterFactoryBean.setUnauthorizedUrl(febsProperties.getShiro().getUnauthorizedUrl());

        LinkedHashMap<String, String> filterChainDefinitionMap = new LinkedHashMap<>();
        // 设置免认证 url
        String[] anonUrls = StringUtils.splitByWholeSeparatorPreserveAllTokens(febsProperties.getShiro().getAnonUrl(), ",");
        for (String url : anonUrls) {
            filterChainDefinitionMap.put(url, "anon");
        }
        // 配置退出过滤器，其中具体的退出代码 Shiro已经替我们实现了
        filterChainDefinitionMap.put(febsProperties.getShiro().getLogoutUrl(), "logout");

        // 除上以外所有 url都必须认证通过才可以访问，未通过认证自动访问 LoginUrl
        filterChainDefinitionMap.put("/**", "user");

        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        return shiroFilterFactoryBean;
    }

    @Bean
    public SecurityManager securityManager(ShiroRealm shiroRealm) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        // 配置 SecurityManager，并注入 shiroRealm
        securityManager.setRealm(shiroRealm);
        // 配置 shiro session管理器
        securityManager.setSessionManager(sessionManager());
        // 配置 缓存管理类 cacheManager
        securityManager.setCacheManager(cacheManager());
        // 配置 rememberMeCookie
        securityManager.setRememberMeManager(rememberMeManager());
        return securityManager;
    }
}
```



### shiro的realm类ShiroRealm

```java
@Conponent
public class ShrioRealm extends AuthorizingRealm{
	/**
     * 授权模块，获取用户角色和权限
     *
     * @param principal principal
     * @return AuthorizationInfo 权限信息
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principal) {
        User user = (User) SecurityUtils.getSubject().getPrincipal();
        String userName = user.getUsername();

        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();

        // 获取用户角色集
        List<Role> roleList = this.roleService.findUserRole(userName);
        Set<String> roleSet = roleList.stream().map(Role::getRoleName).collect(Collectors.toSet());
        simpleAuthorizationInfo.setRoles(roleSet);

        // 获取用户权限集
        List<Menu> permissionList = this.menuService.findUserPermissions(userName);
        Set<String> permissionSet = permissionList.stream().map(Menu::getPerms).collect(Collectors.toSet());
        simpleAuthorizationInfo.setStringPermissions(permissionSet);
        return simpleAuthorizationInfo;
    }

    /**
     * 用户认证
     *
     * @param token AuthenticationToken 身份认证 token
     * @return AuthenticationInfo 身份认证信息
     * @throws AuthenticationException 认证相关异常
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        // 获取用户输入的用户名和密码
        String userName = (String) token.getPrincipal();
        String password = new String((char[]) token.getCredentials());

        // 通过用户名到数据库查询用户信息
        User user = this.userService.findByName(userName);

        if (user == null)
            throw new UnknownAccountException("账号未注册！");
        if (!StringUtils.equals(password, user.getPassword()))
            throw new IncorrectCredentialsException("用户名或密码错误！");
        if (User.STATUS_LOCK.equals(user.getStatus()))
            throw new LockedAccountException("账号已被锁定,请联系管理员！");
        return new SimpleAuthenticationInfo(user, password, getName());
    }
}
```





## JWT

### JWT工具类

```java
public class JwtUtil {

    private static final long EXPIRE_TIME = 5 * 60 * 1000;  // 5分钟过期

    /**
     * 校验token是否正确
     *
     * @return boolean
     * @Param [token, username, password]
     */
    public static boolean verify(String token, String username, String password) {
        try {
            Algorithm algorithm = Algorithm.HMAC256(password);
            JWTVerifier jwtVerifier = JWT.require(algorithm)
                    .withClaim("username", username)
                    .build();

            DecodedJWT jwt = jwtVerifier.verify(token);
            return true;

        } catch (UnsupportedEncodingException e) {
            return false;
        }
    }


    /**
     * 根据token返回username
     *
     * @return java.lang.String
     * @Param [token]
     */
    public static String getUsername(String token) {
        try {
            DecodedJWT jwt = JWT.decode(token);
            return jwt.getClaim("username").asString();
        } catch (JWTDecodeException e) {
            e.printStackTrace();
            return null;
        }
    }

    public static String createSign(String username, String password) {
        Date date = new Date(System.currentTimeMillis() + EXPIRE_TIME);
        try {
            Algorithm algorithm = Algorithm.HMAC256(password);
            // 附带username信息
            return JWT.create()
                    .withClaim("username", username)
                    .withExpiresAt(date)
                    .sign(algorithm);
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
            return null;
        }
    }
}
```

