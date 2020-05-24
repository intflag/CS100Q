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

<!-- tabs:start -->

#### **参考回答**

![](http://images.intflag.com/concurrent02.jpg)

- 新建状态：比如使用 new Thread 方式建立一个线程对象后，这个线程对象就出于新建状态；
- 就绪状态：当线程对象调用了 start 方法以后，就进入了就绪状态，就绪状态的线程处于就绪队列中，要等待 JVM 里线程调度器的调度；
- 运行状态：如果就绪状态的线获得了 CPU 资源，就可以执行 run 方法里面的业务逻辑，此时线程处于运行状态；
- 阻塞状态：如果一个线程执行了睡眠或者挂起等方法，会进入阻塞状态，此时线程会让出 CPU，如果睡眠时间已到或者获得设备资源后可以从新回到就绪状态；
- 死亡状态：一个运行的线程执行完毕或者出现了未知异常会导致线程死亡。

#### **源码详解**



<!-- tabs:end -->

## 3、多线程的使用

?> **面试题：** 实现多线程的方式有哪几种，各有什么优缺点？

<!-- tabs:start -->

#### **参考回答**

- 继承 Thread 类，重写 run 方法；
- 实现 Runnable 接口，然后实现 run 方法；
- 实现 Callable 接口，然后实现 call 方法，使用的时候需要用 FutureTask 包装一下，其实 FutureTask 也间接实现了 Runnable 接口，但这种方式可以通过 FutureTask 的 get 方法获取线程的异步执行结果；
- 一般情况下，选择实现接口的方式会好一些，因为 Java 是单继承，如果选择继承 Thread 类以后就不能再继承其他类，但可以实现多个接口，另外如果是实现接口的方式下获取当前线程对象，需要使用 `Thread.currentThread()`，如果是继承方式的话，直接使用 `this` 关键字就可以得到了。

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
- 因为线程会占用资源，而创建和销毁线程又会带来额外的开销，当然也就降低了性能，如果我们用线程池把若干个线程维护起来，任务来的时候直接从线程池取出一个线程去执行，任务执行结束再把这个线程归还到线程池，那这样就实现了线程的复用；
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
    - <mark>&nbsp;AbortPolicy&nbsp;</mark>：不再接收新任务，直接抛出异常；<mark>&nbsp;CallerRunsPolicy&nbsp;</mark>：提交任务的线程自己处理。<mark>&nbsp;DiscardPolicy&nbsp;</mark>：不处理，直接丢弃；<mark>&nbsp;DiscardOldestPolicy&nbsp;</mark>：丢弃任务队列中排在最前面的任务，并执行当前任务。（排在队列最前面的任务并不一定是在队列中待的时间最长的任务，因为有可能是按照优先级排序的队列）

### 参考资料
- [Java线程池实现原理及其在美团业务中的实践](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)
- [面试官：来！聊聊线程池的实现原理以及使用时的问题](https://juejin.im/post/5dd2d2205188254a0e15b991)

#### **源码详解**



<!-- tabs:end -->

## 5、互斥同步

?> **面试题：** Java 多线程中实现互斥同步的方式有几种，它们各自的实现原理是什么？

<!-- tabs:start -->

#### **参考回答**
### 1）Synchronized
- Synchronized 是 JVM 实现的，可以锁代码块、普通方法和静态方法，还可以锁类；
- 因为 Synchronized 是 JVM 实现，所以看不了源码，但是可以先使用 `javac` 命令编译一个类，然后用 `javap` 命令查看类的字节码，如果是对代码块加锁，那么看到有两个指令： <mark>&nbsp;monitorenter&nbsp;</mark> 和  <mark>&nbsp;monitorexit&nbsp;</mark> ，具体思想是，每个对象都有一个 monitor 监视器，调用 `monitorenter` 就是尝试获取这个对象，成功获取就将计数器 +1，释放就将计数器 -1，如果是重入就将值再 +1；
- 如果是对同步方法加锁的话，字节码没有那两个指令，而是有一个 <mark>&nbsp;ACC_SYNCHRONIZED&nbsp;</mark> 标识，他会在常量池中增加一个这个标识符，获取它的 `monitor`，所以本质上是一样的；
- Synchronized 在获取不到锁的时候会一直阻塞，不可以被中断，但是会自动释放锁；
- 新版本 Java 对 synchronized 进行了很多优化。

### 2）ReentrantLock
- ReentrantLock 是 <mark>&nbsp;java.util.concurrent&nbsp;</mark> 包下的锁，实现了 `Lock` 接口，只能锁代码块；
- ReentrantLock 是基于 <mark>&nbsp;AQS（AbstractQuenedSynchronizer）抽象的队列式同步器&nbsp;</mark> 实现的，需要手动去释放锁，通常方法开始的时候加锁，然后在 `finally` 中释放锁。

#### **源码详解**

### 1）synchronized
**① 同步代码块**
```java
public void fun() {
    synchronized (this) {
        // ...
    }
}
```
它只作用于同一个对象，如果调用两个对象上的同步代码块，就不会进行同步。

对于以下代码，使用 ExecutorService 执行了两个线程，由于调用的是同一个对象的同步代码块，因此这两个线程会进行同步，当一个线程进入同步语句块时，另一个线程就必须等待。

```java
public class SynchronizedTest {

    public void fun1() {
        synchronized (this) {
            for (int i = 0; i < 10; i++) {
                System.out.println("Thread Name: " + Thread.currentThread().getName() + ", i = " + i);
            }
        }
    }

    public static void main(String[] args) {
        SynchronizedTest sync1 = new SynchronizedTest();
        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.execute(() -> sync1.fun1());
        executorService.execute(() -> sync1.fun1());
    }
}
```
```
Thread Name: pool-1-thread-1, i = 0
Thread Name: pool-1-thread-1, i = 1
Thread Name: pool-1-thread-1, i = 2
Thread Name: pool-1-thread-1, i = 3
Thread Name: pool-1-thread-1, i = 4
Thread Name: pool-1-thread-1, i = 5
Thread Name: pool-1-thread-1, i = 6
Thread Name: pool-1-thread-1, i = 7
Thread Name: pool-1-thread-1, i = 8
Thread Name: pool-1-thread-1, i = 9
Thread Name: pool-1-thread-2, i = 0
Thread Name: pool-1-thread-2, i = 1
Thread Name: pool-1-thread-2, i = 2
Thread Name: pool-1-thread-2, i = 3
Thread Name: pool-1-thread-2, i = 4
Thread Name: pool-1-thread-2, i = 5
Thread Name: pool-1-thread-2, i = 6
Thread Name: pool-1-thread-2, i = 7
Thread Name: pool-1-thread-2, i = 8
Thread Name: pool-1-thread-2, i = 9
```
对于以下代码，两个线程调用了不同对象的同步代码块，因此这两个线程就不需要同步。从输出结果可以看出，两个线程交叉执行。
```java
public static void main(String[] args) {
    SynchronizedTest sync1 = new SynchronizedTest();
    SynchronizedTest sync2 = new SynchronizedTest();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> sync1.fun1());
    executorService.execute(() -> sync2.fun1());
}
```
```
Thread Name: pool-1-thread-1, i = 0
Thread Name: pool-1-thread-1, i = 1
Thread Name: pool-1-thread-1, i = 2
Thread Name: pool-1-thread-1, i = 3
Thread Name: pool-1-thread-1, i = 4
Thread Name: pool-1-thread-1, i = 5
Thread Name: pool-1-thread-2, i = 0
Thread Name: pool-1-thread-1, i = 6
Thread Name: pool-1-thread-2, i = 1
Thread Name: pool-1-thread-1, i = 7
Thread Name: pool-1-thread-2, i = 2
Thread Name: pool-1-thread-1, i = 8
Thread Name: pool-1-thread-1, i = 9
Thread Name: pool-1-thread-2, i = 3
Thread Name: pool-1-thread-2, i = 4
Thread Name: pool-1-thread-2, i = 5
Thread Name: pool-1-thread-2, i = 6
Thread Name: pool-1-thread-2, i = 7
Thread Name: pool-1-thread-2, i = 8
Thread Name: pool-1-thread-2, i = 9
```

**② 同步方法**
```java
public synchronized void fun () {
    // ...
}
```
它和同步代码块一样，作用于同一个对象。

**③ 同步静态方法**
```java
public synchronized static void fun() {
    // ...
}
```
作用于整个类。

**④ 同步类**
```java
public void fun() {
    synchronized (SynchronizedTest.class) {
        // ...
    }
}
```
作用于整个类，也就是说两个线程调用同一个类的不同对象上的这种同步语句，也会进行同步。
```java
public class SynchronizedExample {

    public void func2() {
        synchronized (SynchronizedExample.class) {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        }
    }
}
```
```java
public static void main(String[] args) {
    SynchronizedTest sync1 = new SynchronizedTest();
    SynchronizedTest sync2 = new SynchronizedTest();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> sync1.fun2());
    executorService.execute(() -> sync2.fun2());
}
```
```
Thread Name: pool-1-thread-1, i = 0
Thread Name: pool-1-thread-1, i = 1
Thread Name: pool-1-thread-1, i = 2
Thread Name: pool-1-thread-1, i = 3
Thread Name: pool-1-thread-1, i = 4
Thread Name: pool-1-thread-1, i = 5
Thread Name: pool-1-thread-1, i = 6
Thread Name: pool-1-thread-1, i = 7
Thread Name: pool-1-thread-1, i = 8
Thread Name: pool-1-thread-1, i = 9
Thread Name: pool-1-thread-2, i = 0
Thread Name: pool-1-thread-2, i = 1
Thread Name: pool-1-thread-2, i = 2
Thread Name: pool-1-thread-2, i = 3
Thread Name: pool-1-thread-2, i = 4
Thread Name: pool-1-thread-2, i = 5
Thread Name: pool-1-thread-2, i = 6
Thread Name: pool-1-thread-2, i = 7
Thread Name: pool-1-thread-2, i = 8
Thread Name: pool-1-thread-2, i = 9
```

### 2）ReentrantLock
ReentrantLock 是 `java.util.concurrent（J.U.C）` 包中的锁。

```java
public class ReentrantLockTest {

    private ReentrantLock lock = new ReentrantLock();

    public void fun1() {
        lock.lock();
        try {
            for (int i = 0; i < 10; i++) {
                System.out.println("Thread Name: " + Thread.currentThread().getName() + ", i = " + i);
            }
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        ReentrantLockTest sync1 = new ReentrantLockTest();
        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.execute(()->sync1.fun1());
        executorService.execute(()->sync1.fun1());
    }
}
```
```
Thread Name: pool-1-thread-1, i = 0
Thread Name: pool-1-thread-1, i = 1
Thread Name: pool-1-thread-1, i = 2
Thread Name: pool-1-thread-1, i = 3
Thread Name: pool-1-thread-1, i = 4
Thread Name: pool-1-thread-1, i = 5
Thread Name: pool-1-thread-1, i = 6
Thread Name: pool-1-thread-1, i = 7
Thread Name: pool-1-thread-1, i = 8
Thread Name: pool-1-thread-1, i = 9
Thread Name: pool-1-thread-2, i = 0
Thread Name: pool-1-thread-2, i = 1
Thread Name: pool-1-thread-2, i = 2
Thread Name: pool-1-thread-2, i = 3
Thread Name: pool-1-thread-2, i = 4
Thread Name: pool-1-thread-2, i = 5
Thread Name: pool-1-thread-2, i = 6
Thread Name: pool-1-thread-2, i = 7
Thread Name: pool-1-thread-2, i = 8
Thread Name: pool-1-thread-2, i = 9
```

### 3）synchronized 与 ReentrantLock 比较
**① 锁的实现**

synchronized 是 JVM 实现的，ReentrantLock 是 JDK 实现的。

**② 性能**

新版本 Java 对 synchronized 进行了很多优化，例如自旋锁等，synchronized 与 ReentrantLock 大致相同。

**③ 等待可中断**

当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。

ReentrantLock 可中断，而 synchronized 不行。

**④ 公平锁**

公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。

synchronized 中的锁是非公平的，ReentrantLock 默认情况下也是非公平的，但是也可以是公平的。

**⑤ 锁绑定多个条件**

一个 ReentrantLock 可以同时绑定多个 Condition 对象。

### 4）使用选择
除非需要使用 ReentrantLock 的高级功能，否则优先使用 synchronized。这是因为 synchronized 是 JVM 实现的一种锁机制，JVM 原生地支持它，而 ReentrantLock 不是所有的 JDK 版本都支持。并且使用 synchronized 不用担心没有释放锁而导致死锁问题，因为 JVM 会确保锁的释放。

<!-- tabs:end -->

## 6、线程协作
?> **面试题：** Java 是如何实现多线程之间的协作的？

### 1）join

<!-- tabs:start -->

#### **参考回答**

- 可以在线程中调用另一个线程的 join 方法，当前线程会挂起，直到另一个线程执行结束。

#### **源码详解**

在线程中调用另一个线程的 join() 方法，会将当前线程挂起，直到目标线程结束。

join方法有三个重载版本：
```java
join()
join(long millis)                    //参数为毫秒
join(long millis,int nanoseconds)    //第一参数为毫秒，第二个参数为纳秒
```
假如在 main 线程中，调用 thread.join 方法，则 main 方法会等待 thread 线程执行完毕或者等待一定的时间。如果调用的是无参 join 方法，则等待 thread 执行完毕，如果调用的是指定了时间参数的join方法，则等待一定的时间。

```java
public class JoinTest {
    class JoinThread extends Thread {
        @Override
        public void run() {
            try {
                System.out.println("Thread Name: " + this.getName() + ", Sleep Start");
                sleep(5000);
                System.out.println("Thread Name: " + this.getName() + ", Sleep End");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        System.out.println("Thread Name: " + Thread.currentThread().getName() + ", Main Start");
        JoinTest test = new JoinTest();
        JoinTest.JoinThread s1 = test.new JoinThread();
        s1.start();
        //s1.join();        //挂起 main 线程，等待 JoinThread 执行完毕再接着执行 main 线程
        System.out.println("Thread Name: " + Thread.currentThread().getName() + ", Main End");
    }
}
```
不调用 s1.join() 的执行结果：
```
Thread Name: main, Main Start
Thread Name: main, Main End
Thread Name: Thread-0, Sleep Start
Thread Name: Thread-0, Sleep End
```

调用 s1.join() 的执行结果：
```
Thread Name: main, Main Start
Thread Name: Thread-0, Sleep Start
Thread Name: Thread-0, Sleep End
Thread Name: main, Main End
```
可以看出，当调用 s1.join() 方法后，main 线程会进入等待，然后等待 s1 执行完之后再继续执行。

实际上调用 join 方法是调用了 Object 的 wait 方法，这个可以通过查看源码得知：
```java
public final synchronized void join(long millis)
throws InterruptedException {
    long base = System.currentTimeMillis();
    long now = 0;

    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (millis == 0) {
        while (isAlive()) {
            wait(0);
        }
    } else {
        while (isAlive()) {
            long delay = millis - now;
            if (delay <= 0) {
                break;
            }
            wait(delay);
            now = System.currentTimeMillis() - base;
        }
    }
}
```
wait 方法会让线程进入阻塞状态，并且会释放线程占有的锁，并交出 CPU 执行权限。

由于 wait 方法会让线程释放对象锁，所以 join 方法同样会让线程释放对一个对象持有的锁。

<!-- tabs:end -->

### 2）wait() notify() notifyAll()

<!-- tabs:start -->

#### **参考回答**

- 调用 wait 方法也会让线程等待，线程在等待时会被挂起，其他线程调用 notify() 或者 notifyAll() 来唤醒挂起的线程，这些都是 Object 类中的方法；
- 这些方法必须在`同步代码块`或者`同步方法`中使用，否则运行的时候会出现 IllegalMonitorStateException 非法监视状态异常；
- 使用 wait 挂起线程以后，线程会释放锁，因为如果不释放锁，其他线程就不能进入对象的同步代码块或者同步方法中，也就不能执行 notify() 或者 notifyAll() 方法来唤醒挂起的线程，从而造成死锁；
- wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法，wait() 会释放锁，sleep() 不会释放锁。

#### **源码详解**

```java
public class WaitTest {

    public synchronized void before() {
        System.out.println("Thread Name: " + Thread.currentThread().getName()+" before");
        notifyAll();
    }

    public synchronized void after() {
        try {
            wait();
            System.out.println("Thread Name: " + Thread.currentThread().getName()+" after");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();
        WaitTest waitTest = new WaitTest();
        executorService.execute(()->waitTest.after());
        executorService.execute(()->waitTest.before());

    }
}
```
```
Thread Name: pool-1-thread-2 before
Thread Name: pool-1-thread-1 after
```

<!-- tabs:end -->

### 3）await() signal() signalAll()

<!-- tabs:start -->

#### **参考回答**

- java.util.concurrent 类库中提供了 Condition 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其他线程上调用 signal() 或者 signalAll() 方法唤醒等待的线程。
- 相比与 wait() 方法，await() 方法可以指定等待的条件，因此更加灵活。

#### **源码详解**

```java
public class AwaitTest {

    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    public void before() {
        lock.lock();
        try {
            System.out.println("Thread Name: " + Thread.currentThread().getName() + " before");
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public void after() {
        lock.lock();
        try {
            condition.await();
            System.out.println("Thread Name: " + Thread.currentThread().getName() + " after");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();
        AwaitTest awaitTest = new AwaitTest();
        executorService.execute(() -> awaitTest.after());
        executorService.execute(() -> awaitTest.before());
    }
}
```
```java
Thread Name: pool-1-thread-2 before
Thread Name: pool-1-thread-1 after
```

<!-- tabs:end -->

## 7、线程中断
?> **面试题：** Java 中断线程的方式具体有哪些？

## 8、CAS & AQS
?> **面试题：** 谈谈你对 CAS 的理解，Java 8 是如何优化 CAS 性能的？

<!-- tabs:start -->

#### **参考回答**

### 参考资料
- [大白话聊聊Java并发面试问题之Java 8如何优化CAS性能？【石杉的架构笔记】](https://mp.weixin.qq.com/s?__biz=MzU0OTk3ODQ3Ng==&mid=2247484070&idx=1&sn=c1d49bce3c9da7fcc7e057d858e21d69&chksm=fba6eaa5ccd163b3a935303f10a54a38f15f3c8364c7c1d489f0b1aa1b2ef293a35c565d2fda&mpshare=1&scene=1&srcid=0517Jzf4pPxfShe3mewgFLDl&sharer_sharetime=1589726268906&sharer_shareid=2565447dd960ce5d1eaca147e7b93e39&key=042d77279f4726137744ab58f229534d4087388bec935765ec760d286f615f9ea1d6b3882cb6d1f37e76f5df4cab13ca69e46d865c0b9939ec0ed0952f9c9855f031fcd09e2b9d3c16edbe35c5593a4d&ascene=1&uin=ODMxODEyNzEx&devicetype=Windows+10+x64&version=62090070&lang=zh_CN&exportkey=AyOp4FTyqw0H66RJ7howCxc%3D&pass_ticket=udrU14MLSMHdMByTIzdg1n8%2Fx8pZeL9E%2FWhuE%2BcOCfUYXnDgXqXtqGo47o2QxUTB)

#### **源码详解**



<!-- tabs:end -->

?> **面试题：** 谈谈你对 Java 中 AQS 的理解？

<!-- tabs:start -->

#### **参考回答**

### 参考资料
- [Java 并发高频面试题：聊聊你对 AQS 的理解？【石杉的架构笔记】](https://mp.weixin.qq.com/s/zdn54VeNSsabwDd3CBvSoA)

#### **源码详解**



<!-- tabs:end -->

## 9、锁以及锁优化
?> **面试题：** Java 中有那些锁？实现了哪些锁优化技术？

<!-- tabs:start -->

#### **参考回答**

### 参考资料
- [不可不说的Java“锁”事](https://tech.meituan.com/2018/11/15/java-lock.html)
- [不懂什么是 Java 中的锁？看看这篇你就明白了！【石杉的架构笔记】](https://mp.weixin.qq.com/s?__biz=MzU0OTk3ODQ3Ng==&mid=2247486820&idx=1&sn=cdd3ca69c68383a38a48bfd74124f0af&chksm=fba6e567ccd16c71438fe62f40fee9f55746453e91bc12a2c2be47272de84aa1a20dc541c63d&mpshare=1&scene=1&srcid=0517yoHuUp41wrvwCzbB6oaU&sharer_sharetime=1589727926343&sharer_shareid=2565447dd960ce5d1eaca147e7b93e39&key=1f1e787ff7a3f9028b14959bba2dc365d99a3d18a1d04769c87784d8d51fe03f42da291c336e8e5a0e2e5f7cc7108cf40baebbee0813fa48ed9b4e2382e51fa2682f3ac4cddb6c4ff32dc79dfaf4e7b5&ascene=1&uin=ODMxODEyNzEx&devicetype=Windows+10+x64&version=62090070&lang=zh_CN&exportkey=Ayjq6CBMGjJDbamv49c5fTA%3D&pass_ticket=udrU14MLSMHdMByTIzdg1n8%2Fx8pZeL9E%2FWhuE%2BcOCfUYXnDgXqXtqGo47o2QxUTB)
- [一文带你了解 Java 并发中的锁优化和线程池优化！【石杉的架构笔记】](https://mp.weixin.qq.com/s?__biz=MzU0OTk3ODQ3Ng==&mid=2247486831&idx=1&sn=69ca4c63d806f1d22579b3a3df52d3e7&chksm=fba6e56cccd16c7ada14fc23d052de0f2f02c4cc1560f65bdf2c2d473a8a1a0047b35738d911&mpshare=1&scene=1&srcid=0517vTFqkELMfZyIoN8Uk0Ky&sharer_sharetime=1589726717709&sharer_shareid=2565447dd960ce5d1eaca147e7b93e39&key=7696a76dfdc98da9b97c4eb2179ccc28bc842753067f804fa08ee938b7307bfdf685fc9c3901a2b2b9f260bf079c75b1f9bd8d1cbeb71dbf84157afc504b86b87f241940b348d50a0e84b2d1078e8b27&ascene=1&uin=ODMxODEyNzEx&devicetype=Windows+10+x64&version=62090070&lang=zh_CN&exportkey=AytqvL0rdirbI8iwxyaOLzs%3D&pass_ticket=udrU14MLSMHdMByTIzdg1n8%2Fx8pZeL9E%2FWhuE%2BcOCfUYXnDgXqXtqGo47o2QxUTB)

#### **源码详解**



<!-- tabs:end -->


## 10、阻塞队列
?> **面试题：** 什么是阻塞队列，Java 中有哪些阻塞队列？
<!-- tabs:start -->

#### **参考回答**

### 参考资料
- [聊聊并发（七）——Java 中的阻塞队列](https://www.infoq.cn/article/java-blocking-queue)

#### **源码详解**



<!-- tabs:end -->


至少得知道
aba
cas
aqs
unsafe
volatile
sync，常见的各种lock，死锁，
线程池参数和如何合理的去设置，
你必须明白自旋，阻塞，死锁和它如何去定位，
oom如何定位问题，cpu过高如何定位等基本的操作，
你可以没有生产调试经验，但不代表你可以不会top，jps，jstack，jmap这些可能会问的东西。
以及可能衍生的jmm模型和mesi协议等。

