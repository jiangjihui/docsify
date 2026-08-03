# Kafka

Apache Kafka 是一个分布式流处理平台，它最初由 LinkedIn 开发，并于 2011 年开源，随后成为 Apache Software Foundation 下的一个顶级项目。Kafka 主要用于构建实时数据管道和流处理应用程序，它提供了发布/订阅消息传递服务，具有高吞吐量、可靠性和持久性。

## 简介

### 背景

- Kafka最初由LinkedIn公司开发，使用Scala语言编写，目前是Apache的开源项目。
- 它是一个分布式流媒体平台，提供高吞吐量、低延迟的消息传递机制，适用于大规模数据处理和实时数据流场景。

### 核心组件与概念

1. **Broker**：Kafka服务器，负责消息的存储和转发。在Kafka集群中，每个Broker都是一个独立的节点，可以处理消息的读写操作。
2. **Topic**：消息类别，Kafka按照Topic来分类消息。生产者将消息发送到特定的Topic，消费者则订阅这些Topic以获取消息。
3. **Partition**：Topic的分区，一个Topic可以包含多个Partition。Partition是Kafka实现水平扩展和并行处理的关键。每个Partition都是一个有序的、不可变的记录序列，消息被追加到Partition的末尾，并分配一个唯一的偏移量（Offset）。
4. **Offset**：消息在日志中的位置，代表该消息的唯一序号。消费者通过Offset来定位并读取消息。
5. **Producer**：消息生产者，负责将消息发送到Kafka集群中的特定Topic。
6. **Consumer**：消息消费者，从Kafka集群中订阅特定的Topic，并处理所接收的消息。
7. **Consumer Group**：消费者分组，每个Consumer必须属于一个Group。Kafka会将同一个Topic中的消息分发给同一个Group内的不同Consumer，以实现负载均衡和消息的并行处理。
8. **Zookeeper**：保存Kafka集群的元数据，如Broker、Topic、Partition等信息。同时，Zookeeper还负责Broker故障发现、Partition Leader选举和负载均衡等功能。

### 主要特点

1. **高吞吐量**：Kafka能够处理非常高的消息吞吐量，适用于大规模数据处理和实时数据流场景。
2. **低延迟**：Kafka具有较低的消息传递延迟，能够提供快速的消息传递服务。
3. **可伸缩性**：Kafka可以水平扩展，通过增加更多的节点来扩展处理能力和存储容量。
4. **持久性**：Kafka使用磁盘存储消息，确保消息的持久性和可靠性。
5. **高可靠性**：Kafka通过副本机制保证消息的可靠性，即使某些节点发生故障，也不会丢失消息。
6. **分区与并行处理**：Kafka的消息被分成多个分区，每个分区可以在不同的服务器上进行写入和读取，提高了并发性能。

### 适用场景

1. **实时数据流处理**：如实时日志处理、实时监控、实时推荐等。
2. **分布式日志集中存储**：用于收集、存储和分发日志数据，如应用日志、操作日志、系统日志等。
3. **数据集成和数据管道**：作为数据集成和数据管道的中间件，在不同系统之间传递数据。
4. **消息队列和事件驱动架构**：作为消息队列使用，处理异步消息和事件驱动的架构。
5. **大数据处理和流处理**：与大数据处理框架如Hadoop、Spark、Flink等集成，支持大规模数据的处理和分析。

### 优缺点

- **优点**：高吞吐量、低延迟、可伸缩性、持久性、高可靠性、分区与并行处理。
- **缺点**：扩容操作相对复杂、依赖Zookeeper（KRaft模式已解决）、跨分区消息顺序性无法保证。

### 生态系统

- **Kafka Connect**：用于构建和运行可靠的流式数据管道。
- **Kafka Streams**：用于构建流处理应用程序和服务。
- **Kafka MirrorMaker**：用于在 Kafka 集群之间复制数据。
- **Schema Registry**：用于管理和验证 Avro、JSON 等数据格式的模式。

## 架构与存储

### 集群架构

```
Producer ──→ Broker1 ──→ Broker2 ──→ Broker3 ──→ Consumer
                │           │           │
                └───────────┴───────────┘
                        ZooKeeper / KRaft
                 (集群元数据、Controller选举)
```

- **Controller**：集群中某个Broker充当Controller角色，负责Partition Leader选举、Broker上下线处理
- **ZooKeeper / KRaft**：维护集群元数据，Kafka 3.x 开始支持KRaft模式替代ZooKeeper

### 日志存储结构

每个Partition在磁盘上对应一个目录，包含多个Segment文件：

```
topic-order-0/               # Partition 0
├── 00000000000000000000.log  # 消息数据文件
├── 00000000000000000000.index # 偏移量索引文件
├── 00000000000000000000.timeindex # 时间戳索引文件
├── 00000000000005367851.log  # 下一个Segment
├── 00000000000005367851.index
└── 00000000000005367851.timeindex
```

