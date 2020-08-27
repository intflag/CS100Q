# ClickHouse
## 数据库操作
### 1）数据导入导出
```bash
远程导出命令，默认分割符是tab：(本地就ip=127,0,0.1)
echo 'select * from table_name' | curl ip:8123?database=mybi -uroot:password -d @- > table_name.sql

导入数据库的本机执行：
cat table_name.sql | clickhouse-client --query="INSERT INTO database.table_name FORMAT TabSeparated"
```
## 问题记录
### 1）启动报时区相关的错误
报错：
```
Could not determine local time zone: custom time zone file used
```
解决：
```
yum install tzdata
```
### 2）权限或用户问题
报错：
```
Effective user of the process (root) does not match the owner of the data (clickhouse). Run under 'sudo -u clickhouse'
```
解决：
```
sudo -u clickhouse clickhouse-server --config-file=/etc/clickhouse-server/config.xml
```
### 3）用程序写入时类型不一致问题
报错：
```
Exception: Cannot read all data. Bytes read: 90. Bytes expected: 122.: (at row 1)
```
解决：
```

```