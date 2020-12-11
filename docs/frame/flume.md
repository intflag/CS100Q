# Flume
## 普通操作
### 1）常用维护命令
```bash
# 前台启动
bin/flume-ng agent --conf conf --conf-file myconf/test.properties --name a1 -Dflume.root.logger=INFO,console

# 后台启动
nohup bin/flume-ng agent --conf conf --conf-file myconf/test.properties --name a1 -Dflume.root.logger=INFO,console > ./logs/test.log 2>&1 &

```

## 实时同步 Kafka 数据到 HDFS
```bash
# example.conf: A single-node Flume configuration

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = org.apache.flume.source.kafka.KafkaSource
a1.sources.r1.channels = c1
a1.sources.r1.batchSize = 500
a1.sources.r1.batchDurationMillis = 2000
a1.sources.r1.kafka.bootstrap.servers = localhost:9092
a1.sources.r1.kafka.topics = sink-hdfs-topic
# a1.sources.r1.kafka.consumer.group.id = custom.g.id

# Describe the sink
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = hdfs://hadoop:8020/kafka/dump/%Y-%m-%d
a1.sinks.k1.hdfs.fileType= DataStream
a1.sinks.k1.hdfs.writeFormat= Text
# 文件前缀
a1.sinks.k1.hdfs.filePrefix= dump
# 不按照条数生成文件
a1.sinks.k1.hdfs.rollCount = 0
# 文件大小 128M
a1.sinks.k1.hdfs.rollSize = 134217728
# HDFS 上的文件达到10分钟生成一个文件
a1.sinks.k1.hdfs.rollInterval = 600
a1.sinks.k1.hdfs.useLocalTimeStamp = true

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 500

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```