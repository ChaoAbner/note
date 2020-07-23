所有操作推荐通过使用实体来操作。



## 更新操作

### 根据ID更新

```java
public void testUpdateById(){
        // 根据id进行更新
        User user = new User();
        // 设置要更新的数据ID
        user.setId(1L);
        // 设置要更新的字段
        user.setAge(20);
        int i = userMapper.updateById(user);
        System.out.println("result update num >> " + i);
    }
```



### 根据实体更新

- QueryWrapper

```java
 public void testUpdateByQueryWapper(){
        /**
        * @Description: 根据条件更新
        * @Param: []
        * @Return: void
        */
        User user = new User();
        // 设置要更新的字段
        user.setAge(30);
        // 设置条件
        QueryWrapper wrapper = new QueryWrapper();
        wrapper.eq("name", "jone");
        int i = userMapper.update(user, wrapper);
        System.out.println("result update num >> " + i);
    }
```



- UpdateWrapper

```java
public void testUpdateByUpdateWapper(){
        /**
         * @Description: 根据条件更新
         * @Param: []
         * @Return: void
         */
        UpdateWrapper<User> updateWrapper = new UpdateWrapper();

        updateWrapper.set("name", "joker")  // 更新字段（匹配数据库）
                    .set("age", 20)
                    .eq("name", "jone");    // 设置条件
        int i = userMapper.update(null, updateWrapper);     // 不用传入实体
        System.out.println("result update num >> " + i);
    }
```



## 删除操作

### 根据Map删除

```java
public void testDeleteByMap(){    
   Map map = new HashMap<>();    
   map.put("name", "joker");   // 设置条件 and关系   
   map.put("age", "20");    
   int i = userMapper.deleteByMap(map);    
   System.out.println("result update num >> " + i);
}
```

### 通过实体删除(推荐使用)

```java
public void testDeleteByQuerySwrapper(){
    User user = new User();
    user.setId(2L);
    // 将user实体传入QueryWrapper进行包装
    QueryWrapper<User> queryWrapper = new QueryWrapper<>(user);

    int i = userMapper.delete(queryWrapper);
    System.out.println("result update num >> " + i);
}
```

### 根据ID批量删除

```java
public void testDeleteByBatchIds(){
    // 传入数组
    int i = userMapper.deleteBatchIds(Arrays.asList(3L, 4L));
    System.out.println("result update num >> " + i);
}
```





## 查询操作

### selectOne

```java
public void testSelectOne(){
    /**
    * @Description: 根据一个条件获取一条数据
    * @Param: []
    * @Return: void
    */
    QueryWrapper<User> wrapper = new QueryWrapper<>();
    wrapper.eq("name", "james");
    User user = userMapper.selectOne(wrapper);

    System.out.println("data >> " + user.toString());

}
```



### selectCount

```java
public void testSelectCount(){
        /**
        * @Description: 获取查询数据的条数
        * @Param: []
        * @Return: void
        */
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper.gt("age", 18);
        Integer count = userMapper.selectCount(wrapper);
        System.out.println("data count >> " + count);

    }
```



### selectList

```java
public void testSelectList(){
        /**
        * @Description: 获取年龄大于18岁的所有数据
        * @Param: []
        * @Return: void
        */
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper.gt("age", 18);
        List<User> users = userMapper.selectList(wrapper);
        for (User user : users) {
            System.out.println(user.toString());
        }
    }
```



### selectPage

```java
public void testSelectPage(){
        /**
        * @Description: 获取分页数据
        * @Param: []
        * @Return: void
        */
        int currentPage = 1;
        int size = 10;
        Page<User> page = new Page<>(currentPage, size);
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper.gt("age", 16);

        IPage<User> userIPage = userMapper.selectPage(page, wrapper);

        System.out.println("数据总条数 => " + userIPage.getTotal());
        System.out.println("数据总页数 => " + userIPage.getPages());
        System.out.println("数据列表 => ");

        List<User> records = userIPage.getRecords();
        for (User record : records) {
            System.out.println(record);
        }

    }
```



## wrapper

### 条件构造器： QueryWrapper

#### 模糊查询

```java
public void testSelectLike(){
        /**
        * @Description: 模糊查询测试
        * @Param: []
        * @Return: void
        */
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        // 查询名字里又J的
        wrapper.like("name", "j");
        // 查询名字以J结尾的
        
//        wrapper.likeLeft("name", "i");
        // 查询名字以J开头的
        
//        wrapper.likeRight("name", "j");
        // 查询名字里又J的

        List<User> users = userMapper.selectList(wrapper);
        for (User user : users) {
            System.out.println(user);
        }
    }
```

#### 排序

```java
public void testSelectSort(){
    /**
        * @Description: 排序查询
        * @Param: []
        * @Return: void
        */

    QueryWrapper<User> wrapper = new QueryWrapper<>();
    // 按照年龄正序（升序）排序
    //wrapper.orderByAsc("age");
    // 等价于
    //wrapper.orderBy(true, true,"age");

    // 按照年龄降序排序
    wrapper.orderByDesc("age");
    // 等价于
    //wrapper.orderBy(true, false,"age");

    List<User> users = userMapper.selectList(wrapper);

    for (User user : users) {
        System.out.println(user);
    }
}
```

#### 逻辑查询

```java
public void testOr(){
    /**
        * @Description: 逻辑查询测试
        * @Param: []
        * @Return: void
        */

    QueryWrapper<User> wrapper = new QueryWrapper<>();

    // 查询名字为james或者年龄等于28的所有数据
    wrapper.eq("name", "james").or()
        .eq("age", 28);
    List<User> users = userMapper.selectList(wrapper);

    for (User user : users) {
        System.out.println(user);
    }
}		
```

#### 字段查询

```java
public void testSelect(){
    /**
        * @Description: 查询并返回数据的特定字段
        * @Param: []
        * @Return: void
        */
    QueryWrapper<User> wrapper = new QueryWrapper<>();

    // 查询名字为james或者年龄等于28的所有数据，只返回id，name, age字段
    wrapper.eq("name", "james").or()
        .eq("age", 28)
        .select("id", "name", "age");

    List<User> users = userMapper.selectList(wrapper);

    for (User user : users) {
        System.out.println(user);
    }
}
```

