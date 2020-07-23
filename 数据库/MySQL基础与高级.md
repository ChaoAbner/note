## 基础

### 去重

加上关键字 distinct，后面再加上字段名

`select distinct name from role;`



### **+号**

MySQL中的加号只有运算符的功能



### **concat**

将多个字段连接成一个字段来显示

`select concat(first_name, last_name) as name from user;`

结果将**first_name**和**last_name**合成一个name字段来显示

也可用分隔符分隔，如用`-`分隔

`select concat(first_name, '-', last_name) as name from user;`

当传入concat中的字段可能存在有null值的时候，连接的结果显示会全部为null。这时可以用函数`IFNULL`来处理。

比如**last_name**可能有null值，如果有null值，则替换成0。

`select concat(first_name, '-', IFNULL(last_name，0)) as name from user;`



### **条件查询**

`select 查询列表 from 表名 where 筛选条件；`

分类：

1. 条件表达式筛选

   **<, >, =, !=, >=, <=**

2. 按逻辑表达式筛选（用于连接条件表达式）

   **&&, ||, !, and, or, not**

3. 模糊查询

   **like, between and, in, is null**

   `%`匹配任意多的字符，`_`作为通配符匹配一个字符

   `select * from user where name like '%n%';` (查找name字段中间含有字母n的所有数据)

   `select * from user where name like '_n%';` (查找name字段第二个字母为n的所有数据) 

   如果`'_n%'`中想单纯匹配下划线，那么可以使用转移字符`\`

#### 例子

1、**between and** 

`select name, age from user where age < 60 and age > 20;`

此时可以使用**between and** 来代替< >同一个字段的情况。

`select name, age from user where between 20 and 60;`

2、**in**(取出符合范围中的所有值， in中不支持通配符)

`select name, id from user where age = 18 or age = 20 or age = 24;`

使用in

`select name, age from user where age in(18, 20, 24);`



### 排序

使用order by关键字, [asc （升序）| desc（降序）] 

`select * form user order by create_time desc;`

只有**limit**放在order by 的后面，其它都放在order by的前面。

**练习**

![1579404038994](F:\typoraImg\1579404038994.png)



### 常用函数

- 单行函数

  - 字符函数  	

    concat () 	length()	substr() 	upper() 	lower() 	ifnull()	trim()去前后空格  等

  - 数学函数   

    round(num, bit) 四舍五入,bit用来指定保留位数	ceil() 向上去取整 	floor() 向下取整 	truncate() 截断，保留一位

    mod() 取余

  - 时间函数

    now() 返回当前的系统日期+时间 	curdate() 返回当前系统日期	

    curtime() 返回当前时间	year() 传入时间参数，取出年份	month() 取出月份

    str_to_date(str, format) 将str根据format格式转成时间形式 

    date_format(date, format) 将date根据format格式转成字符串形式 

    format格式如下图

    ![1579412867351](F:\typoraImg\1579412867351.png)

  - 其它函数

- 组合函数（做统计使用，又称为统计函数、聚合函数、函数）

  sum 求和、avg 平均值（处理数值型，忽略null值）

  max 最大值、 min最小值、count 计算个数（处理任何类型）

  

  count(*) 用于统计数据总行数

  类似于count(1)的意思是另外加一列全为1的列，再统计这列的行数，也是相当于统计数据的总行数。当然count(2)也行。

  

### 分组查询

group by(field)  根据field先分组，在对每个组来进行查询等操作。

![1579415803111](F:\typoraImg\1579415803111.png)

### 连接查询

又称多表查询，联合多个表来查询

分类：

- 按年代分类：
  - sql192标准 	仅支持内连接
  - sql199标准【推荐】   支持内连接，外连接（左外，右外）交叉连接

- 按功能分类：

  - 内连接：

    等值连接	

    ![1579418590285](F:\typoraImg\1579418590285.png)

    非等值连接	（也是多表查询，连接条件不为等值）

    自连接（单表查询）

    ![1579418942987](F:\typoraImg\1579418942987.png)

  - 外连接：

    左外连接	右外连接	全外连接

  - 交叉连接：

**语法**

![1579483895621](F:\typoraImg\1579483895621.png)

### 子查询

select内部嵌套其它select语句的查询，外部的select称为外查询或主查询，内部的select称为内查询或子查询。其实内查询的结果作为一个集合给外查询。

```sql
select name from user
where group_id in 
(select group_id from group where location_id = 1);
```



### 联合查询

union。连接多个表的结果集，合并成一个结果。

应用场景一般是，查询多个表，表之间没有什么直接联系，但是查询的字段一致。

**特点**

- 多表查询的列数要求一致
- 多表查询的列数的类型和顺序最好一致
- union关键字默认去重，如果使用union all则包含重复项。

**语法**：

`select .. from ...`

`union...`

`union...`

```sql
# 单表联合查询
select name from user where age = 18 or gender = 'male';

