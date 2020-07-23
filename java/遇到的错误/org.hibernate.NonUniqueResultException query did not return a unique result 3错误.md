# org.hibernate.NonUniqueResultException: query did not return a unique result: 3错误

意思是程序意外的返回了多条数据，考虑数据库某些字段或者完全插入了相同的记录。



**解决办法**

1、删除数据库中其他相同的记录

2、采用list()方法返回，但这个时候需要注意实体的类型，不然在运行的时候也会出错。