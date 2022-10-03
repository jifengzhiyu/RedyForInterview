# 三、 Spring底层篇

## 1_什么是Spring?

- Spring是一个企业级java应用框架，他的作用主要是简化软件的**开发以及配置过程，简化项目部署环境**。
  Spring是一个轻量级的控制反转（IOC）和面向切面（AOP）的**容器框架**。
  - 包含并管理Bean的配置和生命周期，这个意义上是一个容器。
  - 将简单的组件配置、组合成为复杂的应用，这个意义上是一个框架。

Spring jar包：

提供sql表达式支持

![image-20220928185658156](Pic/image-20220928185658156.png)

## 2_Spring的优点

1、Spring**低侵入设计**，对业务代码的污染非常低。（大多基于注解，不依赖于spring特定的类）
2、**Spring的DI机制**将对象**之间的关系**交由框架处理，减少组件的耦合。
3、Spring提供了AOP技术，支持将一些通用的功能进行集中式管理，从而提供更好的复用。（配置安全权限）
4、Spring对于主流框架提供了非常好的支持。

## 3_谈谈你对AOP的理解

- AOP面向切面。
- 系统是由许多不同的组件所组成的，每一个组件各负责一块特定功能。除了实现自身核心功能之外，这些组件还经常承担着额外的职责。例如**日志、事务管理和安全**这样的非核心服务经常融入到自身具有核心业务逻辑的组件中去。 这些**系统服务**经常被称为**横切关注点**，因为它们会跨越系统的多个组件。
  当我们需要为分散的对象引入公共行为的时候，OOP（面向对象）则显得无能为力。也就是说，OOP允许你定义**从上到下**的关系，但并不适合定义从**左到右**的关系。
  比如日志代码往往水平地散布在所有对象层次中，而与它所散布到的对象的核心功能毫无关系。
  在OOP设计中，它导致了大量代码的重复，而不利于各个模块的重用。

AOP:将程序中的交叉业务逻辑（比如安全，日志，事务等），封装成一个**切面**，然后注入到目标对象（具体业务逻辑)中去。

- AOP可以对某个对象或某些对象的**功能进行增强**，比如对象中的方法进行增强，可以在执行某个方法前后额外的做一些事情。比如一个接口里面没有写打印日志逻辑，但是自动拥有打印日志功能。

## 4_谈谈你对IOC的理解

- IOC表示控制反转

- IOC容器就是个map，里面存储各种对象，在项目启动的时候会读取配置文件里面的bean节点、@repository、@service、@controller、@component注解的类，根据全类名使用**反射**创建对象放到map里。

  注入时会读取xml里bean节点内的ref属性根据id注入，也会扫描autowired、resource等注解的属性，根据类型或id对象名注入。

- 什么是控制反转：

  对象控制权的转移，把创建对象及给属性赋值 的控制权转移给Spring自动生产。
  我们要做的仅仅是定义类，以及定义哪些**属性**需要Spring来赋值 (比如某个属性上加@Autowired)

  - 优点：减轻了程序员的负担

- 依赖注入：

  依赖注入是实现IOC的方法，IOC容器在运行期间，动态地将某种**依赖关系**注入到对象之中。

  - IOC有三种注入方式：

    1、构造器注入 2、setter方法注入 3、根据注解注入。

## 5_如何实现一个IOC容器

1、配置文件中指定需要扫描的包路径

2、将当前路径下所有以.class结尾的文件添加到一个Set集合中存储

3、遍历这个set集合，反射获取在类上有指定注解类的对象，并将其交给IOC容器，定义Map用来存储这些对象

4、对需要注入的类进行**依赖注入**

## 6_解释下Spring支持的几种bean的作用域

1. singleton（单例模式）：默认，每个IOC**容器**中只有一个bean的实例，由BeanFactory自身来维护。该对象的生命周期是与Spring IOC容器一致的，但在第一次被注入时创建。
2. prototype（原型模式）：为每一个getbean请求提供一个实例。在每次注入时都会创建一个新的对象，容器中有多个实例。
3. request：每个HTTP请求中有一个bean的单例对象，在HTTP请求结束后，bean会随之失效。
4. session：每个session中有一个bean的单例对象，在session过期后，bean会随之失效。
5. application：bean被定义为在容器上下文ServletContext的生命周期中复用一个单例对象。
6. websocket：bean被定义为在websocket的生命周期中复用一个单例对象。

