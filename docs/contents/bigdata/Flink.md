# Apache Flink

## 概述

### Flink 是什么

Apache Flink 是一个开源的分布式流处理框架，专为在无边界和有边界的数据流上进行复杂状态处理和事件处理而设计。Flink 能够在分布式环境中提供高性能的数据流处理能力，并且支持批处理和流处理两种模式，使其成为统一的数据处理平台。

**Flink 的核心优势：**
- 真正的流处理引擎，支持事件驱动型应用
- 精确一次（Exactly-Once）语义的状态一致性保证
- 灵活的窗口操作，支持时间驱动和数据驱动窗口
- 高吞吐量、低延迟的流处理能力
- 统一的批处理和流处理编程模型
- 强大的状态管理和容错机制

### 版本演进

| 版本 | 发布年份 | 主要特性 |
|------|----------|----------|
| Flink 1.0 | 2016 | 初始稳定版本，支持 DataStream API |
| Flink 1.3 | 2017 | Table API 增强，SQL 支持初版 |
| Flink 1.5 | 2018 | Kafka Connector 增强，状态后端优化 |
| Flink 1.6 | 2018 | 状态 TTL 支持，PyFlink 初版 |
| Flink 1.9 | 2019 | Blink SQL 引擎合并，Kubernetes 支持增强 |
| Flink 1.11 | 2020 | CDC 连接器，Hive 集成增强 |
| Flink 1.13 | 2021 | 性能优化，PyFlink 增强，Adaptive Scheduler |
| Flink 1.14 | 2021 | 统一批处理和流处理，PyFlink 生产可用 |
| Flink 1.15 | 2022 | 内存模型优化，Python API 增强 |
| Flink 1.17 | 2023 | 性能提升，Hive 兼容增强 |
| Flink 1.18 | 2023 | 状态管理优化，PyFlink 完善 |
| Flink 1.19 | 2024 | Coordinator 优化，性能进一步提升 |

> **版本选择建议**：生产环境建议使用 Flink 1.17 或 1.18，1.14 之前的版本已不再维护。

### 核心组件

Flink 的基本组件主要包括：

- **DataSource**：数据输入源，可以是文件系统（HDFS、S3）、消息队列（Kafka、RocketMQ）、数据库等
- **Transformation**：对数据的处理转换操作，包括 map、filter、flatMap、keyBy 等
- **DataSink**：数据输出目的地，如 Kafka、HDFS、数据库、文件系统等

### 主要特性

- **流处理与批处理统一**：Flink 将流处理和批处理统一为 DataStream API 和 DataSet API，批处理被定义为有界数据流的特殊处理
- **高吞吐量与低延迟**：通过灵活的执行引擎，能够同时支持高吞吐量和低延迟的流处理任务
- **精确一次容错**：通过 Checkpoint 和 Savepoint 机制，保证状态的一致性和精确一次的语义
- **状态管理**：支持有状态的计算，提供 Keyed State 和 Operator State 两种状态管理方式
- **时间语义**：支持 Event Time、Processing Time 和 Ingestion Time 三种时间语义
- **窗口操作**：提供丰富的窗口机制，包括滚动窗口、滑动窗口、会话窗口等

### 应用场景

Flink 的应用场景非常广泛：

- **实时数据处理**：实时监控、实时报警、实时 ETL
- **实时数据分析**：日志分析、用户行为分析、实时报表
- **事件驱动应用**：物联网、智能交通、金融风控、欺诈检测
- **实时机器学习**：特征工程、在线学习、实时推荐
- **数据管道**：数据同步、数据湖构建、CDC 数据集成

---

## 核心概念与架构

### 集群架构

Flink 采用主从架构，主要包括以下组件：

```
┌─────────────────────────────────────────────────────────────────┐
│                        Flink 集群                                │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────────┐                                          │
│  │   JobManager     │  ◄── 主导者                               │
│  │  - Dispatcher    │      - 接收作业提交                       │
│  │  - JobMaster     │      - 协调 Checkpoint                    │
│  │  - ResourceMgr   │      - 任务调度                            │
│  └────────┬─────────┘                                          │
│           │                                                      │
│           │  ◄── 资源分配                                        │
│           ▼                                                      │
│  ┌──────────────────────────────────────────┐                  │
│  │           TaskManager Slot               │                  │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  │                  │
│  │  │ Slot 1  │  │ Slot 2  │  │ Slot 3  │  │                  │
│  │  │ SubTask │  │ SubTask │  │ SubTask │  │                  │
│  │  └─────────┘  └─────────┘  └─────────┘  │                  │
│  └──────────────────────────────────────────┘                  │
│  ┌──────────────────┐  ┌──────────────────┐                  │
│  │   TaskManager     │  │   TaskManager   │                  │
│  └──────────────────┘  └──────────────────┘                  │
└─────────────────────────────────────────────────────────────────┘
```

