#   6_MySQL

## 1_MySQL是什么

MySQL是一个开放源代码的**关系型数据库管理系统**，可以定制的。

> DBMS:数据库管理系统(Database Management System)

## 2_关系型数据库与非关系型数据库

关系型数据库(RDBMS)：

- 关系型数据库以 行(row) 和 列(column) 的形式存储数据，行和列组成 表(table) ，多个表组成了库(database)。
- 表与表之间的数据有关系(relationship)。
- 关系型数据库绝对是 DBMS 的主流，其中使用最多的 DBMS 分别是 Oracle、 MySQL 和 SQL Server。
- **优势**
  - 可以用SQL语句进行复杂的单表查询、多表查询。
  - 支持事务， 能够更安全实现数据访问。

非关系型数据库(非RDBMS) ：

- **非关系型数据库**，基于键值对 存储数据，不需要经过SQL层的解析， 性能非常高 。
- NoSQL 泛指非关系型数据库

## 3_表的关联关系

- 一对一关联：在实际的开发中应用不多，因为一对一可以创建成一张表。
  - 基础信息表 （常用信息）：学号、姓名、手机号码、班级、系别
  - 档案信息表 （不常用信息）：学号、身份证号码、家庭住址、籍贯、紧急联系人、...
- 一对多关联：客户表和订单表
- 多对多关联：要表示多对多关系，必须创建第三个表，该表通常称为 联接表 ，它将多对多关系划分为两个一对多关系。**将这两个表的主键都插入到第三个表中。**
  “订单”表和“产品”表有一种多对多的关系，这种关系是通过与“订单明细”表建立两个一对多关系来定义的。
- **自我引用**：员工表，员工姓名，部门编号， 主管编号。

## 4_索引的基本原理

- 索引是一种数据结构，用来快速地寻找特定值的记录。如果没有索引，一般来说执行查询时遍历整张表。
- 索引的原理：就是把无序的数据变成有序的查询

1. 把创建了索引的列的内容进行排序
2. 对排序结果生成**倒排表**
3. 在倒排表上 拼上**数据地址链**
4. 在查询的时候，先拿到倒排表内容，再取出数据地址链，从而拿到具体数据

## 5_mysql聚簇和非聚簇索引的区别

- 数据结构都是B+树
- 聚簇索引：
  叶子节点存储数据和索引，找到索引也就找到了数据。
  数据的物理存放顺序与索引顺序是一致的，即：只要索引是相邻的，那么对应的数据一定也是相邻地存放在磁盘上的。
- 非聚簇索引：
  叶子节点存储索引 和 **数据行地址**，也就是说根据索引查找到数据行的位置再取磁盘查找数据。

> 聚簇索引优势： 
>
> > 覆盖索引：只查索引不查数据
>
> 1. 可以**直接获取数据**，相比非聚簇索引需要第二次查询（非覆盖索引的情况下）效率要高
> 2. 适用在**排序场合**
> 3. 适用**范围查询**，因为其数据是按照大小排列的
>
> 劣势： 
>
> 1. **维护索引很昂贵**，特别是插入新行 或者 主键（PRIMARY KEY唯一索引）被更新 导至要分页(page split)的时候。
>    建议在大量插入新行后，选在**负载较低的时间段**，通过OPTIMIZE TABLE优化表。
> 2. 表因为使用UUId（随机ID）作为主键，使数据存储稀疏，扫面慢，所以建议使用int的auto_increment作为主键 
> 3. 如果主键比较大的话，那辅助索引占的空间也会更大
>
> > 辅助索引：除了聚簇索引之外的索引，辅助索引基于聚簇索引，储存聚簇索引的主键id值。使用辅助索引时先找到主键值，再去主键索引找数据。

- InnoDB（MySQL的默认存储引擎）主键索引一定是聚簇索引。
- MyISM（MySQL5.5之前的默认存储引擎）主键索引使用的是非聚簇索引。

如果涉及到**大数据量的排序**、全表扫描、count之类的操作的话，还是MyISAM占优势些，因为**索引所占空间小**，这些操作是需要在内存中完成的。

## 6_mysql索引的数据结构，各自优劣

- 索引的数据结构和**具体存储引擎的实现有关**，在MySQL中使用较多的索引有Hash索引，B+树索引等。
- InnoDB存储引擎的默认索引实现为：B+树索引。
- 哈希索引底层的数据结构就是**哈希表**，因此在绝大多数需求为**单条记录查询**的时候，可以选择哈希索引，性能最快；其余大部分场景，建议选择B+树索引。

## 7_索引设计的原则

查询更快、占用空间更小

1. 适合索引的列是出现在where，或者join … on …子句中的列
2. 基数较小的表，索引效果较差，还要单独维护**索引表**
3. 使用短索引，如果对长字符串列进行索引，应该指定一个**前缀长度**（挑选前面某几个值作为索引），这样能够节省大量索引空间，如果搜索词超过索引前缀长度，则使用索引排除不匹配的行，然后检查其余行是否可能匹配。
4. 索引列不要过多。索引需要额外的磁盘空间，并降低**写操作**的性能。
5. 定义有外键的数据列一定要建立索引。
6. **更新频繁字段**不适合创建索引
7. 若是不能**有效区分数据**的列不适合做索引列(如性别，男女未知，最多也就三种，区分度实在太低)。根据索引值能找到的数据越少越好。
8. 尽量的扩展索引，不要新建索引。
   比如表中已经有a的索引，现在要在b字段添加索引，可以使用**联合索引**（把多个字段组合起来作为一个索引）。

