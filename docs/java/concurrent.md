# Java 并发
## 1、进程和线程

?> **面试题：** 如何理解进程和线程，他们的的区别是什么？
- 一般来说，进程是资源分配的最小单位，线程是 CPU 调度的最小单位，但是这样说太抽象了；
- 平时如果你在后台挂着微信，那在电脑的任务管理器最少可以看到三个进程，微信主进程，小程序进程，微信浏览器进程，拿微信主进程来说，可能你给别人发文字是一个线程，发语音是另一个线程；
- 所以进程管理着多个线程，多个线程会共享进程的资源；
- 进程之间是相互独立的，如果小程序进程崩溃了，不会影响到微信主进程；
- 进程之间通信被称为 IPC，IPC 的方式通常有管道、消息队列、信号量、共享存储、Socket、Streams等；
- 在调度和切换方面，线程上下文切换要比进程上下文切换快得多。

## 2、线程的生命周期

?> **面试题：** 线程的生命周期有哪几种状态？

![](http://images.intflag.com/concurrent02.jpg)

- 新建状态：比如使用 new Thread 方式建立一个线程对象后，这个线程对象就出于新建状态；
- 就绪状态：当线程对象调用了 start 方法以后，就进入了就绪状态，就绪状态的线程处于就绪队列中，要等待 JVM 里线程调度器的调度；
- 运行状态：如果就绪状态的线获得了 CPU 资源，就可以执行 run 方法里面的业务逻辑，此时线程处于运行状态；
- 阻塞状态：如果一个线程执行了睡眠或者挂起等方法，会进入阻塞状态，此时线程会让出 CPU，如果睡眠时间已到或者获得设备资源后可以从新回到就绪状态；
- 死亡状态：一个运行的线程执行完毕或者出现了未知异常会导致线程死亡。

## 3、多线程的使用

?> **面试题：** 实现多线程的方式有哪几种，各有什么优缺点？

<!-- tabs:start -->

#### **参考回答**

- 继承 Thread 类，重写 run 方法；
- 实现 Runnable 接口，然后实现 run 方法；
- 实现 Callable 接口，然后实现 call 方法，使用的时候需要用 FutureTask 包装一下，其实 FutureTask 也间接实现了 Runnable 接口，但这种方式可以通过 FutureTask 的 get 方法获取线程的异步执行结果。

#### **源码详解**

### 1）继承 Thread 类
继承 Thread 类，然后重写 run 方法。
```java
public class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread ID: " + this.getId() + ", Thread Name: " + this.getName() + ", Hello MyThread");
    }

    public static void main(String[] args) {
        System.out.println("Thread ID: " + Thread.currentThread().getId() + ", Thread Name: " + Thread.currentThread().getName());
        MyThread myThread = new MyThread();
        myThread.start();
    }
}
```
```
Thread ID: 1, Thread Name: main
Thread ID: 13, Thread Name: Thread-0, Hello MyThread
```
### 2）实现 Runnable 接口
实现 Runnable 接口，然后实现 run 方法。
```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Thread ID: " + Thread.currentThread().getId() + ", Thread Name: " + Thread.currentThread().getName() + ", Hello MyRunnable");
    }

    public static void main(String[] args) {
        System.out.println("Thread ID: " + Thread.currentThread().getId() + ", Thread Name: " + Thread.currentThread().getName());
        MyRunnable myRunnable = new MyRunnable();
        Thread thread = new Thread(myRunnable);
        thread.start();
    }
}
```
```
Thread ID: 1, Thread Name: main
Thread ID: 11, Thread Name: Thread-0, Hello MyRunnable
```
### 3）实现 Callable 接口
实现 Callable 接口，实现 call 方法并且指定返回值类型，但是在启动时要注意，需要使用 FutureTask 包装一下，然后新建线程启动，因为 Thread 的构造方法接收的是 Runnable，而 FutureTask 间接实现了 Runnable 接口，最后线程执行的结果可以通过 FutureTask 的 get 方法获取到。

Thread 构造方法：
```java
public Thread(Runnable target) {
    init(null, target, "Thread-" + nextThreadNum(), 0);
}
```
FutureTask 类关系：

![](http://images.intflag.com/concurrent01.png)

案例：
```java
public class MyCallable implements Callable<String> {
    @Override
    public String call() throws InterruptedException {
        String res = "Hello MyCallable";
        System.out.println("Thread ID: " + Thread.currentThread().getId() + ", Thread Name: " + Thread.currentThread().getName() + ", " + res);
        Thread.sleep(1000);
        return res + " Return";
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println("Thread ID: " + Thread.currentThread().getId() + ", Thread Name: " + Thread.currentThread().getName());
        MyCallable myCallable = new MyCallable();
        FutureTask<String> futureTask = new FutureTask<>(myCallable);
        Thread thread = new Thread(futureTask);
        thread.start();
        System.out.println("Thread ID: " + Thread.currentThread().getId() + ", Thread Name: " + Thread.currentThread().getName() + ", " + futureTask.get());
    }
}
```
```
Thread ID: 1, Thread Name: main
Thread ID: 11, Thread Name: Thread-0, Hello MyCallable
Thread ID: 1, Thread Name: main, Hello MyCallable Return
```

<!-- tabs:end -->

## 4、线程池原理及使用

?> **面试题：** 什么是线程池？线程池的实现原理是什么？你在生产环境下是如何使用的？

<!-- tabs:start -->

#### **参考回答**

### 1）线程池
- 线程池是一种基于`池化思想`管理线程的工具；
- 因为线程会占用资源，而创建和销毁线程又会带来额外的开销，当然也就降低了性能，如果我们用线程池把若干个线程维护起来，任务来的时候直接从线程池取出一个线程去执行，任务执行结束再把这个线程归还到线程池，那这样就实现了一个线程的复用；
- 线程是稀缺资源，如果不停的创建，很可能会把系统的资源耗尽，所以需要用线程池去统一管理；
- 线程池还具备可扩展性，比如延时定时线程池 ScheduledThreadPoolExecutor，就允许任务延期执行或定期执行。

### 2）实现原理
- 线程池是基于`生产者-消费者`模式实现的，提交任务的一方相当于生产者，线程池相当于消费者；
- 任务提交以后会先判断当前线程数量是否大于核心线程数量，如果没有超过核心线程数就创建新的线程；
- 如果超过了核心线程数并且此时任务队列没有满，就把任务添加到队列中；
- 如果队列满了，会判断当前线程数是否大于最大线程数，如果没有超过最大线程数就会创建一个新的线程来执行任务；
- 如果发现再创建一个线程以后比最大线程数还大，那就执行拒绝策略。

### 3）生产实践
**① Executors**
- 以前实现线程池可以用一个工具类，叫做 Executors，提供工厂方法来创建不同类型的线程池。
- 具体可以创建四种线程池，有 newCachedThreadPool 可缓存线程池、newFixedThreadPool 定长线程池、newScheduledThreadPool 也是定长线程池，可以支持定时任务和周期任务，还可以创建一种单线程化的线程池 newSingleThreadExecutor。

**② ThreadPoolExecutor**
- 但是，看过《阿里巴巴Java开发手册》的人都知道，线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式使用线程池；
- 因为 Executors 创建的线程池有一些弊端，比如`定长线程池`和`单线程化的线程池`允许队列长度为 `(2 ^ 31) - 1`，很可能导致 `OOM`，而`可缓存的线程池`允许创建线程的最大数量也是 `(2 ^ 31) - 1`，也会导致 `OOM`；
- 所以在生产环境中要用 `ThreadPoolExecutor`，它有几个重要的参数：
    - <mark>&nbsp;corePoolSize&nbsp;</mark>：该参数表示的是线程池的核心线程数。当任务提交到线程池时，如果线程池的线程数量还没有达到 corePoolSize，那么就会新创建的一个线程来执行任务，如果达到了，就将任务添加到任务队列中。
    - <mark>&nbsp;maximumPoolSize&nbsp;</mark>：该参数表示的是线程池中允许存在的最大线程数量。当任务队列满了以后，再有新的任务进入到线程池时，会判断再新建一个线程是否会超过 maximumPoolSize，如果会超过，则不创建线程，而是执行拒绝策略。如果不会超过 maximumPoolSize，则会创建新的线程来执行任务。
    - <mark>&nbsp;keepAliveTime&nbsp;</mark>：当线程池中的线程数量大于 corePoolSize 时，那么大于 corePoolSize 这部分的线程，如果没有任务去处理，那么就表示它们是空闲的，这个时候是不允许它们一直存在的，而是允许它们最多空闲一段时间，这段时间就是 keepAliveTime，时间的单位就是 TimeUnit。
    - <mark>&nbsp;unit&nbsp;</mark>：空闲线程允许存活时间的单位，TimeUnit 是一个枚举值，它可以是纳秒、微妙、毫秒、秒、分、小时、天。
    - <mark>&nbsp;workQueue&nbsp;</mark>：任务队列，用来存放任务，该队列的类型是阻塞队列。
    - <mark>&nbsp;threadFactory&nbsp;</mark>：线程池工厂，用来创建线程。通常在实际项目中，为了便于后期排查问题，在创建线程时需要为线程赋予一定的名称，通过线程池工厂，可以方便的为每一个创建的线程设置具有业务含义的名称。
    - <mark>&nbsp;handler&nbsp;</mark>：拒绝策略。当任务队列已满，线程数量达到 maximumPoolSize 后，线程池就不会再接收新的任务了，这个时候就需要使用拒绝策略来决定最终是怎么处理这个任务。默认情况下使用 AbortPolicy，表示无法处理新任务，直接抛出异常。在 ThreadPoolExecutor 类中定义了四个内部类，分别表示四种拒绝策略。我们也可以通过实现 RejectExecutionHandler 接口来实现自定义的拒绝策略。
    - <mark>&nbsp;AbortPocily&nbsp;</mark>：不再接收新任务，直接抛出异常；<mark>&nbsp;CallerRunsPolicy&nbsp;</mark>：提交任务的线程自己处理。<mark>&nbsp;DiscardPolicy&nbsp;</mark>：不处理，直接丢弃；<mark>&nbsp;DiscardOldestPolicy&nbsp;</mark>：丢弃任务队列中排在最前面的任务，并执行当前任务。（排在队列最前面的任务并不一定是在队列中待的时间最长的任务，因为有可能是按照优先级排序的队列）

### 参考
- [Java线程池实现原理及其在美团业务中的实践](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)
- [面试官：来！聊聊线程池的实现原理以及使用时的问题](https://juejin.im/post/5dd2d2205188254a0e15b991)

#### **源码详解**



<!-- tabs:end -->



线程池参数
## 5、Java 多线程中实现互斥同步的方式有几种，它们之间的对比？
## 6、Java 是如何实现多线程之间的协作的？
## 7、Java 中断线程的方式具体有哪些？

阻塞队列 https://www.infoq.cn/article/java-blocking-queue

至少得知道aba，cas，aqs，unsafe，volatile，sync，常见的各种lock，死锁，线程池参数和如何合理的去设置，你必须明白自旋，阻塞，死锁和它如何去定位，oom如何定位问题，cpu过高如何定位等基本的操作，你可以没有生产调试经验，但不代表你可以不会top，jps，jstack，jmap这些可能会问的东西。以及可能衍生的jmm模型和mesi协议等。