| 组件 | 职责 |
|------|------|
| **JobManager** | 负责作业调度、Checkpoint 协调、资源管理等，是 Flink 集群的主导者 |
| **TaskManager** | 负责执行具体的 Task（SubTask），缓存和交换数据流，是 Worker 节点 |
| **ResourceManager** | 负责集群资源的管理，包括 Slot 的分配和释放 |
| **Dispatcher** | 提供 REST 接口提交作业，启动 JobMaster，运行 Flink WebUI |

### 核心概念

- **Stream**：数据流，代表无限的数据序列
- **Operator**：算子，对数据流进行转换操作
- **Task**：算子的并行实例，一个 Task 可以包含多个 SubTask
- **SubTask**：算子的并行执行单元，是调度的基本单位
- **Slot**：TaskManager 的资源分配单位，决定了 Task 的并行执行能力
- **JobGraph**：作业图，程序经过转换后的有向无环图
- **ExecutionGraph**：执行图，JobGraph 经过调度后的并行执行图

### 数据流执行模型

Flink 程序的执行流程如下：

```
DataStream API 代码
        │
        ▼
    StreamGraph  ◄── 用户代码直接转换
        │
        ▼
    JobGraph  ◄── 算子链化优化
        │
        ▼
  ExecutionGraph  ◄── 并行化处理
        │
        ▼
   分布式执行  ◄── SubTask 分发到 TaskManager
```

---

## 编程模型

### 环境配置

**Maven 依赖：**

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-streaming-java</artifactId>
    <version>1.18.1</version>
</dependency>

<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-clients</artifactId>
    <version>1.18.1</version>
</dependency>
```

**StreamExecutionEnvironment：**

```java
// 获取流处理环境
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

// 配置并行度
env.setParallelism(4);

// 配置时间语义
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

// 开启 Checkpoint
env.enableCheckpointing(60000);
env.getCheckpointConfig().setMinPauseBetweenCheckpoints(30000);
env.getCheckpointConfig().setCheckpointTimeout(600000);
env.getCheckpointConfig().setMaxConcurrentCheckpoints(1);

// 设置状态后端
env.setStateBackend(new EmbeddedRocksDBStateBackend());
```

### 数据源（Source）

**从集合创建：**

```java
DataStream<String> stream = env.fromCollection(
    Arrays.asList("hello", "world", "flink")
);

// 从单个元素创建
DataStream<Integer> stream2 = env.fromElements(1, 2, 3, 4, 5);
```

**从文件读取：**

```java
DataStream<String> fileStream = env.readTextFile("hdfs://path/to/file");

// 或者使用 FileSource API
FileSource<String> fileSource = FileSource.forRecordStreamFormat(
    new TextLineInputFormat(), new Path("hdfs://path/to/file")
).build();

DataStream<String> stream = env.fromSource(
    fileSource,
    WatermarkStrategy.noWatermarks(),
    "file-source"
);
```

**从 Kafka 读取：**

```java
Properties props = new Properties();
props.setProperty("bootstrap.servers", "localhost:9092");
props.setProperty("group.id", "flink-consumer");

FlinkKafkaConsumer<String> kafkaConsumer = new FlinkKafkaConsumer<>(
    "topic-name",
    new SimpleStringSchema(),
    props
);

// 设置起始位置
kafkaConsumer.setStartFromEarliest();
kafkaConsumer.setStartFromGroupOffsets();

DataStream<String> kafkaStream = env.addSource(kafkaConsumer);
```

**自定义数据源：**

```java
public class MySource implements SourceFunction<String> {
    private volatile boolean isRunning = true;

    @Override
    public void run(SourceContext<String> ctx) {
        int index = 0;
        while (isRunning) {
            ctx.collect("message-" + index++);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                break;
            }
        }
    }

    @Override
    public void cancel() {
        isRunning = false;
    }
}

// 使用自定义 Source
DataStream<String> customStream = env.addSource(new MySource());
```

### 数据转换（Transformation）

**基础转换：**

```java
// map：将每个元素映射为新元素
DataStream<Integer> mapped = stream.map(new MapFunction<String, Integer>() {
    @Override
    public Integer map(String value) {
        return value.length();
    }
});

// lambda 方式
DataStream<Integer> mapped2 = stream.map(String::length);

// filter：过滤元素
DataStream<String> filtered = stream.filter(new FilterFunction<String>() {
    @Override
    public boolean filter(String value) {
        return value.startsWith("hello");
    }
});

// flatMap：展开每个元素为多个元素
DataStream<String> flatMapped = stream.flatMap(new FlatMapFunction<String, String>() {
    @Override
    public void flatMap(String value, Collector<String> out) {
        for (String word : value.split(" ")) {
            out.collect(word);
        }
    }
});
```

**聚合转换：**

```java
// keyBy：按key分组
KeyedStream<String, String> keyed = stream.keyBy(value -> value.split(",")[0]);

