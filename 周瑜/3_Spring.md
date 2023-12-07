# 3_Spring

## 什么是Spring?

- Spring是一个企业级java应用框架，作用主要是简化软件的**开发以及配置，简化项目部署环境**。 Spring是一个轻量级的控制反转（IOC）和面向切面（AOP）的**容器框架**。
    - 包含并管理Bean的配置和生命周期，这个意义上是一个容器。
    - 将简单的组件配置、组合成为复杂的应用，这个意义上是一个框架。
- Spring jar包：

![image-20220928185658156](Pic/image-20220928185658156.png)

## Spring优点

1. Spring**低侵入设计**，对业务代码的污染非常低。（大多基于注解）
2. **Spring的DI机制**（依赖注入）将对象**之间的关系**交由框架处理，减少组件的耦合。
3. Spring提供AOP技术，支持将一些通用的功能进行**集中式管理**，更好的复用。（配置安全权限）
4. Spring对于主流框架提供了非常好的支持。
6. 方便进行事务操作
7. 降低 API 开发难度

## AOP理解

- AOP思想，**面向切面**，不修改源代码，在主干功能里添加新功能，底层使用**动态代理**（JDK动态代理、CGLIB动态代理）
- 解释：
    - **系统**是由许多不同组件组成，每个组件各负责特定功能，还经常承担着额外的职责，如**日志、事务管理和安全**
    - OOP（面向对象）显得无能为力，导致代码重复，不利于各个模块重用。OOP允许定义**从上到下**关系，不适合定义从**左到右**关系。如日志代码往往水平地散布在所有对象层次中。

- 使用逻辑：AOP将程序中的交叉业务逻辑（比如安全，日志，事务等），封装成一个**切面**，注入到目标对象（具体业务逻辑）中。
- 实现效果：如一个接口里面没有写打印日志逻辑，但是自动拥有打印日志功能。
- 几个术语：
    - 连接点：类里面哪些方法可以被增强，这些方法称为连接点
    - 切入点：实际被真正增强的方法，称为切入点
    - 通知（增强）：实标增强的逻辑部分称为通知（增强）
        - 前置通知（对应的注解是@Before）：在目标方法运行之前运行
        - 后置通知（对应的注解是@After）：在目标方法运行结束之后运行，无论目标方法是正常结束还是异常结束都会执行
        - 返回通知（对应的注解是@AfterReturning）：在目标方法正常返回之后运行
        - 异常通知（对应的注解是@AfterThrowing）：在目标方法运行出现异常之后运行
        - 环绕通知（对应的注解是@Around）：在目标方法的前后都能做增强功能；控制目标方法是否执行；通过返回值修改目标方法的执行结果
    - 切面：把通知应用到切入点过程。

## IOC理解

- IOC思想，控制反转。

- IOC思想基于 IOC 容器，IOC容器是个map，里面存储各种对象。

    - 项目启动时读取配置文件里的bean节点、@repository、@service、@controller、@component注解的类，**根据全类名使用反射创建对象放到map里**。
    -
    依赖注入时读取xml里bean节点内的ref属性根据id注入，也会扫描autowired、resource等注解的属性，根据类型或id对象名注入，即IOC容器自动装配，也是依赖注入，对IOC容器中各个组件的依赖关系赋值，在创建类的过程中给属性赋值。

- 什么是控制反转：

    - 对象控制权的转移，把**创建对象及给属性赋值**的控制权转移给Spring。 我们要做的仅仅是定义类，以及定义哪些**属性**需要Spring来赋值 (比如某个属性上加@Autowired)

    - 优点：减轻了程序员的负担，耦合度降低。

## Spring支持的几种bean的作用域

等同于@Scope注解取值

1. singleton（单例模式）：默认，每个IOC**容器**中只有一个bean的实例，在IOC容器启动后实例化。
2. prototype（原型模式）：IOC容器启动时不会实例化，每次从Spring容器中获取组件对象，都会创建一个新的实例对象并返回，多实例的bean在容器关闭时不销毁。
3. request：每个HTTP请求中有一个bean的单例对象，在HTTP请求结束后，bean会随之失效。
4. session：每个session中有一个bean的单例对象，在session过期后，bean会随之失效。
5. application：不管应用程序中有多少个Spring容器，这个应用程序中**同名的bean**只有一个。

## Spring事务的实现方式和原理？

- Spring框架，有两种使用事务的方式，**一种是编程式（手动写代码），一种是申明式（使用注解）**。

