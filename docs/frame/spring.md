# Spring
## Spring 架构以及设计思想
## Bean 的实例化机制以及循环依赖问题
### 实例化机制
- 实例化包括：对象实例化、对象属性的实例化；
- Spring 实例化 bean 是通过 ApplicationContext.getBean() 方法来进行的；
- 如果要获取的对象依赖了另一个对象，那么其首先会创建当前对象，然后通过递归的调用 ApplicationContext.getBean() 方法来获取所依赖的对象，最后将获取到的对象注入到当前对象中；

### 循环依赖
```java
@Component
public class A {
  private B b;
  public void setB(B b) {
    this.b = b;
  }
}

@Component
public class B {
  private A a;
  public void setA(A a) {
    this.a = a;
  }
}
```

![](http://images.intflag.com/spring001.jpg)

## Spring Bean 的生命周期
- [请别再问Spring Bean的生命周期了](https://www.jianshu.com/p/1dec08d290c1)
- [Spring Bean的生命周期（非常详细）](https://www.cnblogs.com/zrtqsk/p/3735273.html)

## Spring Bean 的作用域
- 通过bean 定义中的scope属性来定义bean的作用阈。
- Spring 框架支持以下五种bean的作用域：
    - Singleton : Bean 在每个 Spring IOC 容器中只有一个实例；
    - Prototype：一个 bean 的定义可以有多个实例；
    - Request：每次http请求都会创建一个bean，该作用域仅在基于web的Spring ApplicationContext情形下有效；
    - Session：在一个HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效；
    - Global-session：在一个全局的HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。 注意： 缺省的Spring bean 的作用域是Singleton。使用 prototype 作用域需要慎重的思考，因为频繁创建和销毁 bean 会带来很大的性能开销。

- 什么时候使用原型模式？

## Spring自动加载哪些类，原理是什么?

## Spring 中是如何处理线程并发的问题的？
## Spring 是如何实现事务的？
spring事务传播级别有几种
## Spring 中用到了哪些设计模式？
## Spring AOP
核心概念，作用与优点，专业术语，应用场景
spring的aop是怎么实现的
## Spring 三级缓存
## SpringMVC 一个请求的流程是什么？
## SpringBoot 启动加载过程?
spring boot和spring的区别？spring boot如何实现自动扫描注入？
## SpringBoot 加载配置文件原理?
## SpringBoot 的自动装配如何实现?
## SpringBoot 优化内部的 Tomcat 怎么做?
## SpringCloud 的组件做一下介绍
## SpringCloud 的服务发现与注册的实现
## spring cloud configration  bus 的 原理