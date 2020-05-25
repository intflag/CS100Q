# Java IO
## 1、BIO NIO AIO
?> **面试题：** 常见的 IO 模型有哪些，各自的原理和优缺点是什么？
<!-- tabs:start -->

#### **参考回答**

### 1）BIO (Blocking I/O)
- <mark>&nbsp;BIO (Blocking I/O)&nbsp;</mark>：同步阻塞 I/O 模型，数据的读取或写入必须阻塞在一个线程内等待其完成；
- 采用 BIO 通信模型的服务端，通常会有一个独立的线程监听客户端的连接，而且为了服务端能同时处理多个客户端请求，就必须采用多线程，主要原因是主要原因是 `socket.accept()`、`socket.read()`、`socket.write()` 涉及的三个主要函数都是同步阻塞的；
- 所以服务端在接收到客户端的连接请求后，会为每个客户端都创建一个新的线程进行链路处理，处理完成后通过输出流返回结果给客户端，然后销毁线程；
- 但是，我们知道在 Java 虚拟机中，线程是宝贵的资源，创建和销毁线程都需要额外的开销，而且，如果某个客户端的连接不做任何事的话，也会造成资源浪费，所以为了对 BIO 这种通信模型做优化，引入了线程池；
- 当服务端监听到有新的客户端接入时，将客户端的 Socket 封装成一个 Task，然后发送到线程池中进行处理，线程池会维护几个核心线程和一个线程队列，通过设置最大线程数和队列大小保证资源是可控的，这样无论多少个客户端并发访问，都不会有资源耗尽的问题，这种优化后的模型被称为伪异步 I/O 通信模型；
- 虽然伪异步 I/O 通信模型采用了线程池实现，因为它的底层仍然是同步阻塞的 BIO 模型，因此无法从根本上解决问题，在活动连接数不是特别高（小于单机1000）的情况下，这种模型是比较不错的，但是，当面对十万甚至百万级连接的时候，传统的 BIO 模型就无能为力了。

### 2）NIO (Non-Blocking I/O)
- <mark>&nbsp;NIO (Non-Blocking I/O)&nbsp;</mark>：同步非阻塞 I/O 模型，在Java 1.4 中引入了 NIO 框架，对应 java.nio 包，提供了 Buffer、Channel、Selector 等抽象；
- 传统 Java IO 的各种流是阻塞的，当一个线程调用 read 或 write 方法时，该线程被阻塞，直到数据被读取，或者数据被完全写入，在此期间这个线程不能做其他事情；
- 而 Java NIO 可以让我们进行非阻塞 I/O 操作，比如说单线程从 Channel 中读取数据到 Buffer 时，线程可以继续做其他事情，当数据读取到 Buffer 中后，再继续处理数据，非阻塞写也一样，一个线程请求写入一些数据到 Channel 中时，不需要等数据完全写入，此时线程可以做其他事情；
- <mark>&nbsp;Buffer（缓冲区）</mark>：IO 是面向流的，NIO 是面向缓存区的，Buffer 是一个对象，它包含一些要写入或者要读出的数据，在面向流的 I/O 中，可以将数据直接写入或者将数据直接读到 Stream 对象中。虽然 Stream 中也有 Buffer 开头的扩展类，但只是流的包装类，还是从流读到缓冲区，而 NIO 却是直接读到 Buffer 中进行操作；
- 在 NIO 厍中，所有数据都是用缓冲区处理的，在读取数据时，它是直接读到缓冲区中的; 在写入数据时，写入到缓冲区中，任何时候访问 NIO 中的数据，都是通过缓冲区进行操作；
- 最常用的缓冲区是 ByteBuffer，一个 ByteBuffer 提供了一组功能用于操作 byte 数组，除了ByteBuffer，还有其他的一些缓冲区，事实上，每一种 Java 基本类型（除了 Boolean 类型）都对应有一种缓冲区；
- <mark>&nbsp;Channel（通道）</mark>：NIO 通过 Channel（通道）进行读写，通道是双向的，可读也可写，而流的读写是单向的，无论读写，通道只能和 Buffer 交互，因为 Buffer，通道可以异步地读写；
- <mark>&nbsp;Selector（选择器）</mark>：NIO 为了提高效率，引入了 Selector（选择器）来实现<mark>&nbsp;多路复用&nbsp;</mark>，选择器的作用是可以用单个线程处理多个通道，所以它只需要比较少的线程就可以处理很多个通道，因为线程之间的切换对于操作系统来说代价是比较高的，所以采用这种方式可以提高效率；
- <mark>&nbsp;NIO 读写数据的方式&nbsp;</mark>：通常 NIO 中的所有 IO 都是从 Channel（通道）开始的：
    - 从通道进行数据读取：创建一个缓冲区，然后请求通道读取数据；
    - 从通道进行数据写入：创建一个缓冲区，填充数据，并要求通道写入数据；
- 生产环境很少使用 JDK 原生的 NIO 类库进行开发，首先编程实现比较复杂，容易出现 Bug，维护成本高，其次 JDK 的 NIO 底层由 epoll 实现，该实现饱受诟病的空轮询 bug 会导致 cpu 飙升 100%。

### 3）AIO (Asynchronous I/O)
- <mark>&nbsp;AIO (Asynchronous I/O)&nbsp;</mark>：也就是 NIO 2，在 Java 7 中引入了 NIO 的改进版 NIO 2，它是异步非阻塞的IO模型；
- 异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作；
- AIO 是异步IO的缩写，虽然 NIO 在网络操作中，提供了非阻塞的方法，但是 NIO 的 IO 行为还是同步的，对于 NIO 来说，我们的业务线程是在 IO 操作准备好时，得到通知，接着就由这个线程自行进行 IO 操作，IO操作本身是同步的。（除了 AIO 其他的 IO 类型都是同步的，这一点可以从操作系统底层 IO 线程模型解释）。

### 参考资料
- [BIO,NIO,AIO 总结【JavaGuide】](https://snailclimb.gitee.io/javaguide/#/docs/java/BIO-NIO-AIO?id=bionioaio-%e6%80%bb%e7%bb%93)
- [Netty入门教程——认识Netty【追那个小女孩】](https://www.jianshu.com/p/b9f3f6a16911)
- [漫话：如何给女朋友解释什么是Linux的五种IO模型？【漫话编程】](https://mp.weixin.qq.com/s?__biz=Mzg3MjA4MTExMw==&mid=2247484746&idx=1&sn=c0a7f9129d780786cabfcac0a8aa6bb7&source=41#wechat_redirect)

#### **源码详解**



<!-- tabs:end -->
