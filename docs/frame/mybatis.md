# MyBatis
### 参考资料
- [MyBatis的工作原理以及核心流程介绍](http://www.mybatis.cn/archives/706.html)

## 工作原理
### SqlSession对象
- 该对象中包含了执行 SQL 语句的所有方法，类似于 JDBC 里面的 Connection；

### Executor接口
- 它将根据 SqlSession 传递的参数动态地生成需要执行的 SQL 语句，同时负责查询缓存的维护，类似于JDBC里面的 Statement / PrepareStatement；

### MappedStatement对象
- 该对象是对映射 SQL 的封装，用于存储要映射的 SQL 语句的 id、参数等信息；

### ResultHandler对象
- 用于对返回的结果进行处理，最终得到自己想要的数据格式或类型，可以自定义返回类型。

## 核心流程
![](http://images.intflag.com/mybatis001.jpg)

### 1）读取MyBatis的配置文件
- mybatis-config.xml 为 MyBatis 的全局配置文件，用于配置数据库连接信息；

### 2）加载映射文件
- 映射文件即 SQL 映射文件，该文件中配置了操作数据库的 SQL 语句，需要在 MyBatis 配置文件 mybatis-config.xml 中加载；
- mybatis-config.xml 文件可以加载多个映射文件，每个文件对应数据库中的一张表；

### 3）构造会话工厂
- 通过 MyBatis 的环境配置信息构建会话工厂 SqlSessionFactory；

### 4）创建会话对象
- 由会话工厂创建 SqlSession 对象，该对象中包含了执行 SQL 语句的所有方法；

### 5）Executor 执行器
- MyBatis 底层定义了一个 Executor 接口来操作数据库，它将根据 SqlSession 传递的参数动态地生成需要执行的 SQL 语句，同时负责查询缓存的维护；

### 6）MappedStatement 对象
- 在 Executor 接口的执行方法中有一个 MappedStatement 类型的参数，该参数是对映射信息的封装，用于存储要映射的 SQL 语句的 id、参数等信息；

### 7）输入参数映射
- 输入参数类型可以是 Map、List 等集合类型，也可以是基本数据类型和 POJO 类型；
- 输入参数映射过程类似于 JDBC 对 preparedStatement 对象设置参数的过程；

### 输出结果映射
- 输出结果类型可以是 Map、List 等集合类型，也可以是基本数据类型和 POJO 类型；
- 输出结果映射过程类似于JDBC对结果集的解析过程；

## 多级缓存
### 一级缓存
![](http://images.intflag.com/mybatis002.jpg)

- 每个 SqlSession 中持有了Executor，每个Executor中有一个LocalCache；
- 当用户发起查询时，MyBatis 根据当前执行的语句生成 MappedStatement，在Local Cache进行查询，如果缓存命中的话，直接返回结果给用户；
- 如果缓存没有命中的话，查询数据库，结果写入Local Cache，最后返回结果给用户；

#### 配置
```xml
<setting name="localCacheScope" value="SESSION"/>

SESSION 或者 STATEMENT，默认是 SESSION 级别，即在一个 MyBatis 会话中执行的所有语句，都会共享这一个缓存，一种是 STATEMENT 级别，可以理解为缓存只对当前执行的这一个 Statement 有效；
```

#### 总结
- MyBatis一级缓存内部设计简单，只是一个没有容量限定的 HashMap，在缓存的功能性上有所欠缺；
- MyBatis 的一级缓存最大范围是 SqlSession 内部，有多个 SqlSession 或者分布式的环境下，数据库写操作会引起脏数据，建议设定缓存级别为 Statement；

### 二级缓存

![](http://images.intflag.com/mybatis004.jpg)

- 一级缓存中，其最大的共享范围就是一个 SqlSession 内部，如果多个 SqlSession 之间需要共享缓存，则需要使用到二级缓存；
- 开启二级缓存后，会使用 CachingExecutor 装饰 Executor，进入一级缓存的查询流程前，先在 CachingExecutor 进行二级缓存的查询；

#### 配置
```xml
<setting name="cacheEnabled" value="true"/>

type：cache使用的类型，默认是PerpetualCache，这在一级缓存中提到过。
eviction： 定义回收的策略，常见的有FIFO，LRU。
flushInterval： 配置一定时间自动刷新缓存，单位是毫秒。
size： 最多缓存对象的个数。
readOnly： 是否只读，若配置可读写，则需要对应的实体类能够序列化。
blocking： 若缓存中找不到对应的key，是否会一直blocking，直到有对应的数据进入缓存。
```

#### 总结
- MyBatis 的二级缓存相对于一级缓存来说，实现了 SqlSession 之间缓存数据的共享，同时粒度更加的细，能够到 namespace 级别，通过 Cache 接口实现类不同的组合，对 Cache 的可控性也更强；
- MyBatis 在多表查询时，极大可能会出现脏数据，有设计上的缺陷，安全使用二级缓存的条件比较苛刻；
- 在分布式环境下，由于默认的 MyBatis Cache 实现都是基于本地的，分布式环境下必然会出现读取到脏数据，需要使用集中式缓存将 MyBatis 的 Cache 接口实现，有一定的开发成本，直接使用 Redis、Memcached 等分布式缓存可能成本更低，安全性也更高。
