# Redis
## 1、简介及优势
?> **面试题：** 简单介绍一下 Redis 以及为什么使用它？

<!-- tabs:start -->

#### **参考回答**

### 1）Redis简介
- Redis 是一个 key-value 数据库，和传统的数据库不同的是，它的数据可以存在内存中，所以读写速度非常快，因此经常用它来对数据进行缓存；
- 除了保存在内存中还可以持久化到磁盘上，另外 Redis 也经常用来做分布式锁；
- Redis 提供多种数据类型支持不同的业务场景，还支持事务、LUA脚本、LRU驱动事件、多种集群方案。

### 2）Redis 优势
- <mark>&nbsp;高性能&nbsp;</mark>：通常情况下，一些频繁使用且很少更新的数据会用 Redis 进行缓存，当第一次访问时会从数据库加载，之后访问会直接从 Redis 中获取，数据库更新时同步更新 Redis 中的数据，采用这种方式可以大大提高程序的性能；
- <mark>&nbsp;高并发&nbsp;</mark>：因为 Redis 中的数据是存储在内存中的，直接操作内存能够承受的请求是远远大于直接访问数据库的，所以可以实现高并发；
- <mark>&nbsp;分布式&nbsp;</mark>：缓存分为本地缓存和分布式缓存，Redis 可以支持分布式缓存，多个实例共用一份缓存，数据拥有一致性。

#### **源码详解**



<!-- tabs:end -->

## 2、单线程模型
?> **面试题：** Redis 是线程安全的吗，实现原理是什么？

<!-- tabs:start -->

#### **参考回答**
- Redis 内部使用了文件事件处理器 `file event handler`，这个文件事件处理器是单线程的，所以说 Redis 是单线程模型，底层采用 epoll，利用 epoll 的多路复用特性对网络请求进行处理；
- Redis 只有网络请求模块是单线程的，所以并发下的网络请求是线程安全的，而其他模块是多线程的;
- 文件事件处理器的结构包含 4 个部分：
    - 多个 socket;
    - IO 多路复用程序;
    - 文件事件分派器;
    - 事件处理器（连接应答处理器、命令请求处理器、命令回复处理器）;
- 多个 socket 可能会并发产生不同的操作，每个操作对应不同的文件事件，但是 IO 多路复用程序会监听多个 socket，会将 socket 产生的事件放入队列中排队，事件分派器每次从队列中取出一个事件，把该事件交给对应的事件处理器进行处理。


#### **源码详解**



<!-- tabs:end -->

## 3、数据结构
?> **面试题：** Redis 有哪几种数据结构，各自的使用场景是什么？

<!-- tabs:start -->

#### **参考回答**
### 1）String
- 常规的 key-value 缓存，key 是字符串，value 可以是字符串也可以是数字；
- 常用命令：set、get、mget、incr、decr。

### 2）Hash
- 保存哈希表，适合存储对象；
- 常用命令：hset、hget、hgetall。

### 3）List
- Redis List 相当于一个队列，和 Java 中的 LinkedList 类似，其实他的实现也是一个双向链表，支持反向查找和遍历；
- 另外可以通过 lrange 命令，就是从某个元素开始读取多少个元素，可以基于这个特性实现分页查询，比如常见的下拉加载下一页，性能比较高。
- 常用命令：lpush、rpush、lpop、rpop、lrange。

### 4）Set
- Redis 的 Set 是 String 类型的无序集合，集合成员是唯一的，这就意味着集合中不能出现重复的数据；
- Redis 中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)；
- Set 可以实现交集、并集、差集的操作，比如：在社交软件中，可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合，Redis可以非常方便的实现如共同关注、共同粉丝、共同喜好等功能，求交集可以用 `sinterstore` 命令。

### 5）Sorted Set
- 和set相比，sorted set 增加了一个权重参数 score，使得集合中的元素能够按 score 进行有序排列；
- 在直播系统中，实时排行信息包含直播间在线用户列表，各种礼物排行榜，弹幕消息（可以理解为按消息维度的消息排行榜）等信息，适合使用 Redis 中的 Sorted Set 结构进行存储；
- 常用命令：zadd、zrange、zrem、zcard。

#### **源码详解**

