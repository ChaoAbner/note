# Java中String的常用操作

## String和StringBuffer的概念

​		String类是字符串常量，是不可更改的常量。而StringBuffer是字符串变量，它的对象是可以扩充和修改的。StringBuffer在进行字符串处理时，不生成新的对象，在内存使用上要优于String类。所以在实际使用时，如果经常需要对一个字符串进行修改，例如插入、删除等操作，使用StringBuffer要更加适合一些。



## String常用操作

### 1、查找字符串中某个位置的字符

`String.charAt(int index)`

```java
String str = "joker-Ye";
// 查找某个字符串某个索引的字符
System.out.println(str.charAt(3));
// 输出e
```

### 2、查找指定字符在字符串中出现的位置

- 从字符串开头开始检索
  - `public int indexOf(String str)`  
- 从字符串开头第fromIndex开始检索
  - ` public int indexOf(String str, int fromIndex)`

- 从字符串末尾开始检索
  - `public int lastIndexOf(String str)`  
- 从字符串末尾第fromIndex开始检索
  - `public int lastIndexOf(String str, int fromIndex)`

- 测试字符串中是否存在某个字符
  - `public boolean contains(CharSequence s)`

**`CharSequence` 为接口，`StringBuffer`、`String`等类都实现了该接口。**

### 3、比较字符串

```java
(1)boolean equals()           
    //判断内容是否相同
(2)int compareTo()        
    //根据两字符串的首字母的asc码差值来判断字符串的大小关系（若相同则比较下一个字母）
(3)int compareToIgnoreCase(String int) 
    //在比较时忽略字母大小写
(4)==                 
    //判断内容与地址是否相同
(5)boolean equalsIgnoreCase() 
    //忽略大小写的情况下判断内容是否相同

//如果想对字符串中的部分内容是否相同进行比较，可以用
(6)reagionMatches()   
    //有两种 public boolean regionMatches(int toffset, String other,int ooffset,int len);表示如果String对象的一个子字符串与参数other的一个子字符串是相同的字符序列，则为true.要比较的String 对象的字符串从索引toffset开始,other的字符串从索引ooffset开始，长度为len。

public boolean reagionMatches(boolean ignoreCase,int toffset,String other,int ooffset,int len);
//用布尔类型的参数指明两个字符串的比较是否对大小写敏感。
```

### 4、检查字符串的起始或者结尾字符

- 字符串开始的两种方法

  - ```java
    public boolean starWith(String prefix,int toffset);
    //如果参数prefix表示的字符串序列是该对象从索引toffset处开始的子字符串，则返回true
    public boolean starWith(String prefix);
    ```

- 字符串结尾的方法

  - ```java
    public boolean endsWith(String suffix);
    ```

    

### 5、截取字符串

```java
public String subString(int beginIndex);
public String subString(int beginIndex, int endIndex);
//返回的字符串是从beginIndex开始到endIndex-1的串

String str = "joker-Ye";
// 根据索引截取字符串
System.out.println(str.substring(1));
// 输出oker-Ye
System.out.println(str.substring(1, 5));
// 输出oker
```



### 6、字符串的替换

```java
public String replace(char oldChar,char newChar);
public String replace(CharSequence target,CharSequence replacement);
//把原来的target子序列替换为replacement序列，返回新串
public String replaceAll(String regex,String replacement);          
//用正则表达式实现对字符串的匹配
```



### 7、取出字符串首尾空格

```java
String trim(String str)
```



### 8、字符串转换

- 将字符串转换成字符数组

```java
public char[] toCharArray();

String str = "joker-Ye"
char [] strList = str.toCharArray();
for (char c : strList) {
	System.out.println(c);
}
// 输出j, o, k, e, r, -, Y, e

```

- 将字符串转换成字符串数组

```java
public String[] split(String regex);//regex 是给定的分隔符匹配

String str = "joker-Ye"
String [] strList = str.split("-");
for (char c : strList) {
	System.out.println(c);
}
// 输出 joker, Ye
```

- 将其它数据类型转换成字符串

```java
public static String valueOf(boolean b);
public static String valueOf(char c);
public static String valueOf(int i);
public static String valueOf(long i);
public static String valueOf(float f);
public static String valueOf(double d);
public static String valueOf(char[] data);
public static String valueOf(Object obj);
```

## **StringBuffer类主要方法的使用**

### **一、可变字符串的创建和初始化**

两种方法:

```java
public StringBuffer();
public StringBuffer(int caoacity);
```

### **二、获取可变字符串长度**

```java
public int length();
public int capacity();
public void setLength(int newLength);
```

### **三、比较可变字符串**

用String 类的equals()方法比较，但是不同。
类Object中的equals()方法比较的是两个对象的地址是否相等，而不仅仅是比较内容，但是

- String类在继承Object类的时候重写了equals()方法，只是比较两个对象的内容是否相等
- StringBuffer类中没有重写Object类的equals()方法，所以比较的是地址和内容。

### **四、追加和插入字符串**

```java
public StringBuffer append(type t);//追加
public StringBuffer insert(int offset,type t);//插入，在offset处加入类型为type的字符串
```

### **五、反转和删除字符串**

```java
 public StringBuffer reverse(); //反转
 public StringBuffer delete(int start,int end); //删除
```

### **六、减少用于可变字符序列的存储空间**

```java
public void trimToSize();
```

### **七、StringBuffer类转换成String类**

```java
public String toString();
```