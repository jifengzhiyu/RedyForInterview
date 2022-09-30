# 三、 Spring底层篇

## 什么是Spring?

Spring:是一个企业级java应用框架，他的作用主要是简化软件的开发以及配置过程，简化项目部署环境。Spring是一个轻量级的控制反转（IoC)和面向切面（AOP）的容器框架。
-包含并管理应用对象(Bean)的配置和生命周期，这个意义上是一个容器。
-将简单的组件配置、组合成为复杂的应用，这个意义上是一个框架。

Spring jar包：

![image-20220928185658156](Pic/image-20220928185658156.png)

Spring的优点：
1、Spring低侵入设计，对业务代码的污染非常低。（大多基于注解，不需要依赖于spring底层基础的类）
2、Spring的DI机制将对象之间的关系交由框架处理，减少组件的耦合。
3、Spring提供了AOP技术，支持将一些通用的功能进行集中式管理，从而提供更好的复用。（配置安全权限）
4、Spring对于主流框架提供了非常好的支持。

## 谈谈你对IOC和AOP的理解

IOC就是控制反转，指创建对象的控制权转移给Spring来进行管理。简单来说，就是应用不用去new对象了，而全部交由Spring自动生产。
所有的对象只要配置好，在容器启动过程中，就会帮我们把这些对象例化好，把依赖关系给建好，

IOC有三种注入方式：

1、构造器注入

2、setter方法注入

3、根据注解注入。

AOP面向切面。用于将那些与业务无关，但却对多个对象产生影响的公共行为（日志，权限认证）。抽取并封装成一个可重用的模块。AOP的核心就是动态代理。动态代理分为JDK的动态代理和CGLIB动态代理。

## 谈谈你对AOP的理解

系统是由许多不同的组件所组成的，每一个组件各负责一块特定功能。除了实现自身核心功能之外，这些组件还经常承担着额外的职责。例如日志、事务管理和安全这样的非核心服务经常融入到自身具有核心业务逻辑的组件中去。 这些系统服务经常被称为横切关注点，因为它们会跨越系统的多个组件。
当我们需要为分散的对象引入公共行为的时候，OOP（面向对象）则显得无能为力。也就是说，OOP允许你定义从上到下的关系，但并不适合定义从左到右的关系。例如在支付接口写打印日志的代码，订单接口也是。
日志代码往往水平地散布在所有对象层次中，而与它所散布到的对象的核心功能毫无关系。
在OOP设计中，它导致了大量代码的重复，而不利于各个模块的重用。
AOP:将程序中的交叉业务逻辑（比如安全，日志，事务等），封装成一个切面，然后注入到目标对象（具体业务逻辑)中去。AOP可以对某个对象或某些对象的功能进行增强，比如对象中的方法进行增强，可以在执行某个方法前后额外的做一些事情。一个接口里面没有写打印日志逻辑，但是自动拥有打印日志功能。

## 谈谈你对IOC的理解

IOC表示控制反转

ioc容器：实际上就是个map（key，value），里面存的是各种对象（在xml里配置的bean节点、@repository、@service、@controller、@component），在项目启动的时候会读取配置文件里面的bean节点，根据全限定类名使用反射创建对象放到map里、扫描到打上上述注解的类还是通过反射创建对象放到map里。

这个时候map里就有各种对象了，接下来我们在代码里需要用到里面的对象时，再通过DI注入（autowired、resource等注解，xml里bean节点内的ref属性，项目启动的时候会读取xml节点ref属性根据id注入，也会扫描这些注解，根据类型或id注入；id就是对象名）。

什么是控制反转：

我们在用Spring的时候，我们需要做什么：
1.建一些类，比如UserService、OrderService
2.用一些注解，比如@Autowired
但是，我们也知道，当程序运行时，用的是具体的UserService对象、OrderService对象，那这些对象是什么时候创建的？谁创建的？包括对象里的属性是什么时候赋的值？ 谁赋的？我们只是写了类而已，所有的这些都是Spring做的，它才是幕后黑手。

这就是控制：
1.控制对象的创建 
2.控制对象内属性的赋值
如果我们不用Spring,那我们得自己来做这两件事，反过来，我们用Spring,这两件事情就不用我们做了，我们要做的仅仅是定义类，以及定义哪些属性需要Spring来赋值 (比如某个属性上加@Autowired)，这就是反转，表示一种对象控制权的转移。

