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

## 3、事务及隔离级别
## 6、SQL 执行细节
## 5、SQL 优化
## 4、分库分表
## 7、MySQL 部署