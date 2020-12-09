# MySQL
## 1、存储引擎
?> **面试题：** MySQL 有哪些存储引擎？默认的存储引擎是什么？

<!-- tabs:start -->

#### **参考回答**
- MySQL 5.7 可以使用 `show engines` 命令来查看提供的所有存储引擎，一共有 9 种；
- 在 MySQL 5.5 之前，MyISAM 是默认的数据库引擎，虽然性能极佳，而且提供了大量的特性，包括全文索引、压缩、空间函数等，但 MyISAM 不支持事务和行级锁，而且最大的缺陷就是崩溃后无法安全恢复；
- 在 5.5 版本之后，MySQL 引入了 InnoDB（事务性数据库引擎），MySQL 5.5 版本后默认的存储引擎为 InnoDB；

### MyISAM 和 InnoDB 的区别
- 锁：MyISAM 只支持表级锁，InnoDB 既支持表级锁也支持行级锁，默认是行级锁；
- 外键：MyISAM 不支持，而 InnoDB 支持；
- 事务和崩溃后的安全恢复：MyISAM 强调性能，每次查询具有原子性，不支持事务，崩溃后无法安全恢复，InnoDB 支持事务和崩溃修复能力；
- 速度：不要轻易相信 “MyISAM 比 InnoDB 快” 之类的经验之谈，这个结论往往不是绝对的，在很多我们已知场景中，InnoDB 的速度都可以让 MyISAM 望尘莫及，尤其是用到了聚簇索引，或者需要访问的数据都可以放入内存的应用；
- 使用场景：大多数时候我们使用的都是 InnoDB 存储引擎，但是在某些情况下使用 MyISAM 也是合适的比如读密集的情况下，（如果你不介意 MyISAM 崩溃恢复问题的话）。

#### **源码详解**

### 1、查看 MySQL 提供的所有存储引擎
```bash
mysql> show engines;
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
9 rows in set (0.00 sec)
```

<!-- tabs:end -->

## 2、MySQL 索引
?> **面试题：** MySQL 索引有哪些，原理和使用场景是什么？

<!-- tabs:start -->

#### **参考回答**
- MySQL 索引使用的数据结构主要有 `BTree 索引`和`哈希索引`，对于哈希索引来说，底层的数据结构就是哈希表，因此在绝大多数需求为单条记录查询的时候，可以选择哈希索引，查询性能最快，其余大部分场景，建议选择 BTree 索引；
- MySQL 的 BTree 索引使用的是 B 树中的 B+Tree，但是对于 MyISAM 和 InnoDB 两种存储引擎的实现方式是不同的；
- MyISAM：B+Tree 叶节点的 data 域存放的是数据记录的地址，在索引检索的时候，首先按照 B+Tree 搜索算法搜索索引，如果指定的 Key 存在，则取出其 data 域的值，然后以 data 域的值为地址读取相应的数据记录，这被称为`非聚簇索引`。
- InnoDB：其数据文件本身就是索引文件，相比 MyISAM，索引文件和数据文件是分离的，其表数据文件本身就是按 B+Tree 组织的一个索引结构，树的叶节点 data 域保存了完整的数据记录，这个索引的 key 是数据表的主键，因此 InnoDB 表数据文件本身就是主索引，这被称为`聚簇索引（或聚集索引）`，而其余的索引都作为辅助索引，辅助索引的data 域存储相应记录主键的值而不是地址，这也是和 MyISAM 不同的地方，在根据主索引搜索时，直接找到 key 所在的节点即可取出数据，在根据辅助索引查找时，则需要先取出主键的值，再走一遍主索引，因此，在设计表的时候，不建议使用过长的字段作为主键，也不建议使用非单调的字段作为主键，这样会造成主索引频繁分裂。 


#### **源码详解**

<!-- tabs:end -->

## 3、事务
?> **面试题：** 什么是事务？事务的特性是什么？并发事务会带来哪些问题？
<!-- tabs:start -->

#### **参考回答**

