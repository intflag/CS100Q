# Java 虚拟机
## 1、运行时数据区
?> **面试题：** 谈谈你对 JVM 内存模型的了解，以及每个区域的用途是什么？

<!-- tabs:start -->

#### **参考回答**
- java虚拟机运行时数据区主要分线程隔离的和线程共享的；
-  <mark>&nbsp;线程隔离&nbsp;</mark>的有：<mark>&nbsp;程序计数器&nbsp;</mark> 、<mark>&nbsp;Java 虚拟机栈&nbsp;</mark> 、<mark>&nbsp;本地方法栈&nbsp;</mark> ；
    - 程序计数器，也叫 PC 寄存器，每个线程都有自己的程序计数器，如果当前执行的是 JVM 的方法，则该寄存器中保存当前执行指令的地址，如果执行的是 native 方法，则 PC 寄存器中为空；
    - 然后每个方法执行的时候都要先创建一个`栈帧`，它里面主要保存着`局部变量表`、`操作数栈`、`常量池引用`等信息，方法从开始调用到执行结束对应着栈帧在 Java 虚拟机中`入栈`和`出栈`的过程；
    - 本地方法栈与虚拟机栈类似，只不过本地方法栈是为本地方法服务的，本地方法一般是 C、C++、汇编实现的，而且虚拟机没有对本地方法的语言、使用方式、数据结构做强制规定，可以由虚拟机自由实现，HotSpot 虚拟机就把二者合二为一了。
-  <mark>&nbsp;线程共享&nbsp;</mark>的有：<mark>&nbsp;堆&nbsp;</mark>和<mark>&nbsp;方法区&nbsp;</mark>，方法区中又包含<mark>&nbsp;运行时常量池&nbsp;</mark>；
    - `Java 堆`被所有`线程共享`，`几乎`所有的`对象实例`都在`堆分配内存`，之所以说几乎，是因为随着 JIT 编译器的发展，在编译时期经过`逃逸分析`，如果发现有些对象没有逃逸出方法，那么就有可能被`优化`成在虚拟机`栈上分配内存`，但这个不是绝对的；
    - Java 堆是`垃圾收集器`管理的主要区域，也被称为 `GC 堆`，现代的垃圾收集器基本都是采用`分代收集算法`，所有堆被分为`新生代 Young Generation`和`老年代 Old Generation`；
    - 堆不需要连续内存，可以动态增加内存，增加失败抛出 OutOfMemoryError 错误；
    - 在 JDK 7 中，方法区用于存放已被加载的类信息、常量、静态变量、即时编译器编译后的代码等数据，从 JDK 1.8 开始，移除永久代，并把方法区移至元空间，它位于本地内存中，而不是虚拟机内存中；
    - 方法区是一个 JVM 规范，永久代与元空间都是其一种实现方式，在 JDK 1.8 之后，原来永久代的数据被分到了堆和元空间中，元空间存储类的元信息，静态变量和常量池等放入堆中。

#### **源码详解**