### 1）String
```bash
127.0.0.1:6379> set hello_redis "Hello Redis"
OK
127.0.0.1:6379> keys *
1) "hello_redis"

127.0.0.1:6379> get hello_redis
"Hello Redis"

127.0.0.1:6379> set hello_int 1
OK
127.0.0.1:6379> get hello_int
"1"
127.0.0.1:6379> incr hello_int
(integer) 2
127.0.0.1:6379> get hello_int
"2"
127.0.0.1:6379> decr hello_int
(integer) 1
127.0.0.1:6379> get hello_int
"1"
127.0.0.1:6379> mget hello_redis hello_int
1) "Hello Redis"
2) "1"
```
### 2）Hash
```bash
127.0.0.1:6379> hmset user_map name "tom" age 18 local "beijing"
OK

127.0.0.1:6379> HGETAll user_map
1) "name"
2) "tom"
3) "age"
4) "18"
5) "local"
6) "beijing"

127.0.0.1:6379> hget user_map name
"tom"
127.0.0.1:6379> HSET user_map name "tom2"
(integer) 0
127.0.0.1:6379> hget user_map name
"tom2"
127.0.0.1:6379> HSET user_map age 25
(integer) 0
127.0.0.1:6379> hget user_map age
"25"
```

### 3）List
```bash
127.0.0.1:6379> lpush hello_list "zhangsan"
(integer) 1
127.0.0.1:6379> lpush hello_list "lisi"
(integer) 2
127.0.0.1:6379> lpush hello_list "wangwu" "zhangliu"
(integer) 4

127.0.0.1:6379> llen hello_list
(integer) 4

127.0.0.1:6379> lindex hello_list 2
"lisi"

127.0.0.1:6379> lpop hello_list
"zhangliu"

127.0.0.1:6379> llen hello_list
(integer) 3

127.0.0.1:6379> lset hello_list 2 "tom"
OK

127.0.0.1:6379> lindex hello_list 2
"tom"
```

### 4）Set
```bash
127.0.0.1:6379> sadd hello_set redis mysql oracle greenplum
(integer) 4

127.0.0.1:6379> scard hello_set
(integer) 4

127.0.0.1:6379> smembers hello_set
1) "mysql"
2) "redis"
3) "greenplum"
4) "oracle"

127.0.0.1:6379> sismember hello_set mysql
(integer) 1

127.0.0.1:6379> sismember hello_set oracle
(integer) 1

127.0.0.1:6379> sismember hello_set zookeeper
(integer) 0

127.0.0.1:6379> srem hello_set redis
(integer) 1

127.0.0.1:6379> smembers hello_set
1) "mysql"
2) "greenplum"
3) "oracle"
```

### 5）Sorted Set
```bash
127.0.0.1:6379> zadd hello_sorted 2 redis 1 mysql 4 greenplum 3 oracle
(integer) 4

127.0.0.1:6379> zcard hello_sorted
(integer) 4

127.0.0.1:6379> zrange hello_sorted 1 4
1) "redis"
2) "oracle"
3) "greenplum"

127.0.0.1:6379> zrange hello_sorted 1 5
1) "redis"
2) "oracle"
3) "greenplum"

127.0.0.1:6379> zrank hello_sorted mysql
(integer) 0

127.0.0.1:6379> zrank hello_sorted redis
(integer) 1

127.0.0.1:6379> zrank hello_sorted mysql
(integer) 0

127.0.0.1:6379> zrank hello_sorted oracle
(integer) 2
```

<!-- tabs:end -->

## 4、设置过期时间
?> **面试题：** 如何使登录失效？如何让验证码在规定时间后过期？

<!-- tabs:start -->

#### **参考回答**
- Redis 有设置过期时间的功能，可以对存储在 Redis 数据库中的值设置一个过期时间；
- 在 set key 的时候，可以设置一个 expire time，通过过期时间来控制这个 key 能够存活的时间；
- Redis 会采用 定期删除 + 惰性删除 两种删除策略；
    - <mark>&nbsp;定期删除&nbsp;</mark>：Redis 默认每隔 100ms 就随机抽取一些设置了过期时间的 key，检查其是否过期，如果过期就进行删除；
    - <mark>&nbsp;惰性删除&nbsp;</mark>：定期删除可能出现 key 已经过期却没有被删除的情况，所以 Redis 还有一个机制，就是在你获取某个设置了过期时间的 key 的时候会检查是否过期，如果已经过期就会删除它，所以叫做惰性删除；
- 但是如果某个 key 过期了并且也没有再使用过它，就有可能还储存在内存中，所以 Redis 还有一个<mark>&nbsp;内存淘汰机制&nbsp;</mark>来解决这个问题。


#### **源码详解**



<!-- tabs:end -->

## 5、内存淘汰机制
?> **面试题：** 你知道 Redis 的内存淘汰机制吗？

<!-- tabs:start -->