- **Segment**：Partition按大小（默认1GB）或时间切分为多个Segment，便于日志清理和查找
- **Offset Index**：稀疏索引，每4KB记录一条，用于快速定位消息位置
- **Time Index**：按时间戳索引，支持按时间消费

### 高性能原理

| 技术 | 说明 |
| --- | --- |
| **顺序写** | 消息追加到日志末尾，无需随机写磁盘，速度接近内存写 |
| **零拷贝** | 使用 `sendfile()` 系统调用，数据从磁盘直接到网卡，不经过用户空间 |
| **页缓存** | 利用OS的Page Cache，写入先到内存再批量刷盘，读取直接命中缓存 |
| **批量压缩** | 生产者批量发送，减少网络IO次数；支持gzip/snappy/lz4/zstd压缩 |
| **分区分段** | 并行读写，Segment便于日志清理和查找 |

## 分区与副本

### 分区策略

生产者发送消息时，决定消息进入哪个Partition：

| 策略 | 说明 | 适用场景 |
| --- | --- | --- |
| 指定Partition | 直接指定分区号 | 需要精确控制 |
| Key哈希 | 相同Key的消息进入同一分区（`hash(key) % partitions`） | 需要保序的消息 |
| 轮询（默认） | Round Robin分配 | 无序要求，均匀分布 |
| 自定义分区器 | 实现 `Partitioner` 接口 | 业务规则分区 |

> **注意**：Key相同保证同一分区有序，但不保证跨分区有序。如果分区数发生变化，Key的映射关系会改变。

### 副本机制

每个Partition有多个副本（Replica），分布在不同Broker上：

- **Leader Replica**：处理该分区所有读写请求
- **Follower Replica**：从Leader拉取数据同步，不处理客户端请求
- **ISR（In-Sync Replicas）**：与Leader保持同步的副本集合，包含Leader自身

**Leader选举**：当Leader宕机时，从ISR中选出一个Follower成为新Leader。

**副本同步流程**：

```
Producer → Leader Replica → 写入本地日志
                              ↓
                    Follower 拉取（Pull模式）
                              ↓
                    Follower 写入本地日志
                              ↓
                    更新 ISR 列表
```

### 副本因子与可靠性

| `acks` 配置 | 行为 | 可靠性 | 吞吐量 |
| --- | --- | --- | --- |
| `acks=0` | 发送后不等响应 | 最低，可能丢数据 | 最高 |
| `acks=1` | Leader写入即返回 | 中等，Leader宕机可能丢 | 中等 |
| `acks=all` | 所有ISR副本写入才返回 | 最高，不丢数据 | 最低 |

## 消费者组

### 消费者组机制

- 同一Consumer Group内的消费者共同消费一个Topic的所有Partition
- 每个Partition只被组内一个消费者消费（确保消息不重复）
- 消费者数量超过Partition数时，多余消费者空闲

```
Topic (3 Partitions)        Consumer Group A
┌──────────┐               ┌──────────────┐
│ P0       │ ────────────→ │ Consumer 1   │ (消费 P0, P1)
│ P1       │ ────────────→ │ Consumer 1   │
│ P2       │ ────────────→ │ Consumer 2   │ (消费 P2)
└──────────┘               └──────────────┘
```

### Rebalance

当消费者加入/离开Group，或Partition数变化时，触发Rebalance——重新分配Partition与消费者的映射关系。

**Rebalance触发条件：**
- 消费者加入或离开Group
- 消费者心跳超时（`session.timeout.ms`）
- 订阅的Topic分区数变化

**Rebalance的问题**：期间所有消费者停止消费，造成短暂不可用。可通过以下方式减少影响：
- 调大 `session.timeout.ms`（默认10s）和 `heartbeat.interval.ms`（默认3s）
- 使用 `StickyAssignor` 分配策略，尽量保持原有分配

### 分区分配策略

| 策略 | 说明 |
| --- | --- |
| RangeAssignor（默认） | 按Partition编号范围连续分配给消费者 |
| RoundRobinAssignor | 轮询分配，跨Topic均匀分布 |
| StickyAssignor | 尽量保持原有分配，Rebalance时最少移动 |

### Offset管理

- **自动提交**：`enable.auto.commit=true`，定期提交（`auto.commit.interval.ms`，默认5s），可能丢消息或重复消费
- **手动提交**：`enable.auto.commit=false`，处理完消息后调用 `commitSync()` 或 `commitAsync()`
- **存储位置**：Kafka 0.9+ 默认存储在 `__consumer_offsets` 内部Topic中

## 生产者与消费者

### 生产者核心配置