## 8_简述MyISAM和InnoDB(最多)的区别

- MySQL数据库存储引擎，最为常用的就是MyISAM和InnoDB两种。
- MyISAM和InnoDB的区别：

1. 存储文件：
   而InnoDB每个表只有一个文件，idb类型的文件。
   MyISAM每个表有两个文件。MYD和MYISAM文件。MYD是数据文件。MYISAM是索引文件。
2. InnoDB支持事务，支持表级锁、行级锁，支持外键；
   MyISAM不支持事务，但是每次查询都是原子的，支持表级锁，即每次操作是对整个表加锁；
3. InnoDB主键索引采用聚簇索引；
   MyISAM主键索引采用非聚簇索引
4. InnoDb不存储总行数；
   MyISAM存储表的总行数
5. InnoDB支持XA事务（可以手动控制回滚与提交）
6. InnoDB支持savePoints（部分回滚机制）

## 关心过业务系统里面的sql耗时吗？统计过慢查询吗？对慢查询都怎么优化过？

在业务系统中，除了使用主键进行的查询，其他的都会在测试库上测试其耗时，慢查询的统计主要由运维在做，会定期将业务中的慢查询反馈给我们。

慢查询的优化首先要搞明白慢的原因是什么？是查询条件没有命中索引？是load了不需要的数据列？还是数据量太大？

所以优化也是针对这三个方向来的，

- 首先分析语句，看看是否查询了额外的数据，可能是查询了多余的行并且抛弃掉了，可能是加载了许多结果中并不需要的列，对语句进行分析以及重写。查了多余的列就去掉。
- 分析语句的执行计划，然后获得其使用索引的情况，之后修改语句或者修改索引，使得语句可以尽可能的命中索引。使用效率高的索引。
- 如果对语句的优化已经无法进行，可以考虑表中的数据量是否太大，如果是的话可以进行横向或者纵向的分表。考虑分库分表。

## 事务的基本特性和隔离级别

事务基本特性ACID分别是：

**原子性**指的是一个事务中的操作要么全部成功，要么全部失败。

**一致性**指的是数据库总是从一个一致性的状态转换到另外一个一致性的状态。比如A转账给B100块钱，假设A只有90块，支付之前我们数据库里的数据都是符合约束的,但是如果事务执行成功了,我们的数据库数据就破坏约束了,因此事务不能成功,这里我们说事务提供了一致性的保证，字段要符合约束。数据的一致性，转账前和转账后，从前面数据状态变为后面数据状态，不能对不上树。一致性是事务要达成的目的。

**隔离性**：定义事务访问相同资源时，资源是否可见。有事务使用资源，不让其他事务使用，直到该事务将数据修改后或提交后，其他事务才能读取。

**持久性**指的是一旦事务提交，所做的修改就会永久保存到数据库中。

隔离性有4个隔离级别，分别是：

- read uncommit 读未提交，可能会读到其他事务未提交的数据，也叫做脏读，之后有可能回滚。用户本来应该读取到id=1的用户age应该是10，结果读取到了其他事务还没有提交的事务，结果读取结果age=20，这就是脏读。
- read commit（oracle默认） 读已提交，两次读取结果不一致，叫做不可重复读。不可重复读解决了脏读的问题，他只会读取已经提交的事务。用户开启事务读取id=1用户，查询到age=10，再次读取发现结果=20，在同一个事务里同一个查询读取到不同的结果叫做不可重复读。
- repeatable read 可重复复读，这是mysql的默认级别，只会读取已经提交的事务，读取数据之后，后面再读取直接用第一次读的值，读多次都一样，但是有可能产生幻读。
- serializable 串行，一般是不会使用的，他会给每一行读取的数据加锁，会导致大量超时和锁竞争的问题。

脏读(Drity Read)：某个事务已更新一份数据，另一个事务在此时读取了同一份数据，由于某些原因，前一个RollBack了操作，则后一个事务所读取的数据就会是不正确的。

不可重复读(Non-repeatable read):在一个事务的两次查询之中数据不一致，这可能是两次查询过程中间插入了一个事务更新的原有的数据。

幻读(Phantom Read):在一个事务的两次查询中数据条数不一致，例如有一个事务查询了几列(Row)数据，而另一个事务却在此时插入了新的几列数据，先前的事务在接下来的查询中，就会发现有几列数据是它先前所没有的。可以使用间隙锁来避免。

## Innodb是如何实现事务的

Innodb通过Buffer Pool,LogBuffer,Redo Log,Undo Log组件来实现事务，以一个updatei语句为例：
1.Innodb在收到一个updatei语句后，会先根据条件找到数据所在的页，并将该页缓存在Buffer Pool中
2.执行updatei语句，修改Buffer Pool中的数据，也就是内存中的数据
3.针对update语句生成一个RedoLog对象，并存入LogBuffer中
4.针对updatei语句生成undolog日志，用于事务回滚
5.如果事务提交，那么则把存在LogBuffer的RedoLog对象进行持久化，后续还有其他机制将Buffer Pool中所修改的数据页持久化到磁盘中
6.如果事务回滚，则利用undolog日志进行回滚