- 事务这个概念在数据库层面，Spring基于数据库中的事务进行了扩展，能更方便操作事务。

- 使用：

    - 在某个方法上增加@Transactional注解，就可以开启事务，这个方法中所有的sql都会在一个事务中执行，要么都成功要么都失败。

- 实现：

    1. Spring基于这个类生成一个**代理对象**，将这代理对象作为bean放在IOC容器中
    2. 调用代理对象的方法时，如果方法上存在@Transactional注解，**代理逻辑**会先把**数据库连接**的自动提交设置为false，然后执行业务逻辑方法

    - 如果执行业务逻辑方法没有出现异常，那么代理逻辑中就会将事务进行提交，如果执行业务逻辑方法出现了异常，会将事务回滚。
    - 针对哪些异常回滚事务是可以配置的，可以利用@Transactional注解中的**rollbackFor**属性进行配置，默认情况下会对RuntimeException和Error进行回滚。

## Spring事务的隔离级别

spring事务隔离级别就是数据库的隔离级别：外加一个默认级别（不同数据库类型自己的默认级别）

- read uncommitted（未提交读）
- read committed（提交读、不可重复读），Oracle默认
- repeatable read（可重复读），mysql默认
- serializable（可串行化）

> 数据库的配置隔离级别是Read Commited,而Spring配置的隔离级别是Repeatable Read，请问这时隔离级别是以哪一个为准？ 以Spring配置的为准，如果spring设置的隔离级别数据库不支持，效果取决于数据库

## Spring中什么时候@Transactional失效（事务失效）

spring事务的原理是AOP，进行了切面增强，那么失效（方法上加的@Transactional失效）的根本原因是这个AOP不起作用了，常见情况：

1. 【重要】发**生自调用**： 使用this调用本类的方法(this通常省略)，此时这个this对象不是**代理对象**，没有代理逻辑来处理@Transactional注解，该注解就会失效。 解法：让this变成代理对象。

2. **@Transactional用于非public的方法上;**
    - 如果要用在非public方法上，可以开启**AspectJ代理模式**。
    - 如果某个方法是private，底层cglib是基于父子类来实现的，子类是不能**重写**父类的private方法，所以无法很好利用代理对象
3. 数据库不支持事务
4. bean没有被spring管理
5. 异常被吃掉，事务不会回滚 或者 抛出的异常没有被定义

## 什么是bean的自动装配，有哪些方式？

- no：xml配置
- byName：根据属性名称 （`@Resource`）
- byType：根据属性类型（`@Autowired`）
- constructor：根据构造函数匹配（根据构造参数类型）

## Spring Bean的生命周期？

简化版：

1. 通过构造器创建 bean 实例（无参数构造器）
2. 为 bean 的属性赋值
3. 把bean实例传递bean**后置处理器**的方法 postProcessBeforelnitialization
4. 调用 bean 的初始化的方法（需要进行配置初始化的方法）
5. 把bean实例传递bean后置处理器的方法 postProcessAfterlnitialization
6. bean 可以使用了（对象获取到了）
7. 当**容器关闭**时候，调用 bean 的销毁的方法（需要配置销毁的方法）

复杂版：（先不背）

> 自定义的组件要想使用Spring容器底层的一些组件，比如ApplicationContext（IOC容器）、底层的BeanFactory等等，那么只需要让自定义组件实现XxxAware接口即可。此时，Spring在创建对象的时候，会调用XxxAware接口中定义的方法注入相关的组件。
>
> [Spring会调用setApplicationContext()方法，并且会将ApplicationContext对象传入到setApplicationContext()方法中，我们只需要在Dog类中定义一个ApplicationContext类型的成员变量来接收setApplicationContext()方法中的参数，那么便可以在Dog类的其他方法中使用ApplicationContext对象了。]
>
> 通过ApplicationContextAware接口我们可以获取到IOC容器。

1. 实例化得到一个对象
2. 设置对象属性
3. **检查Aware的相关接口并设置相关依赖**
4. 初始化前，调用BeanPostProcessor的**初始化前的方法**，其中执行@PostConstruct注解的方法。
5. 调用初始化方法，如果bean的类实现了**InitializingBean**接口，就处理该接口；如果配置自定义的 init-method，就执行init-method指定的方法
6. 初始化后，调用BeanPostProcessor的**初始化后的方法**，进行AOP，后置处理器会判断该bean是否注册了切面，若是，则生成代理对象注入到容器中；
7. 注册Destruction相关回调接口，为了方便对象的销毁
8. 使用bean
9. Spring容器关闭时，如果实现了即调用**DisposableBean**接口中destory()方法，多例bean销毁不会调用该接口方法，多实例bean的生命周期不归Spring容器来管理。
10. 如果配置自定义的destory-method就调用。

