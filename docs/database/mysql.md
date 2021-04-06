# MySQL
## 架构
![](http://images.intflag.com/mysql000.png)

- 连接器：管理连接，权限验证；
    - wait_timeout：默认 8 小时；
    - mysql_reset_connection：MySQL 5.7+ 版本，在不需要重连和权限验证的情况下可以重新初始化资源；
- 查询缓存：SQL 命中缓存直接返回结果；
    - 表只要有一个更新，所以的查询缓存都会失效；
    - SQL_CACHE 或 SQL_NO_CACHE；
    - MySQL 8.0 后该模块被移除；
- 分析器
    - 词法分析：查询语句、更新语句、表名、列名；
    - 语法分析：语句错误；
- 优化器：索引选择、决定表连接的顺序等;
- 执行器：操作数据库引擎，返回结果。

## 存储引擎

### 1、MyISAM 与 InnoDB

|特性|InnoDB|MyISAM|
|:----:|:----:|:----:|
|事务|支持|不支持|
|主键|必须|非必须|
|外键|支持|不支持|

### 2、InnoDB 存储原理

![](http://images.intflag.com/mysql001.png)

- 数据页：数据被分成若干页，以页为单位保存在磁盘中，处理时加载到内存中，各个数据页组成一个双向链表，默认页大小 16 KB，每个数据页中的记录按照主键顺序组成单向链表；
- 页目录：方便按照主键查询记录；
- 槽：相当于分组，每个分组负责若干条记录；
- 记录：记录前面的数字代表当前分组的记录条数；
- 查询：利用二分法，先定位到槽，然后在分组内顺序搜索。

## 索引

### 1、B+Tree
- 叶子节点用来存放数据，聚集索引（主键索引）存放整行记录，非聚集索引（二级索引）存放主键值；
- 上层非叶子节点用来存放目录项，作为索引；
- 所有节点按照索引键大小排序，构成一个双向链表，加速范围查找。

- 主键索引
![](http://images.intflag.com/mysql002.png)

- 二级索引
![](http://images.intflag.com/mysql003.png)

## SQL 执行流程

### 1、重要日志模块

||redo log|binlog|
|:----:|:----:|:----:|
|名称|重做日志|归档日志|
|所属|InnoDB独有|Server 层|
|特性|物理日志，记录数据页的修改|逻辑日志，|
|写入方式|循环写，空间固定，会用完|追加写，达到固定大小后会切换到下一个|

- redo log 的“记账”模式

![](http://images.intflag.com/mysql005.png)

- redo log crash-safe 能力：innodb_flush_log_at_trx_commit={0|1|2}，指定何时将事务日志刷到磁盘，默认为1；
    - 0表示每秒将 log buffer 同步到 os buffer 且从 os buffer 刷到磁盘日志文件中；
    - 1表示每事务提交都将 log buffer 同步到 os buffer 且从 os buffer 刷到磁盘日志文件中；
    - 2表示每事务提交都将 log buffer 同步到 os buffer 但每秒才从 os buffer 刷到磁盘日志文件中。

### 2、update 语句执行流程
- update T set c=c+1 where ID=2;

![](http://images.intflag.com/mysql004.png)

### 3、两阶段提交
- 先写 redo log 但状态为 prepare，再写 binlog，然后调用引擎的提交事务接口，修改 redo log 的状态为 commit；
- 两阶段提交是跨系统维持数据逻辑一致性时常用的一个方案。

## 事务隔离
- 事务：保证一组数据库操作，要么全部成功，要么全部失败；
- ACID：Atomicity、Consistency、Isolation、Durability，即原子性、一致性、隔离性、持久性；
- 多个事务同时执行带来的问题：
    - 脏读（dirty read）：读到其他事务未提交的数据；
    - 不可重复读（non-repeatable read）：前后读取的记录内容不一致；
    - 幻读（phantom read）：前后读取的记录数量不一致；
- 隔离级别：
    - 读未提交（read uncommitted）：一个事务还没提交时，它做的变更就能被别的事务看到；
    - 读提交（read committed）：一个事务提交之后，它做的变更才会被其他事务看到；
    - 可重复读（repeatable read）：一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的，同时未提交变更对其他事务也是不可见的；
    - 和串行化（serializable）：对于同一行记录，“写”会加“写锁”，“读”会加“读锁”，当出现读写锁冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行。

- 示例

```sql
create table T(c int) engine=InnoDB;
insert into T(c) values(1);
```

|时刻|事务1|事务2|读未提交|读提交|可重复读|串行化|
|:----|:----|:----|:----|:----|:----|:----|
|T1|启动事务|启动事务|事务1：c=1<br>事务2：c=1|事务1：c=1<br>事务2：c=1|事务1：c=1<br>事务2：c=1|事务1：c=1<br>事务2：c=1|
|T2||更新 c=2|事务1：c=2 脏读<br>事务2：c=2|事务1：c=1<br>事务2：c=2|事务1：c=1<br>事务2：c=2|事务1：c=1<br>事务2阻塞中|
|T3||提交事务|同上|事务1c=2不可重复度<br>事务2：c=2|事务1：c=1<br>事务2：c=2|事务2阻塞中|
|T4|提交事务||同上|同上|事务1：c=2<br>事务2：c=2|事务1：c=2<br>事务2：c=2|

### MVCC 多版本并发控制
- 事务ID（DB_TRX_ID）
- 回滚指针（DB_ROLL_PT）
- undo log：回滚日志，记录操作的逆过程
- read-view：每个事务在启动时创建

## SQL 优化
## 数据库优化
## 高可用
## 分库分表
## 常用 SQL

### 事务相关
```sql
-- 查看事务隔离级别
select @@global.tx_isolation,@@tx_isolation;

-- 修改全局事务隔离级别 READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE
set global transaction isolation level 隔离级别

-- 修改当前会话事务隔离级别
set session transaction isolation level 隔离级别

-- 查询超过 60 秒的长事务
select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60
```

### 进程及死锁
```sql
-- 按客户端 IP 分组，看哪个客户端的链接数最多
select client_ip,count(client_ip) as client_num from (select substring_index(host,':' ,1) as client_ip from information_schema.processlist ) as connect_info group by client_ip order by client_ip,client_num desc;

-- 查看正在执行的线程，并按 Time 倒排序，看看有没有执行时间特别长的线程
select * from information_schema.processlist where Command != 'Sleep' order by Time desc;

-- 找出所有执行时间超过 5 分钟的线程，拼凑出 kill 语句，方便后面查杀
select concat('kill ', id, ';') from information_schema.processlist where Command != 'Sleep' and Time > 300 order by Time desc;
```

### 慢查询
- 慢查询配置

```sql
-- 查询是否开启慢查询
show variables  like '%slow_query_log%';

-- 临时开启
set global slow_query_log=1;

-- 永久开启
修改my.cnf文件，增加或修改参数slow_query_log 和slow_query_log_file后，然后重启MySQL服务器，如下所示
slow_query_log =1
slow_query_log_file=/tmp/mysql_slow.log

-- 查看当前会话慢查询时间阈值，默认为 10 秒
show variables like 'long_query_time%';

-- 修改阈值
set global long_query_time=4;

-- 查看全局慢查询时间阈值，普通查询方式需要新开一个会话才能看到修改效果
show global variables like 'long_query_time';

-- 查询输出方式，默认为 FILE，表示输出到文件中
show variables like '%log_output%';

-- 修改输出方式，TABLE 表示会输出到表中，也可以使用 FILE,TABLE 逗号分隔的方式另种都配置
set global log_output='TABLE';

-- 查看未使用索引慢查询开启状态
show variables like 'log_queries_not_using_indexes';

-- 开启未使用索引慢查询记录功能
set global log_queries_not_using_indexes=1;

-- 查看管理语句慢查询开启记录
show variables like 'log_slow_admin_statements';

-- 慢查询记录数
show global status like '%Slow_queries%';
```

- 慢查询分析 mysqldumpslow

```bash
mysqldumpslow --help

c: 访问计数

l: 锁定时间

r: 返回记录

t: 查询时间

al:平均锁定时间

ar:平均返回记录数

at:平均查询时间
```

- 常用分析命令

```bash
得到返回记录集最多的10个SQL。

mysqldumpslow -s r -t 10 /database/mysql/mysql06_slow.log

得到访问次数最多的10个SQL

mysqldumpslow -s c -t 10 /database/mysql/mysql06_slow.log

得到按照时间排序的前10条里面含有左连接的查询语句。

mysqldumpslow -s t -t 10 -g “left join” /database/mysql/mysql06_slow.log

另外建议在使用这些命令时结合 | 和more 使用 ，否则有可能出现刷屏的情况。

mysqldumpslow -s r -t 20 /mysqldata/mysql/mysql06-slow.log | more
```

### 表空间
```sql
SELECT table_schema,TABLE_NAME , concat(data_free/1024/1024,"M") FROM `information_schema`.tables WHERE table_schema = 'db_name' and  ENGINE ='innodb'  ORDER BY data_free DESC;
```