## B树和B+树的区别，为什么mysql使用B+树

B树的特点：
1.节点排序
2.一个节点了可以存多个元素，多个元素也排序了
B+树的特点：
1.拥有B树的特点
2.叶子节点之间有指针，mysql范围查找通过指针找到前后节点的数据
3,非叶子节点上的元素在叶子节点上都冗余了，也就是叶子节点中存储了所有的元素，并且排好顺序，非叶子节点存储索引数据
Mysql索引使用的是B+树，因为索引是用来加快查询的，而B+树通过对数据进行排序所以是可以提高查询速度的，然后通过一个节点中可以存储多个元素，从而可以使得
B+树的高度不会太高，在Mysql中一个Innodb页就是一个B+树节点，一个nnodb页默认16kb,所以一般情况下一颗两层的B+树可以存2000万行左右的数据，然后通过利用
B+树叶子节点存储了所有数据并且进行了排序，并且叶子节点之间有指针，可以很好的支持全表扫描，范围查找等SQL语句。

## Mysql锁有哪些，如何理解

按锁粒度分类：
1.行锁：锁某行数据，锁粒度最小，并发度高
2.表锁：锁整张表，锁粒度最大，并发度低
3.间隙锁：锁的是一个区间，某一行数据与某一行数据之间的间隙
图灵学院周
还可以分为：
1.共享锁：也就是读锁，一个事务给某行数据勖加了读锁，其他事务也可以读，但是不能写
2.排它锁：也就是写锁，一个事务给某行数据加了写锁，其他事务可以能读，但是不能加读锁，不能写
还可以分为：
1.乐观锁：并不会真正的去锁某行记录，而是通过一个版本号来实现的
2.悲观锁：上面所的锁都是悲观锁
在事务的隔离级别实现中，就需要利用锁来解决幻读

## Mysql慢查询该如何优化？

1.检查是否走了索引，如果没有则优化SQL利用索
2.检查所利用的索引，是否是最优索引
3.检查所查字段是否都是必须的，是否查询了过多字段，查出了多余数据
4.检查表中数据是否过多，是否应该进行分库分表了
5,检查数据库实例所在机器的性能配置，是否太低，是否可以适当增加资源

## 简述mysql中索引类型及对数据库的性能的影响

普通索引：允许被索引的数据列包含重复的值。
唯一索引：可以保证数据记录的唯一性。
主键：是一种特殊的唯一索引，在一张表中只能定义一个主键索引，主键用于唯一标识一条记录，使用关键字PRIMARY KEY来创建。
联合索引：索引可以覆盖多个数据列，如像INDEX(columnA,columnB)索引。
全文索引：比如文章找词，但是一般不用mysql读功能，用ES技术。通过建立倒排索引，可以极大的提升检索效率，解决判断字段是否包含的问题，是目前搜索引擎使用的一 种关键技术。可以通过ALTER TABLE table name ADD FULLTEXT(column):创建全文索引

索引可以极大的提高数据的查询速度。
通过使用索引，可以在查询的过程中，使用优化隐藏器，提高系统的性能。

但是会降低插入、删除、更新表的速度，因为在执行这些写操作时，还要操作索引文件
索引需要占物理空间，除了数据表占数据空间之外，每一个索引还要占一定的物理空间，如果要建立聚簇索引，那 么需要的空间就会更大，如果非聚集索引很多，一旦聚集索引改变，那么所有非聚集索引都会跟着变。

----

## 书写规范

- 字符串型和日期时间类型的数据可以使用单引号（' '）表示
- 列的别名，尽量使用双引号（" "），而且不建议省略as 
- 每条命令以 ; （navicat）或 \g 或 \G 结束（terminal）

- **推荐采用统一的书写规范：**
  - 数据库名、表名、表别名、字段名、字段别名等都小写
  - SQL 关键字、函数名、绑定变量等都大写

```sql
-- 列的别名
-- 列的别名不能在WHERE中使用。
SELECT employee_id emp_id,
last_name AS lname,
department_id "部门id",
salary * 12 AS "annual sal"
FROM employees;

-- 去除重复行
SELECT DISTINCT department_id 
FROM employees;

-- 过滤数据
-- WHERE子句紧随FROM子句
-- WHERE 需要声明在FROM后，ORDER BY之前。
-- LIMIT 子句必须放在整个SELECT语句的最后！
SELECT employee_id,last_name,salary
FROM employees
WHERE salary > 6000
ORDER BY salary DESC
#limit 0,10;
LIMIT 10;

SELECT (DISTINCT)....,....,....(存在聚合函数)
FROM ... (LEFT / RIGHT)JOIN ....ON 多表的连接条件 
(LEFT / RIGHT)JOIN ... ON .... （习惯上把数据多的放在左边，一般就是左外）
WHERE 不包含聚合函数的过滤条件
GROUP BY ...,....
HAVING 包含聚合函数的过滤条件
ORDER BY ....,...(ASC / DESC )
LIMIT ...,....

#4.2 SQL语句的执行过程：
#FROM ...,...-> ON -> (LEFT/RIGNT  JOIN) -> WHERE -> GROUP BY -> HAVING -> SELECT -> DISTINCT -> 
# ORDER BY -> LIMIT

-- 排序
-- 如果没有使用排序操作，默认情况下查询返回的数据是按照添加数据的顺序显示的。
#练习：显示员工信息，按照department_id的降序排列，salary的升序排列
-- 默认按照升序排列。
SELECT employee_id,salary,department_id
FROM employees
ORDER BY department_id DESC,salary ASC;

-- 分页
-- LIMIT [位置偏移量,] 行数
-- 前10条记录： 
SELECT * FROM 表名 LIMIT 0,10; 
-- （当前页数-1）* 每页条数，每页条数
SELECT * FROM table 
LIMIT(PageNo - 1)*PageSize,PageSize;
# 需求2：每页显示20条记录，此时显示第2页
SELECT employee_id,last_name
FROM employees
LIMIT 20,20;
```