优点：

如果我们自己来负责创建对象，自己来给对象中的属性赋值，会出现什么情况？
比如，现在有三个类：
1.A类，A类里有一个属性c;
2.B类，B类里也有一个属性c;
3.C类
现在程序要运行，这三个类的对象都需要创建出来，并且相应的属性都需要有值，那么除开定义这三个类之外，我们还得写：
1.A a = new A();
2.B b= new B();
3.C c= new C();
4.a.c =c;
5.b.c=c;
这五行代码是不用Spring的情况下多出来的代码，而且，如果类再多一些，类中的属性再多一些，那相应的代码会更多，而且代码会更复杂。所以我们可以发现，我们自己 来控制比交给Spring来控制，我们的代码量以及代码复杂度是要高很多的，反言之，将对象交给Spring来控制，减轻了程序员的负担
总结一下，lOC表示控制反转，表示如果用Spring,那么Spring会负责来创建对象，以及给对象内的属性赋值，也就是如果用Spring,那么对象的控制权会转交给Spring.



全部对象的控制权全部上缴给“第三方”IOC容器，所以，IOC容器成了整个系统的关键核心，它起到了一种类似“粘合剂”的作用，把系统中的所有对象粘合在一起发挥作用，如果没有这个“粘合剂”，对象与对象之间会彼此失去联系，这就是有人把IOC容器比喻成“粘合剂”的由来。

依赖注入：

“获得依赖对象的过程被反转了”。控制被反转之后，获得依赖对象的过程由自身管理变为了由IOC容器主动注入。依赖注入是实现IOC的方法，就是由IOC容器在运行期间，动态地将某种依赖关系注入到对象之中。

## 解释下Spring支持的几种bean的作用域

singleton（单例模式）：默认，每个**容器**中只有一个bean的实例，单例的模式由BeanFactory自身来维护。该对象的生命周期是与Spring IOC容器一致的（但在第一次被注入时才会创建）。

prototype（原型模式）：为每一个bean请求（getbean）提供一个实例。在每次注入时都会创建一个新的对象，容器中有多个实例

request：bean被定义为在每个HTTP请求中创建一个单例对象，也就是说在单个请求中都会复用这一个单例对象。请求结束后该实例对象就会被回收。

session：与request范围类似，确保每个session中有一个bean的单例对象，在session过期后，bean会随之失效。

application：bean被定义为在容器上下文ServletContext的生命周期中复用一个单例对象。

websocket：bean被定义为在websocket的生命周期中复用一个单例对象。

## Spring事务的实现方式和原理以及隔离级别？

在使用Spring框架时，可以有两种使用事务的方式，一种是编程式（手动写代码），一种是申明式（使用注解），@Transactional注解就是申明式的。

首先，事务这个概念是数据库层面的，Spring只是基于数据库中的事务进行了扩展，以及提供了一些能让程序员更加方便操作事务的方式。

比如我们可以通过在某个方法上增加@Transactional注解，就可以开启事务，这个方法中所有的sql都会在一个事务中执行，统一成功或失败。

在一个方法上加了@Transactional注解后，Spring会基于这个类生成一个代理对象，会将这个代理对象作为bean放在IOC容器中，当在使用这个代理对象的方法时，如果这个方法上存在@Transactional注解，那么代理逻辑会先把事务的自动提交设置为false，然后再去执行原本的业务逻辑方法，如果执行业务逻辑方法没有出现异常，那么代理逻辑中就会将事务进行提交，如果执行业务逻辑方法出现了异常，那么则会将事务进行回滚。

当然，针对哪些异常回滚事务是可以配置的，可以利用@Transactional注解中的rollbackFor属性进行配置，默认情况下会对RuntimeException和Error进行回滚。

spring事务隔离级别就是数据库的隔离级别：外加一个默认级别

- read uncommitted（未提交读）
- read committed（提交读、不可重复读），Oracle默认
- repeatable read（可重复读），mysql默认
- serializable（可串行化）

> 数据库的配置隔离级别是Read Commited,而Spring配置的隔离级别是Repeatable Read，请问这时隔离 级别是以哪一个为准？ 以Spring配置的为准，如果spring设置的隔离级别数据库不支持，效果取决于数据库

## spring事务传播机制

多个事务方法相互调用时,事务如何在这些方法间传播