// sum：求和
KeyedStream<Tuple2<String, Integer>, String> keyed2 = stream
    .keyBy(value -> value.f0);
DataStream<Tuple2<String, Integer>> summed = keyed2.sum(1);

// reduce：聚合操作
DataStream<Tuple2<String, Integer>> reduced = keyed2.reduce(
    (value1, value2) -> new Tuple2<>(value1.f0, value1.f1 + value2.f1)
);
```

**多流转换：**

```java
// union：合并多个流
DataStream<Integer> stream1 = env.fromElements(1, 2, 3);
DataStream<Integer> stream2 = env.fromElements(4, 5, 6);
DataStream<Integer> unioned = stream1.union(stream2);

// connect：连接两个流
DataStream<Integer> streamA = env.fromElements(1, 2, 3);
DataStream<String> streamB = env.fromElements("a", "b", "c");

ConnectedStreams<Integer, String> connected = streamA.connect(streamB);
DataStream<String> result = connected.map(
    new MapFunction<Integer, String>() {
        @Override
        public String map(Integer value) {
            return "int: " + value;
        }
    },
    new MapFunction<String, String>() {
        @Override
        public String map(String value) {
            return "string: " + value;
        }
    }
);

// split：分流
SplitStream<String> split = stream.split(new OutputSelector<String>() {
    @Override
    public Iterable<String> select(String value) {
        if (value.contains("error")) {
            return Collections.singletonList("error");
        } else {
            return Collections.singletonList("normal");
        }
    }
});

DataStream<String> errorStream = split.select("error");
DataStream<String> normalStream = split.select("normal");
```

### 数据输出（Sink）

**输出到文件：**

```java
// 写入文本文件
stream.writeAsText("hdfs://path/to/output");

// 使用输出格式化
stream.writeAsText("hdfs://path/to/output", FileSystem.WriteMode.OVERWRITE);

// 使用 FileSink API（推荐）
FileSink<String> fileSink = FileSink
    .forRowFormat(new Path("hdfs://output"), new SimpleStringEncoder<String>())
    .withRollingPolicy(DefaultRollingPolicy.builder()
        .withRolloverInterval(Duration.ofMinutes(5))
        .withInactivityInterval(Duration.ofMinutes(3))
        .withMaxPartSize(1024 * 1024 * 1024)
        .build())
    .build();

stream.sinkTo(fileSink);
```

**输出到 Kafka：**

```java
KafkaSink<String> kafkaSink = KafkaSink.<String>builder()
    .setBootstrapServers("localhost:9092")
    .setRecordSerializer(new KafkaRecordSerializationSchema<String>() {
        @Override
        public ProducerRecord<String, String> serialize(String element, @Nullable Long timestamp) {
            return new ProducerRecord<>("output-topic", element);
        }
    })
    .setDeliveryGuarantee(DeliveryGuarantee.EXACTLY_ONCE)
    .setTransactionalIdPrefix("flink-kafka-")
    .build();

stream.sinkTo(kafkaSink);
```

**输出到 MySQL：**

```java
// 使用 JDBC Sink
stream.addSink(JdbcSink.sink(
    "INSERT INTO orders (order_id, amount, timestamp) VALUES (?, ?, ?) ON DUPLICATE KEY UPDATE amount = ?",
    (statement, value) -> {
        statement.setString(1, value.getOrderId());
        statement.setDouble(2, value.getAmount());
        statement.setTimestamp(3, Timestamp.from(value.getTimestamp()));
        statement.setDouble(4, value.getAmount());
    },
    JdbcExecutionOptions.builder()
        .withBatchSize(1000)
        .withBatchIntervalMs(200)
        .withMaxRetries(3)
        .build(),
    new JdbcConnectionOptions.JdbcConnectionOptionsBuilder()
        .withUrl("jdbc:mysql://localhost:3306/db")
        .withDriverName("com.mysql.cj.jdbc.Driver")
        .withUsername("root")
        .withPassword("password")
        .build()
));
```

**自定义 Sink：**

```java
public class MySink implements SinkFunction<String> {
    @Override
    public void invoke(String value, Context context) {
        // 自定义输出逻辑
        System.out.println("Output: " + value);
    }
}

