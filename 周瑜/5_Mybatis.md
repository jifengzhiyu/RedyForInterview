# 5_Mybatis

## Mybatis是什么

是一个基于Java的持久层框架。

## Mybatis优缺点

优点：

1. 基于 SQL 语句编程，灵活，不会对应用程序或者数据库的**现有设计**造成影响；
2. SQL 写在XML 里，便于统一管理，解除 sql 与程序代码的耦合；
3. 支持编写**动态** SQL 语句， 可重用。
2. 与 JDBC 相比，减少了代码量，**不需要手动开关连接**，设置参数以及获取结果集；
3. 与各种数据库**兼容**（ 因为 MyBatis 使用 JDBC 来连接数据库，所以只要JDBC 支持的数据库 MyBatis 都支持）。
4. 能够与 Spring 很好的**集成**；

缺点：

1. SQL 语句的编写工作量较大， 尤其当字段多、**关联表多**时。
   字段映射时，有时需要额外配置很多映射关系。
2. SQL 语句依赖于数据库， 导致数据库**移植性**差， 不能随意更换数据库。

## MyBatis与Hibernate、JDBC对比(重要)

> ORM(Object Relation Mapping)对象关系映射：Java bean和数据库数据进行双向映射

- JDBC

     1. SQL夹杂在Java代码中，耦合度高，导致硬编码内伤
     2. 代码冗长，开发效率低
- Hibernate

     - Hibernate优点：
          1. **数据库无关性好**；
          2. 有映射关系的维护，是ORM框架，可以通过操作java对象操作数据库；
          3. 开发效率高，更专注于业务。
     - Hibernate缺点：
          1. 学习门槛高；程序中的长难复杂SQL需要绕过框架；
          2. 内部自动生产的SQL，不容易做特殊优化；
          3. 反射操作太多，导致数据库性能下降。
     - MyBatis优点
          - MyBatis优点：
               1. 轻量级，性能出色；
               2. 入门简单；是一个半自动的ORM框架；
               3. 手动编写sql，灵活。
          - MyBatis缺点：
               1. 框架比较简陋，功能尚有缺失
               2. 写sql工作量比较大
- 使用建议：
     - 复杂查询少，**对象模型**要求高，用Hibernate，封装性高
     - 复杂查询多，对象模型要求低，用MyBatis，方便管理、灵活


## #{}和${}区别

- \#{}是**预编译处理、是占位符**， ${}是**字符串替换、是拼接符**。

- Mybatis 在处理#{}时，会将 sql 中的#{}替换为?号，调用 PreparedStatement（预编译）来赋值；

  Mybatis 在处理${}时， 就是把${}替换成变量的值，调用 Statement （非预编译）来赋值；

- #{} 的变量替换是在DBMS (数据库管理系统Database Management System)中、变量替换后，#{} 对应的变量自动**加上单引号**

  ${} 的变量替换是在 DBMS 外、变量替换后，${} 对应的变量不会加上单引号

- 使用#{}可以有效的**防止 SQL 注入**， 提高系统安全性。

## MyBatis缓存

下次查询相同的数据，就会从缓存中直接获取，不会从数据库重新访问

- 一级缓存是**SqlSession**级别的，通过同一个SqlSession查询的数据会被缓存;
- 二级缓存是**SqlSessionFactory**级别，通过同一个SqlSessionFactory创建的SqlSession查询的结果会被缓存；
- 缓存查询的顺序
  1. 先查询二级缓存，因为二级缓存中可能会有其他程序已经查出来的数据，可以拿来直接使用。
  2. 如果二级缓存没有命中，再查询一级缓存
  3. 如果一级缓存也没有命中，则查询数据库
- SqlSession关闭之后，一级缓存中的数据会写入二级缓存

## Mybatis使用

- 需要：
  Java实体类
  Mapper接口
  Mapper映射文件
- MyBatis的核心配置文件为mybatis-config.xml，配置连接数据库的环境，MyBatis的全局配置信息，引入映射文件

```xml
<mappers>
<mapper resource="mappers/UserMapper.xml"/>
</mappers>
```

- MyBatis中的mapper接口相当于以前的dao。但是区别在于，mapper仅仅是接口，我们不需要提供实现类。
  - a>mapper接口的全类名和映射文件的命名空间（namespace）保持一致
    b>mapper接口中方法的方法名和映射文件中编写SQL的标签的id属性保持一致
  

```java
public interface UserMapper {
//添加用户信息
int insertUser();
}
```

- 映射文件的命名规则：
  表所对应的实体类的类名+Mapper.xml
  因此一个映射文件对应一个实体类，对应一张表的操作
  MyBatis映射文件用于编写SQL，访问以及操作表中的数据
  MyBatis映射文件存放的位置是src/main/resources/mappers目录下

```xml
<mapper namespace="com.atguigu.mybatis.mapper.UserMapper">
<!--int insertUser();-->
<insert id="insertUser">
insert into t_user values(null,'张三','123',23,'女')
</insert>
</mapper>
```

````xml
/**
* 根据用户id查询用户信息
* @param id
* @return
*/
<!--User getUserById(@Param("id") int id);-->
<select id="getUserById" resultType="User">
select * from t_user where id = #{id}
</select>

<!--Map<String, Object> getUserToMap(@Param("id") int id);-->
<select id="getUserToMap" resultType="map">
select * from t_user where id = #{id}
</select>
<!--结果：{password=123456, sex=男, id=1, age=23, username=admin}-->
````

---

下面先不背：

## Mybatis的插件运行原理，如何编写一个插件

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

## 
