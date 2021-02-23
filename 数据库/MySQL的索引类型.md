# MySQL的索引类型

MySQL的索引类型主要有四种：

- 普通索引（KEY）
- 唯一索引（UNIQUE）
- 主键索引（PRIMARY）
- 全文索引（FULLTEXT）

## 普通索引

### 介绍

普通索引就是最基本的索引，没有任何的限制，可以认为就是用来加快查询速度的索引。

普通索引又分为**单列索引**和**联合索引**。

通常我们的SQL语句中有`where a = 'xx'`或者 `order by a`的情况都可以对那个字段建立普通索引。

### 使用

- 创建索引

```SQL
- 单列索引
CREATE INDEX index_name ON table(column_name)
- 联合索引
CREATE INDEX index_name ON table(col1, col2, col3)
```

- 修改表结构来添加索引

```SQL
- 单列索引
ALTER TABLE table_name ADD INDEX index_name ON (column_name)
- 联合索引
ALTER TABLE table_name ADD INDEX index_name ON (col1, col2, col3)
```

- 创建表的时候创建索引

```sql
CREATE TABLE `table` (
    `id` int(11) NOT NULL AUTO_INCREMENT ,
    `title` char(255) CHARACTER NOT NULL ,
    `content` text CHARACTER NULL ,
    `time` int(10) NULL DEFAULT NULL ,
    PRIMARY KEY (`id`),
    KEY `单列索引` (col_1)
    KEY `联合索引` (col_1, col_2, col_3)
)
```

**删除索引**

```sql
DROP INDEX index_name ON table
```

## 唯一索引

### 介绍

唯一索引与普通索引类似，一个表中可以有多个唯一索引。不同就是索引列必须唯一，即不能有重复数据，但是允许有空值。

如果是组合索引，则列值的组合必须唯一。

### 使用

- 创建唯一索引

```sql
CREATE UNIQUE INDEX index_name ON table(column_name)
```

- 修改表结构

```sql
ALTER TABLE table_name ADD UNIQUE index_name ON (column_name)
```

- 创建表的时候指定

```sql
CREATE TABLE `table` (
    `id` int(11) NOT NULL AUTO_INCREMENT ,
    `title` char(255) CHARACTER NOT NULL ,
    `content` text CHARACTER NULL ,
    `time` int(10) NULL DEFAULT NULL ,
    UNIQUE indexName (column_name)
);
```

## 主键索引

### 介绍

主键索引是一种特殊的唯一索引，但是主键索引不允许有空值，并且一个表中只有一个主键索引。一般是在建表的时候同时创建主键索引。

### 使用

```sql
CREATE TABLE `table` (
    `id` int(11) NOT NULL AUTO_INCREMENT ,
    `title` char(255) NOT NULL ,
     PRIMARY KEY (`id`)
);
```

## 全文索引

### 介绍

主要用来查找文本中的关键字，而不是直接与索引中的值相比较。

fulltext索引跟其它索引大不相同，它更像是一个搜索引擎，而不是简单的where语句的参数匹配。

fulltext索引配合match against操作使用，而不是一般的where语句加like。

它可以在create table，alter table ，create index使用，不过目前只有char、varchar，text 列上可以创建全文索引。

> 值得一提的是，在数据量较大时候，现将数据放入一个没有全局索引的表中，然后再用CREATE index创建fulltext索引，要比先为一张表建立fulltext然后再将数据写入的速度快很多。

### 使用

- 创建表时添加全文索引

```sql
CREATE TABLE `table` (
    `id` int(11) NOT NULL AUTO_INCREMENT ,
    `title` char(255) CHARACTER NOT NULL ,
    `content` text CHARACTER NULL ,
    `time` int(10) NULL DEFAULT NULL ,
    PRIMARY KEY (`id`),
    FULLTEXT (content)
);
```

- 修改表结构添加全文索引

```sql
ALTER TABLE article ADD FULLTEXT index_content(content)
```

- 创建全文索引

```sql
CREATE FULLTEXT INDEX index_content ON article(content)
```