```sql
-- 多表查询
-- 多表查询的分类
/*
等值连接vs非等值连接
自连接vs非自连接
内连接vs外连接
*/
#自连接的例子：
#练习：查询员工id,员工姓名及其管理者的id和姓名
SELECT emp.employee_id,emp.last_name,mgr.employee_id,mgr.last_name
FROM employees emp ,employees mgr
WHERE emp.`manager_id` = mgr.`employee_id`;

-- JOIN ...ON
-- 它的嵌套逻辑类似我们使用的 FOR 循环
-- 超过三个表禁止 join。需要 join 的字段，数据类型保持绝对一致；多表关联查询时， 保证被关联的字段需要有索引。
#SQL99语法实现内连接：
-- 连续使用
SELECT last_name,department_name,city
FROM employees e JOIN departments d
ON e.`department_id` = d.`department_id`
JOIN locations l
ON d.`location_id` = l.`location_id`;
-- 左/右外连接
SELECT last_name,department_name
FROM employees e LEFT JOIN departments d
ON e.`department_id` = d.`department_id`;
#满外连接：mysql不支持FULL OUTER JOIN

-- **合并查询结果** 利用UNION关键字，可以给出多条SELECT语句，并将它们的结果组合成单个结果集。合并时，两个表对应的列数和数据类型必须相同，并且相互对应。各个SELECT语句之间使用UNION或UNION ALL关键字分隔。
-- UNION 返回两个查询的结果集的并集，去除重复记录
-- UNION ALL 返回两个查询的结果集的并集。对于两个结果集的重复部分，不去重。
# 左下图：满外连接
# 方式1：左上图 UNION ALL 右中图
SELECT employee_id,department_name
FROM employees e LEFT JOIN departments d
ON e.`department_id` = d.`department_id`
UNION ALL
SELECT employee_id,department_name
FROM employees e RIGHT JOIN departments d
ON e.`department_id` = d.`department_id`
WHERE e.`department_id` IS NULL;
```

![image-20221007171322726](Pic/image-20221007171322726.png)

```sql
-- 所有运算符或列值遇到null值，运算的结果都为null

-- 比较运算符
-- 比较的结果为真则返回1，比较的结果为假则返回0，其他情况则返回NULL。
-- 数字用符号 字段用关键字

-- 等于运算符 一个等号;
-- 等于运算符，如果等号两边的值、字符串或表达式中有一个为NULL，则比较结果为NULL。
-- 安全等于运算符（<=>）与等于运算符（=）的作用是相似的， 唯一的区别是‘<=>’可以用来对NULL进行判断。在两个操作数均为NULL时，其返回值为1，而不为NULL；当一个操作数为NULL时，其返回值为0，而不为NULL。

BETWEEN运算符使用的格式通常为 WHERE C (NOT) BETWEEN A AND B，此时，当C大于或等于A，并且C小于或等于B时，结果为1，否则结果为0。 

-- IN运算符
-- IN运算符用于判断给定的值是否是IN列表中的一个值，如果是则返回1，否则返回0。如果给定的值为NULL则结果为NULL。 
WHERE department_id (NOT) IN (NULL,10,20,30);

-- LIKE运算符主要用来匹配字符串，通常用于模糊匹配，如果满足条件则返回1，否则返回0。如果给定的值或者匹配条件为NULL，则返回结果为NULL。 
/*
通配符
“%”：不确定个数的字符0个或多个字符。 
“_”：只能匹配一个字符。
*/
#练习：查询第3个字符是'a'的员工信息
SELECT last_name
FROM employees
WHERE last_name LIKE '__a%';
------------------------------------------------------------------------------------------------------
-- 逻辑运算符
逻辑非（NOT或!）运算符表示当给定的值为0时返回1；当给定的值为非0值时返回0；当给定的值为NULL时，返回NULL。 
逻辑与（AND或&&）运算符是当给定的所有值均为非0值，并且都不为NULL时，返回1；当给定的一个值或者多个值为0时则返回0；否则返回NULL。 
逻辑或（OR或||）运算符是当给定的值都不为NULL，并且任何一个值为非0值时，则返回1，否则返回0；当一个值为NULL，并且另一个值为非0值时，返回1，否则返回NULL；当两个值都为NULL时，返回NULL。 
```

