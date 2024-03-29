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

- `<mark>`&nbsp;高性能&nbsp;`</mark>`：通常情况下，一些频繁使用且很少更新的数据会用 Redis 进行缓存，当第一次访问时会从数据库加载，之后访问会直接从 Redis 中获取，数据库更新时同步更新 Redis 中的数据，采用这种方式可以大大提高程序的性能；
- `<mark>`&nbsp;高并发&nbsp;`</mark>`：因为 Redis 中的数据是存储在内存中的，直接操作内存能够承受的请求是远远大于直接访问数据库的，所以可以实现高并发；
- `<mark>`&nbsp;分布式&nbsp;`</mark>`：缓存分为本地缓存和分布式缓存，Redis 可以支持分布式缓存，多个实例共用一份缓存，数据拥有一致性。

#### **代码详解**

<!-- tabs:end -->

## 2、单线程模型

?> **面试题：** Redis 是线程安全的吗，实现原理是什么？

<!-- tabs:start -->

#### **参考回答**

- Redis 的单线程指 Redis 的网络 IO 和键值对读写由一个线程来完成的；
- Redis 的持久化、异步删除、集群数据同步等功能是由其他线程而不是主线程来执行的，所以严格来说，Redis 并不是单线程；
- Redis 内部使用了文件事件处理器 `file event handler`，这个文件事件处理器是单线程的，所以说 Redis 是单线程模型，底层采用 epoll，利用 epoll 的多路复用特性对网络请求进行处理；
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

#### Redis String 的特点

- Redis实现的SDS支持扩容；
- 包含长度len，获取长度复杂度O(1)；
- 空间预分配；
- 惰性空间释放；

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
  - `<mark>`&nbsp;定期删除&nbsp;`</mark>`：Redis 默认每隔 100ms 就随机抽取一些设置了过期时间的 key，检查其是否过期，如果过期就进行删除；
  - `<mark>`&nbsp;惰性删除&nbsp;`</mark>`：定期删除可能出现 key 已经过期却没有被删除的情况，所以 Redis 还有一个机制，就是在你获取某个设置了过期时间的 key 的时候会检查是否过期，如果已经过期就会删除它，所以叫做惰性删除；
- 但是如果某个 key 过期了并且也没有再使用过它，就有可能还储存在内存中，所以 Redis 还有一个`<mark>`&nbsp;内存淘汰机制&nbsp;`</mark>`来解决这个问题。

#### **代码详解**

<!-- tabs:end -->

## 5、内存淘汰机制

?> **面试题：** 你知道 Redis 的内存淘汰机制吗？

<!-- tabs:start -->

#### **参考回答**

- 在 Redis 配置文件 `redis.conf` 中有一个配置参数 `maxmemory`，通过这个参数可以设置 Redis 能使用的最大内存，如果不配置默认为 0，在64位操作系统下不限制内存大小，在32位操作系统下最多使用3GB内存；
- 如果内存被用完了，Redis 会使用配置的内存淘汰策略对数据进行淘汰，4.0 版本以后共有 8 种策略；
- `<mark>`&nbsp;noeviction&nbsp;`</mark>`：默认策略，对于写请求不再提供服务，直接返回错误（DEL请求和部分特殊请求除外）；
- `<mark>`&nbsp;volatile-lru&nbsp;`</mark>`：在设置了过期时间的 key 中使用 LRU 算法进行淘汰，Redis使用的是近似 LRU 算法，该算法通过随机采样法淘汰数据，每次随机出5（默认）个 key，从里面淘汰掉最近最少使用的 key，在 redis.conf 中有一个叫做 maxmemory-samples 的参数可以控制随机的个数，这个参数越大，越接近严格的 LRU 算法；
- `<mark>`&nbsp;volatile-random&nbsp;`</mark>`：在设置了过期时间的 key 中随机选择数据进行淘汰；
- `<mark>`&nbsp;volatile-lfu&nbsp;`</mark>`：LFU 算法是 Redis4.0 里面新加的一种淘汰策略。它的全称是 Least Frequently Used，它的核心思想是根据 key 的最近被访问的频率进行淘汰，很少被访问的优先被淘汰，被访问的多的则被留下来；
- LFU 算法能更好的表示一个 key 被访问的热度，假如你使用的是 LRU 算法，一个 key 很久没有被访问到，只刚刚是偶尔被访问了一次，那么它就被认为是热点数据，不会被淘汰，而有些 key 将来是很有可能被访问到的则被淘汰了，如果使用 LFU 算法则不会出现这种情况，因为使用一次并不会使一个key成为热点数据；
- `<mark>`&nbsp;volatile-ttl&nbsp;`</mark>`：在设置了过期时间的 key 中挑选将要过期的数据进行淘汰；
- `<mark>`&nbsp;allkeys-lru&nbsp;`</mark>`：在所有 key 中使用 LRU 算法进行淘汰；
- `<mark>`&nbsp;allkeys-random&nbsp;`</mark>`：在所有 key 中随机选择数据进行淘汰；
- `<mark>`&nbsp;allkeys-lfu&nbsp;`</mark>`：在所有 key 中使用 LFU 算法进行淘汰。

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
- Redis 通常有两种持久化方式，一种叫做 `<mark>`&nbsp;RDB&nbsp;`</mark>`，默认也是这种，另一种叫做 `<mark>`&nbsp;AOF&nbsp;`</mark>`。

