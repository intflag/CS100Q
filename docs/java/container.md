# Java 容器
## 0、Java 的容器技术总结
### 1）Collection
![](http://images.intflag.com/container01.png)

**① Set**
- TreeSet：基于红黑树实现，支持有序性操作，例如根据一个范围查找元素的操作。但是查找效率不如 HashSet，HashSet 查找的时间复杂度为 O(1)，TreeSet 则为 O(logN)。

- HashSet：基于哈希表实现，支持快速查找，但不支持有序性操作。并且失去了元素的插入信息，也就是说使用 Iterator 遍历 HashSet 得到的结果是不确定的。

- LinkedHashSet：具有HashSet的查找效率，并且内部使用双向链表维护数据的插入顺序。

**② List**
- ArrayList：基于动态数据实现，支持随机访问。

- Vector：和 ArrayList 类似，但它是线程安全的。

- LinkedList：基于双向链表实现，只能顺序访问，但是可以快速地在链表中间插入和删除元素。不仅如此，LinkedList 可以用作栈、队列和双向队列。

**③ Queue**
- LinkedList：可以用它来实现双向队列。

- PriorityQueue：基于堆结构实现，可以用它来实现优先队列。

### 2）Map
![](http://images.intflag.com/container02.png)

- TreeMap：基于红黑树实现。

- HashMap：基于哈希表实现。

- HashTable：和 HashMap 类似。但它是线程安全的，这意味着同一时刻多个线程同时写入 HashTable 不会导致数据不一致。它是遗留类，不应该去使用它，而是使用效率更高的 ConcurrentHashMap 来支持线程安全，因为 ConcurrentHashMap 引入了分段锁。

- LinkedHashMap：使用双向链表来维护元素的顺序，顺序为插入顺序或者最近最少使用（LRU）顺序。

## 1、ArrayList 和 Vector 的原理和区别是什么？
### 1）ArrayList
<!-- tabs:start -->

#### **参考回答**

- ArrayList 底层是数组实现的，支持随机访问，默认大小是 10；
- 在调用 add 方法添加元素的时候，它底层使用 ensureCapacityInternal 方法来保证数组容量足够，如果发现不够了，就会调用 grow 方法进行扩容，扩充后的容量为：oldCapacity + (oldCapacity >> 1) ，也就是扩容前的 1.5 倍，同时它会使用 Arrays.copyOf 方法拷贝数组，这个操作代价比较高，所以最好在创建 ArrayList 的时候就根据实际情况指定一个大概的容量，避免它频繁扩容；
- 在调用 remove 方法删除元素的时候，实际上是将 index + 1 后面的元素依次复制到 index 位置上，这个操作的时间复杂度是 O(n) ，也是比较高的；
- 在源码里面，存储数据的数组 elementData 被 transient 修饰，所以默认不会序列化，但是它实现了 writeObject 和 readObject 来控制只序列化数组中有元素填充的部分；
- 最后一比较重要的点是，ArrayList 具有一个快速失败的机制，在源码里面有一个 modCount 变量，用来记录 ArrayList 结构发生变化的次数，在进行序列化或者是迭代操作前后，都会判断这个变量是否发生了改变，如果改变了就会抛出 ConcurrentModificationException 并发修改异常。

#### **源码详解**



<!-- tabs:end -->

### 2）Vector
<!-- tabs:start -->

#### **参考回答**

- Vector 底层也是数据实现的，和 ArrayList 类似，而且看过源码就知道，它在 JDK1.0 就有了，而 ArrayList 是 1.2 以后有的；
- 它跟 ArrayList 最大的区别就是，ArrayList 不是线程安全的，而 Vector 是线程安全，因为它的添加，查找，删除等方法都用了 synchronized 关键字进行同步；
- 然后他的默认容量也是 10，它内部保证容量足够的方法叫做 ensureCapacityHelper，如果新建的时候没有使用有参构造，默认扩容后的容量是之前的 2 倍。

#### **源码详解**



<!-- tabs:end -->

## 2、LinkedList 的实现原理以及和 ArrayList 相比优缺点是什么？
<!-- tabs:start -->

#### **参考回答**

- LinkedList 底层是基于双向链表实现的，有一个静态内部类 Node，里面有前驱指针和后继指针，而且每个链表存储了 first 和 last 指针，在查找的时候会判断 index < (size >> 1) ，也就是如果索引靠左，就从 first 指针往后搜索，否则从 last 指针往前搜索；
- 和 ArrayList 相比的话，LinkedList 不支持随机访问，但插入和删除只需要先遍历一遍找到插入或删除的节点，然后改变指针的指向就可以了。

#### **源码详解**



<!-- tabs:end -->

## 3、在不使用 Vector 的情况下如何实现线程安全的 ArrayList？
<!-- tabs:start -->

#### **参考回答**

- 不使用 Vector 的情况下，实现线程安全的 ArrayList 有两种方法，一种是使用 Collections.synchronizedList() 得到一个线程安全的 ArrayList，另一种是使用 concurrent 并发包下的 CopyOnWriteArrayList 类；
- CopyOnWriteArrayList 是读写分离的，写操作在一个复制的数组上进行，读操作还是在原始数组中进行，读写分离，互不影响；
- 写操作前会用 ReentrantLock 加锁，写入后会把引用指向新数组，然后在 finally 里面释放锁；
CopyOnWriteArrayList 大大提高了读操作的性能，适合读多写少的场景，同时它也有缺陷，比如内存占用是原来的两倍左右，还有可能会导致数据不一致的现象（读操作的时候写操作还没有同步到数组中），所以不适合内存敏感以及对实时性要求很高的场景。

#### **源码详解**



<!-- tabs:end -->

## 4、HashMap 和 HashTable 的实现原理和区别以及 JDK8 之后 HashMap的变化？
### 1）HashMap
<!-- tabs:start -->

#### **参考回答**

- HashMap 是 哈希表的一种实现，在 JDK1.7 中，内部包含了一个 Entry 类型的数组 table；
- Entry 里有四个字段，分别是 hash、key、value、next 指针，所以 Entry 是一个单链表，数组的每个位置可以看作一个桶，每个桶存放一个链表，也就是说，HashMap 是用拉链法解决哈希冲突的，并且元素插入链表的方式是头插法；
- HashMap 默认的初始容量是 16，在添加元素的时候，如果发现键值对的数量大于等于一个临界值，就会进行扩容，这个临界值为：容量 * 装载因子，默认是 0.75，扩充后的容量是之前的 2 倍；
- 扩容会调用 resize 方法，会把 oldTable 的所有键值对重新插入 newTable 中，这个操作代价是比较高的，所以和使用 ArrayList 一样，尽量根据实际情况指定一个容量，避免它频繁扩容；
- JDK7 HashMap 的容量一定是 2 的 n 次方，就算构造函数传了一个不是 2 的 n 次方的容量，也会使用 roundUpToPowerOf2 方法，保证是 2 的 n 次方；
- 在计算桶下标时会用 hash & (length-1)，因为在容量是 2 的 n 次方前提下，哈希值和容量减一做与运算的结果，等价于 hash % length，位运算的速度更快；
- 可以存储 null 键值对，因为 null 没有 hashCode 方法，所以会被存储到第 0 个桶内；
- 从 JDK8 开始，当链表的长度 >= 8 的时候，会将链表转为红黑树，插入的时候采用的是尾插法，能够避免出现逆序且链表死循环的问题；
- 至于为什么在长度 >= 8 的时候转，是因为 HashMap 遵循泊松分布，也就是哈希冲突满足小概率事件、独立、稳定三个条件，最后从概率统计的角度出发选择了 8；然后选择转为红黑树的原因是，红黑树在添加、查找、删除元素的时候，时间复杂度都为 O(log n)，比较稳定；
- HashMap 是线程不安全的。

#### **源码详解**



<!-- tabs:end -->

### 2）HashTable
<!-- tabs:start -->

#### **参考回答**

- HashTable 也是哈希表的一种实现，也是一个 Entry 类型的数组存储键值对，比 HashMap 的出现要早，在 JDK1.0 就已经有了，所以最初没有实现 Map 接口（JDK1.2），而是继承了 Dictionary 字典类；
- HashTable 默认容量是 11，然后每次扩容是：(oldCapacity << 1) + 1；
- HashTable 是线程安全的，因为它的 put、get、remove 等方法都用 synchronized 进行同步，但这种方式效率比较低，当一个线程使用 put 方法添加元素的时候，不但不能使用 put 方法，连 get 方法也不能使用，也就是说会竞争同一把锁，竞争越激烈效率越低，所以现在不推荐使用了；
- HashTable 不支持存储 null 键值对。

#### **源码详解**



<!-- tabs:end -->

## 5、ConcurrentHashMap 的实现原理以及它是如何保证线程安全的？
<!-- tabs:start -->

#### **参考回答**

- 在并发环境下应该使用 ConcurrentHashMap，它与 HashMap 的实现类似，但是与 HashTable 在并发下竞争同一把锁不同的是，ConcurrentHashMap 采用了分段锁（Segment），每个分段锁维护着几个桶（HashEntry），多个线程可以同时访问不同分段锁上的桶，从而使其并发度更高（并发度就是 Segment 的个数），默认的并发级别是 16，也就是默认会创建 16 个 Segment；
- 在插入和获取元素的时候，先通过哈希算法定位到 Segment，然后会用变种算法进行一次再哈希，大大减少了哈希冲突发生的概率；
- get 方法高效的原因是因为它所有要使用的共享变量都用 volatile 修饰了，我们都知道，被 volatile 修饰的变量能够在线程之间保持可见性，能够被多线程同时读，并且保证不会读到过期的值，但是只能被单线程写（有一种情况可以被多线程写，就是写入的值不依赖于原值）；
- 它在执行 size 方法计算所有元素个数的时候，会遍历所有 Segment 然后把每个 Segment 中的 count 累加起来，如果连续两次不加锁操作得到的结果一致，那么可以认为这个结果是正确的，否则会对所有 Segment 都加锁；
- JDK8 使用了 CAS 操作来支持更高的并发度，在 CAS 操作失败时使用内置锁 synchronized，并且 JDK8 在链表过长时也会转换为红黑树。

#### **源码详解**



<!-- tabs:end -->

## 6、LinkedHashMap 的实现原理以及如何用它来实现 LRU 缓存？
<!-- tabs:start -->

#### **参考回答**

- LinkedHashMap 继承自 HashMap，所以具有和 HashMap 一样的快速查找特性，在内部维护了一个双向链表，用来实现元素的插入顺序或者 LRU 顺序；
- 它内部有一个变量 accessOrder，默认为 false，维护的是插入顺序，当 accessOrder 为 true 的时候维护 LRU 顺序，每次访问一个节点时，会将这个节点移到链表尾部，保证链表尾部是最近访问的节点，那链表首部就是最近最久未使用的节点。

#### **源码详解**



<!-- tabs:end -->

## 7、你知道 Tomcat 中的 ConcurrentCache 底层是用什么实现的吗？
<!-- tabs:start -->

#### **参考回答**

- Tomcat 中的 ConcurrentCache 使用了 WeakHashMap 来实现缓存功能，WeakHashMap 的 Entry 继承自 WeakReference，被 WeakReference 关联的对象在下一次垃圾回收时会被回收；
- ConcurrentCache 采取的是分代缓存思想：
    - 经常使用的对象放入 eden 中，eden 使用 ConcurrentHashMap 实现，不用担心会被回收（伊甸园）；
    - 不常用的对象放入 longterm，longterm 使用 WeakHashMap 实现，这些老对象会被垃圾收集器回收；
    - 当调用 get() 方法时，会先从 eden 区获取，如果没有找到的话再到 longterm 获取，当从 longterm 获取到就把对象放入 eden 中，从而保证经常被访问的节点不容易被回收；
    - 当调用 put() 方法时，如果 eden 的大小超过了 size，那么就将 eden 中的所有对象都放入 longterm 中，利用虚拟机回收掉一部分不经常使用的对象。

#### **源码详解**



<!-- tabs:end -->

## 8、优先级队列的底层原理是什么，多线程下可以使用吗？
### 1）PriorityQueue
<!-- tabs:start -->

#### **参考回答**

- 在 Java 中，优先级队列 `PriorityQueue` 底层是用`堆`实现的，需要先[了解堆这种数据结构](algorithm/classical?id=_1、如何理解堆，它的应用场景有哪些？)。
- PriorityQueue 是在 JDK 1.5 引入的，无参构造默认自然排序，有参构造需要传入一个 Comparator 比较器，它不允许空值，也不支持不可比较的对象，往队列中添加的对象要么实现 `Comparable` 接口，重写 `compareTo` 方法，要么根据对象的类型创建一个比较器 `Comparator`，实现 `compare` 方法，否则会报错；
- 优先队列默认容量是 11，在使用 add 方法添加元素时，真正调用的是 offer 方法，如果发现队列元素个数大于等于队列容量时，就会调用 grow 方法进行扩容，扩容会判断旧容量是否小于 64，如果小于，会扩容 2 * oldCapacity + 2，否则会扩容 1.5 倍，然后会使用 Arrays.copyOf 方法拷贝数据；
- offer 入队、poll 出队、removeAt 删除指定位置元素方法的时间复杂度都是 O(logn)，remove 删除指定元素方法为 O(n)，peek 方法为 O(1)；
- PriorityQueue 不是线程安全的。

#### **源码详解**



<!-- tabs:end -->

### 2）PriorityBlockingQueue
<!-- tabs:start -->

#### **参考回答**

- PriorityBlockingQueue 实现原理和 PriorityQueue 类似，但它是线程安全的，因为它使用 ReentrantLock 和 Condition 来确保多线程环境下的同步问题，入队、出队、删除元素等方法都会在操作前先获取锁然后加锁，操作完毕后在 finally 中释放锁；
- 它也会动态扩容，扩容方法叫做 tryGrow，扩容前会释放锁，允许在扩容的时候进行入队操作，大大提高了并发性能，但是它采用了 CAS 机制，保证扩容时只能有一个线程进行，避免出现多个线程同时扩容的问题。

#### **源码详解**



<!-- tabs:end -->