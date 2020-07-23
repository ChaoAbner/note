## Swagger介绍

Swagger是当前最好用的Restful API文档生成的开源项目，通过swagger-spring项目

实现了与SpingMVC框架的无缝集成功能，方便生成spring restful风格的接口文档，

同时swagger-ui还可以测试spring restful风格的接口功能。



官方网站为：http://swagger.io/

中文网站：[http://www.sosoapi.com](http://www.sosoapi.com/)



## maven配置

```xml

<dependency>
    <groupId>com.mangofactory</groupId>
    <artifactId>swagger-springmvc</artifactId>
    <version>1.0.2</version>
</dependency>
 
 <dependency>
     <groupId>org.springframework</groupId>
     <artifactId>spring-webmvc</artifactId>
     <version>4.1.6.RELEASE</version>
</dependency>

<!-- 或者 --->
<!-- 建议使用 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.8.0</version>
</dependency>


<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.8.0</version>
</dependency>
```



## 常用注解


最常用的5个注解
@Api：修饰整个类，描述Controller的作用

@ApiOperation：描述一个类的一个方法，或者说一个接口

@ApiParam：单个参数描述

@ApiModel：用对象来接收参数

@ApiProperty：用对象接收参数时，描述对象的一个字段


@ApiResponse：HTTP响应其中1个描述

@ApiResponses：HTTP响应整体描述

@ApiClass

@ApiError

@ApiErrors

@ApiParamImplicit

@ApiParamsImplicit



## 启动项目

默认访问http://localhost:8080/swagger-ui.html  端口号为项目启动端口