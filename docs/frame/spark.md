# Spark
## 常见问题
### 1）超过限制
- 错误信息

```
job aborted due to stage failure: Serialized task 9:0 was 204718480 bytes, which exceeds max allowed: spark.rpc.message.maxSize (134217728 bytes). Consider increasing spark.rpc.message.maxSize or using broadcast variables for large values.
```

- 错误原因
Spark节点间传输的数据过大，超过系统默认的128M，因此需要提高spark.rpc.message.maxSize的大小或者选择用broadcast广播数据。
然而在某些情况下，广播数据并不能契合我们的需求，这时我们可以在提交任务时对spark.rpc.message.maxSize进行配置，调高maxSize即可。

- 解决方案

```
./bin/spark-submit \
--class <main-class>
--master <master-url> \
--deploy-mode <deploy-mode> \
--conf spark.rpc.message.maxSize=256
```
### 2）内存溢出
- 错误信息

```
Caused by: java.lang.OutOfMemoryError: GC overhead limit exceeded
```

- 错误原因
executor 的内存不足，需要设置每个Executor进程的内存。
Executor 内存的大小，很多时候直接决定了 Spark 作业的性能，而且跟常见的 JVM OOM 异常，也有直接的关联

- 解决方案
### 3）依赖问题
- 错误信息

```
Exception in thread "main" java.lang.NoClassDefFoundError: org/apache/kafka/common/serialization/StringDeserializer
```

- 解决方案
```
启动指定 jar 包
--jars lib/spark-streaming-kafka-0-10_2.11-2.4.0.jar,lib/kafka-clients-2.0.0.jar
```

- 解决方案

### 4）连不上 kafka
- 错误信息

```
20/12/03 19:14:40 WARN NetworkClient: [Consumer clientId=consumer-1, groupId=xxxxxx] Connection to node -1 could not be established. Broker may not be available.
```

- 解决方案
```
配置正确的 Broker 地址
broken.server = host1:9092,host2:9092
```

### 5）类型转换异常
- 最后结果中不要包含 null

### 6）Dataset<Row> 转 JavaRDD<T> 异常
- 用 lombok 时不要加 @Accessors(chain = true) 注解
- Dataset<Row> 中字段有 MySQL 中的 datetime 类型