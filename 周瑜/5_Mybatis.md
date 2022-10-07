# 5_Mybatis

## 1_Mybatis的优缺点

优点：

1. 基于 SQL 语句编程，灵活，不会对应用程序或者数据库的**现有设计**造成影响；
   SQL 写在XML 里，便于统一管理，解除 sql 与程序代码的耦合；
   支持编写**动态** SQL 语句， 可重用。
2. 与 JDBC 相比，减少了代码量，**不需要手动开关连接**；
3. 与各种数据库**兼容**（ 因为 MyBatis 使用 JDBC 来连接数据库，所以只要JDBC 支持的数据库 MyBatis 都支持）。
4. 能够与 Spring 很好的**集成**；
5. 提供映射标签， **支持对象与数据库的 ORM 字段关系映射**； 提供**对象关系映射标签， 支持对象关系组件维护**。

缺点：

1. SQL 语句的编写工作量较大， 尤其当字段多、**关联表多**时。
   字段映射时，有时需要额外配置很多映射关系。
2. SQL 语句依赖于数据库， 导致数据库**移植性**差， 不能随意更换数据库。

## 2_MyBatis与Hibernate对比(重要)

- MyBatis与Hibernate都是java使用较多的操作数据库框架。

- Hibernate优点：数据库无关性好；有映射关系的维护，是ORM框架，可以通过操作java对象操作数据库；代码简单，代码量少，更专注于业务。

  Hibernate缺点：学习门槛高，而且怎么设计ORM映射，如何平衡 性能和对象模型，对经验和能力要求高。

- MyBatis优点：入门简单；没有映射关系的维护，是SQL框架；手动编写sql，灵活。

  MyBatis缺点：框架比较简陋，功能尚有缺失，虽然简化了**数据绑定代码**，但是整个底层数据库查询实际还是要自己写的，工作量也比较大，而且不太容易适应**快速数据库修改**。

- 使用建议：复杂查询少，对象模型要求高，用Hibernate，封装性高
  				  复杂查询多，对象模型要求低，用MyBatis，方便管理、灵活

> ORM对象关系映射：Java bean和数据库数据进行双向映射

## 3_#{}和${}的区别是什么？

- \#{}是**预编译处理、是占位符**， ${}是**字符串替换、是拼接符**。

- Mybatis 在处理#{}时，会将 sql 中的#{}替换为?号，调用 PreparedStatement（预编译）来赋值；

  Mybatis 在处理${}时， 就是把${}替换成变量的值，调用 Statement （非预编译）来赋值；

- #{} 的变量替换是在DBMS (数据库管理系统Database Management System)中、变量替换后，#{} 对应的变量自动**加上单引号**

  ${} 的变量替换是在 DBMS 外、变量替换后，${} 对应的变量不会加上单引号

- 使用#{}可以有效的**防止 SQL 注入**， 提高系统安全性。查询字段就用${}。

## 4_简述Mybatis的插件运行原理，如何编写一个插件（先不背）

- 比如分页插件
- Mybatis的插件，指的就是Mybatis的拦截器，只能拦截4个接口
  - Executor（生成sql语句，维护sql语句查询缓存）、
  - ParameterHandler（将sql的java类型参数转换数据库支持类型）、
  - StatementHandler（往sql传参数，对结果集映射） 、
  - ResultSetHandler（接收结果集）
    Mybatis 使用 JDK 的动态代理（方法增强，前/后拦截逻辑）， 为需要拦截的接口生成代理对象以实现接口方法拦截功能， 每当执行这 4 种接口对象的方法时，就会进入拦截方法，具体就是 InvocationHandler 的invoke() 方法， 拦截那些你指定需要拦截的方法。

编写插件： 实现 Mybatis 的 Interceptor 接口并实现 intercept()方法， 方法提供invocation参数来获取代理的目标对象，来获取四个接口拦截的方法内的参数，给插件编写注解， 指定要拦截哪一个接口的哪些方法即可， 在配置文件中配置编写的插件 或 @Component 。

```java
@Intercepts({@Signature(type = StatementHandler.class, method = "query", args = {Statement.class, ResultHandler.class}),
@Signature(type = StatementHandler.class, method = "update", args = {Statement.class}),
@Signature(type = StatementHandler.class, method = "batch", args = { Statement.class })}),
@Component 

//query()执行之前做的操作
invocation.proceed()执行目标类具体的业务逻辑
  //比如拦截StatementHandler的query() = 具体的业务逻辑
//query()执行之后做的操作
```