```sql
-- 函数
/*
内置函数分为两类： 单行函数 、 聚合函数（或分组函数）
单行函数：每行返回一个结果，可以嵌套
聚合函数：输入的是一组数据的集合，输出的是单个值，不能嵌套调用

字符串函数（单行函数）
- MySQL里面，字段名下数据的查询不分大小写，即使查询条件是字段名下数据的小写，大写的数据也会查询出来
- MySQL里面，字符串的角标从1开始

IF(value,value1,value2)如果value的值为TRUE，返回value1，否则返回value2 
IFNULL(value1, value2)如果value1不为NULL，返回value1，否则返回value2
*/

-- 聚合函数
AVG / SUM ：只适用于数值类型的字段（或变量）
-- 不能用于日期，字符串
SELECT AVG(salary),SUM(salary),AVG(salary) * 107
FROM employees;

MAX / MIN :适用于任意数据类型
-- 组函数一般要起别名
SELECT MAX(salary) max_sal ,MIN(salary) mim_sal,AVG(salary) avg_sal,SUM(salary) sum_sal
FROM employees;

/*
COUNT 计算指定字段在查询结构中出现的个数（不包含NULL值的）
count(*)会统计值为 NULL 的行，而 count(列名)不会统计此列为 NULL 值的行。
COUNT(具体字段) 
*/
#需求：查询公司中平均奖金率
SELECT SUM(commission_pct) / COUNT(IFNULL(commission_pct,0)),
AVG(IFNULL(commission_pct,0))
FROM employees;

GROUP BY 的使用
#需求：查询各个部门的平均工资，最高工资
SELECT department_id,AVG(salary),SUM(salary)
FROM employees
GROUP BY department_id

#HAVING的使用 (作用：用来过滤数据的)
#练习：查询各个部门中最高工资比10000高的部门信息

#要求1：如果过滤条件中使用了聚合函数，则必须使用HAVING来替换WHERE。否则，报错。
#要求2：HAVING 必须声明在 GROUP BY 的后面。
#要求3：开发中，我们使用HAVING的前提是SQL中使用了GROUP BY。
SELECT department_id,MAX(salary)
FROM employees
GROUP BY department_id
HAVING MAX(salary) > 10000;
```

```sql
-- 子查询
-- 外查询（或主查询）、内查询（或子查询）
/*
- 子查询（内查询）在主查询之前一次执行完成。
- 子查询的结果被主查询（外查询）使用 。
- 注意事项
  - 子查询要包含在括号内
  - 将子查询放在比较条件的右侧
  - 单行操作符对应单行子查询，多行操作符对应多行子查询
  WHERE比较的就是子查询里面SELECT的内容
  
  子查询中的空值问题，不返回任何行
  */
  
 -- 单行子查询，内查询返回单行
 -- 单行操作符下的子查询
#需求：谁的工资比Abel的高？
#方式3：子查询
SELECT last_name,salary
FROM employees
WHERE salary > (
		SELECT salary
		FROM employees
		WHERE last_name = 'Abel'
		);

-- HAVING中的子查询，CASE中的子查询
HAVING MIN(salary) > (SELECT MIN(salary)   …… );

-- 多行子查询，内查询返回多行，使用多行比较操作符（IN，ANY，ALL）
WHERE  salary IN
                (SELECT   MIN(salary)
                 FROM     employees
                 GROUP BY department_id); 
                 
WHERE job_id <> 'IT_PROG'
AND salary < ANY/ALL (
		SELECT salary
		FROM employees
		WHERE job_id = 'IT_PROG'
		);
		
-- 相关子查询
-- 外部送到子查询一条记录，子查询处理过后与比较操作符比较，为真返回1保留，为假返回0不保留
#题目：查询员工中工资大于本部门平均工资的员工的last_name,salary和其department_id
#方式1：使用相关子查询
SELECT last_name,salary,department_id
FROM employees e1
WHERE salary > (
		SELECT AVG(salary)
		FROM employees e2
		WHERE department_id = e1.`department_id`
		);
		
-- EXISTS 与 NOT EXISTS关键字
-- EXISTS关键字表示如果存在某种条件，则返回TRUE，否则返回FALSE。
#方式3：使用EXISTS（能用IN的考虑使用EXISTS，能用NOT IN的考虑使用NOT EXISTS
SELECT employee_id,last_name,job_id,department_id
FROM employees e1
WHERE EXISTS (
				 -- SELECT *表示送出一条结果
	       SELECT *
	       FROM employees e2
	       WHERE e1.`employee_id` = e2.`manager_id`
	     );
```

```sql
/*
COMMIT 和 ROLLBACK
# COMMIT:提交数据。一旦执行COMMIT，则数据就被永久的保存在了数据库中，意味着数据不可以回滚。
# ROLLBACK:回滚数据。一旦执行ROLLBACK,则可以实现数据的回滚。回滚到最近的一次COMMIT之后。

① DDL（数据定义语言，CREATE 、 DROP 、 ALTER）的操作一旦执行成功，就不可回滚。指令SET autocommit = FALSE 对DDL操作失效。(因为在执行完DDL
    操作之后，一定会执行一次COMMIT。而此COMMIT操作不受SET autocommit = FALSE影响的。)
    
    DDL操作要么成功要么回滚，DDL的原子化
  
② DML（数据操作语言，INSERT 、 DELETE 、 UPDATE 、 SELECT）的操作默认情况，一旦执行，也是不可回滚的。但是，如果在执行DML之前，执行了 
    SET autocommit = FALSE，则执行的DML操作就可以实现回滚。
*/
```

