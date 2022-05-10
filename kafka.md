Kafka 是一个分布式流式处理平台
1. 应用
   1. 消息队列 ：建立实时流数据管道，以可靠地在系统或应用程序之间获取数据。
   2. 数据处理： 构建实时的流数据处理程序来转换或处理数据流。
2. 优势：性能好，系统兼容性好
3. 多分区：Kafka 通过给特定 Topic 指定多个 Partition, 而各个 Partition 可以分布在不同的 Broker 上, 这样便能提供比较好的并发能力（负载均衡）。
4. 多副本：Partition 可以指定对应的 Replica 数, 这也极大地提高了消息存储的安全性, 提高了容灾能力，不过也相应的增加了所需要的存储空间。
5. 发布-订阅模型

## 基础概念
1. Producer
2. Consumer
3. Broker，可以看作是一个独立的 Kafka 实例
4. Topic， Producer 将消息发送到特定的主题，Consumer 通过订阅特定的 Topic(主题) 来消费消息。
5. Partition，属于 Topic 的一部分。一个 Topic 可以有多个 Partition ，并且同一 Topic 下的 Partition 可以分布在不同的 Broker 上，这也就表明一个 Topic 可以横跨多个 Broker 。

## 多副本机制
1. 针对Partition
2. 分区（Partition）中的多个副本之间会有一个叫做 leader 的家伙，其他副本称为 follower。我们发送的消息会被发送到 leader 副本，然后 follower 副本才能从 leader 副本中拉取消息进行同步。

## zookeeper
1. ZooKeeper 主要为 Kafka 提供元数据的管理的功能
2. broker注册、Topic 注册、分区负载均衡


## 问题 
### Kafka 只能保证 Partition(分区) 中的消息有序。
1. 消息在被追加到 Partition(分区)的时候都会分配一个特定的偏移量（offset）。Kafka 通过偏移量（offset）来保证消息在分区内的顺序性。
2. 解决方案
   1. 1 个 Topic 只对应一个 Partition。
   2. （推荐）发送消息的时候指定 key/Partition

### Kafka 如何保证消息不丢失
1. 生产者发送的信息丢失
   1. retry=3
2. 消费者获取的信息丢失
   1. 偏移量offset
3. kafak丢失信息
   1. 分区多副本机制

### Kafka 如何保证消息不重复消费
1. 原因：服务端侧已经消费的数据没有成功提交 offset
2. 消费消息服务做幂等校验，比如 Redis 的set、MySQL 的主键等天然的幂等功能，根据主键查一下，如果数据有了，就别插入了。