stream.addSink(new MySink());
```

### 完整示例

```java
public class WordCountJob {
    public static void main(String[] args) throws Exception {
        // 获取执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        // 设置并行度
        env.setParallelism(2);

        // 数据源：从 Kafka 读取
        Properties props = new Properties();
        props.setProperty("bootstrap.servers", "localhost:9092");
        props.setProperty("group.id", "wordcount-group");

        DataStream<String> input = env.addSource(
            new FlinkKafkaConsumer<>("wordcount-input", new SimpleStringSchema(), props)
        );

        // 转换：分词
        DataStream<String> words = input
            .flatMap((String line, Collector<String> out) -> {
                for (String word : line.split("\\s+")) {
                    out.collect(word);
                }
            })
            .filter(word -> !word.isEmpty());

        // 分组聚合
        DataStream<Tuple2<String, Integer>> counts = words
            .map(word -> Tuple2.of(word, 1))
            .keyBy(t -> t.f0)
            .sum(1);

        // 输出：到 Kafka
        KafkaSink<Tuple2<String, Integer>> sink = KafkaSink.<Tuple2<String, Integer>>builder()
            .setBootstrapServers("localhost:9092")
            .setRecordSerializer(new KafkaRecordSerializationSchema<Tuple2<String, Integer>>() {
                @Override
                public ProducerRecord<String, byte[]> serialize(Tuple2<String, Integer> element, @Nullable Long timestamp) {
                    return new ProducerRecord<>(
                        "wordcount-output",
                        element.f0.getBytes(StandardCharsets.UTF_8),
                        (element.f0 + "," + element.f1).getBytes(StandardCharsets.UTF_8)
                    );
                }
            })
            .build();

        counts.sinkTo(sink);

        // 输出：打印到控制台（用于测试）
        counts.print();

        // 执行作业
        env.execute("WordCount Job");
    }
}
```

---

## 状态管理与容错

### 状态分类

Flink 支持两种类型的状态：

**Keyed State：**

- 作用于 KeyedStream，按 key 分组后的状态
- 只能在使用 keyBy() 后使用
- 自动绑定到当前处理的 key

```java
DataStream<Tuple2<String, Integer>> input = env.fromElements(
    Tuple2.of("a", 1),
    Tuple2.of("b", 2),
    Tuple2.of("a", 3)
);

input.keyBy(t -> t.f0)
    .map(new StatefulMap());
```

```java
public class StatefulMap implements MapFunction<Tuple2<String, Integer>, Tuple2<String, Integer>> {
    // 定义 ValueState
    private ValueState<Integer> countState;

    @Override
    public Tuple2<String, Integer> map(Tuple2<String, Integer> value) throws Exception {
        // 获取当前状态
        Integer currentCount = countState.value();
        if (currentCount == null) {
            currentCount = 0;
        }

        // 更新状态
        currentCount += value.f1;
        countState.update(currentCount);

        return Tuple2.of(value.f0, currentCount);
    }

    @Override
    public void open(Configuration parameters) {
        // 初始化状态
        countState = getRuntimeContext().getState(
            new ValueStateDescriptor<>("count", Integer.class)
        );
    }
}
```

**Operator State：**

- 作用于整个 Operator，跨 key 共享
- 需要实现 CheckpointedFunction 接口
- 支持 ListState、UnionListState、BroadcastState

```java
public class MyOperator implements MapFunction<String, String>, CheckpointedFunction {
    private ListState<String> checkpointedState;

    @Override
    public String map(String value) {
        return value.toUpperCase();
    }

    @Override
    public void snapshotState(FunctionSnapshotContext context) throws Exception {
        // 保存状态到 Checkpoint
        checkpointedState.clear();
        checkpointedState.add("some-state");
    }

    @Override
    public void initializeState(FunctionInitializationContext context) throws Exception {
        // 初始化状态
        ListStateDescriptor<String> descriptor =
            new ListStateDescriptor<>("checkpointed-state", String.class);
        checkpointedState = context.getOperatorStateStore().getListState(descriptor);
    }
}
```

### 状态后端

Flink 提供三种状态后端：

| 状态后端 | 适用场景 | 特点 |
|---------|----------|------|
| **HashMapStateBackend** | 小状态、低延迟 | 状态存储在 TaskManager JVM 堆内存 |
| **EmbeddedRocksDBStateBackend** | 大状态、超大状态 | 状态存储在 RocksDB，支持增量 Checkpoint |
| **FsStateBackend** | 中等状态 | 状态存储在文件系统（内存缓存） |

**配置：**

```java
// 使用 HashMapStateBackend
env.setStateBackend(new HashMapStateBackend());

// 使用 RocksDBStateBackend（需要添加依赖）
env.setStateBackend(new EmbeddedRocksDBStateBackend());

// 使用 FsStateBackend（已废弃，建议使用 HashMapStateBackend + Checkpoint 存储）
// env.setStateBackend(new FsStateBackend("hdfs://checkpoint-path"));
```

### Checkpoint 检查点

Checkpoint 是 Flink 容错的核心机制，通过定期保存状态快照来实现故障恢复。

**配置：**

```java
env.enableCheckpointing(60000);  // 每 60 秒执行一次 Checkpoint