select name from user where age = 18 
union
select name from user where gender = 'male';

# 多表联合查询
select name, gender from cn_people where gender = 'male'
union 
select name, gender from us_people where gender = 'male';

```

### 约束

限制表中的数据，保证表中数据的可靠性和准确性。

- NOT NULL： 不为空
- DEFAULT： 默认，保证字段有默认值
- PRIMARY KEY： 主键，保证该字段的唯一性，非空。
- UNIQUE： 唯一，用于保证字段的值有唯一性，可以为空。
- CHECK： 检查约束，【MySQL不支持】
- FOREIGN KEY：外键，该字段的值来自另外的表的关联列的值。

### 事务

###**含义**
	通过一组逻辑操作单元（一组DML——sql语句），将数据从一种状态切换到另外一种状态

###**特点**
	（ACID）
	原子性：要么都执行，要么都回滚
	一致性：保证数据的状态操作前和操作后保持一致
	隔离性：多个事务同时操作相同数据库的同一个数据时，一个事务的执行不受另外一个事务的干扰
	持久性：一个事务一旦提交，则数据将持久化到本地，除非其他事务对其进行修改

**相关步骤**：

```
1、开启事务
2、编写事务的一组逻辑操作单元（多条sql语句）
3、提交事务或回滚事务
```

#### 事务的分类：

**隐式事务**，没有明显的开启和结束事务的标志

```
比如
insert、update、delete语句本身就是一个事务
```

**显式事务**，具有明显的开启和结束事务的标志

```
1、开启事务
取消自动提交事务的功能

2、编写事务的一组逻辑操作单元（多条sql语句）
insert
update
delete

3、提交事务或回滚事务
```

**使用到的关键字**

```
set autocommit=0;
start transaction;
commit;
rollback;