#### **参考回答**
- 在 Redis 配置文件 `redis.conf` 中有一个配置参数 `maxmemory`，通过这个参数可以设置 Redis 能使用的最大内存，如果不配置默认为 0，在64位操作系统下不限制内存大小，在32位操作系统下最多使用3GB内存；
- 如果内存被用完了，Redis 会使用配置的内存淘汰策略对数据进行淘汰，4.0 版本以后共有 8 种策略；
- <mark>&nbsp;noeviction&nbsp;</mark>：默认策略，对于写请求不再提供服务，直接返回错误（DEL请求和部分特殊请求除外）；
- <mark>&nbsp;volatile-lru&nbsp;</mark>：在设置了过期时间的 key 中使用 LRU 算法进行淘汰，Redis使用的是近似 LRU 算法，该算法通过随机采样法淘汰数据，每次随机出5（默认）个 key，从里面淘汰掉最近最少使用的 key，在 redis.conf 中有一个叫做 maxmemory-samples 的参数可以控制随机的个数，这个参数越大，越接近严格的 LRU 算法；
- <mark>&nbsp;volatile-random&nbsp;</mark>：在设置了过期时间的 key 中随机选择数据进行淘汰；
- <mark>&nbsp;volatile-lfu&nbsp;</mark>：LFU 算法是 Redis4.0 里面新加的一种淘汰策略。它的全称是 Least Frequently Used，它的核心思想是根据 key 的最近被访问的频率进行淘汰，很少被访问的优先被淘汰，被访问的多的则被留下来；
- LFU 算法能更好的表示一个 key 被访问的热度，假如你使用的是 LRU 算法，一个 key 很久没有被访问到，只刚刚是偶尔被访问了一次，那么它就被认为是热点数据，不会被淘汰，而有些 key 将来是很有可能被访问到的则被淘汰了，如果使用 LFU 算法则不会出现这种情况，因为使用一次并不会使一个key成为热点数据；
- <mark>&nbsp;volatile-ttl&nbsp;</mark>：在设置了过期时间的 key 中挑选将要过期的数据进行淘汰；
- <mark>&nbsp;allkeys-lru&nbsp;</mark>：在所有 key 中使用 LRU 算法进行淘汰；
- <mark>&nbsp;allkeys-random&nbsp;</mark>：在所有 key 中随机选择数据进行淘汰；
- <mark>&nbsp;allkeys-lfu&nbsp;</mark>：在所有 key 中使用 LFU 算法进行淘汰。

### 参考资料
- [redis 内存淘汰机制（MySQL里有2000w数据，Redis中只存20w的数据，如何保证Redis中的数据都是热点数据?）](https://snailclimb.gitee.io/javaguide/#/docs/database/Redis/Redis?id=redis-%e5%86%85%e5%ad%98%e6%b7%98%e6%b1%b0%e6%9c%ba%e5%88%b6mysql%e9%87%8c%e6%9c%892000w%e6%95%b0%e6%8d%ae%ef%bc%8credis%e4%b8%ad%e5%8f%aa%e5%ad%9820w%e7%9a%84%e6%95%b0%e6%8d%ae%ef%bc%8c%e5%a6%82%e4%bd%95%e4%bf%9d%e8%af%81redis%e4%b8%ad%e7%9a%84%e6%95%b0%e6%8d%ae%e9%83%bd%e6%98%af%e7%83%ad%e7%82%b9%e6%95%b0%e6%8d%ae)
- [Redis的内存淘汰策略](https://juejin.im/post/5d674ac2e51d4557ca7fdd70)

#### **源码详解**

### 1）noeviction 默认策略

测试将 maxmemory 调到 10kb 后写入数据会发生什么

```bash
# maxmemory <bytes>
maxmemory 10kb
```

重启 Redis 后写入数据

```bash
127.0.0.1:6379> CONFIG GET maxmemory-policy
1) "maxmemory-policy"
2) "noeviction"

127.0.0.1:6379> CONFIG GET maxmemory
1) "maxmemory"
2) "10240"

127.0.0.1:6379> hmset user_map2 name "tom" age 18 local "beijing" local2 "beijing" local3 "beijing" local4 "beijing" local5 "beijing" local6 "beijing" local7 "beijing" local8 "beijing" local9 "beijing" local10 "beijing" local11 "beijing" local12 "beijing" local13 "beijing" local14 "beijing" local15 "beijing" local16 "beijing" local17 "beijing"
(error) OOM command not allowed when used memory > 'maxmemory'.
```

<!-- tabs:end -->



Redis，必须会的，我这方便还算懂得多点，可以和面试官大战几个回合吧，应该不至于上来被打趴下，

单线程模型，aof，rdb，rewrite，

主从，cluster，

哪些类型，不要再说常规的5个了，多说几个让你区别其他小哥，

包含一些缓存常见的问题击穿、穿透、雪崩、数据一致性等，你必须会，不会基本没戏，

一致性hash，

布隆过滤器的原理，为此我还去了解了geohash的原理以及google s2的原理，

底层数据结构sds和跳表等，你多学点，准没错。