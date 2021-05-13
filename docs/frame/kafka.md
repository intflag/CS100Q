# Kafka
## 常用命令
### 1）常用维护命令
```bash
# 启动 zookeeper 服务
bin/zookeeper-server-start.sh config/zookeeper.properties

# 启动/停止Kafka集群服务
bin/kafka-server-start.sh -daemon config/server.properties
bin/kafka-server-stop.sh -daemon config/server.properties

# topic 创建/删除
bin/kafka-topics.sh --create --zookeeper 127.0.0.1:2181 --replication-factor 1 --partitions 3 --topic test
bin/kafka-topics.sh --delete --zookeeper 127.0.0.1:2181 --topic test

# 查询所有 topic
bin/kafka-topics.sh --zookeeper 127.0.0.1:2181 --list

# 查看topic副本信息
bin/kafka-topics.sh --describe --zookeeper 127.0.0.1:2181 --topic test
bin/kafka-topics.sh --zookeeper 127.0.0.1:2181 --alter --partitions 20 --topic test

# 控制台生产者
bin/kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic bametl-cop-lltf

# 控制台消费者
bin/kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic test | grep -E '201800001345'

# 平衡
bin/kafka-preferred-replica-election.sh --zookeeper 127.0.0.1:2181

# Kafka查看最大偏移量（总消息个数） -1 偏移量最大值 -2 偏移量最小值
bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list 127.0.0.1:9092 --topic test --time -1

# 修改topic的partition数量（只能增加不能减少）
./bin/kafka-topics.sh --zookeeper 127.0.0.1:2181 --alter --partitions 20 --topic test

# 列出所有主题中的所有用户组
./bin/kafka-consumer-groups.sh --bootstrap-server 127.0.0.1:9092 --list

# 查看某个group的消费信息
./bin/kafka-consumer-groups.sh --bootstrap-server 127.0.0.1:9092 --describe --group groupName
```

## Kafka 高效文件存储机制
### 顺序 IO
- Kafka 使用磁盘作为存储介质，读写操作涉及到寻址过程，随机 IO 需要多次寻址，而顺序 IO 只需要寻址一次可以连续的读写下去；
- 对于每个分区，Kafka 把收到的消息顺序的写入对应的 log 文件中，一个文件写满了就开启一个新的文件顺序写下去，消费的时候从某个全局的位置开始，也就是说某一个 log 文件中的某一个位置开始，顺序的把消息读出来，充分利用磁盘顺序 IO 的特性，极大的提升了读写性能；

### PageCache 加速消息读写
- PageCache 是现代操作系统都具有的基本特性，就是在操作系统的内存中给磁盘上的文件建立的缓存，应用程序在调用系统的 API 读写文件时，并不会直接去读写磁盘上的文件，实际上操作的都是 PageCache；
- PageCache 使用 LRU 淘汰策略，通常情况下，消息刚刚写入到服务端就会被消费，读取的时候，对于这种刚刚写入的 PageCache，命中率会非常高；

### ZeroCopy 零拷贝技术
- 通常服务端处理消费的大致逻辑是：从文件读到内存，然后把消息通过网络发送到客户端；
- 这个过程数据一般会做 2-3 次复制，从文件复制到 PageCache，如果命中缓存可以省略，从 PageCache 复制到应用程序内存中，最后复制到 Socket 缓存区；
- Kafka 使用零拷贝技术可以把这个复制次数减少一次，DMA 控制器会把 PageCache 中的数据之间复制到 Socket，不需要 CPU 参与，速度更快。