a方法调用b方法，a、b方法里面都有事务，这两个事务如何运行（a回滚，b回滚吗）

> 方法A是一个事务的方法，方法A执行过程中调用了方法B，那么方法B有无事务以及方法B对事务的要求不同都 会对方法A的事务具体执行造成影响，同时方法A的事务对方法B的事务执行也有影响，这种影响具体是什么就 由两个方法所定义的事务传播类型所决定。

下面都是@Transactional的枚举参数，修饰被调用者

REQUIRED(Spring默认的事务传播类型)：如果调用方没有事务，则被调用方新建一个事务，如果调用方存在事务，则加入这个事务。a调用b，a没有事务，b新建事务

SUPPORTS：调用方存在事务，则加入调用方事务，如果调用方没有事务，就以非事务方法执行。

MANDATORY：调用方存在事务，则加入当前事务，如果调用方事务不存在，则抛出异常。

REQUIRES_NEW：创建一个新事务，如果存在调用方事务，则挂起该事务。各自处理各自的事物，需要回滚只回滚自己

NOT_SUPPORTED：以非事务方式执行,如果调用方存在事务，则挂起当前事务

NEVER：都不使用事务，如果调用方事务存在，则抛出异常

NESTED：如果调用方事务存在，则在嵌套事务中执行，a是父事务，b是子事物，否则REQUIRED的操作一样（开启一个事务）

> 和REQUIRES_NEW的区别 
>
> REQUIRES_NEW是新建一个事务并且新开启的这个事务与原有事务无关，而NESTED则是当前存在事务时（我 们把当前事务称之为父事务）会开启一个嵌套事务（称之为一个子事务）。 在NESTED情况下父事务回滚时， 子事务也会回滚，而在REQUIRES_NEW情况下，原有事务回滚，不会影响新开启的事务。 
>
> 和REQUIRED的区别 
>
> REQUIRED情况下，调用方存在事务时，则被调用方和调用方使用同一事务，那么被调用方出现异常时，由于共用一个事务，所以无论调用方是否catch其异常，事务都会回滚。而在NESTED情况下，被调用方发生异常时，调用方可以catch其异常，这样只有子事务回滚，父事务不受影响；父事务回滚，子事务一定回滚。

## Spring中什么时候@Transactional会失效（事务失效）

sprig事务的原理是AOP,进行了切面增强，那么失效（方法上加的@Transactional失效）的根本原因是这个AOP不起作用了，常见情况有如下几种
1、【重要】发生自调用，类里面使用this调用本类的方法(this通常省略)，此时这个this对象不是代理类，而是UserService对象本身，就没有代理逻辑来处理这个@Transactional注解，该注解就会失效。 解决方法很简单，让那个this变成UserService的代理类即可！

不是由Spring事务的代理对象执行该方法

2、方法不是public的：@Transactional只能用于public的方法上，否则事务不会失效，如果要用在非public方法上，可以开启AspectJ代理模式。同时如果某个方法是privatel的，那么@Transactionali也会失效，因为底层cglib是基于父子类来实现的，子类是不能重载父类的private方法的，所以无法很好的利用代理，也会导致@Transactianals失效

 3、数据库不支持事务 

4、bean没有被spring管理

5、异常被吃掉，事务不会回滚或者抛出的异常没有被定义，默认为RuntimeException

## 什么是bean的自动装配，有哪些方式？

- 开启自动装配，只需要在xml配置文件中定义“autowire”属性。

```xml
<bean id="cutomer" class="com.xxx.xxx.Customer" autowire="" />
```