savepoint  断点
commit to 断点
rollback to 断点
```

#### 事务的隔离级别:

**事务并发问题如何发生**？

```
当多个事务同时操作同一个数据库的相同数据时
```

**事务的并发问题有哪些**？

```
脏读：一个事务读取到了另外一个事务未提交的数据
不可重复读：同一个事务中，多次读取到的数据不一致
幻读：一个事务读取数据时，另外一个事务进行更新，导致第一个事务读取到了没有更新的数据
```

**如何避免事务的并发问题**？

```
通过设置事务的隔离级别
1、READ UNCOMMITTED
2、READ COMMITTED 可以避免脏读
3、REPEATABLE READ 可以避免脏读、不可重复读和一部分幻读
4、SERIALIZABLE可以避免脏读、不可重复读和幻读
```

**设置隔离级别**：

```sql
set session|global  transaction isolation level 隔离级别名;
```

**查看隔离级别**：

```sql
select @@transaction_isolation;
```

#### 事务ACID属性

![1579685169412](F:\typoraImg\1579685169412.png)

## 高级

### MySQL逻辑架构

- 连接层 

  客户端等连接服务，完成一些安全认证，授权等操作。引入了**连接池**的概念.

- 服务层

  完成核心服务，SQL操作、分析、优化等，以及缓存的查询。内置函数的执行。

- 引擎层

  存储引擎层，负责MySQL真正的数据存储和提取。常见**InnoDB**和**MyISAM**

- 存储层

  文件系统，硬盘。

### 存储引擎

查看当前存储引擎

```sql
show variables like "storage_engine";
```

查看数据库支持的存储引擎

```sql
show engines;
```

![1579569457692](F:\typoraImg\1579569457692.png)

#### MyISAM和InnoDB

**![1579569591884](F:\typoraImg\1579569591884.png)**

### 查询变慢

- SQL相关	
  - 没有建索引，导致全表查询
  - 查询语句写的烂
  - 太多表join
- 服务器相关
  - 服务器调优以及各个参数的设置（缓冲，线程数）

### 常见join

![1579570912871](F:\typoraImg\1579570912871.png)

### 索引

索引即是有序的快速查找的数据结构。

#### 种类

- 单值索引

  一个索引是包含一列

- 复合索引

  一个索引包含多列，通常我们都建立的是复合索引，根据经常查找的字段来建立。

- 唯一索引

  索引值的值必须唯一，但允许有空值。id默认是唯一索引。

#### **优势**

- 加快数据查找速度，降低IO成本
- 通过索引对数据进行排序（B+树），降低数据排序成本。降低了CPU的消耗

#### **劣势**

- 索引也是一张表，是占用存储空间的，保存了主键和索引字段。通常建立符合索引不宜建立太长的索引。
- 索引加快了数据查找的速度，但是更新、插入、删除操作却会变慢，因为每次都会影响索引的结构。
- 需要花费时间来建立和优化索引。

#### **语法**

![1579703209200](F:\typoraImg\1579703209200.png)



### explain

#### 用法

 explain + SQL语句

#### 用途

- 查看表的读取顺序
- 数据读取操作的操作类型
- 那些索引可以使用
- 哪些索引被实际使用
- 表之间的引用
- 每张表有多少行被优化

#### 字段

- id

- select_type 

- table type 

- possible_keys  

- key 

- key_len 

- ref 

- rows 

- Extra


### 索引优化

#### 索引失效

- 全值匹配
- **最左前缀法则**
- 不要在索引列上做任何操作（查找时使用计算、函数、（自动或手动）的类型转换）
- 索引列范围查询后面的索引会失效
- 尽量使用覆盖索引（即访问索引的查询（建立的所有索引列））,尽量不用*
- 使用!= 或 <>的时候会导致全表扫描而不用索引。
- is null 或者is not null也无法使用索引。
- like的通配符%出现在左边的时候无法使用索引如（%ike...）,会导致全表扫描。
- 字符串不加单引号导致索引失效。（即类型转换了）
- 少用or，or连接会导致索引失效

![1579680384230](F:\typoraImg\1579680384230.png)

### 锁

锁是用于协调进程和线程的并发访问资源的机制。

![1579683621572](F:\typoraImg\1579683621572.png)

![1579683646294](F:\typoraImg\1579683646294.png)

### Show Profile

诊断SQL，分析当前会话中的语句执行造成的资源消耗情况，可以作为对SQL调优的参考。

默认情况下属于关闭状态，并保存最近**15**次SQL运行的结果。

**查看当前MySQL是否支持profile**

```sql
show variables like "profiling%";
```

**开启profile**

```sql
set profiling=on;
```

**查看结果**

```sql
show profiles;
```

**诊断SQL**

![1579694998967](F:\typoraImg\1579694998967.png)

```sql
show profile cpu,block io for query;
```

**prifile中的耗时操作**

- `convert heap to MyISAM` 查询结果太大， 内存不够往磁盘上搬
- `creating tem table` 创建临时表（拷贝数据到临时表，删除临时表）
- `copying to tem table on disk` 把内存中的临时表复制到磁盘，危险！
- `locked` 被加锁了

#### 锁粒度

- 表锁（偏读）

  ```sql
  # 给一张表加上读锁或者写锁
  
  # 读锁是共享锁，多个连接或客户端都可读取访问，但是不能修改。加读锁的session不能读其他的表。当前sesstion插入或者更新锁定的表都会报错，其它session插入或者更新或阻塞等待锁的释放。
  
  # 写锁是排他锁，只有加锁的session才能进行读和写操作，其它session不能读和写，会被阻塞。
  # 当前session不能读和修改其它的表。
  
  # 简言之，读锁会阻塞写，写锁会阻塞读和写。
  lock table tablename read/wirte;
  
  # 解锁
  unlock tables;
  ```

- 行锁（偏写）

  MySQL默认自动提交，可以先将自动提交关闭，`set autocommit 0`，当前session修改第四行，其它session就修改第四行就会被阻塞，知道当前session被提交。

### 索引失效导致行锁变表锁

​		索引失效会导致行锁编程表锁。比如修改varchar类型但是传入整形，使数据库内部进行类型转换导致索引失效，此时修改成功但是未提交的情况下会锁住整个表。其它session无法对该表的其它数据进行修改。

### 间隙锁

间隙锁插入的问题

```sql
# user表有两列 age(int) 和 name(varchar)
# 假设user表的age为12的为空，A session执行
update user set name='Lee' where age < 18 and age > 10;

# B session执行,此时会发生阻塞，直到A session提交后这个插入命令才被执行
insert into user value (12, 'Mike');
```

当我们使用范围条件而不是相等条件的时候，刚好范围中可能有不存在的值，就叫`间隙（GAP）`，InnoDB会对这些范围内的索引项进行加锁；当另一个事务对这些间隙进行插入操作的时候，就会导致阻塞的问题。

InnoDB对这个“间隙”加锁，这个就叫做间隙锁（Next-key锁）

#### 危害

 因为InnoDB会锁定范围内所有的索引键值，即使这个键值并不存在，导致一些不存在的键值被无辜锁定，所以这是无法插入锁定键值范围内的任何数据。