// Checkpoint 配置
CheckpointConfig checkpointConfig = env.getCheckpointConfig();
checkpointConfig.setMinPauseBetweenCheckpoints(30000);  // 最小间隔
checkpointConfig.setCheckpointTimeout(600000);  // Checkpoint 超时
checkpointConfig.setMaxConcurrentCheckpoints(1);  // 最大并发数
checkpointConfig.setTolerableCheckpointFailureNumber(3);  // 容忍失败次数
checkpointConfig.setCheckpointStorage("hdfs://checkpoint-path");  // Checkpoint 存储

// 增量 Checkpoint（RocksDB）
checkpointConfig.enableIncrementalCheckpointing();
```

**Checkpoint 策略：**

```java
// 精确一次语义（默认）
env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);

// 至少一次语义（低延迟场景）
env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.AT_LEAST_ONCE);
```

### Savepoint 保存点

Savepoint 是手动触发的 Checkpoint，用于计划内停机和迁移。

**触发 Savepoint：**

```bash
# 使用 Flink CLI
flink savepoint <job-id> <savepoint-path>

# 停止作业并触发 Savepoint
flink stop --savepoint <savepoint-path> <job-id>
```

**从 Savepoint 恢复：**

```bash
# 从 Savepoint 恢复作业
flink run -s <savepoint-path> <jar-file>

# 使用指定 Savepoint 恢复，允许忽略无法恢复的状态
flink run -s <savepoint-path> --allowNonRestoredState <jar-file>
```

### 故障恢复

当 TaskManager 失败时，Flink 自动从最近的 Checkpoint 恢复：

1. JobManager 检测到 TaskManager 失败
2. 重新调度受影响的 Task 到可用的 TaskManager
3. 从最近的 Checkpoint 恢复状态
4. 继续处理数据

---

## 窗口与时间语义

### 时间语义

Flink 支持三种时间语义：

| 时间语义 | 说明 | 适用场景 |
|---------|------|----------|
| **Event Time** | 事件实际发生的时间 | 需要精确结果的场景，如日志分析 |
| **Processing Time** | 数据被处理的时间 | 实时性要求高、允许近似结果的场景 |
| **Ingestion Time** | 数据进入 Flink 的时间 | 介于两者之间 |

**配置：**

```java
// 设置时间语义
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

// 设置 Watermark 生成策略
WatermarkStrategy<String> watermarkStrategy = WatermarkStrategy
    .forMonotonousTimestamps()  // 升序时间戳
    .withTimestampAssigner((event, timestamp) -> event.getTimestamp());  // 提取时间戳
```

### Watermark 水位线

Watermark 用于处理乱序事件：

```java
// 升序 Watermark（事件时间顺序）
WatermarkStrategy<MyEvent> ascendingStrategy = WatermarkStrategy
    .<MyEvent>forMonotonousTimestamps()
    .withTimestampAssigner((event, timestamp) -> event.getTimestamp());

// 乱序 Watermark（允许延迟 5 秒）
WatermarkStrategy<MyEvent> outOfOrderStrategy = WatermarkStrategy
    .<MyEvent>forBoundedOutOfOrderness(Duration.ofSeconds(5))
    .withTimestampAssigner((event, timestamp) -> event.getTimestamp());

// 自定义 Watermark
WatermarkStrategy<MyEvent> customStrategy = WatermarkStrategy
    .<MyEvent>forGenerator((context) -> new PeriodicWatermarkGenerator())
    .withTimestampAssigner((event, timestamp) -> event.getTimestamp());

// 在数据流上应用 Watermark
DataStream<MyEvent> eventsWithWatermark = stream.assignTimestampsAndWatermarks(watermarkStrategy);
```

### 窗口分类

**滚动窗口（Tumbling Window）：**

```java
// 滚动时间窗口（5秒）
DataStream<MyEvent> windowed = stream
    .keyBy(MyEvent::getKey)
    .window(TumblingEventTimeWindows.of(Time.seconds(5)));

// 滚动计数窗口（100条）
DataStream<MyEvent> countWindowed = stream
    .keyBy(MyEvent::getKey)
    .countWindow(100);
```

**滑动窗口（Sliding Window）：**

```java
// 滑动时间窗口（窗口大小10秒，滑动步长5秒）
DataStream<MyEvent> slidingWindowed = stream
    .keyBy(MyEvent::getKey)
    .window(SlidingEventTimeWindows.of(Time.seconds(10), Time.seconds(5)));
```

**会话窗口（Session Window）：**

```java
// 会话窗口（间隔超过3秒则认为会话结束）
DataStream<MyEvent> sessionWindowed = stream
    .keyBy(MyEvent::getKey)
    .window(EventTimeSessionWindows.withGap(Time.seconds(3)));
```

**全局窗口（Global Window）：**

```java
// 全局窗口（需要配合自定义触发器使用）
DataStream<MyEvent> globalWindowed = stream
    .keyBy(MyEvent::getKey)
    .window(GlobalWindows.create());