## Spring中Bean线程安全吗？

- Spring中的Bean默认是单例模式，并没有针对Bean做线程安全的处理，所以：

1. 如果Bean是无状态（bean**内部属性**不会变化）的，那么Bean则是线程安全的

2. 如果Bean是有状态的，那么Bean则不是线程安全的

   ​ 做法：

    - 改变**bean的作用域**，把 "singleton"改为"protopyte"，这样每次请求Bean就相当于是 new Bean() 。
    - 不在bean中声明任何有状态的**实例变量或类变量**，如果必须如此，那么就使用**ThreadLocal**把变量变为**线程私有**
      ，如果bean的实例变量或类变量需要在多个线程之间共享，那么就只能使用synchronized、lock、CAS等线程同步方法了。

- 另外，Bean是不是线程安全，跟Bean的作用域没有关系，得看这个Bean对象本身的各种方法属性是不是线程安全（方法有没有加锁等）。

## Spring容器启动流程

1. 扫描得到所有的BeanDefinition对象，存在一个Map中
2. 筛选出**非懒加载的单例**利用BeanDefinition创建Bean，对于多例Bean会在每次获取Bean时利用BeanDefinition去创建
3. 利用BeanDefinition创建Bean就是Bean的创建生命周期，这期间包括了**合并BeanDefinition**、**推断构造方法**、实例化、属性注入、初始化前、初始化、初始化后等步骤 其中AOP发生在初始化后
4. 非懒加载的单例Bean创建完了之后，Spring会发布一个**容器启动事件**，表示Spring启动结束

后面先不背

1.
在源码中会更复杂，比如源码中会提供一些模板方法，让子类来实现，比如源码中还涉及到一些BeanFactoryPostProcessor和BeanPostProcessor的注册，Spring的扫描就是通过BenaFactoryPostProcessor来实现的，依赖注入就是通过BeanPostProcessor来实现的
2. 在Spring启动过程中还会去处理@Import等注解

---

## Spring框架中设计模式？

- 工厂模式：BeanFactory
- 原型模式：bean的**作用域**中有prototype（原型模式），为每一个getbean请求提供一个实例。
- 单例模式：bean的作用域中默认每个IOC**容器**中只有一个bean的实例。
- 构造器模式：BeanDefinitionBuilder（BeanDefinition构造器）
- 适配器模式：AdvisorAdapter（把Advisor适配成MethodInterceptor）
- 访问者模式：PropertyAccessor（属性访问器，用来访问和设置某个对象的某个属性）
- 装饰器模式：BeanWrapper （比单纯的Bean对象功能更加强大）
- 代理模式：生成了代理对象的地方就用到了代理模式，有AOP、@Configuration、@Lazy
- 观察者模式：ApplicationListener（事件监听机制）
- 策略模式：BeanNameGenerator （beanName生成器）
- 模板方法模式：AbstractApplicationContext（onRefresh() 子类可以做一些额外的初始化）
- 责任链模式：DefaultAdvisorChainFactory（负责构造一条AdvisorChain,代理对象执行某个方法时会依次经过AdvisorChain中的每个Advisor）

简单工厂：由一个工厂类根据传入的参数，动态决定应该创建哪一个产品类。

> Spring中的**BeanFactory**就是简单工厂模式的体现，根据传入一个唯一的标识来获得Bean对象，但是否是在传入参数后创建还是传入参数前创建这个要根据具体情况来定。

工厂方法：

> - 一般情况下，Spring是通过反射机制利用bean的class属性指定实现类来实例化bean的。在某些情况下，实例化bean过程比较复杂，Spring为此提供了一个FactoryBean的工厂类接口，用户可以通过实现该接口定制实例化bean的逻辑。
> - spring会在使用getBean()调用获得该bean时，会自动调用该bean的getObject()方法，所以返回的不是factory这个bean，而是这个 bean.getOjbect()方法的返回值。

单例模式：保证一个类仅有一个实例，并提供一个访问它的全局访问点

> spring对单例的实现： spring中的单例模式完成了后半句话，即提供了全局的访问点BeanFactory。但没有从构造器级别去控制单例，这是因为spring管理的是任意的java对象。