### 参考资料
- [事务【JavaGuide】](https://snailclimb.gitee.io/javaguide/#/docs/database/MySQL?id=%e4%bb%80%e4%b9%88%e6%98%af%e4%ba%8b%e5%8a%a1)


#### **源码详解**

<!-- tabs:end -->

## 4、隔离级别
?> **面试题：** MySQL 隔离级别有哪些，默认的是什么？
<!-- tabs:start -->

#### **参考回答**

### 参考资料
- [事务隔离级别有哪些?MySQL的默认隔离级别是?【JavaGuide】](https://snailclimb.gitee.io/javaguide/#/docs/database/MySQL?id=%e4%ba%8b%e5%8a%a1%e9%9a%94%e7%a6%bb%e7%ba%a7%e5%88%ab%e6%9c%89%e5%93%aa%e4%ba%9bmysql%e7%9a%84%e9%bb%98%e8%ae%a4%e9%9a%94%e7%a6%bb%e7%ba%a7%e5%88%ab%e6%98%af)


#### **源码详解**

<!-- tabs:end -->

## 5、锁机制与 InnoDB 锁算法
?> **面试题：** MySQL 有哪些锁？InnoDB 锁算法原理是什么？
<!-- tabs:start -->

#### **参考回答**

### 参考资料
- [锁机制与InnoDB锁算法【JavaGuide】](https://snailclimb.gitee.io/javaguide/#/docs/database/MySQL?id=%e9%94%81%e6%9c%ba%e5%88%b6%e4%b8%8einnodb%e9%94%81%e7%ae%97%e6%b3%95)


#### **源码详解**



<!-- tabs:end -->

## 6、SQL 执行细节
<!-- tabs:start -->

#### **参考回答**

### 参考资料
- [一条SQL语句在MySQL中如何执行的【JavaGuide】](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485097&idx=1&sn=84c89da477b1338bdf3e9fcd65514ac1&chksm=cea24962f9d5c074d8d3ff1ab04ee8f0d6486e3d015cfd783503685986485c11738ccb542ba7&token=79317275&lang=zh_CN#rd)


#### **源码详解**



<!-- tabs:end -->

## 7、SQL 优化
?> **面试题：** 对 SQL 做过优化吗？怎么做的？
<!-- tabs:start -->

#### **参考回答**

### 参考资料
- [一次非常有意思的sql优化经历【风过无痕的博客】](https://www.cnblogs.com/tangyanbo/p/4462734.html)


#### **源码详解**



<!-- tabs:end -->

## 8、分库分表
?> **面试题：** 项目中有分库分表吗？
<!-- tabs:start -->

#### **参考回答**

### 参考资料
- [大表优化【JavaGuide】](https://snailclimb.gitee.io/javaguide/#/docs/database/MySQL?id=%e5%a4%a7%e8%a1%a8%e4%bc%98%e5%8c%96)


#### **源码详解**



<!-- tabs:end -->

## 9、MySQL 部署

## 10、常用 SQl
### 1）查询进程
```sql
select * from information_schema.processlist
```

## 10、MySQL 数据库调优
### 1）脏页
?> **面试题：** 对 MySQL 数据库做过调优吗？怎么做的？
<!-- tabs:start -->

#### **参考回答**

### 参考资料


#### **源码详解**
### 1）InnoDB 的 IO 能力
- 测试磁盘的 IOPS

```bash
fio -filename=$filename -direct=1 -iodepth 1 -thread -rw=randrw -ioengine=psync -bs=16k -size=500M -numjobs=10 -runtime=10 -group_reporting -name=mytest 
```

- 设置 InnoDB 的 IO 能力

```bash
-- 查看
show variables like '%innodb_io_capacity%';

+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| innodb_io_capacity     | 200   |
| innodb_io_capacity_max | 2000  |
+------------------------+-------+

-- 设置成磁盘的 IOPS

set global innodb_io_capacity=1000;
set global innodb_io_capacity_max=10000;

```

### 2）脏页控制策略
- 开启 show_compatibility_56

```bash

-- 查看

show variables like '%show_compatibility_56%';

+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| show_compatibility_56 | OFF   |
+-----------------------+-------+

-- 开启
set global show_compatibility_56=on;

-- 验证
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| show_compatibility_56 | ON    |
+-----------------------+-------+
```

- 查看脏页比例

```bash

use information_schema;

select VARIABLE_VALUE into @a from global_status where VARIABLE_NAME = 'Innodb_buffer_pool_pages_dirty';select VARIABLE_VALUE into @b from global_status where VARIABLE_NAME = 'Innodb_buffer_pool_pages_total';select round(100*@a/@b,2) as '脏页比例';
```

- 查看脏页比例上限

```bash
show variables like '%innodb_max_dirty_pages_pct%';

+--------------------------------+-----------+
| Variable_name                  | Value     |
+--------------------------------+-----------+
| innodb_max_dirty_pages_pct     | 75.000000 |
| innodb_max_dirty_pages_pct_lwm | 0.000000  |
+--------------------------------+-----------+

平时要多关注脏页比例，不要让它经常接近 75%
```

### 3）redo log

<!-- tabs:end -->

## 10、使用 Canal 同步 MySQL binlog 到 Kafka
### 1）binlog
?> **面试题：** binlog 是什么？用什么作用？具体原理？
<!-- tabs:start -->

#### **参考回答**

### 参考资料


#### **源码详解**
### 1）开启 MySQL binlog

- 查询是否开启

```sql
mysql> show variables like 'log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | OFF   |
+---------------+-------+
1 row in set (0.00 sec)
```

- 编辑 MySQL 配置文件(/etc/my.cnf)

```bash
[mysqld]
# 打开binlog
log-bin=mysql-bin
# 选择ROW(行)模式
binlog-format=ROW
# 配置MySQL replaction需要定义，不要和canal的slaveId重复
server_id=1
```

- 重启 MySQL
```bash
# 5.5.7+版本命令
service mysql restart
```

- 验证

```sql
-- 查看是否打开binlog模式
mysql> show variables like 'log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | ON    |
+---------------+-------+
1 row in set (0.00 sec)

-- 查看binlog日志文件列表
mysql> show binary logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |       107 |
+------------------+-----------+
1 row in set (0.00 sec)

-- 查看当前正在写入的binlog文件
mysql> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |      107 |              |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)
```

### 2）创建 Canal 用户并授权
```sql
-- 创建用户
create user 'canal'@'%' identified by 'canal';
-- 授权 *.*表示所有库
grant SELECT, REPLICATION SLAVE, REPLICATION CLIENT on *.* to 'canal'@'%' identified by 'canal';
```

### 3）在 Kafka 创建 Topic
```bash
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 3 --topic canal-test
```

### 4）安装 Canal
- 下载地址：https://github.com/alibaba/canal/releases
- 将canal.deployer 复制到固定目录并解压

```bash
mkdir -p /usr/local/canal
cp canal.deployer-1.1.5-SNAPSHOT.tar.gz /usr/local/canal/
cd /usr/local/canal/
tar -zxvf canal.deployer-1.1.5-SNAPSHOT.tar.gz
ll

drwxr-xr-x 2 root root 4096 Dec  9 09:45 bin
drwxr-xr-x 5 root root 4096 Dec  9 09:45 conf
drwxr-xr-x 2 root root 4096 Dec  9 09:45 lib
drwxrwxrwx 2 root root 4096 Aug 22 13:17 logs
drwxrwxrwx 2 root root 4096 Aug 22 13:17 plugin
```

- 修改配置文件

a. 修改instance 配置文件 vi conf/example/instance.properties

```bash
# position info
canal.instance.master.address=127.0.0.1:3306

# username/password
canal.instance.dbUsername=canal
canal.instance.dbPassword=canal
canal.instance.connectionCharset = GBK

# mq config
canal.mq.topic=canal-test
canal.mq.partition=3
```

b. 修改canal 配置文件vi /usr/local/canal/conf/canal.properties

```bash
# tcp, kafka, rocketMQ, rabbitMQ
canal.serverMode = kafka
canal.mq.partitionsNum=3
```

- 启动 canal
```bash
./bin/startup.sh
```

- 验证
```bash

# 在数据库中插入/修改/删除一条数据

# 启动一个消费者进行查看
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic canal-test
```

<!-- tabs:end -->