```

### 窗口函数

**增量聚合函数：**

```java
// 使用 aggregate 进行增量聚合
DataStream<Integer> result = stream
    .keyBy(value -> value.getKey())
    .window(TumblingEventTimeWindows.of(Time.minutes(1)))
    .aggregate(
        new AggregateFunction<Integer, Integer, Integer>() {
            @Override
            public Integer createAccumulator() {
                return 0;
            }

            @Override
            public Integer add(Integer value, Integer accumulator) {
                return accumulator + 1;
            }

            @Override
            public Integer getResult(Integer accumulator) {
                return accumulator;
            }

            @Override
            public Integer merge(Integer a, Integer b) {
                return a + b;
            }
        }
    );
```

**全量窗口函数：**

```java
// 使用 process 进行全量处理
DataStream<String> result = stream
    .keyBy(value -> value.getKey())
    .window(TumblingEventTimeWindows.of(Time.minutes(1)))
    .process(new ProcessWindowFunction<String, String, String, TimeWindow>() {
        @Override
        public void process(
            String key,
            Context context,
            Iterable<String> elements,
            Collector<String> out
        ) {
            int count = 0;
            for (String element : elements) {
                count++;
            }
            out.collect("Key: " + key + ", Count: " + count);
        }
    });
```

### 迟到数据处理

```java
// 允许迟到 5 秒
DataStream<MyEvent> windowed = stream
    .keyBy(MyEvent::getKey)
    .window(TumblingEventTimeWindows.of(Time.minutes(1)))
    .allowedLateness(Time.seconds(5))
    .process(new LateDataProcessFunction());

// 侧输出流处理迟到数据
OutputTag<MyEvent> lateDataTag = new OutputTag<MyEvent>("late-data"){};

DataStream<String> result = stream
    .keyBy(MyEvent::getKey)
    .window(TumblingEventTimeWindows.of(Time.minutes(1)))
    .sideOutputLateData(lateDataTag)
    .process(new ProcessWindowFunction<MyEvent, String, String, TimeWindow>() {
        // 主窗口处理逻辑
    });

// 获取侧输出流
DataStream<MyEvent> lateData = result.getSideOutput(lateDataTag);
```

---

## 与 Kafka 集成

### Maven 依赖

```xml
<!-- Flink Kafka Connector -->
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka</artifactId>
    <version>1.18.1</version>
</dependency>

<!-- Kafka Client -->
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>3.6.1</version>
</dependency>
```

### 读取 Kafka 数据

```java
Properties props = new Properties();
props.setProperty("bootstrap.servers", "localhost:9092");
props.setProperty("group.id", "flink-consumer");
props.setProperty("auto.offset.reset", "earliest");
props.setProperty("enable.auto.commit", "true");

// 创建 Kafka Source
KafkaSource<String> kafkaSource = KafkaSource.<String>builder()
    .setBootstrapServers("localhost:9092")
    .setTopics("input-topic")
    .setGroupId("flink-consumer")
    .setStartingOffsets(OffsetsInitializer.earliest())
    .setValueOnlyDeserializer(new SimpleStringSchema())
    .build();

DataStream<String> stream = env.fromSource(
    kafkaSource,
    WatermarkStrategy.noWatermarks(),
    "Kafka Source"
);
```

### 写入 Kafka 数据

```java
// 创建 Kafka Sink
KafkaSink<String> kafkaSink = KafkaSink.<String>builder()
    .setBootstrapServers("localhost:9092")
    .setRecordSerializer(new KafkaRecordSerializationSchema<String>() {
        @Override
        public ProducerRecord<String, String> serialize(String element, @Nullable Long timestamp) {
            // 可以自定义 key 和 topic
            return new ProducerRecord<>("output-topic", element);
        }
    })
    .setDeliveryGuarantee(DeliveryGuarantee.EXACTLY_ONCE)
    .setTransactionalIdPrefix("flink-kafka-tx-")
    .setTransactionTimeout(900000)  // 15 分钟
    .build();

stream.sinkTo(kafkaSink);
```

### Exactly-Once 语义

Flink + Kafka 保证端到端的精确一次语义：

```java
// Source: 从 Checkpoint 恢复偏移量
KafkaSource<String> kafkaSource = KafkaSource.<String>builder()
    .setBootstrapServers("localhost:9092")
    .setTopics("input-topic")
    .setGroupId("flink-consumer")
    .setStartingOffsets(OffsetsInitializer.committedOffsets())
    .setValueOnlyDeserializer(new SimpleStringSchema())
    .setBoundedTimeout(Duration.ofMillis(1000))  // 设置结束时间
    .build();

// 开启 Checkpoint
env.enableCheckpointing(5000);
env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
env.getCheckpointConfig().setMinPauseBetweenCheckpoints(500);

