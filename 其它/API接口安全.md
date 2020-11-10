# 背景

我们的APP、前后端分离项目基本都是采用API接口形式与服务器进行数据通信，传输的数据被偷窥、被抓包、被伪造时有发生。

**在未进行安全处理的开放 API 接口存在诸多的风险问题，如以下三种常见场景：**

1、场景一
A 公司开发的开放 API 未对接口进行安全控制，有黑客通过爬虫程序调用开放 API 查询客户信息的接口，由于没有安全验证，A 公司的客户数据通过客户信息查询 API 流出，竞争公司拥有了这批客户数据，对 A 公司造成损失。

2、场景二
A 公司开发的开放 API 未对接口进行防篡改控制，有客户购买价值 1 万元的产品，现有黑客通过技术手段，对客户提交的表单进行篡改，将 1 万元的价格改成 100 元，对公司造成了经济损失。

3、场景三
A 公司开发的开放 API 未进行接口安全控制，有黑客截取了客户的用户名，密码信息。黑客用客户的用户名，密码登录，查阅客户的相关隐私信息，盗刷了客户的信用卡，给客户造成了损失。

这三种常见未进行开放 API 接口校验的场景对公司以及客户造成损失的都是不可修补的，也反映了普遍开放式 API 接口存在的安全问题。那么，在编写开放 API 接口时如何保障数据传输安全性？

**开放API接口安全性需要考虑的事项总结如下：**
1、请求来源(身份)是否合法？
2、请求参数是否被篡改？
3、请求的唯一性(不可复制、DDOS)？



# 解决思路

## 问题1：请求来源是否合法

解决方案：

1、Token授权机制

2、appKey，appSecret，accessToken校验机制



### Token授权机制

Token授权机制是我们平时最常接触的判断用户的方式，最常用于前后端分离的应用。

HTTP协议是无状态的，一次请求结束，连接就会断开，下次服务器再次接受到请求，即使是同一个用户发送的，服务器也无法判断是哪个用户发送的请求。

对于我们的系统，是需要状态管理的，以便于服务器能够准确的知道HTTP请求是哪个用户发起的，从而判断他是否有权限继续这个请求，即访问我们的API接口。

#### 设计方案

Token的设计方案通常是用户在客户端使用用户名和密码登录系统后，服务器会给客户端返回一个Token，并将这个Token以键值对的形式存储（通常存储在Redis），后续客户端对需要授权的模块的所有操作都要带上这个Token，服务器接收到请求后会对请求携带的Token进行校验，如果Token合法，说明是授权的请求。

主要流程如下：