```sql
-- 数据处理之增删改
#1. 添加数据
INSERT INTO emp1(id,NAME,salary)
VALUES
(4,'Jim',5000),
(5,'张俊杰',5500);

#方式2：将查询结果插入到表中
INSERT INTO emp1(id,NAME,salary,hire_date)
#查询语句
SELECT employee_id,last_name,salary,hire_date FROM ……

#2. 更新数据 （或修改数据）
# UPDATE .... SET .... WHERE ...
UPDATE emp1
SET hire_date = CURDATE(),salary = 6000
WHERE id = 4;

#3. 删除数据 DELETE FROM .... WHERE....
DELETE FROM emp1
WHERE id = 1;
```

```sql
整形常用 INT
其他数值常用 DECIMAL 定点数类型，因为浮点类型有误差
简短 或 固定长度用CHAR，其他VARCHAR
```

```sql
约束
① not null (非空约束)【单列】
② unique  (唯一性约束，允许出现多个空值：null)【可多列组合值唯一】
③ primary key (主键约束，唯一约束+非空约束，一定只要有一个主键约束，主键名PRIMARY，不要修改主键字段的值)【可多列复合主键】
④ foreign key (外键约束) (非空且唯一，必须引用/参考主表的主键或唯一约束的列)限定某个表的某个字段的引用完整性。比如：员工表的员工所在部门的选择，必须在部门表能找到对应的部分。主表（父表）：被引用的表，被参考的表。
当创建外键约束时，系统默认会在所在的列上建立对应的普通索引，删除外键约束后，必须手动删除对应的索引。
约束关系是针对主从表双方的。
外键约束有约束等级，最好是采用:     
FOREIGN KEY (deptid) REFERENCES dept(did)  ON UPDATE CASCADE ON DELETE SET NULL

⑤ default (默认值约束)

自增列 auto_increment
某个字段的值自增，默认从1开始（包括1），一个表最多只能有一个自增长列；
自增长列约束的列必须是键列（主键列，唯一键列）；
自增约束的列的数据类型必须是整数类型。
```

```sql
-- 视图
① 视图，可以看做是一个虚拟表，本身是不存储数据的。
视图的本质，就可以看做是存储起来的SELECT语句
② 视图中SELECT语句中涉及到的表，称为基表
③ 针对视图做DML操作，会影响到对应的基表中的数据。反之亦然。对视图数据的更改，都是通过对实际数据表里数据的操作来完成的。
④ 视图本身的删除，不会导致基表中数据的删除。
⑤ 视图的应用场景：针对于小型项目，不推荐使用视图。针对于大型项目，可以考虑使用视图。
⑥ 视图的优点：简化查询; 控制数据的访问；缺点：数据表结构改变，维护不方便，加之嵌套视图

#2.2 针对于多表
CREATE OR REPLACE VIEW vu_emp_dept
AS
SELECT e.employee_id,e.department_id,d.department_name
FROM emps e JOIN depts d
ON e.`department_id` = d.`department_id`;
-- 当我们创建好一张视图之后，还可以在它的基础上继续创建视图。
#6. 删除视图
DROP VIEW IF EXISTS vu_emp2,vu_emp3;
```

![image-20220627002125766](Pic/image-20220627002125766.png)

```sql
存储过程
存储过程预先存储在 MySQL 服务器上，需要执行的时候，客户端只需要向服务器端发出调用存储过程的命令，服务器端就可以把预先存储好的这一系列 SQL 语句全部执行。
【强制】禁止使用存储过程，存储过程难以调试和扩展，更没有移植性。

INOUT ：当前参数既可以为输入参数，也可以为输出参数。
characteristics 表示创建存储过程时指定的对存储过程的约束条件，其取值信息如下：
1. BEGIN…END：BEGIN…END 中间包含了多个语句，每个语句都以（;）号为结束符。 
2. DECLARE：DECLARE 用来声明变量，使用的位置在于 BEGIN…END 语句中间，而且需要在其他语句使用之前进行变量的声明。
3. SET：赋值语句，用于对变量进行赋值。 
4. SELECT… INTO：把从数据表中查询的结果存放到变量中，也就是为变量赋值。
“DELIMITER //”语句的作用是将MySQL的结束符设置为//，并以“END //”结束存储过程。存储过程定义完毕之后再使用“DELIMITER ;”恢复默认结束符。DELIMITER也可以指定其他符号作为结束符。

DELIMITER $
CREATE PROCEDURE 存储过程名(IN|OUT|INOUT 参数名 参数类型,...)
[characteristics ...]
BEGIN
sql语句1;
sql语句2; 
END $
DELIMITER ;

#类型4：带 IN 和 OUT
#举例6：创建存储过程show_someone_salary2()，查看“emps”表的某个员工的薪资，
#并用IN参数empname输入员工姓名，用OUT参数empsalary输出员工薪资。
DELIMITER //

CREATE PROCEDURE show_someone_salary2(IN empname VARCHAR(20),OUT empsalary DECIMAL(10,2))
BEGIN
	SELECT salary INTO empsalary
	FROM employees
	WHERE last_name = empname;
END //

DELIMITER ;

#调用
SET @empname = 'Abel';
CALL show_someone_salary2(@empname,@empsalary);

SELECT @empsalary;
----------------------------------------------------------------------
#类型5：带 INOUT
#举例7：创建存储过程show_mgr_name()，查询某个员工领导的姓名，并用INOUT参数“empname”输入员工姓名，
#输出领导的姓名。
DELIMITER $

CREATE PROCEDURE show_mgr_name(INOUT empname VARCHAR(25))
BEGIN

	SELECT last_name INTO empname
	FROM employees
	WHERE employee_id = (
				SELECT manager_id
				FROM employees
				WHERE last_name = empname
				);
END $

DELIMITER ;

#调用
SET @empname := 'Abel';
CALL show_mgr_name(@empname);

SELECT @empname;
```

