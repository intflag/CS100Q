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
- [Spring 中经典的 9 种设计模式](https://zhuanlan.zhihu.com/p/114244039)
- [SpringBoot自动装配原理分析，手写starter组件](https://www.jianshu.com/p/914df825dc76)
- [SpringBoot 应用程序启动过程探秘](https://www.codesheep.cn/2018/09/04/springboot-startup-process/)
- [淘宝一面：“说一下 Spring Boot 自动装配原理呗？](https://www.cnblogs.com/javaguide/p/springboot-auto-config.html)

## Spring 架构
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

## Spring IOC
### Spring Bean 创建流程
![](http://images.intflag.com/spring002.jpg)

- createBeanInstance：实例化，其实也就是调用对象的构造方法实例化对象；
- populateBean：填充属性，这一步主要是对 bean 的依赖属性进行注入(@Autowired)；
- initializeBean：回调一些形如 initMethod、InitializingBean 等方法；

### Spring Bean 生命周期
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

### Spring 三级缓存
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

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
  // Quick check for existing instance without full singleton lock
  Object singletonObject = this.singletonObjects.get(beanName);
  if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
    singletonObject = this.earlySingletonObjects.get(beanName);
    if (singletonObject == null && allowEarlyReference) {
      synchronized (this.singletonObjects) {
        // Consistent creation of early reference within full singleton lock
        singletonObject = this.singletonObjects.get(beanName);
        if (singletonObject == null) {
          singletonObject = this.earlySingletonObjects.get(beanName);
          if (singletonObject == null) {
            ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
            if (singletonFactory != null) {
              singletonObject = singletonFactory.getObject();
              this.earlySingletonObjects.put(beanName, singletonObject);
              this.singletonFactories.remove(beanName);
            }
          }
        }
      }
    }
  }
  return singletonObject;
}
```

- 1）先从一级缓存 singletonObjects 中去获取，如果获取到就直接 return；
- 2）如果获取不到或者对象正在创建中（isSingletonCurrentlyInCreation()），那就再从二级缓存 earlySingletonObjects 中获取，如果获取到就直接 return；
- 3）如果还是获取不到，且允许 singletonFactories（allowEarlyReference=true）通过 getObject() 获取，就从三级缓存 singletonFactory.getObject() 获取，如果获取到了就从三级缓存 singletonFactories 中移除，并且放进二级缓存 earlySingletonObjects，其实也就是从三级缓存移动到了二级缓存；

### Spring Bean 循环依赖问题
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
  - 在 Bean 初始化完成后，即 Bean 执行完初始化方法（init-method），Spring 通过切点对 Bean 类中的方法进行匹配，若匹配成功，则会为该 bean 生成代理对象，并将代理对象返回给容器;

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

### TransactionDefinition 事务属性
事务属性：
- 隔离级别
- 传播行为
- 任务超时
- 回滚规则
- 是否只读

1）隔离级别

|常量名称 |常量值|常量解释|
|:----|:----:|:----|
|ISOLATION_DEFAULT|-1|使用后端数据库默认的隔离级别，Mysql 默认为可重复读，Oracle 默认为读提交|
|ISOLATION_READ_UNCOMMITTED|1|读未提交，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读|
|ISOLATION_READ_COMMITTED|2|读提交，允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生|
|ISOLATION_REPEATABLE_READ|4|可重复读，对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生|
|ISOLATION_SERIALIZABLE|8|串行化，所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能|

2）传播行为：

|常量名称 |常量值|常量解释|
|:----|:----:|:----|
|PROPAGATION_REQUIRED|0|如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务|
|PROPAGATION_SUPPORTS|1|如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行|
|PROPAGATION_MANDATORY|2|如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常（mandatory：强制性）|
|PROPAGATION_REQUIRES_NEW|3|创建一个新的事务，如果当前存在事务，则把当前事务挂起|
|PROPAGATION_NOT_SUPPORTED|4|以非事务方式运行，如果当前存在事务，则把当前事务挂起|
|PROPAGATION_NEVER|5|以非事务方式运行，如果当前存在事务，则抛出异常|
|PROPAGATION_NESTED|6|如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于 PROPAGATION_REQUIRED|

3）事务超时属性

- 所谓事务超时，就是指一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务；
- 在 TransactionDefinition 中以 TIMEOUT_DEFAULT（int 类型） 的值来表示超时时间，其单位是秒；

4）事务只读属性

- 事务的只读属性是指，对事务性资源进行只读操作或者是读写操作；
- TransactionDefinition 中以 isReadOnly（boolean 类型）来表示该事务是否只读

### TransactionStatus 事务运行状态
```java
public interface TransactionStatus{
    boolean isNewTransaction(); // 是否是新的事物
    boolean hasSavepoint(); // 是否有恢复点
    void setRollbackOnly();  // 设置为只回滚
    boolean isRollbackOnly(); // 是否为只回滚
    boolean isCompleted; // 是否已完成
} 
```
### 事务实现原理
- 通过 AOP 在方法执行前后增加数据库事务的操作;

## Spring 线程并发
- 只有无状态的 Bean 才可以在多线程环境下共享，有状态的 Bean 会有线程安全问题；
- ThreadLocal 会为每一个线程提供一个独立的变量副本，从而隔离了多个线程对数据的访问冲突；

## Spring 设计模式
|设计模式 |相关类|
|:----|:----|
|简单工厂|BeanFactory|
|工厂方法|FactoryBean|
|单例模式|DefaultSingletonBeanRegistry.getSingleton()|
|适配器模式|SpringMVC 中的 HandlerAdatper|
|装饰器模式|xxxWrapper 和 xxxDecorator|
|代理模式|AOP 底层实现|
|观察者模式|ApplicationListener|
|策略模式|Resource 资源访问接口|
|模板方法模式|JdbcTemplate|

## SpringMVC 请求流程
![](http://images.intflag.com/spring006.jpg)

流程说明：

- 1）用户的浏览器发出了一个请求，这个请求经过互联网到达了我们的服务器，Servlet 容器首先接待了这个请求，并将该请求委托给 DispatcherServlet 进行处理；
- 2）接着 DispatcherServlet 将该请求传给了处理器映射组件 HandlerMapping，并获取到适合该请求的拦截器和处理器；
- 3）在获取到处理器后，DispatcherServlet 还不能直接调用处理器的逻辑，需要进行对处理器进行适配；
- 4）处理器适配成功后，DispatcherServlet 通过处理器适配器 HandlerAdapter 调用处理器的逻辑，并获取返回值 ModelAndView；
- 5）之后，DispatcherServlet 需要根据 ModelAndView 解析视图，解析视图的工作由 ViewResolver 完成，若能解析成功，ViewResolver 会返回相应的视图对象 View；
- 6）在获取到具体的 View 对象后，最后一步要做的事情就是由 View 渲染视图，并将渲染结果返回给用户。

组件说明：

|组件 |说明|
|:----|:----|
|DispatcherServlet|前端控制器，BeanFacSpring MVC 的核心组件，是请求的入口，负责协调各个组件工作|
|HandlerMapping|处理器映射器，内部维护了一些 <访问路径, 处理器> 映射，负责为请求找到合适的处理器|
|HandlerAdapter|处理器适配器，Spring 中的处理器的实现多变，比如用户处理器可以实现 Controller 接口，也可以用 @RequestMapping 注解将方法作为一个处理器等，这就导致 Spring 不知道怎么调用用户的处理器逻辑，所以这里需要一个处理器适配器，由处理器适配器去调用处理器的逻辑|
|ViewResolver|视图解析器，用于将视图名称解析为视图对象 View|
|View|视图对象，用于将模板渲染成 html 或其他类型的文件，比如 InternalResourceView 可将 jsp 渲染成 html|

## SpringBoot
### 自动装配原理
![](http://images.intflag.com/spring007.jpg)
#### @SpringBootConfiguration
- 来源于 @Configuration；
- 将当前类标注为配置类，并将当前类里以 @Bean 注解标记的方法的实例注入到 Srping 容器中，实例名即为方法名；

#### @ComponentScan
- 用于定义 Spring 的扫描路径；
- 如果不配置扫描路径 basePackages 参数，那么 Spring 就会默认扫描当前类所在的包及其子包中的所有标注了 @Component，@Service，@Controller 等注解的类；

#### @EnableAutoConfiguration
- @AutoConfigurationPackage：读取到我们在最外层的 @SpringBootApplication 注解中配置的扫描路径（没有配置则默认当前包及其子包），然后把扫描路径下面的类都加到数组中返回；
- @Import(AutoConfigurationImportSelector.class)
  - 间接实现了 ImportSelector 类的 selectImports() 方法；
  - 扫描所有 jar 包下的 META-INF/spring.factories 文件，利用反射自动将需要的类加载到容器中；

#### SPI 机制
- SPI，Service Provider Interface。即：接口服务的提供者；
- 应该面向接口（抽象）编程，而不是面向具体的实现来编程，这样一旦我们需要切换到当前接口的其他实现就无需修改代码；
- 如：数据库驱动 DriverManager  类，数据库驱动的 jar 包下面的 META-INF/services/ 下有一个文件 java.sql.Driver，里面记录了当前需要加载的驱动；

### 启动流程
- 对 SpringApplication 类进行实例化，然后初始化；
  - 项目应用类型；
  - 监听器；
  - 初始化器；
  - 启动类集合；
  - 类加载器；

![](http://images.intflag.com/spring008.jpg)
![](http://images.intflag.com/spring009.jpg)

- 执行 run 方法；

![](http://images.intflag.com/spring010.jpg)

### 内置 Tomcat 优化
- server.tomcat.threads.max 最大工作线程数：默认为 200，不是越大越好，通常为 核数的 200 - 250 倍；
- server.tomcat.threads.min-spare 最小工作线程数：默认10，可以适当增大一些，以便应对突然增长的访问量；
- server.tomcat.max-connections 最大连接数：默认为 8192；
- server.tomcat.accept-count 等待队列长度，默认100，队列也做缓冲池用，但也不能无限长，不但消耗内存，而且出队入队也消耗CPU；