## 7_Spring事务的实现方式和原理？

- 在使用Spring框架时，有两种使用事务的方式，一种是编程式（手动写代码），一种是申明式（使用注解）。
- 事务这个概念是数据库层面的，Spring只是基于数据库中的事务进行了扩展，能让程序员更加方便操作事务。
- 比如我们可以通过在某个方法上增加@Transactional注解，就可以开启事务，这个方法中所有的sql都会在一个事务中执行，要么都成功要么都失败。
- 在一个方法上加了@Transactional注解后，Spring会基于这个类生成一个**代理对象**，会将这个代理对象作为bean放在IOC容器中，当在使用这个代理对象的方法时，如果这个方法上存在@Transactional注解，那么**代理逻辑**会先把事务的自动提交设置为false，然后再去执行原本的业务逻辑方法，如果执行业务逻辑方法没有出现异常，那么代理逻辑中就会将事务进行提交，如果执行业务逻辑方法出现了异常，那么则会将事务进行回滚。
- 针对哪些异常回滚事务是可以配置的，可以利用@Transactional注解中的**rollbackFor**属性进行配置，默认情况下会对RuntimeException和Error进行回滚。

## 8_Spring事务的隔离级别

spring事务隔离级别就是数据库的隔离级别：外加一个默认级别（不同数据库类型自己的默认级别）

- read uncommitted（未提交读）
- read committed（提交读、不可重复读），Oracle默认
- repeatable read（可重复读），mysql默认
- serializable（可串行化）

> 数据库的配置隔离级别是Read Commited,而Spring配置的隔离级别是Repeatable Read，请问这时隔离 级别是以哪一个为准？ 以Spring配置的为准，如果spring设置的隔离级别数据库不支持，效果取决于数据库

## 9_spring事务传播机制

> 下面都是@Transactional的7个枚举参数，修饰被调用者（方法B）

a()调用b()时

- REQUIRED（**required**）(Spring默认的事务传播类型)：
  - a()没有事务，则b()新建事务；
  - a()有事务，则b()加入a()的事务，a()b()有异常就回滚。
- SUPPORTS（**supports**）：
  - a()没有事务，则b()也没有事务；
  - a()有事务，则b()加入a()的事务。
- MANDATORY（**mandatory**）：
  - a()没有事务，则b()抛出异常；
  - a()有事务，则b()加入a()的事务。
- REQUIRES_NEW（**requires-new**）：
  - **b()创建事务**，若a()有事务则挂起该事务；各自处理各自的事物，需要回滚只回滚自己
- NOT_SUPPORTED（**not-supported**）：
  - **b()没有事务**，若a()有事务则挂起该事务
- NEVER（**never**）：
  - 都不使用事务，如果a()有事务，则抛出异常
- NESTED（**nested**）：
  - a()有事务，则在嵌套事务中执行，a是父事务，b是子事务；父事务回滚时， 子事务也会回滚；子事物回滚时，父事务可以catch其异常
  - a()没有事务，则b()新建事务

## 10_Spring中什么时候@Transactional会失效（事务失效）

sprig事务的原理是AOP，进行了切面增强，那么失效（方法上加的@Transactional失效）的根本原因是这个AOP不起作用了，常见情况有如下几种

1. 【重要】发**生自调用**：
   使用this调用本类的方法(this通常省略)，此时这个this对象不是**代理对象**，而是UserService对象本身，就没有代理逻辑来处理这个@Transactional注解，该注解就会失效。 
   解法让this变成UserService的代理对象。

2. **@Transactional用于非public的方法上;**
   - 如果要用在非public方法上，可以开启**AspectJ代理模式**。
   - 同时如果某个方法是private，底层cglib是基于父子类来实现的，子类是不能**重载**父类的private方法，所以无法很好利用代理对象
3.  数据库不支持事务 
4. bean没有被spring管理
5. 异常被吃掉，事务不会回滚 或者 抛出的异常没有被定义

## 11_什么是bean的自动装配，有哪些方式？

- 开启自动装配，只需要在xml配置文件中定义“autowire”属性。

```xml
<bean id="cutomer" class="com.xxx.xxx.Customer" autowire="" />
```