### 1）RDB

- 通过建立快照来保存数据在某个时间点上的副本，可以对快照备份，也可以把快照同步到其他服务器，还可以根据快照进行数据恢复；
- Redis 提供了两个命令来生成快照，save 和 bgsave，save 在主线程执行，会导致阻塞，bgsave 创建子线程执行，避免阻塞；
- 避免阻塞不代表可以正常处理写操作，为了能修改正在执行快照的数据，Redis 会借助操作系统的写时复制技术（Copy-On-Write, COW），把要修改的数据复制一份，主线程在数据副本上修改，bgsave 子线程可以继续把原来的数据写入 RDB 文件；
- RDB 是 Redis 的默认持久化方式，默认配置是：
  - `<mark>`&nbsp;save 900 1&nbsp;`</mark>`：在900秒（15分钟）之后，如果至少有1个key发生变化，Redis就会自动触发BGSAVE命令创建快照；
  - `<mark>`&nbsp;save 300 10&nbsp;`</mark>`：在300秒（5分钟）之后，如果至少有10个key发生变化，Redis就会自动触发BGSAVE命令创建快照；
  - `<mark>`&nbsp;save 60 10000&nbsp;`</mark>`：在60秒（1分钟）之后，如果至少有10000个key发生变化，Redis就会自动触发BGSAVE命令创建快照。

### 2）AOF

- AOF 是 append-only file 的缩写，就是只追加文件，并且 Redis 是写后日志，只有命令成功执行先写入内存后，才会把这个命令写入硬盘中的 AOF 文件；
- Redis 的 AOF 持久化具体有三种方式：
  - `<mark>`&nbsp;appendfsync always&nbsp;`</mark>`：同步写回：每个写命令执行完，立马同步地将日志写回磁盘，性能最低，可靠性最高；
  - `<mark>`&nbsp;appendfsync everysec&nbsp;`</mark>`：每秒写回：每个写命令执行完，只是先把日志写到 AOF 文件的内存缓冲区，每隔一秒把缓冲区中的内容写入磁盘，性能适中，宕机时丢失1秒内的数据；
  - `<mark>`&nbsp;appendfsync no&nbsp;`</mark>`：操作系统控制写回：每个写命令执行完，只是先把日志写到 AOF 文件的内存缓冲区，由操作系统决定何时将缓冲区内容写回磁盘，性能最高，宕机时丢失数据较多。
- Redis 默认没有开启 AOF 持久化，redis.conf 配置参数 `<mark>`&nbsp;appendonly&nbsp;`</mark>` 默认为 no，如果设置为 yes 就代表开启，AOF 文件默认名称为 appendonly.aof，可以配置，存储位置和 RDB 一样，是通过配置项 `<mark>`&nbsp;dir&nbsp;`</mark>` 设置的；
- 避免 AOF 日志文件太大，Redis 还提供了 AOF 重写机制，直接根据数据库里数据的最新状态，生成这些数据的插入命令，作为新日志。这个过程通过后台线程完成，避免了对主线程的阻塞

### 3）混合持久化

- Redis 4.0 开始支持 RDB 和 AOF 的混合持久化（可以通过配置项 aof-use-rdb-preamble 开启）；
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

## 8、缓存常见问题

?> **面试题：** 关于【缓存穿透、缓存击穿、缓存雪崩、热点数据失效、缓存一致性】问题的解决方案？

<!-- tabs:start -->

#### **参考回答**

### 1）缓存穿透

