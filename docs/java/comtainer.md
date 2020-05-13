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

## 5、ConcurrentHashMap 的实现原理以及它是如何保证线程安全的？
## 6、LinkedHashMap 的实现原理以及如何用它来实现 LRU 缓存？
## 7、Tomcat 中的 ConcurrentCache 底层是用什么实现的？
## 8、优先级队列的底层原理是什么？