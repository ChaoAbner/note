`@RequestParam` 和 `@PathVariable` 注解是用于从`request`中接收请求的，两个都可以接收参数，关键点不同的是`@RequestParam` 是从`request`里面拿取值，而 `@PathVariable` 是从一个`URI模板`里面来填充

`@RequestParam`
看下面一段代码：

`http://localhost:8080/springmvc/hello/101?param1=10&param2=20`

根据上面的这个URL，你可以用这样的方式来进行获取



```java
public String getDetails(
    @RequestParam(value="param1", required=true) String param1,
        @RequestParam(value="param2", required=false) String param2){
...
}
```

`@RequestParam` 支持下面四种参数

defaultValue 如果本次请求没有携带这个参数，或者参数为空，那么就会启用默认值
name 绑定本次参数的名称，要跟URL上面的一样
required 这个参数是不是必须的
value 跟name一样的作用，是name属性的一个别名
`@PathVariable`
这个注解能够识别URL里面的一个模板，我们看下面的一个URL

`http://localhost:8080/springmvc/hello/101?param1=10&param2=20`

上面的一个url你可以这样写：



```java
@RequestMapping("/hello/{id}")
    public String getDetails(@PathVariable(value="id") String id,
    @RequestParam(value="param1", required=true) String param1,
    @RequestParam(value="param2", required=false) String param2){
.......
}
```


区别很明显了

@PathParam
这个注解是和spring的pathVariable是一样的，也是基于模板的，但是这个是jboss包下面的一个实现，上面的是spring的一个实现，都要导包

@QueryParam
@QueryParam 是 JAX-RS 本来就提供的，和Spring的RequestParam作用一致

@ResponseBody
responseBody表示服务器返回的时候以一种什么样的方式进行返回， 将内容或对象作为 HTTP 响应正文返回，值有很多，一般设定为json

@RequestBody
一般是post请求的时候才会使用这个请求，把参数丢在requestbody里面