- 缓存和数据库都查询不到某条数据，但是请求每次都会打到数据库上面去，这种查询不存在数据的现象称为缓存穿透；
- 一般情况下有 2 种方案解决缓存穿透的问题：
  - `<mark>`&nbsp;缓存空值&nbsp;`</mark>`：之所以会发生穿透，就是因为缓存中没有存储这些空数据的key，从而导致每次查询都到数据库去了，所以我们可以为这些 key 对应的值设置为 null，丢到缓存里面去。后面再出现查询这个 key 的请求的时候，直接返回 null；
  - `<mark>`&nbsp;[BloomFilter 布隆过滤器](algorithm/classical?id=_3、布隆过滤)&nbsp;`</mark>`：是一种在海量数据中判断某个元素是否存在的数据结构，可以在查询缓存之前使用布隆过滤器进行判断，如果 key 合法在走缓存，如果不合法直接返回错误信息；
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
- `<mark>`&nbsp;设置不同的失效时间&nbsp;`</mark>`：为了避免这些热点的数据集中失效，可以在设置缓存过期时间的时候，尽量让过期时间错开，比如在一个基础的时间上加上或者减去一个范围内的随机值；
- `<mark>`&nbsp;互斥锁&nbsp;`</mark>`：在第一个请求去查询数据库的时候对它加一个互斥锁，其余的查询请求都会被阻塞住，直到锁被释放，从而保护数据库，但是也是由于它会阻塞其他的线程，此时系统吞吐量会下降，需要结合实际的业务去考虑是否要这么做。

### 5、缓存一致性

- 一般情况下我们都是这样使用缓存的：先读缓存，缓存没有的话，就读数据库，然后取出数据后放入缓存，同时返回响应，这种方式很明显会存在缓存和数据库的数据不一致的情况；
- 你只要用缓存，就可能会涉及到缓存与数据库双存储双写，你只要是双写，就一定会有数据一致性的问题，那么你如何解决一致性问题？
- 一般来说，就是如果你的系统不是严格要求缓存+数据库必须一致性的话，缓存可以稍微的跟数据库偶尔有不一致的情况，最好不要做这个方案，读请求和写请求串行化，串到一个内存队列里去，这样就可以保证一定不会出现不一致的情况；
- 串行化之后，就会导致系统的吞吐量会大幅度的降低，用比正常情况下多几倍的机器去支撑线上的一个请求。

### 参考资料：

