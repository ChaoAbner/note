## BigDecimal的使用

在生产过程中我们进行一些精度计算（比如货币），不能通过Float和Double来进行。

而是需要通过BigDecimal来进行这些精度计算。



### 构建BigDecimal

```java
BigDecimal BigDecimal(double d); //不允许使用,精度不能保证
BigDecimal BigDecimal(String s); //常用,推荐使用
static BigDecimal valueOf(double d); //常用,推荐使用
```



### 方法

|         方法         | 描叙                                       |
| :------------------: | ------------------------------------------ |
|   add(BigDecimal)    | BigDecimal对象中的值相加，然后返回这个对象 |
| subtract(BigDecimal) | BigDecimal对象中的值相减，然后返回这个对象 |
| multiply(BigDecimal) | BigDecimal对象中的值相乘，然后返回这个对象 |
|  divide(BigDecimal)  | BigDecimal对象中的值相除，然后返回这个对象 |
|      toString()      | 将BigDecimal对象的数值转换成字符串         |
|    doubleValue()     | 将BigDecimal对象中的值以双精度数返回       |
|     floatValue()     | 将BigDecimal对象中的值以单精度数返回       |
|     longValue()      | 将BigDecimal对象中的值以长整数返回         |
|      intValue()      | 将BigDecimal对象中的值以整数返回。         |