// Sink: Exactly-Once
KafkaSink<String> kafkaSink = KafkaSink.<String>builder()
    .setBootstrapServers("localhost:9092")
    .setRecordSerializer(new KafkaRecordSerializationSchema<String>() {
        @Override
        public ProducerRecord<String, String> serialize(String element, @Nullable Long timestamp) {
            return new ProducerRecord<>("output-topic", element);
        }
    })
    .setDeliveryGuarantee(DeliveryGuarantee.EXACTLY_ONCE)
    .setTransactionalIdPrefix("flink-kafka-")
    .build();
```

### Kafka Connector 配置

| 配置项 | 说明 |
|--------|------|
| `bootstrap.servers` | Kafka broker 地址列表 |
| `group.id` | 消费者组 ID |
| `auto.offset.reset` | 偏移量重置策略 |
| `enable.auto.commit` | 自动提交偏移量 |
| `transactional.id.prefix` | 事务 ID 前缀 |
| `transaction.timeout` | 事务超时时间 |

---

## 配置与优化

### 集群配置

**flink-conf.yaml 关键配置：**

```yaml
# TaskManager 配置
taskmanager.numberOfTaskSlots: 4
taskmanager.memory.process.size: 4096m
taskmanager.memory.heap.size: 2048m
taskmanager.memory.managed.fraction: 0.4

# JobManager 配置
jobmanager.memory.process.size: 2048m

# Checkpoint 配置
execution.checkpointing.interval: 60s
execution.checkpointing.mode: EXACTLY_ONCE
execution.checkpointing.timeout: 10min
execution.checkpointing.min-pause: 30s
execution.checkpointing.max-concurrent-checkpoints: 1

# 状态后端
state.backend: rocksdb
state.checkpoints.dir: hdfs://checkpoint/fnk
state.savepoints.dir: hdfs://savepoints/fnk

# RocksDB 配置
state.backend.incremental: true
rocksdb.memory.managed: true
```

### 内存模型

Flink 1.18 的内存模型：

```
┌─────────────────────────────────────────────────────────────────┐
│                      TaskManager 内存                          │
├───────────────────────────────┬─────────────────────────────────┤
│        Heap Memory            │        Off-Heap Memory          │
│  ┌─────────────────────────┐  │  ┌─────────────────────────┐   │
│  │    Network Memory       │  │  │    Managed Memory       │   │
│  │    (25%)               │  │  │    (RocksDB 状态)        │   │
│  │                        │  │  │    (40%)                │   │
│  └─────────────────────────┘  │  └─────────────────────────┘   │
│  ┌─────────────────────────┐  │                                 │
│  │    State Backend       │  │                                 │
│  │    (HashMap 状态)       │  │                                 │
│  │    (剩余堆内存)         │  │                                 │
│  └─────────────────────────┘  │                                 │
└───────────────────────────────┴─────────────────────────────────┘
```

### 并行度设置

**算子级别并行度：**

```java
// 设置整个 Job 的全局并行度
env.setParallelism(4);

// 设置某个算子的并行度
stream.map(...).setParallelism(8);

// 设置数据源的并行度
env.addSource(...).setParallelism(4);

// 设置 Sink 的并行度
stream.sinkTo(...).setParallelism(2);
```

**并行度优先级**（从高到低）：
1. 算子级别设置
2. 执行环境级别设置
3. 提交作业时的参数
4. 配置文件默认值

```bash
# 提交作业时设置并行度
flink run -p 4 <jar-file>
```

### 性能调优

**算子链（Operator Chaining）：**

```java
// 启用算子链（默认开启）
env.enableOperatorChaining();

// 禁用特定算子的链
stream.map(...).disableChaining();

// 从某个算子开始新链
stream.map(...).startNewChain();
```

**缓冲区（Buffer）：**

```java
// 设置网络缓冲区超时
env.setBufferTimeout(100);  // 毫秒

// 设置 Task 之间的缓冲区数量
env.getConfig().setDefaultNetworkBufferDuration(100);
```

**背压（Backpressure）：**

```bash
# 查看背压状态
flink list

# 查看具体 Task 的背压
# 通过 Web UI 查看 Task Manager -> Task -> Back Pressure
```

**反序列化优化：**

```java
// 使用 Kryo 序列化器（默认）
env.getConfig().setDefaultKryoSerializer();

// 注册自定义序列化器
env.getConfig().registerTypeWithKryoSerializer(MyClass.class, new MySerializer());
```

---

## 部署与运行

### Standalone 集群

**集群启动：**

```bash
# 启动 Flink 集群
./bin/start-cluster.sh

# 停止 Flink 集群
./bin/stop-cluster.sh
```

**提交作业：**

```bash
# 提交作业
./bin/flink run <jar-file>

# 提交作业并指定参数
./bin/flink run -p 4 -c com.example.Job <jar-file> --arg1 value1

# 取消作业
./bin/flink cancel <job-id>