适配器模式：

> Spring定义了一个适配接口，使得每一种Controller有一种对应的适配器实现类，让适配器代替 controller执行相应的方法。这样在扩展Controller时，只需要增加一个适配器类就完成了SpringMVC 的扩展了。

装饰器模式：动态地给一个对象添加一些额外的职责。就增加功能来说，Decorator模式相比生成子类更为灵活。

> Spring中用到的包装器模式在类名上有两种表现：一种是类名中含有Wrapper，另一种是类名中含有 Decorator。

动态代理：

> 切面在应用运行的时刻被织入。一般情况下，在织入切面时，AOP容器会为目标对象创建动态地创建一个代理对象。SpringAOP就是以这种方式织入切面的。
>
> 织入：把切面应用到目标对象并创建新的代理对象的过程。

观察者模式：

> spring的事件驱动模型使用的是观察者模式 ，Spring中Observer模式常用的地方是listener的实现。

策略模式：

> Spring框架的资源访问Resource接口。该接口提供了更强的资源访问能力，Spring 框架本身大量使用了 Resource 接口来访问底层资源。

模板方法：父类定义了骨架（调用哪些方法及顺序），某些特定方法由子类实现。

最大的好处：代码复用，减少重复代码。除了子类要实现的特定方法，其他方法及方法调用顺序都在父类中预先写好了。

> refresh方法

## 如何实现一个IOC容器

1. 配置文件中指定需要扫描的包路径
2. 将当前路径下所有以.class结尾的文件添加到一个Set集合中存储
3. 遍历这个set集合，反射获取在类上有指定注解类的对象，并将其交给IOC容器，定义Map用来存储这些对象
4. 对需要注入的类进行**依赖注入**

下面看看

- Spring IOC容器在启动的时候，会先保存所有注册进来的bean的定义信息，BeanFactory就会按照这些bean的定义信息来为我们创建对象。 两种方式来编写这些bean的定义信息： 使用XML配置文件的方式来注册bean；
  使用注解来注册bean。
- 当IOC容器中有保存一些bean的定义信息的时候，它便会在合适的时机来创建这些bean，而且主要有两个合适的时机，分别如下： 就是在用到某个bean的时候。 统一创建所有剩下的单实例bean的时候。
- BeanPostProcessor（即后置处理器）。在Spring中其实是有非常多的后置处理器的，它们一般都是在我们bean初始化前后进行逻辑增强的。
- Spring的事件驱动模型。它涉及到了两个元素：
    - ApplicationListener：它是用来做事件监听 ApplicationEventMulticaster：事件派发器。它就是来帮我们进行事件派发的

## spring事务传播机制

> 下面都是@Transactional的7个枚举参数，修饰被调用者（方法B）

a()调用b()时

- ‼️REQUIRED（**required**）(Spring默认的事务传播类型)：
    - a()没有事务，则b()新建事务；
    - a()有事务，则b()加入a()的事务，a()b()有异常就回滚。
- ‼️SUPPORTS（**supports**）：
    - a()没有事务，则b()也没有事务；
    - a()有事务，则b()加入a()的事务。
- MANDATORY（**mandatory**）：
    - a()没有事务，则b()抛出异常；
    - a()有事务，则b()加入a()的事务。
- ‼️REQUIRES_NEW（**requires-new**）：
    - **b()创建事务**，若a()有事务则挂起该事务；各自处理各自的事物，需要回滚只回滚自己
- NOT_SUPPORTED（**not-supported**）：
    - **b()没有事务**，若a()有事务则挂起该事务
- NEVER（**never**）：
    - 都不使用事务，如果a()有事务，则抛出异常
- NESTED（**nested**）：
    - a()有事务，则在嵌套事务中执行，a是父事务，b是子事务；父事务回滚时， 子事务也会回滚；子事物回滚时，父事务可以catch其异常
    - a()没有事务，则b()新建事务

## ApplicationContext和BeanFactory有什么区别

- BeanFactory是Spring中非常核心的组件，可以生成并维护Bean
- 而ApplicationContext接口继承了BeanFactory接口，它还继承了以下接口
    - EnvironmentCapable 获取**系统环境变量**（获取操作系统、jvm环境变量的键值对、property文件的键值对）、
    - MessageSource 国际化、
    - ApplicationEventPublisher **事件发布**、
    - ResourcePatternResolver **资源解析器**（获取网络的某个网页、系统的某个文件）