```sql
-- 存储函数
存储函数是 用户自己定义 的，而内部函数是MySQL 的开发者定义的。
FUNCTION中总是默认为IN参数。(因为存储函数一定存在return语句)

CREATE FUNCTION 函数名(参数名 参数类型,...) 
RETURNS 返回值类型 
[characteristics ...]
BEGIN函数体 
#函数体中肯定有 RETURN 语句 
END

#举例2：创建存储函数，名称为email_by_id()，参数传入emp_id，该函数查询emp_id的email，
#并返回，数据类型为字符串型。
#创建函数前执行此语句，保证函数的创建会成功
SET GLOBAL log_bin_trust_function_creators = 1;
#声明函数
DELIMITER //
CREATE FUNCTION email_by_id(emp_id INT)
RETURNS VARCHAR(25)
BEGIN
	RETURN (SELECT email FROM employees WHERE employee_id = emp_id);
END //
DELIMITER ;
#调用
SELECT email_by_id(101);

SET @emp_id := 102;
SELECT email_by_id(@emp_id);
```

```sql
-- 变量
- 在MySQL数据库的存储过程和函数中，可以使用变量来存储查询或计算的中间结果数据，或者输出最终的结果数据。
- 在 MySQL 数据库中，变量分为 系统变量（变量由系统定义） 以及 用户自定义变量。

系统变量分为全局系统变量（需要添加 global 关键字）以及会话系统变量（需要添加 session 关键字），有时也把全局系统变量简称为全局变量，有时也把会话系统变量称为local变量。如果不写，默认会话级别。静态变量（在 MySQL 服务实例运行期间它们的值不能使用 set 动态修改）属于特殊的全局系统变量。
每一个MySQL客户机成功连接MySQL服务器后，都会产生与之对应的会话。这些会话系统变量的初始值是全局系统变量值的复制。
MySQL 中的系统变量以 两个“@” 开头，其中“@@global”仅用于标记全局系统变量，“@@session”仅用于标记会话系统变量。“@@”首先标记会话系统变量，如果会话系统变量不存在，则标记全局系统变量。

#查看指定的系统变量的值
SELECT @@global.变量名;
#查看指定的会话变量的值 
SELECT @@session.变量名;
#或者
SELECT @@变量名;

-- 修改系统变量的值
#方式1：
SET @@global.变量名=变量值;
#方式2：
SET GLOBAL 变量名=变量值;

#为某个会话变量赋值 
#针对于当前会话是有效的，一旦结束会话，重新建立起新的会话，就失效了。
#方式1：
SET @@session.变量名=变量值;
#方式2：
SET SESSION 变量名=变量值;
--------------------------------------------------------------------------------
MySQL 中的用户变量以 一个“@” 开头。根据作用范围不同，又分为会话用户变量和局部变量。
局部变量：只在 BEGIN 和 END 语句块中有效。局部变量只能在存储过程和函数中使用。

-- 会话用户变量
#会话用户变量不用声明数据类型
#查看某个未声明的会话用户变量时，将得到NULL值
#方式1：
SET @m1 = 1;
SET @m2 := 2;
SET @sum := @m1 + @m2;

#方式2：
SELECT @count := COUNT(*) FROM employees;

#方式3：
SELECT AVG(salary) INTO @avg_sal FROM employees;
-- 查看用户变量的值
SELECT @avg_sal;

-- 会话用户变量可以当函数返回值
#2. 创建函数ename_salary(),根据员工姓名，返回它的工资
DELIMITER $
CREATE FUNCTION ename_salary(emp_name VARCHAR(15))
RETURNS DOUBLE
BEGIN
	#声明变量
	SET @sal = 0; #定义了一个会话用户变量
	#赋值
	SELECT salary INTO @sal FROM employees WHERE last_name = emp_name;	
	RETURN @sal;
END $
DELIMITER ;
#调用
SELECT ename_salary('Abel');
SELECT @sal;
------------------------------------------------------------------------------------
-- 局部变量
-- 局部变量如果没有DEFAULT子句，初始值为NULL
-- 需要指定数据类型
-- 位置：只能放在 BEGIN ... END 中，而且只能放在第一句
DELIMITER //
CREATE PROCEDURE test_var()

BEGIN
	#1、声明局部变量
	DECLARE a INT DEFAULT 0;
	DECLARE b INT ;
	#DECLARE a,b INT DEFAULT 0;
	DECLARE emp_name VARCHAR(25);
	#2、赋值
	SET a = 1;
	SET b := 2;
	SELECT last_name INTO emp_name FROM employees WHERE employee_id = 101;
	#3、使用
	SELECT a,b,emp_name;	
END //

DELIMITER ;
#调用存储过程
CALL test_var();
```

