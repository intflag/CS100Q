# 设计模式
> 笔记整理自极客时间王铮老师的[《设计模式之美》](https://time.geekbang.org/column/intro/250)课程。

## 1、单例模式（创建型）
### a、饿汉式
```java
public class SingletonHunger {
    private static final SingletonHunger INSTANCE = new SingletonHunger();

    private SingletonHunger() {}

    public static SingletonHunger getInstance() {
        return INSTANCE;
    }
}
```
### b、懒汉式
```java
public class SingletonLazy {
    private static SingletonLazy instance = null;
    private SingletonLazy(){}

    public static synchronized SingletonLazy  GetInstance(){
        if (instance == null) {
            instance = new SingletonLazy();
        }
        return instance;
    }
}
```
### c、懒汉式-双重检测
```java
public class SingletonDoubleChecking {

    private static SingletonDoubleChecking instance = null;

    private SingletonDoubleChecking(){}

    public static SingletonDoubleChecking getInstance(){
        if (instance == null) {
            synchronized (SingletonDoubleChecking.class) {
                if (instance == null) {
                    instance = new SingletonDoubleChecking();
                }
            }
        }
        return instance;
    }
}
```
### d、懒汉式-静态内部类
```java
public class SingletonStaticInner {

    private SingletonStaticInner() {}

    private static class SingletonHolder {
        private static final SingletonStaticInner INSTANCE = new SingletonStaticInner();
    }

    public static SingletonStaticInner getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```

## 2、工厂模式（创建型）
## 3、建造者模式（创建型）
- 创建同一种类型的复杂对象，可以选择不同参数，定制化创建对象；

```java
public class ResourcePoolConfig {
    private String poolName;
    private int maxTotal;
    private int maxIdle;
    private int minIdle;

    public ResourcePoolConfig(Builder builder) {
        this.poolName = builder.poolName;
        this.maxTotal = builder.maxTotal;
        this.maxIdle = builder.maxIdle;
        this.minIdle = builder.minIdle;
    }

    public static class Builder {
        private String poolName = "taskPool";
        private int maxTotal = 8;
        private int maxIdle = 8;
        private int minIdle = 8;

        public ResourcePoolConfig build() {
            // 校验逻辑放到这里来做，包括必填项校验、依赖关系校验、约束条件校验等
            if (!(poolName != null && poolName.length() > 0)) {
                throw new IllegalArgumentException("poolName can not be blank");
            }
            return new ResourcePoolConfig(this);
        }

        public Builder setPoolName(String poolName) {
            this.poolName = poolName;
            return this;
        }

        public Builder setMaxTotal(int maxTotal) {
            this.maxTotal = maxTotal;
            return this;
        }

        public Builder setMaxIdle(int maxIdle) {
            this.maxIdle = maxIdle;
            return this;
        }

        public Builder setMinIdle(int minIdle) {
            this.minIdle = minIdle;
            return this;
        }
    }

    public static void main(String[] args) {
        ResourcePoolConfig config = new Builder()
                .setPoolName("")
                .setMaxTotal(16)
                .setMaxIdle(16)
                .setMinIdle(8)
                .build();
    }
}
```

## 4、原型模式（创建型）
## 5、代理模式（结构型）
### a、代理模式的原理与实现
- 在不改变原始类（被代理对象）的情况，引入代理类给原始类增加功能；

- 方式一：代理类和原始类实现相同接口

```java
public interface IService {
    void fun();
}

public class Service implements IService {
    @Override
    public void fun() {
        System.out.println("fun");
    }
}

public class AProxyService implements IService{
    private Service service;

    public AProxyService(Service service) {
        this.service = service;
    }

    @Override
    public void fun() {
        System.out.println("start proxy...");
        service.fun();
        System.out.println("end proxy...");
    }

    public static void main(String[] args) {
        IService service = new AProxyService(new Service());
        service.fun();
    }
}
```

- 方式二：代理类继承原始类

```java
public class BProxyService extends Service {

    public void fun() {
        System.out.println("start proxy...");
        super.fun();
        System.out.println("end proxy...");
    }

    public static void main(String[] args) {
        Service service = new BProxyService();
        service.fun();
    }
}
```

- 缺点：每个类都要创建一个代理类，增加了维护成本的开发成本，不推荐。

### b、动态代理的原理与实现
- 运行过程中动态的创建原始类对应的代理类；

- 方式一：基于接口的 JDK 动态代理

```java
public class JdkDynamicProxy {
    public Object createProxy(Object proxiedObject) {
        Class<?>[] interfaces = proxiedObject.getClass().getInterfaces();
        DynamicProxyHandler proxyHandler = new DynamicProxyHandler(proxiedObject);
        return Proxy.newProxyInstance(proxiedObject.getClass().getClassLoader(), interfaces, proxyHandler);
    }

    private static class DynamicProxyHandler implements InvocationHandler {

        private Object proxiedObject;

        public DynamicProxyHandler(Object proxiedObject) {
            this.proxiedObject = proxiedObject;
        }

        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            System.out.println("start proxy...");
            Object invoke = method.invoke(proxiedObject, args);
            System.out.println("end proxy...");
            return invoke;
        }
    }

    public static void main(String[] args) {
        JdkDynamicProxy proxy = new JdkDynamicProxy();
        IService service = (IService) proxy.createProxy(new Service());
        service.fun();
    }
}
```

- 方式二：基于子类的 Cglib 动态代理

```java
public class CglibDynamicProxy {
    public Object createProxy(Object proxiedObject) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(proxiedObject.getClass());
        enhancer.setCallback(new DynamicProxyHandler(proxiedObject));
        return  enhancer.create();
    }

    private static class DynamicProxyHandler implements MethodInterceptor {
        private Object proxiedObject;

        public DynamicProxyHandler(Object proxiedObject) {
            this.proxiedObject = proxiedObject;
        }

        @Override
        public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
            System.out.println("start proxy...");
            Object invoke = method.invoke(proxiedObject, args);
            System.out.println("end proxy...");
            return invoke;
        }
    }

    public static void main(String[] args) {
        CglibDynamicProxy proxy = new CglibDynamicProxy();
        Service service = (Service) proxy.createProxy(new Service());
        service.fun();
    }
}
```

### a、应用场景

## 6、桥接模式（结构型）
## 7、装饰器模式（结构型）
## 8、适配器模式（结构型）
## 9、门面模式（结构型）
## 10、组合模式（结构型）
## 11、享元模式（结构型）
## 12、观察者模式（行为型）

### a、原理
- 观察者模式又被称为发布订阅模式；
- 在对象之间定义一个或者多个依赖，当一个对象状态发生改变时，所有依赖的对象都会自动收到通知；

### a、实现

- 方式一：经典实现

```java
public interface Subject {
    void registerObserver(Observer observer);
    void removeObserver(Observer observer);
    void notifyObservers(String message);
}

public interface Observer {
    void update(String message);
}

public class ConcreteObserverOne implements Observer {
    @Override
    public void update(String message) {
        System.out.println("ConcreteObserverOne update message: " + message);
    }
}

public class ConcreteObserverTwo implements Observer{
    @Override
    public void update(String message) {
        System.out.println("ConcreteObserverTwo message: " + message);
    }
}

public class ConcreteSubject implements Subject{

    private final List<Observer> observers = new ArrayList<>();

    @Override
    public void registerObserver(Observer observer) {
        observers.add(observer);
    }

    @Override
    public void removeObserver(Observer observer) {
        observers.remove(observer);
    }

    @Override
    public void notifyObservers(String message) {
        for (Observer observer : observers) {
            observer.update(message);
        }
    }

    public static void main(String[] args) {
        ConcreteSubject subject = new ConcreteSubject();
        subject.registerObserver(new ConcreteObserverOne());
        subject.registerObserver(new ConcreteObserverTwo());
        subject.notifyObservers("user login");
    }
}
```

### a、应用场景

## 13、模板模式（行为型）
### a、原理

- 在方法中制定一个算法骨架或者业务流程，让某些步骤在子类中实现；
- 模板模式可以让子类在不改变算法整体结构或者业务流程的情况下，重新定制某些步骤。

### b、实现
```java
public abstract class AbstractClass {
  public final void templateMethod() {
    //...
    method1();
    //...
    method2();
    //...
  }
  
  protected abstract void method1();
  protected abstract void method2();
}

public class ConcreteClass1 extends AbstractClass {
  @Override
  protected void method1() {
    //...
  }
  
  @Override
  protected void method2() {
    //...
  }
}

public class ConcreteClass2 extends AbstractClass {
  @Override
  protected void method1() {
    //...
  }
  
  @Override
  protected void method2() {
    //...
  }
}

AbstractClass demo = ConcreteClass1();
demo.templateMethod();
```
### c、应用场景

- 复用
    - Java InputStream：read() 方法。
    - Java AbstractList：addAll() 函数可以看作模板方法，add() 是子类需要重写的方法，尽管没有声明为 abstract 的，但函数实现直接抛出了 UnsupportedOperationException 异常。
- 扩展
    - Java HttpServlet：service() 方法是模板方法，doGet()、doPost() 是子类需要重写的方法。 
    - JUnit TestCase：runBare() 函数是模板方法，它定义了执行测试用例的整体流程：先执行 setUp() 做些准备工作，然后执行 runTest() 运行真正的测试代码，最后执行 tearDown() 做扫尾工作。

## 14、策略模式（行为型）
## 15、责任链模式（行为型）
## 16、状态模式（行为型）
## 17、迭代器模式（行为型）
## 18、访问者模式（行为型）
## 18、备忘录模式（行为型）
## 19、命令模式（行为型）
## 20、解释器模式（行为型）
## 21、中介模式（行为型）