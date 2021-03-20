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

#### **代码详解**



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


#### **代码详解**



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

#### **代码详解**

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


#### **代码详解**



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

#### **代码详解**

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

## 6、持久化机制

?> **面试题：** 谈一谈你对 Redis 持久化机制的理解？

<!-- tabs:start -->

#### **参考回答**

- Redis 的数据全部存储在内存中，如果突然宕机，数据就会全部丢失，因此必须有一套机制来保证 Redis 的数据不会因为故障而丢失，这种机制就是 Redis 的持久化机制，它会将内存中的数据库状态保存到磁盘中；
- Redis 通常有两种持久化方式，一种叫做 <mark>&nbsp;RDB&nbsp;</mark>，默认也是这种，另一种叫做 <mark>&nbsp;AOF&nbsp;</mark>。

### 1）RDB
- 通过建立快照来保存数据在某个时间点上的副本，可以对快照备份，也可以把快照同步到其他服务器，还可以根据快照进行数据恢复；
- RDB 是 Redis 的默认持久化方式，默认配置是：
    - <mark>&nbsp;save 900 1&nbsp;</mark>：在900秒（15分钟）之后，如果至少有1个key发生变化，Redis就会自动触发BGSAVE命令创建快照；
    - <mark>&nbsp;save 300 10&nbsp;</mark>：在300秒（5分钟）之后，如果至少有10个key发生变化，Redis就会自动触发BGSAVE命令创建快照；
    - <mark>&nbsp;save 60 10000&nbsp;</mark>：在60秒（1分钟）之后，如果至少有10000个key发生变化，Redis就会自动触发BGSAVE命令创建快照。

### 2）AOF
- AOF 是 append-only file 的缩写，就是只追加文件，每执行一条会改变 Redis 中数据的命令时，就会把这个命令写入硬盘中的 AOF 文件；
- Redis 默认没有开启 AOF 持久化，redis.conf 配置参数 <mark>&nbsp;appendonly&nbsp;</mark> 默认为 no，如果设置为 yes 就代表开启，AOF 文件默认名称为 appendonly.aof，可以配置，存储位置和 RDB 一样，是通过配置项 <mark>&nbsp;dir&nbsp;</mark> 设置的；
- Redis 的 AOF 持久化具体有三种方式：
    - <mark>&nbsp;appendfsync always&nbsp;</mark>：每次有数据修改发生时都会写入AOF文件,这样会严重降低Redis的速度；
    - <mark>&nbsp;appendfsync everysec&nbsp;</mark>：每秒钟同步一次，显示地将多个写命令同步到硬盘，默认是这种，同时为了兼顾数据和写入性能，也推荐这种方式，让 Redis 每秒同步一次 AOF 文件，Redis 性能几乎没受到任何影响，而且这样即使出现系统崩溃，用户最多只会丢失一秒之内产生的数据；
    - <mark>&nbsp;appendfsync no&nbsp;</mark>：让操作系统决定何时进行同步。

### 3）混合持久化
- Redis 4.0 开始支持 RDB 和 AOF 的混合持久化（默认关闭，可以通过配置项 aof-use-rdb-preamble 开启）；
- 如果把混合持久化打开，AOF 重写的时候就直接把 RDB 的内容写到 AOF 文件开头，这样做的好处是可以结合 RDB 和 AOF 的优点, 快速加载同时避免丢失过多的数据，当然缺点也是有的，AOF 里面的 RDB 部分是压缩格式不再是 AOF 格式，可读性较差。

#### **代码详解**



<!-- tabs:end -->

## 7、Redis 事务

?> **面试题：** Redis 是怎么解决事务的？

<!-- tabs:start -->

#### **参考回答**

- Redis 通过 MULTI、EXEC、WATCH 等命令来实现事务（transaction）功能，事务提供了一种将多个命令请求打包，然后一次性、按顺序地执行多个命令的机制，并且在事务执行期间，服务器不会中断事务而改去执行其他客户端的命令请求，它会将事务中的所有命令都执行完毕，然后才去处理其他客户端的命令请求；
- 在传统的关系式数据库中，常常用 ACID 性质来检验事务功能的可靠性和安全性，在 Redis 中，事务总是具有原子性（Atomicity）、一致性（Consistency）和隔离性（Isolation），并且当 Redis 运行在某种特定的持久化模式下时，事务也具有持久性（Durability）；
- Redis 在事务失败时不进行回滚，而是继续执行余下的命令，Redis 认为失败的命令是由编程错误造成的，而这些错误应该在开发的过程中被发现，而不应该出现在生产环境中，另一方面正是因为不需要对回滚进行支持，所以 Redis 的内部可以保持简单且快速。

