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
## 高级操作
### 1）kafka 表引擎实时接收数据

```bash

# 1、在 kafka 中创建 topic

./bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic ck_test --partitions 3 --replication-factor 3

# 2、在 ClickHouse 每个节点中创建 Kafka 引擎表和与之对应的分布式表

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

CREATE TABLE test_ck_table_all AS test_ck_table ENGINE = Distributed(intell, default, test_ck_table);

# 3、在 ClickHouse 每个节点中创建物化视图
drop table if exists test_ck_view;
CREATE MATERIALIZED VIEW test_ck_view TO test_ck_table AS (SELECT key,tag,t.id as id, t.code as code, t.name as name FROM test_kafka_table ARRAY JOIN datas AS t);

# 4、写入 topic 数据测试

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
