# 设计模式
> 笔记整理自极客时间王铮老师的[《设计模式之美》](https://time.geekbang.org/column/intro/250)课程。

## 1、单例模式（创建型）
## 2、工厂模式（创建型）
## 3、建造者模式（创建型）
## 4、原型模式（创建型）
## 5、代理模式（结构型）
## 6、桥接模式（结构型）
## 7、装饰器模式（结构型）
## 8、适配器模式（结构型）
## 9、门面模式（结构型）
## 10、组合模式（结构型）
## 11、享元模式（结构型）
## 12、观察者模式（行为型）

## 13、模板模式（行为型）
### 1、定义

- 在方法中制定一个算法骨架或者业务流程，让某些步骤在子类中实现；
- 模板模式可以让子类在不改变算法整体结构或者业务流程的情况下，重新定制某些步骤。

### 2、实现
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
### 3、作用

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