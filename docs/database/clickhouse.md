# ClickHouse
## 数据库操作
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