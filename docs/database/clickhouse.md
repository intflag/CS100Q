# ClickHouse
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