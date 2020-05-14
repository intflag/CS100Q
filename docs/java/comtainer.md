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
**参考回答：**
- ArrayList 底层是数组实现的，支持随机访问，默认大小是 10；
- 在调用 add 方法添加元素的时候，它底层使用 ensureCapacityInternal 方法来保证数组容量足够，如果发现不够了，就会调用 grow 方法进行扩容，扩充后的容量为：oldCapacity + (oldCapacity >> 1) ，也就是扩容前的 1.5 倍，同时它会使用 Arrays.copyOf 方法拷贝数组，这个操作代价比较高，所以最好在创建 ArrayList 的时候就根据实际情况指定一个大概的容量，避免它频繁扩容；
- 在调用 remove 方法删除元素的时候，实际上是将 index + 1 后面的元素依次复制到 index 位置上，这个操作的时间复杂度是 O(n) ，也是比较高的；
- 在源码里面，存储数据的数组 elementData 被 transient 修饰，所以默认不会序列化，但是它实现了 writeObject 和 readObject 来控制只序列化数组中有元素填充的部分；
- 最后一比较重要的点是，ArrayList 具有一个快速失败的机制，在源码里面有一个 modCount 变量，用来记录 ArrayList 结构发生变化的次数，在进行序列化或者是迭代操作前后，都会判断这个变量是否发生了改变，如果改变了就会抛出 ConcurrentModificationException 并发修改异常。

### 2）Vector
**参考回答：**
- Vector 底层也是数据实现的，和 ArrayList 类似，而且看过源码就知道，它在 JDK1.0 就有了，而 ArrayList 是 1.2 以后有的；
- 它跟 ArrayList 最大的区别就是，ArrayList 不是线程安全的，而 Vector 是线程安全，因为它的添加，查找，删除等方法都用了 synchronized 关键字进行同步；
- 然后他的默认容量也是 10，它内部保证容量足够的方法叫做 ensureCapacityHelper，如果新建的时候没有使用有参构造，默认扩容后的容量是之前的 2 倍。

## 2、LinkedList 的实现原理以及和 ArrayList 相比优缺点是什么？
**参考回答：**
- LinkedList 底层是基于双向链表实现的，有一个静态内部类 Node，里面有前驱指针和后继指针，而且每个链表存储了 first 和 last 指针，在查找的时候会判断 index < (size >> 1) ，也就是如果索引靠左，就从 first 指针往后搜索，否则从 last 指针往前搜索；
- 和 ArrayList 相比的话，LinkedList 不支持随机访问，但插入和删除只需要先遍历一遍找到插入或删除的节点，然后改变指针的指向就可以了。

## 3、在不使用 Vector 的情况下如何实现线程安全的 ArrayList？
**参考回答：**
- 不使用 Vector 的情况下，实现线程安全的 ArrayList 有两种方法，一种是使用 Collections.synchronizedList() 得到一个线程安全的 ArrayList，另一种是使用 concurrent 并发包下的 CopyOnWriteArrayList 类；
- CopyOnWriteArrayList 是读写分离的，写操作在一个复制的数组上进行，读操作还是在原始数组中进行，读写分离，互不影响；
- 写操作前会用 ReentrantLock 加锁，写入后会把引用指向新数组，然后在 finally 里面释放锁；
CopyOnWriteArrayList 大大提高了读操作的性能，适合读多写少的场景，同时它也有缺陷，比如内存占用是原来的两倍左右，还有可能会导致数据不一致的现象（读操作的时候写操作还没有同步到数组中），所以不适合内存敏感以及对实时性要求很高的场景。

## 4、HashMap 和 HashTable 的实现原理和区别以及 JDK8 之后 HashMap的变化？
### 1）HashMap
**参考回答：**
- HashMap 是 哈希表的一种实现，在 JDK1.7 中，内部包含了一个 Entry 类型的数据 table；
- Entry 里有四个字段，分别是 hash、key、value、next 指针，所以 Entry 是一个单链表，数组的每个位置可以看作一个桶，每个桶存放一个链表，也就是说，HashMap 是用拉链法解决哈希冲突的，并且元素插入链表的方式是头插法；
- HashMap 默认的初始容量是 16，在添加元素的时候，如果发现键值对的数量大于等于一个临界值，就会进行扩容，这个临界值为：容量 X 装载因子，默认是 0.75，扩充后的容量是之前的 2 倍；
- 扩容会调用 resize 方法，会把 oldTable 的所有键值对重新插入 newTable 中，这个操作代价是比较高的，所以和使用 ArrayList 一样，尽量根据实际情况指定一个容量，避免它频繁扩容；
- JDK7 HashMap 的容量一定是 2 的 n 次方，及时构造函数传了一个不是 2 的 n 次方的容量，也会使用 roundUpToPowerOf2 方法，保证是 2 的 n 次方；
- 在计算桶下标时会用 hash & (length-1)，因为在容量是 2 的 n 次方前提下，哈希值和容量减一做与运算的结果，等价于 hash % length，位运算的速度更快；
- 可以存储 null 键值对，因为 null 没有 hashCode 方法，所以会被存储到第 0 个桶内；
- 从 JDK8 开始，当链表的长度 >= 8 的时候，会将链表转为红黑树，插入的时候采用的是尾插法，能够避免出现逆序且链表死循环的问题；
- 至于为什么在长度 >= 8 的时候转，是因为 HashMap 遵循泊松分布，也就是哈希冲突满足小概率事件、独立、稳定三个条件，最后从概率统计的角度出发选择了 8；然后选择转为红黑树的原因是，红黑树在添加、查找、删除元素的时候，时间复杂度都为 O(log n)，比较稳定；
- HashMap 是线程不安全的。

### 2）HashTable
**参考回答：**
- HashTable 也是哈希表的一种实现，也是一个 Entry 类型的数组存储键值对，比 HashMap 的出现要早，在 JDK1.0 就已经有了，所以最初没有实现 Map 接口（JDK1.2），而是继承了 Dictionary 字典类；
- HashTable 默认容量是 11，然后每次扩容是：(oldCapacity << 1) + 1；
- HashTable 是线程安全的，因为它的 put、get、remove 等方法都用 synchronized 进行同步，但这种方式效率比较低，当一个线程使用 put 方法添加元素的时候，不但不能使用 put 方法，连 get 方法也不能使用，也就是说会竞争同一把锁，竞争越激烈效率越低，所以现在不推荐使用了；

## 5、ConcurrentHashMap 的实现原理以及它是如何保证线程安全的？
**参考回答：**
- 在并发环境下应该使用 ConcurrentHashMap，它与 HashMap 的实现类似，但是与 HashTable 在并发下竞争同一把锁不同的是，ConcurrentHashMap 采用了分段锁（Segment），每个分段锁维护着几个桶（HashEntry），多个线程可以同时访问不同分段锁上的桶，从而使其并发度更高（并发度就是 Segment 的个数），默认的并发级别是 16，也就是默认会创建 16 个 Segment；
- 
## 6、LinkedHashMap 的实现原理以及如何用它来实现 LRU 缓存？
## 7、Tomcat 中的 ConcurrentCache 底层是用什么实现的？
## 8、优先级队列的底层原理是什么？