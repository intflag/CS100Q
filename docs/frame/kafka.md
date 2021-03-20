# Kafka
## 普通操作
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