# 查看作业状态
./bin/flink list
```

### Yarn 集群

**启动 Session 集群：**

```bash
# 创建一个 Yarn Session
./bin/yarn-session.sh -n 4 -s 4 -tm 4096 -d

# 参数说明：
# -n: TaskManager 数量
# -s: 每个 TaskManager 的 Slot 数量
# -tm: TaskManager 内存
# -d: 后台运行
```

**提交作业到 Yarn：**

```bash
# 提交到 Yarn Session
./bin/flink run <jar-file>

# 直接提交到 Yarn（Per-Job 模式）
./bin/flink run -m yarn-cluster <jar-file>
```

### Kubernetes 部署

**部署配置：**

```yaml
# flink-configuration-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: flink-config
data:
  flink-conf.yaml: |
    taskmanager.numberOfTaskSlots: 4
    state.backend: rocksdb
    execution.checkpointing.interval: 60s
```

```yaml
# jobmanager-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flink-jobmanager
spec:
  replicas: 1
  selector:
    matchLabels:
      component: jobmanager
  template:
    metadata:
      labels:
        component: jobmanager
    spec:
      containers:
      - name: jobmanager
        image: flink:1.18.1
        ports:
        - containerPort: 8081
        command: ["/bin/bash", "-c", "jobmanager.sh start-cluster"]
```

---

## 常见问题与 Tips

### 常见问题

**Q: Flink 作业一直处于 RUNNING 状态但没有输出？**

A: 检查以下几点：
1. 确认数据源是否有数据输入
2. 检查并行度设置是否正确
3. 查看 Web UI 确认 Task 是否正常运行
4. 检查日志是否有错误信息

**Q: Checkpoint 失败怎么办？**

A: 常见原因和解决方案：
1. Checkpoint 存储路径不可用 - 检查 HDFS/NFS 配置
2. 状态过大 - 调整内存或使用 RocksDB
3. 超时 - 增加 checkpoint timeout

**Q: 如何处理背压？**

A: 解决方案：
1. 增加 TaskManager 的 Slot 数量
2. 调整算子并行度
3. 优化算子逻辑，减少处理时间
4. 增加缓冲区大小

**Q: 状态在 Checkpoint 后丢失？**

A: 检查：
1. 状态后端配置是否正确
2. Checkpoint 是否成功完成
3. 是否使用了正确的方式恢复（Savepoint/Checkpoint）

**Q: Watermark 策略如何选择？**

A: 建议：
- 升序时间戳：使用 `forMonotonousTimestamps`
- 已知最大延迟：使用 `forBoundedOutOfOrderness`
- 复杂场景：自定义 WatermarkGenerator

### Tips

**1. 使用 DataStream API 时启用算子链**

```java
env.enableOperatorChaining();
```

**2. 合理设置 Checkpoint 间隔**

生产环境建议 60-300 秒。

**3. 使用 ProcessTime 时注意时区**

```java
env.setStreamTimeCharacteristic(TimeCharacteristic.ProcessingTime);
// 或者
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
// 并配置时区
env.getConfig().setLocalTimeZone(TimeZone.getTimeZone("Asia/Shanghai"));
```

**4. 状态过期清理**

```java
// 状态 TTL 配置
StateTtlConfig ttlConfig = StateTtlConfig.newBuilder(Time.hours(1))
    .setUpdateType(StateTtlConfig.UpdateType.OnReadAndWrite)
    .setStateVisibility(StateTtlConfig.StateVisibility.NeverReturnExpired)
    .build();

ValueStateDescriptor<String> descriptor = new ValueStateDescriptor<>("state", String.class);
descriptor.enableTimeToLive(ttlConfig);
```

**5. 使用BroadcastState实现动态规则**

```java
// 广播流
DataStream<Rule> ruleStream = env.addSource(...).broadcast();

// 非广播流
DataStream<Event> eventStream = env.addSource(...);

// 合并
BroadcastStream<Rule> broadcastStream = ruleStream.broadcast(
    new MapStateDescriptor<>("rules", String.class, Rule.class)
);

eventStream.connect(broadcastStream)
    .process(new BroadcastProcessFunction<...>() {
        // 处理逻辑
    });
```

**6. 监控和指标**

```java
// 添加内置指标
env.getCheckpointConfig().setTolerableCheckpointFailureNumber(3);
env.getCheckpointConfig().registerCheckpointMetricsCheckpointListener();
```

---

## 参考资料

- [Apache Flink 官方文档](https://flink.apache.org/)
- [Flink 中文文档](https://flink-learning.org.cn/)
- [Flink SQL 文档](https://nightlies.apache.org/flink/flink-docs-release-1.18/zh/)
- [Flink GitHub](https://github.com/apache/flink)
- [Kafka Connect](https://docs.confluent.io/kafka-connect-flink/current/)