- [阿里一面：关于【缓存穿透、缓存击穿、缓存雪崩、热点数据失效】问题的解决方案【石杉的架构笔记】](https://juejin.im/post/5c9a67ac6fb9a070cb24bf34)

#### **代码详解**

<!-- tabs:end -->

## 9、Redis 分布式锁

?> **面试题：** 如何解决 Redis 的并发竞争 Key 问题？

<!-- tabs:start -->

#### **参考回答**

- 利用 SETNX 命令，同时带上 EX 或 PX 选项，用来设置键值对的过期时间；

```bash
SET key value [EX seconds | PX milliseconds] [NX]

# 加锁, unique_value作为客户端唯一性的标识
SET lock_key unique_value NX PX 10000

```

- 其中，unique_value 是客户端的唯一标识，可以用一个随机生成的字符串来表示，PX 10000 则表示 lock_key 会在 10s 后过期，以免客户端在这期间发生异常而无法释放锁;
- 应用：防止接口重复提交

### 参考资料

- [Redis 分布式锁的实现原理](https://shishan100.gitee.io/docs/#/./docs/distributed/page3)
- [每秒上千订单场景下的分布式锁高并发优化实践](https://shishan100.gitee.io/docs/#/./docs/distributed/page4)

#### **代码详解**

<!-- tabs:end -->

## 10、Redis 部署模式

?> **面试题：** Redis 有哪几种部署方式？

### 1）主从模式

![](http://images.intflag.com/redis010.jpg)

- 读操作：主库、从库都可以接收；
- 写操作：首先到主库执行，然后，主库将写操作同步给从库

#### 实现方式

- 从库执行：replicaof 主库IP 主库端口号（5.0 以前使用 slaveof）

```bash
replicaof  172.16.19.3  6379
```

#### 原理

![](http://images.intflag.com/redis011.jpg)

- 第一阶段是主从库间建立连接、协商同步的过程；
- 第二阶段，主库将所有数据同步给从库，从库收到数据后，在本地完成数据加载，这个过程依赖于内存快照生成的 RDB 文件；
- 第三个阶段，主库会把第二阶段执行过程中新收到的写命令，再发送给从库。具体的操作是，当主库完成 RDB 文件发送后，就会把此时 replication buffer 中的修改操作发给从库，从库再重新执行这些操作；

### 2）哨兵模式

#### 哨兵机制

![](http://images.intflag.com/redis012.jpg)

- 监控：
  - 周期性地给所有的主从库发送 PING 命令，检测它们是否仍然在线运行；
  - 如果从库没有在规定时间内响应哨兵的 PING 命令，哨兵就会把它标记为“下线状态”；
  - 同样，如果主库也没有在规定时间内响应哨兵的 PING 命令，哨兵就会判定主库下线，然后开始自动切换主库的流程；
- 选主：按照一定的规则选择一个从库实例，把它作为新的主库；
- 通知：
  - 在执行通知任务时，哨兵会把新主库的连接信息发给其他从库，让它们执行 replicaof 命令，和新主库建立连接，并进行数据复制；
  - 同时，哨兵会把新主库的连接信息通知给客户端，让它们把请求操作发到新主库上；

#### 主观下线和客观下线

- 主观下线：哨兵进程会使用 PING 命令检测它自己和主、从库的网络连接情况，用来判断实例的状态，如果哨兵发现主库或从库对 PING 命令的响应超时了，那么，哨兵就会先把它标记为“主观下线”；
- 客观下线：部署多个哨兵节点，根据一定的规则给从库打分，最后选择分数最高的从库为新主库；

#### 原理

- 哨兵直接建立网络连接是通过发布/订阅机制实现的；

![](http://images.intflag.com/redis013.jpg)

- 哨兵和从库建立网络是通过向主库发送 INFO 命令实现的；

![](http://images.intflag.com/redis014.jpg)

### 3）集群模式

- 纵向扩展（增加硬件配置），会导致内存变大，进而导致生成 rdb 文件 fork 子进程时阻塞时间边长，影响性能，同时硬件配置不能无限增加；
- 横向扩展（数据分片），不用担心单个实例的硬件配置，但是需要解决切片数据如何分布、客户端如何访问的问题；

![](http://images.intflag.com/redis015.jpg)

#### 切片数据分布规则

- Redis Cluster 方案，采用哈希槽来处理数据和实例之间的映射关系，一个切片集群共有 16384 个哈希槽，这些哈希槽类似于数据分区，每个键值对都会根据它的 key，被映射到一个哈希槽中；
- 按照 CRC16 算法计算一个 16 bit 的值，然后，再用这个 16bit 值对 16384 取模，得到 0~16383 范围内的模数，每个模数代表一个相应编号的哈希槽；
- 部署 Redis Cluster 方案时，可以使用 cluster create 命令创建集群，此时，Redis 会自动把这些槽平均分布在集群实例上。例如，如果集群中有 N 个实例，那么，每个实例上的槽个数为 16384/N 个；
- 也可以使用 cluster meet 命令手动建立实例间的连接，形成集群，再使用 cluster addslots 命令，根据不同实例的配置，手动指定每个实例上的哈希槽个数，需要注意，在手动分配哈希槽时，需要把 16384 个槽都分配完，否则 Redis 集群无法正常工作；

#### 客户端访问规则

- 哈希槽没有重新分配

  - 集群建立以后，Redis 实例会把自己的哈希槽信息发给和它相连的其他实例，来完成哈希槽的扩散；
  - 客户端访问任何一个实例都能获得所有的哈希槽信息；
- 哈希槽重新分配

  - 在集群中，实例有新增或删除，Redis 需要重新分配哈希槽；
  - 为了负载均衡，Redis 需要把哈希槽在所有实例上重新分布一遍；
  - 如果客户端给一个实例发送数据读写操作时，这个实例上并没有相应的数据，会返回 MOVED 迁移命令，包含新实例的地址，客户端再请求新实例获取数据，同时缓存新的哈希槽映射信息；
- 哈希槽正在分配中

  - 返回 ASK 正在迁移命令，同时返回最新地址，客户端需要给实例 3 发送 ASKING 命令，然后再发送操作命令，但不进行缓存；

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

## 11、Redis 常用命令

- 客户端连接

```bash
./redis-cli -h 127.0.0.1 -p 6379 -a Passw0rd
```

- 批量删除 key

```bash
./src/redis-cli -a pwd keys "simple*" | xargs ./src/redis-cli -a pwd del
```

- 响应延迟

```bash
./src/redis-cli --intrinsic-latency 120

Max latency so far: 1 microseconds.
Max latency so far: 8 microseconds.
Max latency so far: 10 microseconds.
Max latency so far: 12 microseconds.
Max latency so far: 13 microseconds.
Max latency so far: 16 microseconds.
Max latency so far: 26 microseconds.
Max latency so far: 41 microseconds.
Max latency so far: 42 microseconds.
Max latency so far: 1592 microseconds.

2324349292 total runs (avg latency: 0.0516 microseconds / 51.63 nanoseconds per run).
Worst run took 30836x longer than the average latency.
```

- swap 情况查询

```bash
ps -ef|grep redis 得到 pid
cd /proc/36052/
cat smaps | egrep '^(Swap|Size)'

Size:                 24 kB
Swap:                  0 kB
Size:                 88 kB
Swap:                  0 kB
Size:                312 kB
Swap:                  8 kB
Size:            9014860 kB
Swap:                520 kB
Size:              16388 kB
Swap:                  0 kB
```

## 12、Redis 优化

<!-- tabs:start -->

#### **参考回答**

### 1、Pipeline 批量操作

### 2、Redis 碎片整理

- info memory：查询 Redis 的内存使用情况 info memory

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
