# hashCode()和equals()

## 规范

![image-20200705103205526](http://img.fosuchao.com/image-20200705103205526.png)

### 反例

```java
// 具有随机性
class H {
	@Override
	public int hashCode() {
		return Math.random();
	}
}
```

## 为什么要有规范

定义这些规范的原因涉及到hash储存，例如Java中的HashMap和HashSet等等。

这里我们举HashMap为例

![image-20200705103511552](http://img.fosuchao.com/image-20200705103511552.png)

- 我们查找数据的时候，一般是根据hashcode()方法的值进行运算，求出相应存储的桶的位置。
- 然后再在链表中通过equals()方法来对比是否是相同的对象，相同说明找到。

### 结论

hashcode相同，equals不一定返回true

equals返回true，两个对象的hashcode一定相同

## 扩展

![image-20200705105108992](http://img.fosuchao.com/image-20200705105108992.png)