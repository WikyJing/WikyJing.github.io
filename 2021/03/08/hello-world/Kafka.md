# Kafka

## kafka安装

```bash
1.安装kafka
brew install kafka
2.修改server.properties
vi /usr/local/etc/kafka/server.properties
增加一行配置如下：
#listeners = PLAINTEXT://your.host.name:9092
listeners=PLAINTEXT://localhost:9092
3.启动zookeeper 
zkServer start
4. 以server.properties的配置，启动kafka
在kafka的bin目录下(/usr/local/Cellar/kafka/2.3.0/libexec/bin)：执行命令：./kafka-server-start.sh /usr/local/etc/kafka/server.properties
5.新建session，查看kafka的topic
在kafka的bin目录下：执行命令：./kafka-topics.sh --list --zookeeper localhost:2181
6.启动kafka生产者
在kafka的bin目录下：
执行命令：./kafka-console-producer.sh --topic [topic-name]  --broker-listlocalhost:9092(第2步修改的listeners)
7.启动kafka消费者
在kafka的bin目录下：执行命令：./kafka-console-consumer --bootstrap-serverlocalhost:9092 —topic [topic-name]
```

##  本地kafka启动

```bash
# 先启动zk
zkServer start
# 再启动kafka
./kafka-server-start.sh /usr/local/etc/kafka/server.properties
```



## 管理

```bash
## 创建主题（4个分区，2个副本）
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 2 --partitions 4 --topic test
```

## 查询

```bash
## 查询集群描述
bin/kafka-topics.sh --describe --zookeeper localhost:2181

## topic列表查询
bin/kafka-topics.sh --zookeeper 127.0.0.1:2181 --list

## topic列表查询（支持0.9版本+）
bin/kafka-topics.sh --list --bootstrap-server localhost:9092

## 新消费者列表查询（支持0.9版本+）
bin/kafka-consumer-groups.sh --new-consumer --bootstrap-server localhost:9092 --list

## 新消费者列表查询（支持0.10版本+）
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list

## 显示某个消费组的消费详情（仅支持offset存储在zookeeper上的）
bin/kafka-run-class.sh kafka.tools.ConsumerOffsetChecker --zookeeper localhost:2181 --group test

## 显示某个消费组的消费详情（0.9版本 - 0.10.1.0 之前）
bin/kafka-consumer-groups.sh --new-consumer --bootstrap-server localhost:9092 --describe --group test-consumer-group

## 显示某个消费组的消费详情（0.10.1.0版本+）
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-group
```

## 分区操作（kafka-consumer-groups.sh）

```bash
# --to-earliest：把位移调整到分区当前最小位移
./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group test-group --reset-offsets --all-topics --to-earliest --execute
# --to-latest：把位移调整到分区当前最新位移
./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group test-group --reset-offsets --all-topics --to-latest --execute
# --to-current：把位移调整到分区当前位移
./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group test-group --reset-offsets --all-topics --to-current --execute
# --to-offset <offset>： 把位移调整到指定位移处
./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group test-group --reset-offsets --all-topics --to-offset 500000 --execute
# --shift-by N： 把位移调整到当前位移 + N处，注意N可以是负数，表示向前移动
./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group test-group --reset-offsets --all-topics --shift-by -100000 --execute
# --to-datetime <datetime>：把位移调整到大于给定时间的最早位移处，datetime格式是yyyy-MM-ddTHH:mm:ss.xxx，比如2017-08-04T00:00:00.000
./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group test-group --reset-offsets --all-topics --to-datetime 2017-08-04T14:30:00.000
# --by-duration <duration>：把位移调整到距离当前时间指定间隔的位移处，duration格式是PnDTnHnMnS，比如PT0H5M0S（将所有分区位移调整为30分钟之前的最早位移）
./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group test-group --reset-offsets --all-topics --by-duration PT0H30M0S
# --dry-run：把group <personListener>在topic <TPerson1>的所有分区的offset设置到了最后，从而可以跳过所有数据,用了--dry-run选项，不会真正执行。用--execute选项才真正执行
./kafka-consumer-groups --bootstrap-server localhost:9092 --group personListener --topic TPerson1 --reset-offsets --to-latest --dry-run
```