#### **代码详解**



<!-- tabs:end -->

## 8、缓存三大问题

?> **面试题：** 关于【缓存穿透、缓存击穿、缓存雪崩、热点数据失效】问题的解决方案？

<!-- tabs:start -->

#### **参考回答**
### 1）缓存穿透
- 缓存和数据库都查询不到某条数据，但是请求每次都会打到数据库上面去，这种查询不存在数据的现象称为缓存穿透；
- 一般情况下有 2 种方案解决缓存穿透的问题：
    - <mark>&nbsp;缓存空值&nbsp;</mark>：之所以会发生穿透，就是因为缓存中没有存储这些空数据的key，从而导致每次查询都到数据库去了，所以我们可以为这些 key 对应的值设置为 null，丢到缓存里面去。后面再出现查询这个 key 的请求的时候，直接返回 null；
    - <mark>&nbsp;[BloomFilter 布隆过滤器](algorithm/classical?id=_3、布隆过滤)&nbsp;</mark>：是一种在海量数据中判断某个元素是否存在的数据结构，可以在查询缓存之前使用布隆过滤器进行判断，如果 key 合法在走缓存，如果不合法直接返回错误信息；
- 针对于一些恶意攻击，攻击带过来的大量 key 是不存在的，那么我们采用第一种方案就会缓存大量不存在 key 的数据，可以采用第二种方案直接把不合法的 key 过滤掉。

### 2）缓存击穿
- 在平常高并发的系统中，大量的请求同时查询一个 key 时，此时这个 key 正好失效了，就会导致大量的请求都打到数据库上面去，这种现象我们称为缓存击穿，会造成某一时刻数据库请求量过大，压力剧增；
- 缓存击穿是由于多个线程同时去查询数据库的某条数据，我们可以在第一个查询数据的请求上使用一个互斥锁来锁住它，其他的线程走到这一步拿不到锁就等着，等第一个线程查询到了数据，然后做缓存，后面的线程进来发现已经有缓存了，就直接走缓存。

### 3）缓存雪崩
- 缓存雪崩的情况是说，当某一时刻发生大规模的缓存失效的情况，比如你的缓存服务宕机了，会有大量的请求进来直接打到 DB上面，结果就是 DB 撑不住，挂掉；
- 预防和解决缓存雪崩可以从三个方面入手：
    - 事前：尽量保证整个 redis 集群的高可用性，发现机器宕机尽快补上。选择合适的内存淘汰策略；
    - 事中：本地 ehcache 缓存 + hystrix 限流 & 降级，避免 DB 崩掉；比如一秒来了5000个请求，我们可以设置假设只能有一秒 2000 个请求能通过这个组件，那么其他剩余的 3000 请求就会走限流逻辑，然后去调用我们自己开发的降级组件（降级），比如设置的一些默认值呀之类的，以此来保护最后的 DB 不会被大量的请求给打死；
    - 事后：利用 redis 持久化机制保存的数据尽快恢复缓存。

### 4）热点数据失效
- 通常在设置缓存的时候，一般会给缓存设置一个失效时间，过了这个时间，缓存就失效了，但是对于一些热点的数据来说，当缓存失效以后会存在大量的请求过来，然后打到数据库去，从而可能导致数据库崩溃的情况；
- <mark>&nbsp;设置不同的失效时间&nbsp;</mark>：为了避免这些热点的数据集中失效，可以在设置缓存过期时间的时候，尽量让过期时间错开，比如在一个基础的时间上加上或者减去一个范围内的随机值；
- <mark>&nbsp;互斥锁&nbsp;</mark>：在第一个请求去查询数据库的时候对它加一个互斥锁，其余的查询请求都会被阻塞住，直到锁被释放，从而保护数据库，但是也是由于它会阻塞其他的线程，此时系统吞吐量会下降，需要结合实际的业务去考虑是否要这么做。

