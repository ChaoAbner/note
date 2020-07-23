## 添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.0.29</version>
</dependency>
```

## application配置

```yaml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource	#使用阿里的数据源
    url: jdbc:mysql://127.0.0.1:3306/campus？useUnicode=true&characterEncoding=UTF-8&useSSL=false
    username: root
    password: 123456
```

## 常见错误

#### 1、You must configure either the server or JDBC driver (via the serverTimezone conf）

解决方法： 在url后面添加`serverTimezone=UTC`