## 发送和消费

```bash
## 生产者
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test

## 消费者
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test

## 新生产者（支持0.9版本+）
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test --producer.config config/producer.properties

## 新消费者（支持0.9版本+）
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --new-consumer --from-beginning --consumer.config config/consumer.properties

## 高级点的用法
bin/kafka-simple-consumer-shell.sh --brist localhost:9092 --topic test --partition 0 --offset 1234  --max-messages 10
```

## 平衡leader

```bash
bin/kafka-preferred-replica-election.sh --zookeeper zk_host:port/chroot
```

## kafka自带压测命令

```bash
bin/kafka-producer-perf-test.sh --topic test --num-records 100 --record-size 1 --throughput 100  --producer-props bootstrap.servers=localhost:9092
```

## 增加副本

1. 创建规则json

   ```bash
   cat > increase-replication-factor.json <<EOF
   {"version":1, "partitions":[
   {"topic":"__consumer_offsets","partition":0,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":1,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":2,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":3,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":4,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":5,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":6,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":7,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":8,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":9,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":10,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":11,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":12,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":13,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":14,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":15,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":16,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":17,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":18,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":19,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":20,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":21,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":22,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":23,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":24,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":25,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":26,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":27,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":28,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":29,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":30,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":31,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":32,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":33,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":34,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":35,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":36,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":37,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":38,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":39,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":40,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":41,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":42,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":43,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":44,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":45,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":46,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":47,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":48,"replicas":[0,1]},
   {"topic":"__consumer_offsets","partition":49,"replicas":[0,1]}]
   }
   EOF
   ```

2. 执行

   ```bash
   bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --reassignment-json-file increase-replication-factor.json --execute
   ```

3. 验证

   ```bash
   bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --reassignment-json-file increase-replication-factor.json --verify
   ```




## Log Compaction

Only need the latest message with the same key:

`https://towardsdatascience.com/log-compacted-topics-in-apache-kafka-b1aa1e4665a7`



```bash
./kafka-topics.sh --zookeeper 127.0.0.1:2181 --create --topic test_distinct_topic --partitions 3 --replication-factor 1 --config "cleanup.policy=compact" --config "delete.retention.ms=100" --config "segment.ms=100" --config "min.cleanable.dirty.ratio=0.01"
# cleanup.policy=compact 配置清理策略为compact
# delete.retention.ms=100 对于近来的消息会存在100ms，过了100ms，重复key的消息就会被清除
# segment.ms=100 kafka消息是以segment形式存在的，默认是以大小进行分区(默认1G)，也可以设置使用时间进行分区
# min.cleanable.dirty.ratio=0.01         dirty ratio = the number of bytes in the head / total number of bytes in the log(tail + head)
```

```
./kafka-console-producer.sh --broker-list localhost:9092 --topic latest-product-price --property parse.key=true --property key.separator=:

./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic latest-product-price --property print.key=true --property key.separator=: --from-beginning
```

**topic是由partition组成的，partition是由segment组成的**

## Recap

* Data in Kafka is stored in topics
* Topics are partitioned
* Each partition is further divided into segments
* Each segment has a log file to store the actual message and an index file to store the position of the messages in the log file
* Various partitions of a topic can be on different brokers but a partition is always tied to a single broker
* Replicated partitions are passive. You can consume messages from them only when the leader is down



```
alias KafkaServer='bash /usr/local/Cellar/kafka/2.3.0/libexec/bin/kafka-server-start.sh /usr/local/etc/kafka/server.properties'
```



```
/usr/local/Cellar/kafka/2.4.1/bin/zookeeper-server-start /usr/local/etc/kafka/zookeeper.properties
/usr/local/Cellar/kafka/2.4.1/bin/kafka-server-start /usr/local/etc/kafka/server.properties
```

