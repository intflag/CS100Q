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

## Kafka 无消息丢失机制

### 生产者无消息丢失
- ack 机制：生产者每次发送消息后都有一个确认反馈机制，通过配置生产者的 acks 参数可以控制等级
    - 1：默认值，表示 leader 副本接收到就认为发送成功；
    - 0：生产者发送后直接返回，实际使用中不推荐；
    - -1 或 all：表示 leader 副本接收成功，并且所有 follower 副本同步成功才认为消息发送成功；
- 重试机制：设置 retries 重试次数为一个较大的值，当发送失败时生产者会进行重试，kafka 2.4 版本以后默认设置为Integer.MAX_VALUE；
- 回调机制：永远使用带有回调方法的 send API，一旦消息提交失败可以有针对性的处理；

### 消费者无消息丢失
- 手动提交位移：将 enable.auto.commit 自动提交位移设置为 false，手动在消息消费完成后提交；

### Broker 无消息丢失
- unclean.leader.election.enable = false，它控制的是哪些 Broker 有资格竞选分区的 Leader，如果一个 Broker 落后原先的 Leader 太多，那么它一旦成为新的 Leader，必然会造成消息的丢失，所以一般都要将该参数设置成 false，即不允许这种情况的发生；
- 设置 replication.factor >= 3，将副本多保存几份；
- 设置 min.insync.replicas > 1，控制的是消息至少要被写入到多少个副本才算是“已提交”，和 acks=all 全部接收是有区别，acks=all 指的是可以正常工作的副本个数，如果 ISR 里只有一个副本，那么 acks=all 就为 1，而 Broker 这个参数起到了一个保底的措施；
- 保证 replication.factor >= min.insync.replicas + 1，因为如果他们的值相等，那么只要有一个副本挂机整个分区就无法正常工作了；

## Kafka 副本机制

### 副本设计思想
- 每个分区都有一个 Leader 副本，其余的为 Follower 副本；
- 只有 Leader 副本对外提供服务，所有消费者和生产者的读写请求只发往 Leader 副本所在的 Broker，Follower 副本只从领导者副本拉取消息；
- 当Follower 副本挂了之后，Kafka 借助 ZooKeeper 提供的监控功能力，可以实时感知到，然后开始 Leader 选举，从 Follower 副本中选一个作为新的 Leader，老 Leader 副本重启回来后，只能作为 Follower 副本加入到集群中；

### 不支持读写分离的原因
- 会带来数据实时性和一致性问题；
- 因为 Kafka 使用磁盘存储数据，Follwer 副本异步向 Leader 副本拉取消息，延时较高；
- 如果消费者从多个 Follower 副本获取消息可能出现消息一会儿存在、一会儿不存在的现象；

### ISR 副本集合
- 位于 ISR 集合中的副本都是与 Leader 同步的副本，并且 Leader 副本一定位于 ISR 中；
- 只有 Follower 副本落后 Leader 副本的时间不连续超过 replica.lag.time.max.ms 参数值，才认为是与 Leader 同步的，这个值默认为 10s；   