```sql
- 定义条件 是事先定义程序执行过程中可能遇到的问题， 处理程序定义了在遇到问题时应当采取的处理方式，并且保证存储过程或函数在遇到警告或错误时能继续执行。这样可以增强存储程序处理问题的能力，避免程序异常停止运行。
- 说明：定义条件和处理程序在存储过程、存储函数中都是支持的。
- 在存储过程中未定义条件和处理程序，且当存储过程中执行的SQL语句报错时，MySQL数据库会抛出错误，并退出当前SQL逻辑，不再向下继续执行。

-- 定义条件
定义条件就是给MySQL中的错误码命名，这有助于存储的程序代码更清晰。它将一个 错误名字 和 指定的 错误条件 关联起来。这个名字可以随后被用在定义处理程序的 DECLARE HANDLER 语句中。

DECLARE 错误名称 CONDITION FOR 错误码（或错误条件）
#方式1：使用MySQL_error_code
DECLARE Field_Not_Be_NULL CONDITION FOR 1048;
#方式2：使用sqlstate_value
DECLARE Field_Not_Be_NULL CONDITION FOR SQLSTATE '23000'

-- 定义处理程序 
DECLARE 处理方式 HANDLER FOR 错误类型 处理语句
处理方式：处理方式有3个取值：
- CONTINUE ：表示遇到错误不处理，继续执行。
- EXIT ：表示遇到错误马上退出。
- UNDO ：表示遇到错误后撤回之前的操作。MySQL中暂时不支持这样的操作。
错误类型有很多种

-- 捕获mysql_error_value
DECLARE CONTINUE HANDLER FOR 1146 SET @info = 'NO_SUCH_TABLE';
#先定义条件，再调用
DECLARE no_such_table CONDITION FOR 1146;
DECLARE CONTINUE HANDLER FOR no_such_table SET @info = 'NO_SUCH_TABLE';
```

```sql
-- 流程控制
针对于MySQL 的流程控制语句主要有 3 类。注意：只能用于存储过程。
- 条件判断语句 ：IF 语句和 CASE 语句
- 循环语句 ：LOOP、WHILE 和 REPEAT 语句
- 跳转语句 ：ITERATE 和 LEAVE 语句

-- IF 使用在begin end中
DECLARE age INT DEFAULT 20;
	
	IF age > 40
		THEN SELECT '中老年';
	ELSEIF age > 18
		THEN SELECT '青壮年';
	ELSEIF age > 8
		THEN SELECT '青少年';
	ELSE
		SELECT '婴幼儿';
	END IF;
	
-- CASE
DELIMITER //
CREATE PROCEDURE test_case()
BEGIN
	#演示1：case ... when ...then ...
	/*
	declare var int default 2;
	
	case var
		when 1 then select 'var = 1';
		when 2 then select 'var = 2';
		when 3 then select 'var = 3';
		else select 'other value';
	end case;
	*/
	#演示2：case when ... then ....
	DECLARE var1 INT DEFAULT 10;
	CASE 
	WHEN var1 >= 100 THEN SELECT '三位数';
	WHEN var1 >= 10 THEN SELECT '两位数';
	ELSE SELECT '个数位';
	END CASE; 
-- END [case]（如果是放在begin end中需要加上case，如果放在select后面不需要）
END //
DELIMITER ;
#调用
CALL test_case();

-- LOOP 
DELIMITER //
CREATE PROCEDURE test_loop()

BEGIN
	#声明局部变量
	DECLARE num INT DEFAULT 1;
	loop_label:LOOP
		#重新赋值
		SET num = num + 1;
		#可以考虑某个代码程序反复执行。（略）
		IF num >= 10 THEN LEAVE loop_label;
		END IF;
	END LOOP loop_label;
	#查看num
	SELECT num;
END //

DELIMITER ;
#调用
CALL test_loop();

-- WHILE
DELIMITER //
CREATE PROCEDURE test_while()

BEGIN	
	#初始化条件
	DECLARE num INT DEFAULT 1;
	#循环条件
	WHILE num <= 10 DO
		#循环体（略）
		#迭代条件
		SET num = num + 1;
	END WHILE;
	#查询
	SELECT num;
END //

DELIMITER ;
-- REPEAT
-- REPEAT 循环首先会执行一次循环
DELIMITER //
CREATE PROCEDURE test_repeat()
BEGIN
	#声明变量
	DECLARE num INT DEFAULT 1;
	REPEAT
		SET num = num + 1;
		UNTIL num >= 10
	END REPEAT;
	#查看
	SELECT num;
END //
DELIMITER ;

-- LEAVE
LEAVE语句：可以用在循环语句内，或者以 BEGIN 和 END 包裹起来的程序体内，表示跳出循环或者跳出程序体的操作。如果你有面向过程的编程语言的使用经验，你可以把 LEAVE 理解为 break。
LEAVE 标记名

-- ITERATE
ITERATE语句：只能用在循环语句（LOOP、REPEAT和WHILE语句）内，表示重新开始循环，将执行顺序转到语句段开头处。如果你有面向过程的编程语言的使用经验，你可以把 ITERATE 理解为 continue，意思为“再次循环”。
ITERATE label
```





没写：单行函数。数据库、表整体的创建、增、删。部分数据类型。一些查看修改删除，约束、变量。Check约束。游标，触发器。


