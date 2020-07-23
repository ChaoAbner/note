# Netty源码本地报错问题

netty下载源码，想运行案例的时候发现报错了。

原因是common下的util缺少了collection包。

![](http://img.fosuchao.com/20200307185740.png)

解决方法如下：

1、在根`pom`中添加<skip>true</skip>

```xml
<project>
  ...
  <build>
    ...
    <plugins>
      ...
      <plugin>
        <artifactId>maven-checkstyle-plugin</artifactId>
        <configuration>
            <! -- 添加 -->
          <skip>true</skip>	
        </configuration>
      </plugin>
      ...
    </plugins>
    ...
  </build>
  ...
</project>
```

2、然后在重新编译common的maven文件

```maven
mvn compile -Dcheckstyle.skip=true
```

![](http://img.fosuchao.com/20200307190110.png)

提示编译成功即可。

可以发现在编译后生成的target下多了一个collection包。

![](http://img.fosuchao.com/20200307190216.png)