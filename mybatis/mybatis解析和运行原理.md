#### SqlSession下的四大对象

SqlSesstion的执行过程是通过Executor、StatementHandle、ParameterHandler和ResultSetHandler来完成数据库操作和结果返回的。

- Executor代表执行器，由它调度StatementHandler、ParameterHandler、ResultSetHandler等来执行对应的SQL。其中StatementHandler最重要。
- StatemengHandler的作用是使用数据库的Statement执行操作。
- ParameterHandler是处理SQL参数的。
- ResultSetHandler临时进行数据集（ResultSet）的封装返回处理的。
  