![image-20201110095105934](http://img.fosuchao.com/image-20201110095105934.png)

#### Token设计要求

1、应用内一定要唯一，否则会出现授权混乱，A用户看到了B用户的数据；

2、每次生成的Token一定要不一样，防止被记录，授权永久有效；

3、一般Token对应的是Redis的key，value存放的是这个用户相关缓存信息，比如：用户的id（user_id）；

4、要设置Token的过期时间，过期后需要客户端重新登录，获取新的Token，如果Token有效期设置较短，会反复需要用户登录，体验比较差，我们一般采用Token过期后，客户端静默登录的方式，当客户端收到Token过期后，客户端用本地保存的用户名和密码在后台静默登录来获取新的Token，还有一种是单独出一个刷新Token的接口，但是一定要注意刷新机制和安全问题；

#### 安全隐患

如果用户的Token被劫持后，可以用来伪造请求和篡改参数

解决方法：在服务端配置IP白名单，确保这个Token来自同一个IP。



### appKey，appSecret，accessToken校验机制

如果我们平时有用过微信开放平台或者其他的开放平台，我们就会发现这些开放平台基本都是使用这种身份校验机制，这是用来对openApi访问的一种授权机制。

我们如果想调用发布方的API接口的话，通常需要我们创建一个调用方的应用，这个应用会有一个appKey和appSecret，appKey是公开的，就是调用方的身份认证标识，appSecret是密钥，属于私密的，后续会用于加签。获取到了这两个信息后，真正要调用接口一般还需要两步：

1、使用appKey等参数按照开放平台的要求请求获取accessToken的接口，获取的accessToken一般有过期时间。

2、拿到accessToken后，携带accessToken去访问具体的openApi，然后平台会通过这个accessToken识别到调用方的信息，去判断授权情况等。





## 问题2：请求参数是否被篡改

解决方案：

1、使用签名的方式（sign）

### 使用签名的方式（sign）

将请求中的所有url参数按照字典升序排序，拼接生成字符串（即key1=value1&key2=value2...），再加上secretKey等到最终的字符串，最后使用加密算法（比如MD5，SHA1）生成一个签名（sign），然后在请求中添加这个请求参数，服务端获取了请求参数后，再对这些请求参数进行同样的操作并生成sign，再与客户端传过来的sign进行对比，如果不相等，则说明参数被篡改。

#### 方案分析

从上面的具体流程可以看出，最关键的在于secretKey，这个密钥是不参与通信的，是只有调用方和发布方才知道的，通过约定的加密算法加密后，发布方可以根据对应的调用方查询出相应的secretKey，然后加密后对比sign即可。一般来说只要secretKey不会泄漏，那么这个方法就是安全的。



## 问题3：请求的唯一性（防重放）

解决方案：

1、时间戳超时机制

2、请求严格校验

2、接口限流



### 时间戳超时机制

用户每次请求都带上当前时间的时间戳timestamp，服务端接收到timestamp后跟当前时间进行比对，如果时间差大于一定时间（比如30秒），则认为该请求失效。时间戳超时机制是防御重复调用和爬取数据的有效手段。

这里需要注意的地方是保证客户端和服务端的“当前时间”是一致的，我们采取的方式是客户端第一次连接服务端时请求一个接口获取服务端的当前时间A1，再和客户端的当前时间B1做一个差异化计算（A1-B1=AB），得出差异值AB，客户端再后面的请求中都是传B1+AB给到服务端。

#### 简单实现

```java
// timeStamp是客户端从Header传过来的值
Long timeStamp = RequestHeaderContext.getInstance().getTimeStamp();
boolean checkTime = checkTime(timeStamp, 30 * 1000);
if (!checkTime) {
    return responseErrorAPISecurity(response);
}

// checkTime方法
public static boolean checkTime(Long time, Integer variable){
    Long currentTimeMillis = System.currentTimeMillis();
    Long addTime = currentTimeMillis + variable;
    Long subTime = currentTimeMillis - variable;
    if (addTime > time && time > subTime){
        return true;
    }
    return false;
}
```

#### 分析

 如果有人使用已经劫持的URL进行DOS攻击和爬取数据，那么他也只能最多使用30s； 



### 请求严格校验

使用一个nonce字段，nonce表示一个随机唯一字符串，每次的接口请求对应一个nonce字段。

请求之后，服务端生成一个nonce，保存在Redis中，当后面的请求访问的时候，判断能否在Redis中找到对应的nonce，如果有则阻止请求，没有则放行请求。

这种方法就是通过记录所有的请求来防止接口重放，然而让服务器去保存所有请求的nonce，无疑对服务器会造成很大的压力。



### 接口限流

可以对某个调用方进行限流，比如1分钟只能调用10次，超出的请求都会被阻止。直到下分钟次数才会被重置。

具体的实现方式参考链接：

[限流算法实现](https://www.jianshu.com/p/1dbc1a19fa8c)

[接口限流算法：漏桶算法&令牌桶算法](https://www.ymq.io/2018/08/11/RateLimiter/)



## 其他相关

使用OAuth2.0授权方式

具体流程如下：

![0001](http://img.fosuchao.com/0001.jpg)

详情参考

[OAuth2.0认证和授权机制讲解](https://zhuanlan.zhihu.com/p/20913727)

[腾讯开放平台](https://wiki.open.qq.com/wiki/mobile/OAuth2.0%E7%AE%80%E4%BB%8B)

[维基百科-开放授权](https://zh.wikipedia.org/zh/%E5%BC%80%E6%94%BE%E6%8E%88%E6%9D%83)



# 总结

API的安全对于我们的应用来说非常重要，如果不对API做必要的安全策略，那么我们的数据很可能被人利用，对于用户和公司来说都是极大的损失。



参考资料

[API安全接口安全设计]([https://maizitoday.github.io/post/api%E5%AE%89%E5%85%A8%E6%8E%A5%E5%8F%A3%E5%AE%89%E5%85%A8%E8%AE%BE%E8%AE%A1/](https://maizitoday.github.io/post/api安全接口安全设计/))

[API接口之安全篇](https://www.cnblogs.com/xingxia/p/API_secrute.html)
[安全|API接口安全性设计（防篡改和重复调用）](https://cloud.tencent.com/developer/article/1369607)

[网关、开放平台如何设计appKey,appSecret，accessToken的生成和校验机制](https://blog.csdn.net/ywg_1994/article/details/103518806)