```properties
# Broker地址
bootstrap.servers=localhost:9092
# Key序列化器
key.serializer=org.apache.kafka.common.serialization.StringSerializer
# Value序列化器
value.serializer=org.apache.kafka.common.serialization.StringSerializer
# ACK确认级别 (0/1/all)
acks=all
# 重试次数
retries=3
# 批量发送大小（字节）
batch.size=16384
# 批量等待时间（毫秒）
linger.ms=5
# 缓冲区大小
buffer.memory=33554432
```

### 消费者核心配置

```properties
# Broker地址
bootstrap.servers=localhost:9092
# 反序列化器
key.deserializer=org.apache.kafka.common.serialization.StringDeserializer
value.deserializer=org.apache.kafka.common.serialization.StringDeserializer
# 消费者组ID
group.id=order-service
# 自动提交Offset
enable.auto.commit=false
# 消费起始位置（earliest/latest/none）
auto.offset.reset=earliest
# 一次拉取最大记录数
max.poll.records=500
# 两次poll最大间隔（超过则触发Rebalance）
max.poll.interval.ms=300000
```

### Exactly-Once 语义

Kafka 通过幂等性（Idempotence）和事务（Transaction）实现 Exactly-Once：

**幂等性生产者**（防止生产者重试导致重复）：

```properties
enable.idempotence=true
```

每个Producer分配一个PID，Broker通过 `<PID, Partition, SeqNum>` 去重。

**事务**（跨Partition原子写入）：

```java
producer.initTransactions();
try {
    producer.beginTransaction();
    producer.send(record1);
    producer.send(record2);
    producer.commitTransaction();
} catch (Exception e) {
    producer.abortTransaction();
}
```

## Kafka 3.x 新特性

### KRaft 模式

Kafka 2.8 开始引入 KRaft（Kafka Raft），3.3 标记为生产可用，完全移除 ZooKeeper 依赖：

| 对比项 | ZooKeeper 模式 | KRaft 模式 |
| --- | --- | --- |
| 元数据存储 | ZooKeeper 集群 | Kafka 内部Topic `@metadata` |
| Controller选举 | ZooKeeper 临时节点 | Raft 协议选举 |
| 运维复杂度 | 需额外维护ZK集群 | 只需维护Kafka集群 |
| Partition数支持 | 数万级 | 数百万级 |
| 启动速度 | 分钟级 | 秒级 |

**KRaft 启动配置：**

```properties
# server.properties
node.id=1
process.roles=broker,controller
controller.quorum.voters=1@host1:9093,2@host2:9093,3@host3:9093
listeners=PLAINTEXT://:9092,CONTROLLER://:9093
```

### 其他 3.x 特性

- **分层存储（Tiered Storage）**：热数据存本地，冷数据自动迁移到S3/HDFS
- **增量协作Rebalance**：减少Rebalance期间的停顿
- **移除ZooKeeper依赖**：Kafka 4.0 完全不再支持ZK模式

## 常用命令

### Topic 管理

```bash
# 创建Topic（3分区，2副本）
kafka-topics.sh --create --topic order \
  --partitions 3 --replication-factor 2 \
  --bootstrap-server localhost:9092

# 查看Topic列表
kafka-topics.sh --list --bootstrap-server localhost:9092

# 查看Topic详情
kafka-topics.sh --describe --topic order --bootstrap-server localhost:9092

# 修改分区数（只能增加）
kafka-topics.sh --alter --topic order --partitions 6 --bootstrap-server localhost:9092

# 删除Topic
kafka-topics.sh --delete --topic order --bootstrap-server localhost:9092
```

### 消费者组

```bash
# 查看所有消费者组
kafka-consumer-groups.sh --list --bootstrap-server localhost:9092

# 查看消费者组详情（含Offset和Lag）
kafka-consumer-groups.sh --describe --group order-service \
  --bootstrap-server localhost:9092

# 重置Offset（earliest/latest）
kafka-consumer-groups.sh --reset-offsets --group order-service \
  --topic order --to-earliest --execute \
  --bootstrap-server localhost:9092
```

### 生产与消费

```bash
# 控制台生产者
kafka-console-producer.sh --topic order --bootstrap-server localhost:9092

# 控制台消费者（从头消费）
kafka-console-consumer.sh --topic order --from-beginning \
  --bootstrap-server localhost:9092

# 查看Topic消息数量
kafka-run-class.sh kafka.tools.GetOffsetShell \
  --broker-list localhost:9092 --topic order
```

### 集群运维

```bash
# 查看Broker配置
kafka-configs.sh --describe --broker 1 --bootstrap-server localhost:9092

# 查看集群元数据
kafka-metadata.sh snapshot --snapshot /tmp/kafka-logs/meta.properties

# 查看LogDirs信息
kafka-log-dirs.sh --describe --bootstrap-server localhost:9092
```