- autowire属性有五种装配的方式：

  - **缺省情况下**，通过“ref”属性手动设定

    ```
    手动装配：以value或ref的方式明确指定属性值都是手动装配。 
    需要通过‘ref’属性来连接bean。
    ```

    - byName ---根据bean的属性名称进行自动装配。

      ```xml
      Cutomer的属性名称是person，Spring会将bean id为person的bean通过setter方法进行自动装配。
      <bean id="cutomer" class="com.xxx.xxx.Cutomer" autowire="byName"/> 
      <bean id="person" class="com.xxx.xxx.Person"/>
      ```

    - byType---根据bean的类型进行自动装配。

  
    ```xml
    Cutomer的属性person的类型为Person，Spirng会将Person类型通过setter方法进行自动装配。
    <bean id="cutomer" class="com.xxx.xxx.Cutomer" autowire="byType"/>
    <bean id="person" class="com.xxx.xxx.Person"/>
    ```
  
    -  constructor---如果一个bean与构造器参数的类型形同，则进行自动装配，否则导致异常。
  
  
    ```xml
    Cutomer构造函数的参数person的类型为Person，Spirng会将Person类型通过构造方法进行自动装配。
    <bean id="cutomer" class="com.xxx.xxx.Cutomer" autowire="construtor"/>
    <bean id="person" class="com.xxx.xxx.Person"/>
    ```

    -  autodetect---如果有默认的构造器，则通过constructor方式进行自动装配，否则使用byType方式进行自动装配。
  

@Autowired注解方式自动装配bean，可以在**字段、setter方法、构造方法**上使用。

## 12_描述一下Spring Bean的生命周期？

1. 解析类得到BeanDefinition
2. 如果有多个构造方法，则要推断**构造方法**
3. 确定好构造方法后，进行实例化得到一个对象
4. 对对象中的加了@Autowired注解的属性进行注入
5. **处理回调Aware接口**，比如BeanNameAware，BeanFactoryAware
6. 初始化前，调用BeanPostProcessor的**初始化前的方法**，@PostConstruct注解的方法在BeanPostProcessor**前置处理器**中被执行。
7. 调用初始化方法，如果bean的类实现了**InitializingBean**接口，就处理该接口
8. 初始化后，进行AOP
9. 如果当前创建的bean是单例的则会把bean放入单例池
10. 使用bean
11. Spring容器关闭时调用**DisposableBean**中destory()方法

## 13_Spring中Bean是线程安全的吗？

- Spring中的Bean默认是单例模式，并没有针对Bean做线程安全的处理，所以： 

1. 如果Bean是无状态（bean**内部属性**不会变化）的，那么Bean则是线程安全的 

2. 如果Bean是有状态的，那么Bean则不是线程安全的

   ​	做法：

   - 改变**bean的作用域**，把 "singleton"改为"protopyte"，这样每次请求Bean就相当于是 new Bean() 。
   - 不要在bean中声明任何有状态的**实例变量或类变量**，如果必须如此，那么就使用**ThreadLocal**把变量变为**线程私有**，如果bean的实例变量或类变量需要在多个线程之间共享，那么就只能使用synchronized、lock、CAS等线程同步方法了。

- 另外，Bean是不是线程安全，跟Bean的作用域没有关系，得看这个Bean对象本身的各种方法属性是不是线程安全（方法有没有加锁等）。

## 14_ApplicationContext和BeanFactory有什么区别

- BeanFactory是Spring中非常核心的组件，表示Bean工厂，可以生成Bean,维护Bean
- 而ApplicationContext接口继承了BeanFactory接口,所以ApplicationContext拥有BeanFactory 所有的特点，也是一个Bean工厂，
  但是ApplicationContext除开继承了BeanFactory,之外，还继承了诸如EnvironmentCapable、MessageSource、ApplicationEventPublisher、ResourcePatternResolver接口，从而ApplicationContext还有获取系统环境变量（获取操作系统、jvm环境变量的键值、property文件的键值）、国际化、事件发布、资源解析器（获取网络的某个网页、文系统的某个文件）功能，这是BeanFactory)所不具备的

## Spring中的事务是如何实现的

