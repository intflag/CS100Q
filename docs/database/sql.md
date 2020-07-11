# SQL 优化
## information_schema.processlist
### 1）按客户端 IP 分组，看哪个客户端的链接数最多
```sql
select client_ip,count(client_ip) as client_num from (select substring_index(host,':' ,1) as client_ip from information_schema.processlist ) as connect_info group by client_ip order by client_ip,client_num desc;
```
### 2）查看正在执行的线程，并按 Time 倒排序，看看有没有执行时间特别长的线程
```sql
select * from information_schema.processlist where Command != 'Sleep' order by Time desc;
```
### 3）找出所有执行时间超过 5 分钟的线程，拼凑出 kill 语句，方便后面查杀
```sql
select concat('kill ', id, ';') from information_schema.processlist where Command != 'Sleep' and Time > 300 order by Time desc;
```
## 慢查询
### 1）开启慢查询
```sql
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