- autowire属性有五种装配的方式：

  -  缺省情况下，自动配置是通过“ref”属性手动设定

    ```
    手动装配：以value或ref的方式明确指定属性值都是手动装配。 
    需要通过‘ref’属性来连接bean。
    ```

    - byName-根据bean的属性名称进行自动装配。

      ```xml
      Cutomer的属性名称是person，Spring会将bean id为person的bean通过setter方法进行自动装配。
      <bean id="cutomer" class="com.xxx.xxx.Cutomer" autowire="byName"/> 
      <bean id="person" class="com.xxx.xxx.Person"/>
      ```

  - byType-根据bean的类型进行自动装配。

    ```xml
    Cutomer的属性person的类型为Person，Spirng会将Person类型通过setter方法进行自动装配。
    <bean id="cutomer" class="com.xxx.xxx.Cutomer" autowire="byType"/>
    <bean id="person" class="com.xxx.xxx.Person"/>
    ```

  - constructor-类似byType，不过是应用于构造器的参数。如果一个bean与构造器参数的类型形同，则进行自动装配，否则导致异常。

    ```xml
    Cutomer构造函数的参数person的类型为Person，Spirng会将Person类型通过构造方法进行自动装配。
    <bean id="cutomer" class="com.xxx.xxx.Cutomer" autowire="construtor"/>
    <bean id="person" class="com.xxx.xxx.Person"/>
    ```

  - autodetect-如果有默认的构造器，则通过constructor方式进行自动装配，否则使用byType方式进行自动装配。

    ```xml
    如果有默认的构造器，则通过constructor方式进行自动装配，否则使用byType方式进行自动装配。
    ```

@Autowired注解方式自动装配bean，可以在字段、setter方法、构造函数上使用。

## 描述一下Spring Bean的生命周期？

1、解析类得到BeanDefinition

2、如果有多个构造方法，则要推断构造方法

3、确定好构造方法后，进行实例化得到一个对象

4、对对象中的加了@Autowired注解的属性进行属性填充

5、处理回调Aware接口，比如BeanNameAware，BeanFactoryAware

6、初始化前，调用BeanPostProcessor的初始化前的方法，@PostConstruct注解后的方法在BeanPostProcessor前置处理器中就被执行了。

7、调用初始化方法，如果bean的类实现了InitializingBean接口，就处理该接口

8、初始化后，进行AOP

9、如果当前创建的bean是单例的则会把bean放入单例池

10、使用bean

11、Spring容器关闭时调用DisposableBean中destory()方法

## Spring中Bean是线程安全的吗？

Spring中的Bean默认是单例模式，并没有针对Bean做线程安全的处理，所以： 

1, 如果Bean是无状态的，那么Bean则是线程安全的 

2,如果Bean是有状态的（bean内部属性会变化），那么Bean则不是线程安全的
最简单的办法就是改变bean的作用域把 "singleton"改为’‘protopyte’ 这样每次请求Bean就相当于是 new Bean() 这样就可以保证线程的安全了。
不要在bean中声明任何有状态的实例变量或类变量，如果必须如此，那么就使用ThreadLocal把变量变为线程私有的，如果bean的实例变量或类变量需要在多个线程之间共享，那么就只能使用synchronized、lock、CAS等这些实现线程同步的方法了。

另外，Bean是不是线程安全，跟Bean的作用域没有关系，Bean的作用域只是表示Bean的生命周期范围，对于任何生命周期的Bean都一个对象，这个对象是不是线程安全的，还是得看这个Bean对象本身的各种方法属性是不是线程安全（方法有没有加锁等）。

## ApplicationContext和BeanFactory有什么区别

BeanFactory是Spring中非常核心的组件，表示Bean工厂，可以生成Bean,维护Bean,而ApplicationContext接口继承了BeanFactory接口,所以ApplicationContext拥有BeanFactory 所有的特点，也是一个Bean工厂，但是ApplicationContext除开继承了BeanFactory,之外，还继承了诸如EnvironmentCapable、MessageSource、ApplicationEventPublisher、ResourcePatternResolver接口，从而ApplicationContext还有获取系统环境变量（获取操作系统、jvm环境变量的键值、property文件的键值）、国际化、事件发布、资源解析器（获取网络的某个网页、文系统的某个文件）功能，这是BeanFactory)所不具备的

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

## 如何实现一个IOC容器

1、配置文件配置包扫描路径

2、递归包扫描获取.class文件

3、反射、确定需要交给IOC管理的类

4、对需要注入的类进行依赖注入

- 配置文件中指定需要扫描的包路径
- 定义一些注解，分别表示访问控制层、业务服务层、数据持久层、依赖注入注解、获取配置文件注解
- 从配置文件中获取需要扫描的包路径，获取到当前路径下的文件信息及文件夹信息，我们将当前路径下所有以.class结尾的文件添加到一个Set集合中进行存储
- 遍历这个set集合，获取在类上有指定注解的类，并将其交给IOC容器，定义一个安全的Map用来存储这些对象
- 遍历这个IOC容器，获取到每一个类的实例，判断里面是有有依赖其他的类的实例，然后进行递归注入