1.Spring事务底层是基于数据库事务和AOP机制的
2.首先对于使用了@Transactional注解的Bean,Spring会创建一个代理对象作为Bean
3.当调用代理对象的方法时，会先判断该方法上是否加了@Transactional注解
4.如果加了，那么则利用事务管理器创建一个数据库连接
5.并且修改数据库连接的autocommit属性为false,禁止此连接的自动提交，这是实现Spring事务非常重要的一步
6.然后执行当前方法，方法中会执行sql
7.执行完当前方法后，如果没有出现异常就直接提交事务
8.如果出现了异常，并且这个异常是需要回滚的就会回滚事务，否则仍然提交事务 
9,Spring事务的隔离级别对应的就是数据库的隔离级别
10.Spring事务的传播机制是Spring事务自己实现的，也是Spring事务中最复杂的
11.Spring事务的传播机制是基于数据库连接来做的，一个数据库连接一个事务，如果传播机制配置为需要新开一个事务，那么实际上就是先建立一个数据库连接，在此新 数据库连接上执行sql

## Spring容器启动流程是怎样的

1.在创建Spring容器，也就是启动Spring时：
2.首先会进行扫描，扫描得到所有的BeanDefinition对象，并存在一个Map中
3.然后筛选出非懒加载的单例BeanDefinition进行创建Bean,对于多例Bean不需要在启动过程中去进行创建，对于多例Bean会在每次获取Bean时利用BeanDefinition去创建
4.利用BeanDefinition创建Bean就是Bean的创建生命周期，这期间包括了合并BeanDefinition、推断构造方法、实例化、属性填充、初始化前、初始化、初始化后等步骤
其中AOP就是发生在初始化后这一步骤中
5.非懒加载的单例Bean创建完了之后，Spring会发布一个容器启动事件，表示Spring启动结束
6.在源码中会更复杂，比如源码中会提供一些模板方法，让子类来实现，比如源码中还涉及到一些BeanFactoryPostProcessor和BeanPostProcessor的注册，Spring的扫描就是通过BenaFactoryPostProcessor来实现的，依赖注入就是通过BeanPostProcessor来实现的
7.在Spring启动过程中还会去处理@Import等注解

## **Spring** 框架中都用到了哪些设计模式？

简单工厂：由一个工厂类根据传入的参数，动态决定应该创建哪一个产品类。

> Spring中的BeanFactory就是简单工厂模式的体现，根据传入一个唯一的标识来获得Bean对象，但是否是在传入参数后创建还是传入参数前创建这个要根据具体情况来定。 

工厂方法：

> 实现了FactoryBean接口的bean是一类叫做factory的bean。其特点是，spring会在使用getBean()调 用获得该bean时，会自动调用该bean的getObject()方法，所以返回的不是factory这个bean，而是这个 bean.getOjbect()方法的返回值。

单例模式：保证一个类仅有一个实例，并提供一个访问它的全局访问点

> spring对单例的实现： spring中的单例模式完成了后半句话，即提供了全局的访问点BeanFactory。但没有从构造器级别去控制单例，这是因为spring管理的是任意的java对象。

适配器模式：

> Spring定义了一个适配接口，使得每一种Controller有一种对应的适配器实现类，让适配器代替 controller执行相应的方法。这样在扩展Controller时，只需要增加一个适配器类就完成了SpringMVC 的扩展了。

装饰器模式：动态地给一个对象添加一些额外的职责。就增加功能来说，Decorator模式相比生成子类更为灵活。

> Spring中用到的包装器模式在类名上有两种表现：一种是类名中含有Wrapper，另一种是类名中含有 Decorator。

动态代理：

> 切面在应用运行的时刻被织入。一般情况下，在织入切面时，AOP容器会为目标对象创建动态的创建一个代理 对象。SpringAOP就是以这种方式织入切面的。 
>
> 织入：把切面应用到目标对象并创建新的代理对象的过程。

观察者模式：

> spring的事件驱动模型使用的是 观察者模式 ，Spring中Observer模式常用的地方是listener的实现。

策略模式：

> Spring框架的资源访问Resource接口。该接口提供了更强的资源访问能力，Spring 框架本身大量使用了 Resource 接口来访问底层资源。

模板方法：父类定义了骨架（调用哪些方法及顺序），某些特定方法由子类实现。

最大的好处：代码复用，减少重复代码。除了子类要实现的特定方法，其他方法及方法调用顺序都在父类中预先写好了。

> refresh方法

