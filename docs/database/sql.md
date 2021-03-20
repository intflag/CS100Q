# SQL 优化
## 进程及死锁
### 1）按客户端 IP 分组，看哪个客户端的链接数最多
```bash
select client_ip,count(client_ip) as client_num from (select substring_index(host,':' ,1) as client_ip from information_schema.processlist ) as connect_info group by client_ip order by client_ip,client_num desc;
```
### 2）查看正在执行的线程，并按 Time 倒排序，看看有没有执行时间特别长的线程
```bash
select * from information_schema.processlist where Command != 'Sleep' order by Time desc;
```
### 3）找出所有执行时间超过 5 分钟的线程，拼凑出 kill 语句，方便后面查杀
```bash
select concat('kill ', id, ';') from information_schema.processlist where Command != 'Sleep' and Time > 300 order by Time desc;
```
## 慢查询
### 1）开启慢查询
```bash
-- 查询是否开启慢查询
show variables  like '%slow_query_log%';

+---------------------+------------------------------------------------------+
| Variable_name       | Value                                                |
+---------------------+------------------------------------------------------+
| slow_query_log      | OFF                                                  |
| slow_query_log_file | /data1/mysql57/mysql/data/host-172-26-94-68-slow.log |
+---------------------+------------------------------------------------------+

-- 临时开启
set global slow_query_log=1;

+---------------------+------------------------------------------------------+
| Variable_name       | Value                                                |
+---------------------+------------------------------------------------------+
| slow_query_log      | ON                                                   |
| slow_query_log_file | /data1/mysql57/mysql/data/host-172-26-94-68-slow.log |
+---------------------+------------------------------------------------------+

-- 永久开启
修改my.cnf文件，增加或修改参数slow_query_log 和slow_query_log_file后，然后重启MySQL服务器，如下所示

slow_query_log =1

slow_query_log_file=/tmp/mysql_slow.log

```

### 2）慢查询时间设置
```bash
-- 查看慢查询时间阈值，默认为 10 秒
show variables like 'long_query_time%';

+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+

-- 修改阈值
set global long_query_time=4;

-- 全局查看修改后的结果，普通查询方式需要新开一个会话才能看到修改效果
show global variables like 'long_query_time';

+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 11.000000 |
+-----------------+-----------+
```

### 3）慢查询日志输出方式
```bash
-- 查询输出方式，默认为 FILE，表示输出到文件中
show variables like '%log_output%';

+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_output    | FILE  |
+---------------+-------+

-- 修改输出方式，TABLE 表示会输出到表中，也可以使用 FILE,TABLE 逗号分隔的方式另种都配置
set global log_output='TABLE';

show variables like '%log_output%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_output    | TABLE |
+---------------+-------+

select * from mysql.slow_log;

+---------------------+---------------------------+------------+-----------+-----------+---------------+----+----------------+-----------+-----------+-----------------+-----------+
| start_time          | user_host                 | query_time | lock_time | rows_sent | rows_examined | db | last_insert_id | insert_id | server_id | sql_text        | thread_id |
+---------------------+---------------------------+------------+-----------+-----------+---------------+----+----------------+-----------+-----------+-----------------+-----------+
| 2016-06-16 17:37:53 | root[root] @ localhost [] | 00:00:03   | 00:00:00  |         1 |             0 |    |              0 |         0 |         1 | select sleep(3) |         5 |
| 2016-06-16 21:45:23 | root[root] @ localhost [] | 00:00:05   | 00:00:00  |         1 |             0 |    |              0 |         0 |         1 | select sleep(5) |         2 |
+---------------------+---------------------------+------------+-----------+-----------+---------------+----+----------------+-----------+-----------+-----------------+-----------+
```

### 4）未使用索引慢查询
系统变量 log-queries-not-using-indexes：未使用索引的查询也被记录到慢查询日志中（可选项）。如果调优的话，建议开启这个选项。另外，开启了这个参数，其实使用 full index scan 的 sql 也会被记录到慢查询日志。

```bash
-- 查看
show variables like 'log_queries_not_using_indexes';
+-------------------------------+-------+
| Variable_name                 | Value |
+-------------------------------+-------+
| log_queries_not_using_indexes | OFF   |
+-------------------------------+-------+

-- 开启
set global log_queries_not_using_indexes=1;
 
show variables like 'log_queries_not_using_indexes';
+-------------------------------+-------+
| Variable_name                 | Value |
+-------------------------------+-------+
| log_queries_not_using_indexes | ON    |
+-------------------------------+-------+
```

### 5）慢管理语句
系统变量 log_slow_admin_statements 表示是否将慢管理语句例如 ANALYZE TABLE 和 ALTER TABLE 等记入慢查询日志
```bash
show variables like 'log_slow_admin_statements';
+---------------------------+-------+
| Variable_name             | Value |
+---------------------------+-------+
| log_slow_admin_statements | OFF   |
+---------------------------+-------+
```

### 6）慢查询记录数
```bash
show global status like '%Slow_queries%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| Slow_queries  | 3     |
+---------------+-------+
```

### 7）慢查询分析工具
- mysqldumpslow

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
## 脏页
### 磁盘能力 innodb_io_capacity
- innodb_io_capacity 会告诉 InnoDB 你的磁盘能力，通常设置成磁盘的 IOPS

```bash
-- 查看 innodb_io_capacity
mysql> show global variables like '%capacity%';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| innodb_io_capacity     | 200   |
| innodb_io_capacity_max | 2000  |
+------------------------+-------+
```

## 事务隔离级别
### 设置隔离级别
- 命令：SET [SESSION | GLOBAL] TRANSACTION ISOLATION LEVEL {READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE}

```bash
-- 查看当前级别
select @@global.tx_isolation,@@tx_isolation;
+-----------------------+-----------------+
| @@global.tx_isolation | @@tx_isolation  |
+-----------------------+-----------------+
| REPEATABLE-READ       | REPEATABLE-READ |
+-----------------------+-----------------+

-- 修改
set global transaction isolation level read committed;

select @@global.tx_isolation,@@tx_isolation;
+-----------------------+-----------------+
| @@global.tx_isolation | @@tx_isolation  |
+-----------------------+-----------------+
| READ-COMMITTED        | REPEATABLE-READ |
+-----------------------+-----------------+


```

## 查询优化
### innodb_buffer_pool_size

```sql
mysql> show variables  like '%innodb_buffer_pool_size%';
+-------------------------+-----------+
| Variable_name           | Value     |
+-------------------------+-----------+
| innodb_buffer_pool_size | 134217728 |
+-------------------------+-----------+
1 row in set (0.00 sec)

mysql> set global innodb_buffer_pool_size = 8589934592;
Query OK, 0 rows affected (0.30 sec)

mysql> show variables  like '%innodb_buffer_pool_size%';
+-------------------------+-----------+
| Variable_name           | Value     |
+-------------------------+-----------+
| innodb_buffer_pool_size | 134217728 |
+-------------------------+-----------+
1 row in set (0.00 sec)

mysql> show variables  like '%innodb_buffer_pool_size%';
+-------------------------+------------+
| Variable_name           | Value      |
+-------------------------+------------+
| innodb_buffer_pool_size | 8589934592 |
+-------------------------+------------+
1 row in set (0.00 sec)

```

## 表空间
```sql
SELECT table_schema,TABLE_NAME , concat(data_free/1024/1024,"M") FROM `information_schema`.tables WHERE table_schema = 'db_name' and  ENGINE ='innodb'  ORDER BY data_free DESC;
```