### 参考资料
- [Metaspace 之一：Metaspace整体介绍（永久代被替换原因、元空间特点、元空间内存查看分析方法）](https://www.cnblogs.com/duanxz/p/3520829.html)

<!-- tabs:end -->

## 2、垃圾回收
?> **面试题：** JVM 是怎么进行对象回收判断的，具体的垃圾回收算法是什么？

<!-- tabs:start -->

#### **参考回答**

### 1）对象回收判断
- 有<mark>&nbsp;引用计数器算法&nbsp;</mark>，原理主要是给对象添加一个引用计数器，每当对象增加一个引用计数器就加 1，引用失效计数器就减 1，引用计数器为 0 的对象可以被回收，但是，当两个对象出现循环引用的时候，计数器永远不为 0，导致对象无法被回收，所以虚拟机现在都不采用这种算法；
- 还有一种是<mark>&nbsp;可达性分析算法&nbsp;</mark>，原理是以 GC Roots 为起始点进行搜索，可达的对象都是存活的，不可达的就是能被回收的，现在的虚拟机大多采用的是这种算法。

### 2）垃圾回收算法
- <mark>&nbsp;标记-清除&nbsp;</mark>算法：标记阶段会根据对象的存活状态在对象头部打上标记，清除阶段回收对象取消标志位，另外，还会将当前回收的区块和前一个连续的空闲区块合并，但是在分配的时候会将大区块分隔成多个小区块，这样就会产生不连续的内存碎片，导致无法给大对象分配内存；
- <mark>&nbsp;标记-整理&nbsp;</mark>算法：让所有存活的对象都往一端移动，然后直接清理边界以外的内存，虽然这样不会产生内存碎片，但是需要移动大量对象，处理效率比较低；
- <mark>&nbsp;复制&nbsp;</mark>算法：原理是将内存划分成大小相等的两块，每次只是用其中一块，当这一块内存用完就将还存活的对象复制到另一块上，然后清理之前那一块，但这种效率只有 50%；
- 现在的商业虚拟机都是采用这种算法回收新生代，但不是分成相等的两块，而是一块较大的 <mark>&nbsp;Eden&nbsp;</mark> 空间和两块较小的 <mark>&nbsp;Survivor&nbsp;</mark> 空间，每次使用一块 Eden 和一块 Survivor ，回收的时候，将 Eden 和 Survivor 中还存活着的对象全部复制到另一块 Survivor 上，最后清理 Eden 和使用过的那一块 Survivor；
- HotSpot 虚拟机的 Eden 和 Survivor 大小比例默认为 <mark>&nbsp;8:1&nbsp;</mark>，保证了内存的利用率达到 90%。如果每次回收时有超过 10% 的对象存活，那么一块 Survivor 就不够用了，此时需要依赖于老年代进行空间分配担保，也就是借用老年代的空间存储放不下的对象。
- <mark>&nbsp;分代收集&nbsp;</mark>：现在的商业虚拟机基本都采用分代收集，一般将堆划分成<mark>&nbsp;新生代&nbsp;</mark>和<mark>&nbsp;老年代&nbsp;</mark>，新生代通常使用`复制算法`，老年代使用`标记-清除`或者`标记-整理`算法；

#### **源码详解**



<!-- tabs:end -->

## 3、垃圾收集器
?> **面试题：** JVM 有哪些垃圾收集器，各自的优缺点及使用场景是什么？

<!-- tabs:start -->

#### **参考回答**
常见的垃圾收集器有 7 种，新生代收集器有：Serial、ParNew、Parallel Scavenge，老年代收集器有：Serial Old、Parallel Old、CMS，堆内存垃圾收集器有：G1；

### 1）新生代收集器
- <mark>&nbsp;Serial&nbsp;</mark> 回收的范围是新生代，采用单线程和复制算法进行垃圾收集，它在收集时所有的用户线程都要等待，适合在 Client 场景下使用，能与其搭配的老年代收集器是 CMS 与 Serial Old；
- <mark>&nbsp;ParNew&nbsp;</mark> 收集的范围也是新生代，但它使用多线程和复制算法进行垃圾收集，相当于 Serial 收集器的多线程版本，适合在 Server 场景下使用，能与其搭配的老年代收集器是 CMS 与 Serial Old；
- <mark>&nbsp;Parallel Scavenge&nbsp;</mark> 同样用来回收新生代，采用多线程和复制算法进行垃圾收集，但是它和 ParNew 不同的是，它会尽可能的缩短垃圾收集时用户线程的停顿时间，来达到一个可控制的吞吐量，比如程序一共运行了 100 分钟，垃圾回收占用 1 分钟，那吞吐量就等于 99%；
- 它还可以通过一个开关参数打开 GC 自适应的调节策略（GC Ergonomics），默认是开启的，这样就不需要手工指定新生代的大小（-Xmn）、Eden 和 Survivor 区的比例、晋升老年代对象年龄等细节参数了，虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或者最大的吞吐量；
- 适合比较注重吞吐量，能高效利用 CPU 进行计算，且不需要太多交互场景，因为如果频繁交互，就需要缩短停顿时间，垃圾回收更频繁，吞吐量降低；

### 2）老年代收集器
- <mark>&nbsp;Serial Old &nbsp;</mark> 收集器是 Serial 的老年代版本，采用单线程和标记-整理算法进行垃圾收集，适合在 Client 场景下使用，与 Parallel Scavenge 收集器搭配，作为 CMS 收集器的后备预案；
- <mark>&nbsp;Paraller Old&nbsp;</mark> 是 Parallel Scavenge 的老年代版本，采用多线程和标记-整理算法进行垃圾收集；
- <mark>&nbsp;CMS (Concurrent Mark Sweep)&nbsp;</mark> 从名字上就可以知道它采用的是标记-清除算法，垃圾回收主要包含 4 个步骤：
    - 初始标记：标记一下 GC Roots 能直接关联到的对象，速度较快，需要停顿；
    - 并发标记：进行 GC Roots Tracing，标记出全部的垃圾对象，耗时较长，不需要停顿；
    - 重新标记：为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，需要停顿。
    - 并发清除：用标记-清除算法清除垃圾对象，耗时较长，不需要停顿。
- CMS 收集器也有一些缺点：
    - 并发阶段会和工作现场争抢 CPU 资源；
    - 采用标记-清除算法会产生空间碎片，虚拟机提供一个参数，用于当 CMS 顶不住需要进行 FullGC 时整理空间碎片，但是整理的过程是用户线程是得停止工作的，所以停顿的时间会变长；
    - 浮动垃圾问题，因为在并发清理的时候允许用户线程继续执行，而执行就可能产生新的垃圾进入老年代，所以需要预留一部分空间给这些浮动垃圾，而当这些浮动垃圾过多在 CMS 运行期间爆了，那 CMS 就会出现 Concurrent Mode Failure ，这是时候就得后备的 Serial Old 上来重新进行老年代的垃圾收集，所以停顿的时间就更长了。
    
### 3）G1 收集器
- <mark>&nbsp;G1 (Garbage-First)&nbsp;</mark> 是 HotSpot JDK1.7 时出现的可商用垃圾收集器，它是一款面向服务端应用的垃圾收集器，在多 CPU 和大内存的场景下有很好的性能，HotSpot 团队研发它的目的是为了在以后能替换掉 JDK 1.5 中发布的 CMS 收集器；
- 堆被分为新生代和老年代，其它收集器进行收集的范围要么是新生代，要么是老年代，而 G1 可以直接对新生代和老年代一起回收，能这样做的原因是 G1 把堆划分成多个大小相等的独立区域 Region ，新生代和老年代不再物理隔离，每个 Region 可以单独进行垃圾回收；
- 这种方式带来的好处是可以预测停顿时间，就是把每个 Region 每次垃圾回收的时间和获得的空间记录下来，当做下一次的参考，并维护了一个优先列表，每次根据允许的收集时间，优先回收价值最大的 Region；
- 但也有一个问题，就是如果一个 Region 中的对象引用了另外一个 Region 中的对象，在做可达性分析的时候就需要全堆扫描，效率太低，为了解决这个问题，给每个 Region 加了一个 Remembered Set，用来记录该 Region 对象引用的其他对象所在的 Region，这样就避免了全堆扫描；
- 如果不计算维护 Remembered Set 的操作，G1 收集器的运作大致可划分为以下几个步骤：
    - 初始标记（Initial Marking）：仅仅只是标记一下 GC Roots 能直接关联到的对象，并且修改 TAMS（Nest Top Mark Start）的值，让下一阶段用户程序并发运行时，能在正确可以的 Region 中创建对象，此阶段需要停顿线程，但耗时很短；
    - 并发标记（Concurrent Marking）：从 GC Root 开始对堆中对象进行可达性分析，找到存活对象，此阶段耗时较长，但可与用户程序并发执行；
    - 最终标记（Final Marking）：为了修正在并发标记期间因用户程序继续运作而导致标记产生变动的那一部分标记记录，虚拟机将这段时间对象变化记录在线程的 Remembered Set Logs 里面，最终标记阶段需要把 Remembered Set Logs 的数据合并到 Remembered Set 中，这阶段需要停顿线程，但是可多线程执行。
    - 筛选回收（Live Data Counting and Evacuation）：首先对各个 Region 中的回收价值和成本进行排序，根据用户所期望的 GC 停顿时间来制定回收计划，此阶段其实也可以做到与用户程序一起并发执行，但是因为只回收一部分 Region，时间是用户可控制的，而且停顿用户线程将大幅度提高收集效率。

#### **源码详解**

![](http://images.intflag.com/jvm08.png)

以上是 HotSpot 虚拟机中的 7 个垃圾收集器，连线表示垃圾收集器可以配合使用。

- 单线程与多线程：单线程指的是垃圾收集器只使用一个线程，而多线程使用多个线程；
- 串行与并发：串行指的是垃圾收集器与用户程序交替执行，这意味着在执行垃圾收集的时候需要停顿用户程序；并发指的是垃圾收集器和用户程序同时执行。除了 CMS 和 G1 之外，其它垃圾收集器都是以串行的方式执行。

|收集器|范围|运行模式|算法|目标|适用场景|
|:----:|:----:|:----:|:----:|:----:|:----:|
|Serial|新生代|单线程串行|复制算法|响应速度优先|单CPU环境下的Client模式|
|ParNew|新生代|多线程串行|复制算法|响应速度优先|多CPU环境时在Server模式下与CMS配合|
|Parallel Scavenge|新生代|多线程串行|复制算法|吞吐量优先|在后台运算而不需要太多交互的任务|
|Serial Old|老年代|多线程串行|标记-整理|响应速度优先|单CPU环境下的Client模式、CMS的后备预案|
|Parallel Old|老年代|多线程串行|标记-整理|吞吐量优先|在后台运算而不需要太多交互的任务|
|CMS|老年代|并发|标记-清除|响应速度优先|集中在互联网站或B/S系统服务端上的Java应用|
|G1|both|并发|标记-整理+复制算法|响应速度优先|面向服务端应用，将来替换CMS|

### 1）Serial 收集器

![](http://images.intflag.com/jvm09.png)

Serial 翻译为串行，也就是说它以串行的方式执行。

它是单线程的收集器，只会使用一个线程进行垃圾收集工作。

它的优点是简单高效，在单个 CPU 环境下，由于没有线程交互的开销，因此拥有最高的单线程收集效率。

它是 Client 场景下的默认新生代收集器，因为在该场景下内存一般来说不会很大。它收集一两百兆垃圾的停顿时间可以控制在一百多毫秒以内，只要不是太频繁，这点停顿时间是可以接受的。

### 2）PerNew 收集器

![](http://images.intflag.com/jvm10.png)

它是 Serial 收集器的多线程版本。

它是 Server 场景下默认的新生代收集器，除了性能原因外，主要是因为除了 Serial 收集器，只有它能与 CMS 收集器配合使用。

### 3）Parallel Scavenge 收集器

与 ParNew 一样是多线程收集器。

其它收集器目标是尽可能缩短垃圾收集时用户线程的停顿时间，而它的目标是达到一个可控制的吞吐量，因此它被称为“吞吐量优先”收集器。这里的吞吐量指 CPU 用于运行用户程序的时间占总时间的比值。

停顿时间越短就越适合需要与用户交互的程序，良好的响应速度能提升用户体验。而高吞吐量则可以高效率地利用 CPU 时间，尽快完成程序的运算任务，适合在后台运算而不需要太多交互的任务。

缩短停顿时间是以牺牲吞吐量和新生代空间来换取的：新生代空间变小，垃圾回收变得频繁，导致吞吐量下降。

可以通过一个开关参数打开 GC 自适应的调节策略（GC Ergonomics），就不需要手工指定新生代的大小（-Xmn）、Eden 和 Survivor 区的比例、晋升老年代对象年龄等细节参数了。虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或者最大的吞吐量。

### 4）Serial Old 收集器

![](http://images.intflag.com/jvm11.png)

是 Serial 收集器的老年代版本，也是给 Client 场景下的虚拟机使用。如果用在 Server 场景下，它有两大用途：

- 在 JDK 1.5 以及之前版本（Parallel Old 诞生以前）中与 Parallel Scavenge 收集器搭配使用。
- 作为 CMS 收集器的后备预案，在并发收集发生 Concurrent Mode Failure 时使用。

### 5）Parallel Old 收集器

![](http://images.intflag.com/jvm12.png)

是 Parallel Scavenge 收集器的老年代版本。

在注重吞吐量以及 CPU 资源敏感的场合，都可以优先考虑 Parallel Scavenge 加 Parallel Old 收集器。

### 6）CMS 收集器

![](http://images.intflag.com/jvm13.png)

CMS（Concurrent Mark Sweep），Mark Sweep 指的是标记 - 清除算法。

分为以下四个流程：

- 初始标记：仅仅只是标记一下 GC Roots 能直接关联到的对象，速度很快，需要停顿。
- 并发标记：进行 GC Roots Tracing 的过程，它在整个回收过程中耗时最长，不需要停顿。
- 重新标记：为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，需要停顿。
- 并发清除：不需要停顿。
在整个过程中耗时最长的并发标记和并发清除过程中，收集器线程都可以与用户线程一起工作，不需要进行停顿。

具有以下缺点：

- 吞吐量低：低停顿时间是以牺牲吞吐量为代价的，导致 CPU 利用率不够高。
- 无法处理浮动垃圾，可能出现 Concurrent Mode Failure。浮动垃圾是指并发清除阶段由于用户线程继续运行而产生的垃圾，这部分垃圾只能到下一次 GC 时才能进行回收。由于浮动垃圾的存在，因此需要预留出一部分内存，意味着 CMS 收集不能像其它收集器那样等待老年代快满的时候再回收。如果预留的内存不够存放浮动垃圾，就会出现 Concurrent Mode Failure，这时虚拟机将临时启用 Serial Old 来替代 CMS。
- 标记 - 清除算法导致的空间碎片，往往出现老年代空间剩余，但无法找到足够大连续空间来分配当前对象，不得不提前触发一次 Full GC。

### 7）G1 收集器

G1（Garbage-First），它是一款面向服务端应用的垃圾收集器，在多 CPU 和大内存的场景下有很好的性能。HotSpot 开发团队赋予它的使命是未来可以替换掉 CMS 收集器。

堆被分为新生代和老年代，其它收集器进行收集的范围都是整个新生代或者老年代，而 G1 可以直接对新生代和老年代一起回收。

![](http://images.intflag.com/jvm14.png)

G1 把堆划分成多个大小相等的独立区域（Region），新生代和老年代不再物理隔离。

![](http://images.intflag.com/jvm15.png)

通过引入 Region 的概念，从而将原来的一整块内存空间划分成多个的小空间，使得每个小空间可以单独进行垃圾回收。这种划分方法带来了很大的灵活性，使得可预测的停顿时间模型成为可能。通过记录每个 Region 垃圾回收时间以及回收所获得的空间（这两个值是通过过去回收的经验获得），并维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的 Region。

每个 Region 都有一个 Remembered Set，用来记录该 Region 对象的引用对象所在的 Region。通过使用 Remembered Set，在做可达性分析的时候就可以避免全堆扫描。

![](http://images.intflag.com/jvm16.png)

如果不计算维护 Remembered Set 的操作，G1 收集器的运作大致可划分为以下几个步骤：

- 初始标记
- 并发标记
- 最终标记：为了修正在并发标记期间因用户程序继续运作而导致标记产生变动的那一部分标记记录，虚拟机将这段时间对象变化记录在线程的 Remembered Set Logs 里面，最终标记阶段需要把 Remembered Set Logs 的数据合并到 Remembered Set 中。这阶段需要停顿线程，但是可并行执行。
筛选回收：首先对各个 Region 中的回收价值和成本进行排序，根据用户所期望的 GC 停顿时间来制定回收计划。此阶段其实也可以做到与用户程序一起并发执行，但是因为只回收一部分 Region，时间是用户可控制的，而且停顿用户线程将大幅度提高收集效率。

具备如下特点：

- 空间整合：整体来看是基于“标记 - 整理”算法实现的收集器，从局部（两个 Region 之间）上来看是基于“复制”算法实现的，这意味着运行期间不会产生内存空间碎片。
- 可预测的停顿：能让使用者明确指定在一个长度为 M 毫秒的时间片段内，消耗在 GC 上的时间不得超过 N 毫秒。

### 参考资料
- [cyc2018 垃圾收集器](https://cyc2018.github.io/CS-Notes/#/notes/Java%20%E8%99%9A%E6%8B%9F%E6%9C%BA?id=%e5%9e%83%e5%9c%be%e6%94%b6%e9%9b%86%e5%99%a8)

<!-- tabs:end -->


## 4、内存分配和回收策略
?> **面试题：** 对象在内存上的分配策略具体有哪些？
- <mark>&nbsp;对象优先在 Eden 分配&nbsp;</mark>：大多数情况下，对象在新生代 Eden 区中分配，当Eden区没有足够空间进行分配时，虚拟机将发起一次 Minor GC；
- <mark>&nbsp;大对象直接进入老年代&nbsp;</mark>：因为对象优先在新生代 Eden 区分配的原则，大对象在分配时很可能出现空间不足，这时候需要移动大量小的年轻对象进入年老代，这对 GC 代价很高，所以 JVM 中有一个阈值，如果发现对象大小超过了这个值会直接将对象分配到老年代，当然这个值可以手动修改；
- <mark>&nbsp;长期存活的对象进入老年代&nbsp;</mark>：虚拟机为每个对象都维护一个年龄。如果对象在 Eden 区，经过一次 GC 后依然存活，则被移动到 Survivor 区中，对象年龄加 1，以后，如果对象每经过一次 GC 依然存活，则年龄再加 1，当对象年龄达到阈值时，就移入年老代，默认阈值是15；
- <mark>&nbsp;动态年龄判定&nbsp;</mark>：如果在 Survivor 空间中相同年龄所有对象大小的总和大于Survivor 空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代，不需要等到15岁。
- <mark>&nbsp;空间分配担保&nbsp;</mark>：在发生 Minor GC 之前，虚拟机先检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果大于的话，就认为 Minor GC 是安全的，如果不成立的话虚拟机会查看 HandlePromotionFailure 的值是否允许担保失败，如果允许那么就会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将尝试着进行一次 Minor GC，如果小于，或者 HandlePromotionFailure 的值不允许冒险，那么就要进行一次 Full GC。

?> **面试题：** 什么时候进行 Minor GC，什么时候 Full GC？

- 对于 Minor GC，其触发条件非常简单，当 Eden 空间满时，就将触发一次 Minor GC；
- 而 Full GC 比较复杂，有以下条件：
    - <mark>&nbsp;调用 System.gc()&nbsp;</mark>：只是建议虚拟机执行 Full GC，调用了以后不一定真正执行；
    - <mark>&nbsp;老年代空间不足&nbsp;</mark>：因为超过阈值的大对象会直接在老年代分配，所以当老年代空间不足就会引发 Full GC，所以尽量不要创建太大的对象和数组，除此之外，可以通过一个虚拟机参数调大新生代的大小，让对象尽量在新生代就被回收掉，还可以调大对象进入老年代的年龄，让对象在新生代多活一段时间；
    - <mark>&nbsp;空间分配担保失败&nbsp;</mark>：因为使用复制算法的 Minor GC 需要老年代的内存空间作担保，如果担保失败会执行一次 Full GC；
    - <mark>&nbsp;JDK 1.7 及以前的永久代空间不足&nbsp;</mark>：在 JDK 1.7 及以前，HotSpot 虚拟机中的方法区是用永久代实现的，永久代中存放的为一些 Class 的信息、常量、静态变量等数据，当系统中要加载的类、反射的类和调用的方法较多时，永久代可能会被占满，在未配置为采用 CMS GC 的情况下也会执行 Full GC，如果经过 Full GC 仍然回收不了，那么虚拟机会抛出 java.lang.OutOfMemoryError，为避免以上原因引起的 Full GC，可采用的方法为增大永久代空间或转为使用 CMS GC；
    - <mark>&nbsp;Concurrent Mode Failure&nbsp;</mark>：执行 CMS GC 的过程中同时有对象要放入老年代，而此时老年代空间不足（可能是 GC 过程中浮动垃圾过多导致暂时性的空间不足），便会报 Concurrent Mode Failure 错误，并触发 Full GC。


## 5、类加载机制
?> **面试题：** 你知道一个类的加载过程以及类加载器和双亲委派模型吗？

<!-- tabs:start -->

#### **参考回答**
### 1）类加载过程
- 类是在运行期间第一次被使用的时候动态加载到 JVM 中的；
- 一个类的加载过程包括加载、验证、准备、解析、初始化 5 个阶段；
-<mark>&nbsp; 加载&nbsp;</mark>：主要完成三件事：
    - 通过类的全类名获取一个类的二进制字节流；
    - 将该字节流表示的静态存储结构转换为方法区的运行时存储结构；
    - 在内存中生成一个代表该类的 Class 对象，作为方法区中该类各种数据的访问入口；
- <mark>&nbsp;验证&nbsp;</mark>：根据 Java 虚拟机规范，来验证你加载进来的 .class 字节码文件中的内容是否符合规范，防止字节码被篡改；
- <mark>&nbsp;准备&nbsp;</mark>：为类变量（静态成员变量）分配内存并设置初始值，普通成员变量不会在准备阶段分配内存，它会在实例化时随对象一起被分配在堆中，应该注意到，实例化不是类加载的一个过程，类加载发生在所有实例化操作之前，并且类加载只进行一次，实例化可以进行多次；
- <mark>&nbsp;解析&nbsp;</mark>：将常量池的符号引用替换为直接引用；
- <mark>&nbsp;初始化&nbsp;</mark>：给类变量（静态成员变量）真正赋值一个值，初始化阶段是虚拟机执行类构造器 `<clinit>()` 方法的过程，`<clinit>()` 方法是由编译器自动收集类中所有类变量的赋值动作和静态语句块中的语句合并产生的，编译器收集的顺序由语句在源文件中出现的顺序决定；
- `<clinit>()` 方法是线程安全的，如果多个线程同时初始化一个类，只会有一个线程执行这个类的 `<clinit>()` 方法，其它线程都会阻塞等待，直到活动线程执行 `<clinit>()` 方法完毕。如果在一个类的 `<clinit>()` 方法中有耗时的操作，就可能造成多个线程阻塞，在实际过程中此种阻塞很隐蔽；
- <mark>&nbsp;初始化时机&nbsp;</mark>：
    - 使用 `new` 实例化对象；
    - 包含 `main()` 方法的主类，必须是立马初始化;
    - 如果初始化一个类的时候，发现他的父类还没初始化，那么必须先初始化他的父类。

### 2）类加载器和双亲委派模型
- 一般有 4 种类加载器：
    - 启动类加载器（Bootstrap ClassLoader）：负责加载 `jre1.8.0_102\lib` 下的核心类；
    - 扩展类加载器（Extension ClassLoader）：负责加载 `jre1.8.0_102\lib\ext` 下的扩展类；
    - 应用程序类加载器（Application ClassLoader）：负责去加载 `ClassPath` 环境变量所指定的路径中的类;
    - 自定义类加载器：根据自己的需求加载类；
- JVM的类加载器是有亲子层级结构的，就是说启动类加载器是最上层的，扩展类加载器在第二层，第三层是应用程序类加载器，最后一层是自定义类加载器；
- 双亲委派模型：假设你的应用程序类加载器需要加载一个类，他首先会委派给自己的父类加载器去加载，最终传导到顶层的类加载器去加载，但是如果父类加载器在自己负责加载的范围内，没找到这个类，那么就会下推加载权利给自己的子类加载器。

### 3）整体流程图
![](http://images.intflag.com/jvm20.png)

### 参考资料
- [面试官对于JVM类加载机制的猛烈炮火，你能顶住吗？【石杉的架构笔记】](https://mp.weixin.qq.com/s?__biz=MzU0OTk3ODQ3Ng==&mid=2247485642&idx=1&sn=14542b61ed71e94732f71ab4956049d4&chksm=fba6e0c9ccd169df1329d12712db786076d1da4aaed6a07dca6d45ef3321d6ac6ab27485ad77&mpshare=1&scene=1&srcid=0522QVPE50kBGwFSjDyyOUVQ&sharer_sharetime=1590160642289&sharer_shareid=2565447dd960ce5d1eaca147e7b93e39&key=234ddfccebaed6d3de40636d141901de96ca03cb8f267d0848296f81fc8a3a0bc99846409c200e17a5ebfc82d2a97d13d65178bfa46582c3441573f46e88e0483be368dd74d08364020d64300999ea04&ascene=1&uin=ODMxODEyNzEx&devicetype=Windows+10+x64&version=62090070&lang=zh_CN&exportkey=AwOKF7muaLBI4q7MmhLpmVg%3D&pass_ticket=Ymvfb4S%2Bj%2FEZLlpZpZ9MIKY7FY4b2NSv9nuifvFmC5UNwlNncYmf9nJtN%2BF0jb5X)

#### **源码详解**

类是在运行期间第一次使用时动态加载的，而不是一次性加载所有类。因为如果一次性加载，那么会占用很多的内存。

### 1）类的生命周期

![](http://images.intflag.com/jvm17.png)

包括以下 7 个阶段：

- 加载（Loading）
- 验证（Verification）
- 准备（Preparation）
- 解析（Resolution）
- 初始化（Initialization）
- 使用（Using）
- 卸载（Unloading）

### 2）类加载过程
包含了加载、验证、准备、解析和初始化这 5 个阶段。

**① 加载**

加载是类加载的一个阶段，注意不要混淆。

加载过程完成以下三件事：

- 通过类的完全限定名称获取定义该类的二进制字节流。
- 将该字节流表示的静态存储结构转换为方法区的运行时存储结构。
- 在内存中生成一个代表该类的 Class 对象，作为方法区中该类各种数据的访问入口。

其中二进制字节流可以从以下方式中获取：

- 从 ZIP 包读取，成为 JAR、EAR、WAR 格式的基础。
- 从网络中获取，最典型的应用是 Applet。
- 运行时计算生成，例如动态代理技术，在 java.lang.reflect.Proxy 使用 ProxyGenerator.generateProxyClass 的代理类的二进制字节流。
- 由其他文件生成，例如由 JSP 文件生成对应的 Class 类。

**② 验证**

确保 Class 文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。

**③ 准备**

类变量是被 static 修饰的变量，准备阶段为类变量分配内存并设置初始值，使用的是方法区的内存。

实例变量不会在这阶段分配内存，它会在对象实例化时随着对象一起被分配在堆中。应该注意到，实例化不是类加载的一个过程，类加载发生在所有实例化操作之前，并且类加载只进行一次，实例化可以进行多次。

初始值一般为 0 值，例如下面的类变量 value 被初始化为 0 而不是 123。

```java
public static int value = 123;
```

如果类变量是常量，那么它将初始化为表达式所定义的值而不是 0。例如下面的常量 value 被初始化为 123 而不是 0。

```java
public static final int value = 123;
```

**④ 解析**

将常量池的符号引用替换为直接引用的过程。

其中解析过程在某些情况下可以在初始化阶段之后再开始，这是为了支持 Java 的动态绑定。

**⑤ 初始化**

初始化阶段才真正开始执行类中定义的 Java 程序代码。初始化阶段是虚拟机执行类构造器 `<clinit>()` 方法的过程。在准备阶段，类变量已经赋过一次系统要求的初始值，而在初始化阶段，根据程序员通过程序制定的主观计划去初始化类变量和其它资源。

`<clinit>()` 是由编译器自动收集类中所有类变量的赋值动作和静态语句块中的语句合并产生的，编译器收集的顺序由语句在源文件中出现的顺序决定。特别注意的是，静态语句块只能访问到定义在它之前的类变量，定义在它之后的类变量只能赋值，不能访问。例如以下代码：

```java
public class Test {
    static {
        i = 0;                // 给变量赋值可以正常编译通过
        System.out.print(i);  // 这句编译器会提示“非法向前引用”
    }
    static int i = 1;
}
```

由于父类的 `<clinit>()` 方法先执行，也就意味着父类中定义的静态语句块的执行要优先于子类。例如以下代码：

```java
static class Parent {
    public static int A = 1;
    static {
        A = 2;
    }
}

static class Sub extends Parent {
    public static int B = A;
}

public static void main(String[] args) {
     System.out.println(Sub.B);  // 2
}
```

接口中不可以使用静态语句块，但仍然有类变量初始化的赋值操作，因此接口与类一样都会生成 `<clinit>()` 方法。但接口与类不同的是，执行接口的 `<clinit>()` 方法不需要先执行父接口的 `<clinit>()` 方法。只有当父接口中定义的变量使用时，父接口才会初始化。另外，接口的实现类在初始化时也一样不会执行接口的 `<clinit>()` 方法。

虚拟机会保证一个类的 `<clinit>()` 方法在多线程环境下被正确的加锁和同步，如果多个线程同时初始化一个类，只会有一个线程执行这个类的 `<clinit>()` 方法，其它线程都会阻塞等待，直到活动线程执行 `<clinit>()` 方法完毕。如果在一个类的 `<clinit>()` 方法中有耗时的操作，就可能造成多个线程阻塞，在实际过程中此种阻塞很隐蔽。

### 3）类初始化时机

**① 主动引用**

虚拟机规范中并没有强制约束何时进行加载，但是规范严格规定了有且只有下列五种情况必须对类进行初始化（加载、验证、准备都会随之发生）：

- 遇到 new、getstatic、putstatic、invokestatic 这四条字节码指令时，如果类没有进行过初始化，则必须先触发其初始化。最常见的生成这 4 条指令的场景是：
    - 使用 new 关键字实例化对象的时候；
    - 读取或设置一个类的静态字段（被 final 修饰、已在编译期把结果放入常量池的静态字段除外）的时候；
    - 以及调用一个类的静态方法的时候。
- 使用 java.lang.reflect 包的方法对类进行反射调用的时候，如果类没有进行初始化，则需要先触发其初始化。
- 当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。
- 当虚拟机启动时，用户需要指定一个要执行的主类（包含 main() 方法的那个类），虚拟机会先初始化这个主类；
- 当使用 JDK 1.7 的动态语言支持时，如果一个 java.lang.invoke.MethodHandle 实例最后的解析结果为 REF_getStatic, REF_putStatic, REF_invokeStatic 的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化；

**② 被动引用**

以上 5 种场景中的行为称为对一个类进行主动引用。除此之外，所有引用类的方式都不会触发初始化，称为被动引用。被动引用的常见例子包括：

- 通过子类引用父类的静态字段，不会导致子类初始化。

```java
System.out.println(SubClass.value);  // value 字段在 SuperClass 中定义
```

- 通过数组定义来引用类，不会触发此类的初始化。该过程会对数组类进行初始化，数组类是一个由虚拟机自动生成的、直接继承自 Object 的子类，其中包含了数组的属性和方法。

```java
SuperClass[] sca = new SuperClass[10];
```

- 常量在编译阶段会存入调用类的常量池中，本质上并没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化。

```java
System.out.println(ConstClass.HELLOWORLD);
```

### 4）类与类加载器
两个类相等，需要类本身相等，并且使用同一个类加载器进行加载。这是因为每一个类加载器都拥有一个独立的类名称空间。

这里的相等，包括类的 Class 对象的 equals() 方法、isAssignableFrom() 方法、isInstance() 方法的返回结果为 true，也包括使用 instanceof 关键字做对象所属关系判定结果为 true。

### 5）类加载器的分类
从 Java 虚拟机的角度来讲，只存在以下两种不同的类加载器：

- 启动类加载器（Bootstrap ClassLoader），使用 C++ 实现，是虚拟机自身的一部分；

- 所有其它类的加载器，使用 Java 实现，独立于虚拟机，继承自抽象类 java.lang.ClassLoader。

从 Java 开发人员的角度看，类加载器可以划分得更细致一些：

- 启动类加载器（Bootstrap ClassLoader）此类加载器负责将存放在 `<JRE_HOME>\lib` 目录中的，或者被 -Xbootclasspath 参数所指定的路径中的，并且是虚拟机识别的（仅按照文件名识别，如 rt.jar，名字不符合的类库即使放在 lib 目录中也不会被加载）类库加载到虚拟机内存中。启动类加载器无法被 Java 程序直接引用，用户在编写自定义类加载器时，如果需要把加载请求委派给启动类加载器，直接使用 null 代替即可。

- 扩展类加载器（Extension ClassLoader）这个类加载器是由 ExtClassLoader（sun.misc.Launcher$ExtClassLoader）实现的。它负责将 `<JAVA_HOME>/lib/ext` 或者被 java.ext.dir 系统变量所指定路径中的所有类库加载到内存中，开发者可以直接使用扩展类加载器。

- 应用程序类加载器（Application ClassLoader）这个类加载器是由 AppClassLoader（sun.misc.Launcher$AppClassLoader）实现的。由于这个类加载器是 ClassLoader 中的 getSystemClassLoader() 方法的返回值，因此一般称为系统类加载器。它负责加载用户类路径（ClassPath）上所指定的类库，开发者可以直接使用这个类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

### 6）双亲委派模型
应用程序是由三种类加载器互相配合从而实现类加载，除此之外还可以加入自己定义的类加载器。

下图展示了类加载器之间的层次关系，称为双亲委派模型（Parents Delegation Model）。该模型要求除了顶层的启动类加载器外，其它的类加载器都要有自己的父类加载器。这里的父子关系一般通过组合关系（Composition）来实现，而不是继承关系（Inheritance）。

![](http://images.intflag.com/jvm18.png)

**① 工作过程**

一个类加载器首先将类加载请求转发到父类加载器，只有当父类加载器无法完成时才尝试自己加载。

**② 好处**

使得 Java 类随着它的类加载器一起具有一种带有优先级的层次关系，从而使得基础类得到统一。

例如 java.lang.Object 存放在 rt.jar 中，如果编写另外一个 java.lang.Object 并放到 ClassPath 中，程序可以编译通过。由于双亲委派模型的存在，所以在 rt.jar 中的 Object 比在 ClassPath 中的 Object 优先级更高，这是因为 rt.jar 中的 Object 使用的是启动类加载器，而 ClassPath 中的 Object 使用的是应用程序类加载器。rt.jar 中的 Object 优先级更高，那么程序中所有的 Object 都是这个 Object。

**③ 实现**

以下是抽象类 java.lang.ClassLoader 的代码片段，其中的 loadClass() 方法运行过程如下：先检查类是否已经加载过，如果没有则让父类加载器去加载。当父类加载器加载失败时抛出 ClassNotFoundException，此时尝试自己去加载。

```java
public abstract class ClassLoader {
    // The parent class loader for delegation
    private final ClassLoader parent;

    public Class<?> loadClass(String name) throws ClassNotFoundException {
        return loadClass(name, false);
    }

    protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    c = findClass(name);
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }

    protected Class<?> findClass(String name) throws ClassNotFoundException {
        throw new ClassNotFoundException(name);
    }
}
```

### 7）自定义类加载器实现
以下代码中的 FileSystemClassLoader 是自定义类加载器，继承自 java.lang.ClassLoader，用于加载文件系统上的类。它首先根据类的全名在文件系统上查找类的字节代码文件（.class 文件），然后读取该文件内容，最后通过 defineClass() 方法来把这些字节代码转换成 java.lang.Class 类的实例。

java.lang.ClassLoader 的 loadClass() 实现了双亲委派模型的逻辑，自定义类加载器一般不去重写它，但是需要重写 findClass() 方法。

```java
public class FileSystemClassLoader extends ClassLoader {

    private String rootDir;

    public FileSystemClassLoader(String rootDir) {
        this.rootDir = rootDir;
    }

    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] classData = getClassData(name);
        if (classData == null) {
            throw new ClassNotFoundException();
        } else {
            return defineClass(name, classData, 0, classData.length);
        }
    }

    private byte[] getClassData(String className) {
        String path = classNameToPath(className);
        try {
            InputStream ins = new FileInputStream(path);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            int bufferSize = 4096;
            byte[] buffer = new byte[bufferSize];
            int bytesNumRead;
            while ((bytesNumRead = ins.read(buffer)) != -1) {
                baos.write(buffer, 0, bytesNumRead);
            }
            return baos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    private String classNameToPath(String className) {
        return rootDir + File.separatorChar
                + className.replace('.', File.separatorChar) + ".class";
    }
}
```

<!-- tabs:end -->

## 6、JVM 调优
?> **面试题：** 工作中对 JVM 做了哪些调优操作？

<!-- tabs:start -->

#### **参考回答**
|配置参数|功能|
|:----|:----|
|-Xms|初始堆大小。如：-Xms256m|
|-Xmx|最大堆大小。如：-Xmx512m|
|-XX:+HeapDumpOnOutOfMemoryError|让虚拟机在发生内存溢出时 Dump 出当前的<br>内存堆转储快照，以便分析用|
|-XX:+UseConcMarkSweepGC|指定老年代的GC收集器为 CMS|

#### **源码详解**



<!-- tabs:end -->


各种oom的种类，
jvm调优经验，没有你也要做过，自己去设置启动参数，知道常见参数的含义，