### 参考资料：
- [阿里一面：关于【缓存穿透、缓存击穿、缓存雪崩、热点数据失效】问题的解决方案【石杉的架构笔记】](https://juejin.im/post/5c9a67ac6fb9a070cb24bf34)

#### **代码详解**



<!-- tabs:end -->


## 9、Redis 分布式锁

?> **面试题：** 如何解决 Redis 的并发竞争 Key 问题？

<!-- tabs:start -->

#### **参考回答**
- 所谓 Redis 的并发竞争 Key 的问题也就是多个系统同时对一个 key 进行操作，但是最后执行的顺序和我们期望的顺序不同，这样也就导致了结果的不同，使用 Redis 分布式锁可以解决这个问题，具体方式有两种：


- 使用 redis 的 setnx()、expire() 方法，用于分布式锁：
    - 1、setnx(lockkey, 1) 如果返回 0，则说明占位失败，如果返回 1，则说明占位成功；
    - 2、expire() 命令对 lockkey 设置超时时间，为的是避免死锁问题；
    - 3、执行完业务代码后，可以通过 delete 命令删除 key；


- 这个方案其实是可以解决日常工作中的需求的，但从技术方案的探讨上来说，可能还有一些可以完善的地方，比如，如果在第一步 setnx 执行成功后，在 expire() 命令执行成功前，发生了宕机的现象，那么就依然会出现死锁的问题；


- 还可以使用redis的setnx()、get()、getset()方法，用于分布式锁，解决死锁问题：
    - 1、setnx(lockkey, 当前时间+过期超时时间)，如果返回 1 则获取锁成功，如果返回 0 则没有获取到锁，转向 2；
    - 2、get(lockkey) 获取值 oldExpireTime，并将这个 value 值与当前的系统时间进行比较，如果小于当前系统时间，则认为这个锁已经超时，可以允许别的请求重新获取，转向 3；
    - 3、计算 newExpireTime = 当前时间 + 过期超时时间，然后 getset(lockkey, newExpireTime) 会返回当前 lockkey 的值 currentExpireTime；
    - 4、判断 currentExpireTime 与 oldExpireTime 是否相等，如果相等，说明当前 getset 设置成功，获取到了锁，如果不相等，说明这个锁又被别的请求获取走了，那么当前请求可以直接返回失败，或者继续重试；
    - 5、在获取到锁之后，当前线程可以开始自己的业务处理，当处理完毕后，比较自己的处理时间和对于锁设置的超时时间，如果小于锁设置的超时时间，则直接执行 delete 释放锁；如果大于锁设置的超时时间，则不需要再锁进行处理。

### 参考资料
- [分布式锁的实现之 redis 篇【小米信息部技术团队】](https://xiaomi-info.github.io/2019/12/17/redis-distributed-lock/)
- [Java分布式锁三种实现方案【蓝汀华韶】](https://www.jianshu.com/p/535efcab356d)

#### **代码详解**



<!-- tabs:end -->


## 10、缓存一致性

?> **面试题：** 如何保证缓存与数据库双写时的数据一致性？

<!-- tabs:start -->

#### **参考回答**
- 一般情况下我们都是这样使用缓存的：先读缓存，缓存没有的话，就读数据库，然后取出数据后放入缓存，同时返回响应，这种方式很明显会存在缓存和数据库的数据不一致的情况；
- 你只要用缓存，就可能会涉及到缓存与数据库双存储双写，你只要是双写，就一定会有数据一致性的问题，那么你如何解决一致性问题？
- 一般来说，就是如果你的系统不是严格要求缓存+数据库必须一致性的话，缓存可以稍微的跟数据库偶尔有不一致的情况，最好不要做这个方案，读请求和写请求串行化，串到一个内存队列里去，这样就可以保证一定不会出现不一致的情况；
- 串行化之后，就会导致系统的吞吐量会大幅度的降低，用比正常情况下多几倍的机器去支撑线上的一个请求。


#### **代码详解**



<!-- tabs:end -->


## 11、Redis 部署模式
?> **面试题：** Redis 有哪几种部署方式？

<!-- tabs:start -->

#### **参考回答**



#### **代码详解**

### 1、Redis 单点安装

```bash
# 后台启动
daemonize yes

# 允许远程连接
bind 0.0.0.0

# 设置密码
requirepass admin123
```

<!-- tabs:end -->


### 参考资料
- [Redis的几种部署模式概述](https://mp.weixin.qq.com/s/mc5sK8bRtl1IqZsoVnGO_w)
- [Redis 4 种模式简单了解](https://www.zhangaoo.com/article/redis-cluster-sentinel)
- [这可能是目前最全的Redis高可用技术解决方案总结](https://juejin.im/entry/5b7a27ade51d4538d5174b83)

## 12、Redis 常用命令
- 客户端连接

```bash
./redis-cli -h 127.0.0.1 -p 6379 -a Passw0rd
```
- 批量删除 key

```bash
./src/redis-cli -a pwd keys "simple*" | xargs ./src/redis-cli -a pwd del
```

## 13、Redis 优化

<!-- tabs:start -->

#### **参考回答**

### 1、Pipeline 批量操作

### 2、Redis 碎片整理

- info memory：查询 Redis 的内存使用情况
    - used_memory：是 Redis 中的数据占用的内存。
    - used_memory_rss：是 Redis 向操作系统申请的内存。
    - mem_fragmentation_ratio：就是内存碎片率，mem_fragmentation_ratio = used_memory_rss / used_memory

- 内存碎片如何产生的？
    - Redis 内部有自己的内存管理器，为了提高内存使用的效率，来对内存的申请和释放进行管理。
    - Redis 中的值删除的时候，并没有把内存直接释放，交还给操作系统，而是交给了 Redis 内部的内存管理器。
    - Redis 中申请内存的时候，也是先看自己的内存管理器中是否有足够的内存可用。
    - Redis 的这种机制，提高了内存的使用率，但是会使Redis中有部分自己没在用，却不释放的内存，导致了内存碎片的发生。
    
- 碎片率的意义
    - 大于1：说明内存有碎片，一般在1到1.5之间是正常的。
    - 大于1.5：说明内存碎片率比较大，需要考虑是否要进行内存碎片清理，要引起重视。
    - 小于1：说明已经开始使用交换内存，也就是使用硬盘了，正常的内存不够用了，需要考虑是否要进行内存的扩容。

#### **代码详解**

### 1、Pipeline 批量操作

```java
@Test
public void simpleSinkTest() {
    long st = DateUtils.getCurrentTimeMillis();
    for (int i = 0; i < 100000; i++) {
        jedis.set("simple" + i, "simple-value" + i);
    }
    System.out.println("simpleSinkTest set time=" + DateUtils.getExecuteTime(st));
    List<String> keys = new LinkedList<>();
    for (int i = 0; i < 100000; i++) {
        keys.add(jedis.get("simple" + i));
    }
    List<String> res = jedis.mget(keys.toArray(new String[0]));
    System.out.println("simpleSinkTest get time=" + DateUtils.getExecuteTime(st) + " conut=" + res.size());
}

@Test
public void pipelineSinkTest() {
    long st = System.currentTimeMillis();
    Pipeline pipeline = jedis.pipelined();
    for (int i = 0; i < 100000; i++) {
        pipeline.set("pipeline" + i, "pipeline-value" + i);
    }
    pipeline.sync();
    System.out.println("pipelineSinkTest set time=" + DateUtils.getExecuteTime(st));

    Pipeline pipeline2 = jedis.pipelined();
    for (int i = 0; i < 100000; i++) {
        pipeline2.get("pipeline" + i);
    }
    List<Object> res = pipeline2.syncAndReturnAll();
    System.out.println("pipelineSinkTest get time=" + DateUtils.getExecuteTime(st) + " conut=" + res.size());
}
```
```
simpleSinkTest set time=19
simpleSinkTest get time=38 conut=100000

pipelineSinkTest set time=0
pipelineSinkTest get time=0 conut=100000
```


### 2、Redis 碎片整理

- 重启：所有版本都适用，Redis服务器重启后，Redis会将没用的内存归还给操作系统，碎片率会降下来。
- 修改配置：Redis 版本 > 4.0

```bash
# 开启自动碎片清理功能
config set activedefrag yes

# 将配置写入配置文件，防止重启失效
config rewrite

# 手动整理碎片
memory purge
```

<!-- tabs:end -->