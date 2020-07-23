# Java 代码编译和执行的整个过程

Java 代码编译是由 Java 源码编译器来完成，流程图如下所示：

![img](https://wiki.jikexueyuan.com/project/java-vm/images/javadebug.gif)

Java 字节码的执行是由 JVM 执行引擎来完成，流程图如下所示：

![img](https://wiki.jikexueyuan.com/project/java-vm/images/jvmdebug.gif)

Java 代码编译和执行的整个过程包含了以下三个重要的机制：

- Java 源码编译机制
- 类加载机制
- 类执行机制

## Java 源码编译机制

Java 源码编译由以下三个过程组成：

- 分析和输入到符号表
- 注解处理
- 语义分析和生成 class 文件

流程图如下所示：

![img](https://wiki.jikexueyuan.com/project/java-vm/images/workflow.gif)

最后生成的 class 文件由以下部分组成：

- 结构信息。包括 class 文件格式版本号及各部分的数量与大小的信息。
- 元数据。对应于 Java 源码中声明与常量的信息。包含类/继承的超类/实现的接口的声明信息、域与方法声明信息和常量池。
- 方法信息。对应 Java 源码中语句和表达式对应的信息。包含字节码、异常处理器表、求值栈与局部变量区大小、求值栈的类型记录、调试符号信息。

