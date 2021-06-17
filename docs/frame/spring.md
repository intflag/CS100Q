# Spring
### 参考资料
- [田小波的技术博客-Spring 系列文章](http://www.tianxiaobo.com/categories/java-framework/spring/)
- [一文告诉你Spring是如何利用"三级缓存"巧妙解决Bean的循环依赖问题的](https://cloud.tencent.com/developer/article/1497692)
- [请别再问Spring Bean的生命周期了](https://www.jianshu.com/p/1dec08d290c1)
- [Spring Bean的生命周期（非常详细）](https://www.cnblogs.com/zrtqsk/p/3735273.html)
- [Spring Bean 作用域](https://www.w3cschool.cn/wkspring/nukv1ice.html)
- [Spring总结以及在面试中的一些问题](https://www.cnblogs.com/wang-meng/p/5701982.html)
- [spring(基础20) threadLocal在spring框架中的运用](https://blog.csdn.net/zengdeqing2012/article/details/77098994)
- [Spring 中使用自定义的 ThreadLocal 存储导致的坑](https://juejin.cn/post/6844903847916371981)
- [浅析Spring中AOP的实现原理—动态代理](https://www.cnblogs.com/tuyang1129/p/12878549.html)
- [spring 事务原理](https://xie.infoq.cn/article/52f38883e28821c9cf0a608ea)
- [可能是最漂亮的Spring事务管理详解](https://juejin.cn/post/6844903608224333838)

## Spring 架构以及设计思想
### Spring 架构
![](http://images.intflag.com/spring000.jpg)

- 核心容器
  - spring-core 和 spring-beans 提供框架的基础部分，包括 IOC 功能，BeanFactory 是一个复杂的工厂模式的实现，将配置和特定的依赖从实际程序逻辑中解耦；
  - context 模块建立在 core 和 beans 模块的基础上，增加了对国际化的支持、事件广播、资源加载和创建上下文，ApplicationContext 是 context 模块的重点；
  - spring-context-support 提供对常见第三个库的支持，集成到 spring 上下文中，比如缓存(ehcache,guava)、通信(javamail)、调度(commonj,quartz)、模板引擎等(freemarker,velocity)；
  - spring-expression 模块提供了一个强大的表达式语言用来在运行时查询和操作对象图，这种语言支持对属性值、属性参数、方法调用、数组内容存储、集合和索引、逻辑和算数操作及命名变量，并且通过名称从spring的控制反转容器中取回对象。
- AOP和服务器工具
  - spring-aop 模块提供面向切面编程实现，单独的 spring-aspects 模块提供了 aspectj 的集成和适用；
  - spring-instrument 提供一些类级的工具支持和 ClassLoader 级的实现，用于服务器；
  - spring-instrument-tomcat 针对 tomcat 的 instrument 实现。
- 消息组件
  - spring 框架4包含了spring-messaging模块，从 spring 集成项目中抽象出来，比如 Messge、MessageChannel、MessageHandler 及其他用来提供基于消息的基础服务；
- 数据访问/集成
  - spring-jdbc 模块提供了不需要编写冗长的JDBC代码和解析数据库厂商特有的错误代码的 JDBC 抽象出；
  - spring-tx 模块提供可编程和声明式事务管理；
  - spring-orm 模块提供了领先的对象关系映射API集成层，如 JPA、Hibernate等；
  - spring-oxm 模块提供抽象层用于支持Object/XML maping的实现，如 JAXB、XStream等；
  - spring-jms 模块包含生产和消费消息的功能，从 Spring4.1 开始提供集成 spring-messaging 模块。
- Web
  - spring-web 模块提供了基本的面向 web 开发的集成功能，例如多文件上传、使用 servert listeners 和 web 开发应用程序上下文初始化 IOC 容器。也包含 HTTP 客户端以及spring 远程访问的支持的 web 相关部分；
  - spring-webmvc 包含 spring 的 model-view-controller 和 REST web services 实现的 Web 应用程序；
  - spring-webmvc-portlet 模块提供了 MVC 模式的 portlet 实现，protlet 与 Servlet 的最大区别是请求的处理分为 action 和 render 阶段，在一个请求中，action 阶段只执行一次，但 render 阶段可能由于用户的浏览器操作而被执行多次。

## Spring Bean
### Bean 创建流程
![](http://images.intflag.com/spring002.jpg)

- createBeanInstance：实例化，其实也就是调用对象的构造方法实例化对象；
- populateBean：填充属性，这一步主要是对 bean 的依赖属性进行注入(@Autowired)；
- initializeBean：回调一些形如 initMethod、InitializingBean 等方法；

### Bean 生命周期
![](http://images.intflag.com/spring005.jpg)

#### 扩展点（影响多个 Bean）
- InstantiationAwareBeanPostProcessor 实例化 Bean 后处理器
  - postProcessBeforeInstantiation：Bean 实例化前执行；
  - postProcessAfterInstantiation：Bean 实例后、属性赋值前执行；
- BeanPostProcessor Bean 后处理器
  - postProcessBeforeInitialization：Bean 属性赋值后、Bean 初始化前执行；
  - postProcessAfterInitialization：Bean 初始化前执行；

#### 扩展点（影响单个 Bean）
- Aware
  - Aware Group1
    - BeanNameAware
    - BeanClassLoaderAware
    - BeanFactoryAware
  - Aware Group2
    - EnvironmentAware
    - EmbeddedValueResolverAware
    - ApplicationContextAware(ResourceLoaderAware\ApplicationEventPublisherAware\MessageSourceAware)
- 生命周期
  - InitializingBean
  - DisposableBean

### 三级缓存
```java
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {
	...
	// 从上至下 分表代表这“三级缓存”
	private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256); //一级缓存
	private final Map<String, Object> earlySingletonObjects = new HashMap<>(16); // 二级缓存
	private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16); // 三级缓存
	...
	
	/** Names of beans that are currently in creation. */
	// 这个缓存也十分重要：它表示bean创建过程中都会在里面呆着~
	// 它在Bean开始创建时放值，创建完成时会将其移出~
	private final Set<String> singletonsCurrentlyInCreation = Collections.newSetFromMap(new ConcurrentHashMap<>(16));

	/** Names of beans that have already been created at least once. */
	// 当这个Bean被创建完成后，会标记为这个 注意：这里是set集合 不会重复
	// 至少被创建了一次的  都会放进这里~~~~
	private final Set<String> alreadyCreated = Collections.newSetFromMap(new ConcurrentHashMap<>(256));
}
```

- singletonObjects：用于存放完全初始化好的 bean，从该缓存中取出的 bean 可以直接使用；
- earlySingletonObjects：提前曝光的单例对象的 cache，存放原始的 bean 对象（尚未填充属性），用于解决循环依赖；
- singletonFactories：单例对象工厂的 cache，存放 bean 工厂对象，用于解决循环依赖；

### Bean 循环依赖问题
```java
@Service
public class A {
    @Autowired
    private B b;
}

@Service
public class B {
    @Autowired
    private A a;
}
```
#### 简单理解版本
![](http://images.intflag.com/spring001.jpg)

#### 源码分析版本
![](http://images.intflag.com/spring003.jpg)

- 1）使用 context.getBean(A.class)，旨在获取容器内的单例 A (若 A 不存在，就会走 A 这个 Bean 的创建流程)，显然初次获取 A 是不存在的，因此走 A 的创建之路；
- 2）实例化 A（注意此处仅仅是实例化），并将它放进缓存（此时A已经实例化完成，已经可以被引用了）；
- 3）初始化 A：@Autowired 依赖注入 B（此时需要去容器内获取 B）；
- 4）为了完成依赖注入 B，会通过 getBean(B) 去容器内找B。但此时 B 在容器内不存在，就走向 B 的创建之路；
- 5）实例化 B，并将其放入缓存。（此时 B 也能够被引用了）；
- 6）初始化 B，@Autowired 依赖注入 A（此时需要去容器内获取 A）；
- 7）此处重要：初始化B时会调用 getBean(A) 去容器内找到 A，上面我们已经说过了此时候因为 A 已经实例化完成了并且放进了缓存里，所以这个时候去看缓存里是已经存在A的引用了的，所以 getBean(A) 能够正常返回；
- 8）B 初始化成功（此时已经注入 A 成功了，已成功持有 A 的引用了），return（注意此处 return 相当于是返回最上面的 getBean(B) 这句代码，回到了初始化 A 的流程中）；
- 9）因为 B 实例已经成功返回了，因此最终 A 也初始化成功；
- 10）到此，B 持有的已经是初始化完成的 A，A 持有的也是初始化完成的 B，完美；

### Spring Bean 的作用域
- 通过bean 定义中的scope属性来定义bean的作用阈。
- Spring 框架支持以下五种 bean 的作用域：
    - singleton : 单例类型，在创建容器时就同时自动创建了一个 Bean 的对象，每次获取到的对象都是同一个对象，是 Spring 中的默认配置；
    - prototype：原型类型，它在我们创建容器的时候并没有实例化，而是当我们调用 getBean 获取 Bean 的时候才会去创建一个对象，而且我们每次获取到的对象都不是同一个对象；
    - request：每次 HTTP 请求都会创建一个新的 Bean，该作用域仅适用于 WebApplicationContext环境；
    - session：同一个 HTTP Session 共享一个 Bean，不同 Session 使用不同的 Bean，仅适用于 WebApplicationContext 环境；
    - global-session：在一个全局的 HTTP Session 中，一个 Bean 定义对应一个实例。该作用域仅在基于 Web 的 Spring ApplicationContext 情形下有效；

- 什么时候使用原型模式？
  - 有状态使用单例模式，无状态使用原型模式；

## Spring 线程并发问题
- 只有无状态的 Bean 才可以在多线程环境下共享，有状态的 Bean 会有线程安全问题；
- ThreadLocal 会为每一个线程提供一个独立的变量副本，从而隔离了多个线程对数据的访问冲突；

## Spring AOP
### 概念和作用
- 面向切面编程，可以理解为程序的设计模式；
- 将通用的功能从业务逻辑中抽离出来，使代码解耦，提高代码的灵活性和扩展性；
- 日志记录，性能统计，安全控制，事务处理，异常处理等扩展。

### AOP 术语
- 连接点（Joinpoint）：用于执行拦截器链中的下一个拦截器逻辑，通常情况一个方法调用就是一个连接点；
- 切点（Pointcut）：表示如何选择连接点（在哪里调用），使用 AspectJ 表达式，如：execution(* *.find*(..))；
- 通知（Advice）：表示在何时调用
  - 前置通知（Before advice）：在目标方法调用前执行通知；
  - 后置通知（After advice）：在目标方法完成后、返回前执行通知；
  - 返回通知（After returning advice）：在目标方法执行成功后，调用通知；
  - 异常通知（After throwing advice）：在目标方法抛出异常后，执行通知；
  - 环绕通知（Around advice）：在目标方法调用前后均可执行自定义逻辑；
- 切面（Aspect）：切面只是一个概念，表示对什么方法（where）在何时（when 前置还是后置，或者环绕）执行什么样的横切逻辑（how）
- 织入（Weaving）：织入就是在切点的引导下，将通知逻辑插入到方法调用上，使得我们的通知逻辑在方法调用时得以执行
  - 实现后置处理器 BeanPostProcessor 接口；
  - 在 Bean 初始化完成后，即 Bean 执行完初始化方法（init-method）,Spring 通过切点对 Bean 类中的方法进行匹配,若匹配成功，则会为该 bean 生成代理对象，并将代理对象返回给容器;

### AOP 实现原理
- Spring 默认使用 JDK 的动态代理实现 AOP，类如果实现了接口，Spring 就会使用这种方式实现动态代理；
- 若需要代理的类没有实现接口，此时 JDK 的动态代理将没有办法使用，于是 Spring 会使用 CGLib 的动态代理来生成代理对象；
- [JDK 动态代理、CGLib 动态代理参考这里](programming/designPattern?id=_5、代理模式（结构型）)

## Spring 事务
### PlatformTransactionManager 事务管理器
- 事务管理器接口本身只有三个方法：
  - getTransaction：获取当前激活的事务或者创建一个事务；
  - commit：提交当前事务；
  - rollback：回滚当前事务；
- PlatformTransactionManager 有很多具体的实现：
  - DataSourceTransactionManager，使用 JDBC 与 ibatis 进行持久化
  - JtaTransactionManager：使用分布式事务
  - JpaTransactionManager：使用 jpa
  - HibernateTransactionManager：使用 hibernate

### TransactionDefinition 事务属性定义

### TransactionStatus 事务运行状态


## Spring 中用到了哪些设计模式？
核心概念，作用与优点，专业术语，应用场景
spring的aop是怎么实现的
## SpringMVC 一个请求的流程是什么？
## SpringBoot 启动加载过程?
spring boot和spring的区别？spring boot如何实现自动扫描注入？
## SpringBoot 加载配置文件原理?
## SpringBoot 的自动装配如何实现?
- https://www.codesheep.cn/2018/09/04/springboot-startup-process/

## SpringBoot 优化内部的 Tomcat 怎么做?
## SpringCloud 的组件做一下介绍
## SpringCloud 的服务发现与注册的实现
## spring cloud configration  bus 的 原理