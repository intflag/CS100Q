# ClickHouse
## 普通操作
### 1）数据导入导出
```bash
远程导出命令，默认分割符是tab：(本地ip=127.0.0.1)
echo 'select * from table_name' | curl ip:8123?database=mybi -uroot:password -d @- > table_name.sql

导入数据库的本机执行：
cat table_name.sql | clickhouse-client --query="INSERT INTO database.table_name FORMAT TabSeparated"
```
### 2）删除或更新数据
```bash
# 删除
ALTER TABLE [db.]table DELETE WHERE filter_expr

如：

# 更新
ALTER TABLE [db.]table UPDATE column1 = expr1 [, ...] WHERE filter_expr

# 注意：
1.、这两条命令必须在版本号大于1.1.54388才可以使用，适用于 mergeTree 引擎

2、这两条命令是异步执行的，可以通过查看表 system.mutations 来查看命令的是否执行完毕
select * from system.mutations where table='test_update';
```

### 3）系统表

- 查看数据库容量、行数、压缩率

```
SELECT sum(rows) AS `total_count`, formatReadableSize(sum(data_uncompressed_bytes)) AS `original_size`, formatReadableSize(sum(data_compressed_bytes)) AS `compressed_size`, round((sum(data_compressed_bytes) / sum(data_uncompressed_bytes)) * 100, 0) AS `compression_ratio` FROM system.parts

┌─total_count─┬─original_size─┬─compressed_size─┬─compression_ratio─┐
│ 27130807069 │ 2.76 TiB      │ 482.91 GiB      │                17 │
└─────────────┴───────────────┴─────────────────┴───────────────────┘
```

- 查看数据表容量、行数、压缩率

```
SELECT table AS `table_name`, sum(rows) AS `total_count`, formatReadableSize(sum(data_uncompressed_bytes)) AS `original_size`, formatReadableSize(sum(data_compressed_bytes)) AS `compressed_size`, round((sum(data_compressed_bytes) / sum(data_uncompressed_bytes)) * 100, 0) AS `compression_ratio` FROM system.parts WHERE table IN ('temp_1') GROUP BY table
```

## 高级操作
### 1）kafka 表引擎实时接收数据

```bash

# 1、在 kafka 中创建 topic

./bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic ck_test --partitions 3 --replication-factor 3

# 2、在 ClickHouse 每个节点中创建 Kafka 引擎表

drop table if exists test_kafka_table;

CREATE TABLE test_kafka_table(
    `tag` String COMMENT 'tag',
    `key` Int64 COMMENT 'key',
    `datas` Nested
    (
        `id` Int64,
        `code` String,
        `name` String
    ) 
) ENGINE = Kafka() 
SETTINGS kafka_broker_list = 'localhost:9092',
kafka_topic_list = 'ck_test',
kafka_group_name = 'ck_group',
kafka_format = 'JSONEachRow',
kafka_skip_broken_messages = 100;

# 3、创建数据表及对应分布式表
drop table if exists test_ck_table;
CREATE TABLE `test_ck_table` (
    `key` Int64 COMMENT 'key',
    `tag` String COMMENT 'tag',
    `id` Int64 COMMENT 'id',
    `code` String COMMENT 'code',
    `name` String COMMENT 'name'
) ENGINE = ReplicatedReplacingMergeTree(
    '/clickhouse/tables/{shard}/test_ck_table',
    '{replica}'
)
ORDER BY
    (id, code) PARTITION BY (id) SETTINGS storage_policy = 'allData';


CREATE TABLE test_ck_table_all AS test_ck_table ENGINE = Distributed(intell, default, test_ck_table);

# 4、在 ClickHouse 每个节点中创建物化视图
drop table if exists test_ck_view;
CREATE MATERIALIZED VIEW test_ck_view TO test_ck_table AS (SELECT key,tag,t.id as id, t.code as code, t.name as name FROM test_kafka_table ARRAY JOIN datas AS t);

# 5、写入 topic 数据测试

{ "tag": "202010291636", "key": 202000001906, "datas.id": [8, 9], "datas.code": ["a8", "a9"], "datas.name": ["n8", "n9"] }

select * from test_ck_table;

SELECT *
FROM test_ck_table

┌──────────key─┬─tag──────────┬─id─┬─code─┬─name─┐
│ 202000001906 │ 202010291636 │  9 │ a9   │ n9   │
└──────────────┴──────────────┴────┴──────┴──────┘
┌──────────key─┬─tag──────────┬─id─┬─code─┬─name─┐
│ 202000001906 │ 202010291636 │  8 │ a8   │ n8   │
└──────────────┴──────────────┴────┴──────┴──────┘

2 rows in set. Elapsed: 0.003 sec.
```

### 2）MySQL 表引擎数据无缝对接

```bash
# 1、MySQL 准备测试表

CREATE TABLE `t_ch_test` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL COMMENT '名称',
  `val` varchar(255) DEFAULT NULL COMMENT '值',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=gbk;

insert into t_ch_test(name,val) values('a','1'),('b','2'),('c','3');

# 2、ClickHouse 创建 MySQL 引擎表

CREATE TABLE `t_ch_test` on cluster 集群名称 (
  `id` Int64,
  `name` String,
  `val` String
) ENGINE = MySQL('127.0.0.1:3306', '数据库名称', 't_ch_test', 'root', '123456');

# 3、2、ClickHouse 测试
insert into t_ch_test(name,val) values('d','4'),('e','5');
select * from t_ch_test;

┌─id─┬─name─┬─val─┐
│  1 │ a    │ 1   │
│  2 │ b    │ 2   │
│  3 │ c    │ 3   │
│  4 │ d    │ 4   │
│  5 │ e    │ 5   │
└────┴──────┴─────┘

# 只会在 Clickhouse 上删除该表
drop table t_ch_test;
```

## 问题记录
### 1）启动报时区相关的错误
报错：
```bash
Could not determine local time zone: custom time zone file used
```
解决：
```bash
yum install tzdata
```
### 2）权限或用户问题
报错：
```bash
Effective user of the process (root) does not match the owner of the data (clickhouse). Run under 'sudo -u clickhouse'
```
解决：
```bash
sudo -u clickhouse clickhouse-server --config-file=/etc/clickhouse-server/config.xml
```
### 3）用程序写入时类型不一致问题
报错：
```bash
Exception: Cannot read all data. Bytes read: 90. Bytes expected: 122.: (at row 1)
```
解决：
```
查看代码写入字段个数、顺序、类型是否一致
```

### 4）Table is in readonly mode
- 问题原因：因为zookeeper压力太大，表处于“read only mode”模式，导致插入失败
- 解决方案
    - 在zookeeper中将dataLogDir存放目录应该与dataDir分开，可单独采用一套存储设备来存放ZK日志；
    - 做好zookeeper集群和clickhouse集群的规划，可以多套zookeeper集群服务一套clickhouse集群。