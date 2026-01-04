# Kafka 面试题与答案

### Kafka 基础概念
* **什么是 Kafka？**
    * 分布式流处理平台，主要用于构建实时数据管道和流应用。
    * 特点：高吞吐、低延迟、可扩展、持久化、高可靠。
* **Kafka 的核心组件有哪些？**
    * **Producer**：生产者，发送消息。
    * **Consumer**：消费者，拉取消息。
    * **Broker**：Kafka 节点，存储消息。
    * **Topic**：逻辑上的消息分类（主题）。
    * **Partition**：物理上的分片，一个 Topic 可以有多个 Partition，实现负载均衡。
    * **Consumer Group**：消费者组，组内消费者共同消费一个 Topic，Partition 只能被组内一个消费者消费。
    * **Zookeeper/KRaft**：元数据管理（新版本逐渐移除 ZK 转向 KRaft 模式）。

### 架构与原理
* **为什么 Kafka 吞吐量这么高？（高性能设计）**
    * **顺序写磁盘**：利用磁盘顺序 I/O 性能远高于随机 I/O 的特性。
    * **零拷贝 (Zero Copy)**：使用 `sendfile` 系统调用，减少内核态与用户态的数据拷贝。
    * **页缓存 (Page Cache)**：利用操作系统缓存，减少磁盘读取。
    * **批量发送 (Batching)**：Producer 累积一批消息一次发送，减少网络开销。
    * **分区并发**：Topic 分多个 Partition，支持多消费者并发处理。

* **ISR、AR、OSR 是什么？**
    * **AR (Assigned Replicas)**：所有副本。
    * **ISR (In-Sync Replicas)**：与 Leader 保持同步的副本集合（包括 Leader）。
    * **OSR (Out-of-Sync Replicas)**：落后太多的副本。
    * **HW (High Watermark)**：高水位，ISR 中所有副本都同步了的 offset，消费者只能看到 HW 之前的消息。
    * **LEO (Log End Offset)**：日志末端位移，下一条消息写入的位置。

### 可靠性与数据一致性
* **Kafka 如何保证消息不丢失？**
    * **Producer 端**：设置 `acks=all`（或 -1），确保 ISR 中所有副本写入成功。重试机制 `retries > 0`。
    * **Broker 端**：
        * `unclean.leader.election.enable=false`（禁止非 ISR 副本选主）。
        * `min.insync.replicas > 1`（ISR 最少副本数，保证至少写入多个节点）。
        * `replication.factor >= 3`（多副本冗余）。
    * **Consumer 端**：关闭自动提交 `enable.auto.commit=false`，业务处理完成后手动提交 offset。

* **ACK 机制（0, 1, -1/all）的区别？**
    * **acks=0**：发送即成功，不等待 Broker 确认。性能最高，但易丢消息。
    * **acks=1**：Leader 写入成功即确认。Leader 挂掉且未同步到 Follower 时会丢消息。
    * **acks=all (-1)**：ISR 中所有副本都写入成功才确认。最安全，但延迟较高。

* **Kafka 如何保证消息幂等性？**
    * **幂等性 Producer**：开启 `enable.idempotence=true`。Producer 被分配 PID，消息带序列号，Broker 自动去重（仅限单分区单会话）。
    * **事务 (Transactional API)**：支持跨分区原子写入，保证 `consume-process-produce` 模式下的 Exactly Once 语义。

### 常见生产问题
* **消息积压怎么办？**
    * **临时扩容**：新建一个 Topic（Partition 是原来的 N 倍），启动转发 Consumer 将积压消息转发到新 Topic，再开启 N 倍的 Worker 消费新 Topic。
    * **优化消费端**：增加 Consumer 数量（不超过 Partition 数），提高批量处理能力，减少耗时 IO。
    * **排查 Bug**：检查 Consumer 是否死锁、频繁 Rebalance 或抛异常导致无法提交 Offset。

* **如何保证消息顺序消费？**
    * **全局顺序**：Topic 设置 1 个 Partition（吞吐量低，不推荐）。
    * **局部顺序（Key 顺序）**：发送时指定 Key，Hash 相同的 Key 会进入同一 Partition。Partition 内消息有序，Consumer 单线程处理或内存队列保序。

* **什么是 Rebalance（重平衡）？如何避免？**
    * **定义**：Consumer Group 成员变化（增减 Consumer）或订阅 Topic 变化时，重新分配 Partition 的过程。会导致 Stop The World（消费暂停）。
    * **触发原因**：
        * Consumer 宕机或心跳超时（`session.timeout.ms`）。
        * 处理太慢导致 Poll 间隔超时（`max.poll.interval.ms`）。
        * 增减 Consumer 或 Topic 分区变化。
    * **避免策略**：
        * 调大 `session.timeout.ms`（避免网络抖动误判）。
        * 调大 `max.poll.interval.ms`（确保业务逻辑能处理完）。
        * 优化业务逻辑，避免单次处理过久。

### 高级特性
* **Kafka 的零拷贝原理？**
    * 传统模式：磁盘 -> 内核 Buffer -> 用户 Buffer -> Socket Buffer -> 网卡。
    * 零拷贝（Sendfile）：磁盘 -> 内核 Buffer -> (描述符) -> Socket Buffer -> 网卡。减少了 2 次 CPU 拷贝和 2 次上下文切换。

* **Kafka 与 RabbitMQ 的区别？**
    * **Kafka**：吞吐量极高，基于 Pull 模式，适合日志收集、流计算、大数据场景。消息持久化保留。
    * **RabbitMQ**：延迟极低，基于 Push 模式，支持复杂路由（Exchange），适合任务队列、即时通讯、复杂业务流程。

### Kafka 进阶与内核面试题

#### 1. Kafka 的存储机制（Log, Segment, Index）是怎样的？
* **Log**：每个 Partition 对应一个文件夹，里面包含多个 Segment 文件。
* **Segment**：为了防止 Log 文件过大，Kafka 将其切分为 Segment。每个 Segment 包含：
    * `.log`：实际存储消息的数据文件。
    * `.index`：位移索引文件（Offset -> Position）。
    * `.timeindex`：时间戳索引文件（Timestamp -> Offset）。
* **查找过程**：
    1. 根据 Offset 二分查找定位到 `.index` 文件。
    2. 在 `.index` 中找到小于等于目标 Offset 的最大物理 Position。
    3. 去 `.log` 文件中从该 Position 开始顺序扫描，找到目标消息。

#### 2. 什么是 Page Cache？Kafka 为什么要重度依赖它？
* **原理**：Linux 内核会将空闲内存用作磁盘缓存（Page Cache）。Kafka 写入消息时，直接写入 Page Cache 即返回（异步刷盘），读取时也优先读 Cache。
* **优势**：
    * **避免 JVM GC**：数据在堆外内存（内核态），不占用 Java 堆内存。
    * **重启不丢热数据**：Broker 重启后，Page Cache 依然存在（只要 OS 不重启）。

#### 3. 详细解释 Kafka 的零拷贝（Zero Copy）在 sendfile 中的体现
* **传统读写**：Disk -> Kernel Buffer -> User Buffer -> Socket Buffer -> NIC Buffer (4 次拷贝，4 次上下文切换)。
* **Kafka sendfile**：Disk -> Kernel Buffer -> (Descriptor) -> NIC Buffer。
    * 数据直接在内核态传输，数据不再经过 User Space（用户态）。
    * 减少了 2 次 CPU 拷贝和 2 次上下文切换，极大提升网卡发送效率。

#### 4. Kafka 如何实现 Exactly Once（精确一次）语义？
* 结合了 **幂等性（Idempotence）** 和 **事务（Transactions）**。
* **幂等性**：`enable.idempotence=true`。Producer 有 PID，消息有 Sequence Number。Broker 拒绝重复 SN 的消息。保证单分区、单会话不丢不重。
* **事务**：`isolation.level=read_committed`。
    * 引入 Transaction Coordinator。
    * 发送消息前开启事务，发送完提交事务。
    * 消费者只能读取到已提交事务（Committed）的消息。
    * 解决了跨分区写入和“消费-处理-生产”流程的原子性。

#### 5. Controller 的作用是什么？它是怎么选举的？
* **作用**：集群的大脑，负责管理所有 Broker 的状态。
    * 监听 ZK/KRaft，处理 Broker 上线/下线。
    * 负责 Partition 的 Leader 选举和 ISR 维护。
    * 将元数据推送到其他 Broker。
* **选举（ZK 模式）**：所有 Broker 启动时尝试在 ZK 创建临时节点 `/controller`，先创建成功的当选。

#### 6. 为什么 Kafka 不支持读写分离？（为什么消费者只读 Leader？）
* **一致性问题**：主从同步有延迟，读 Follower 可能读不到最新数据（除非接受最终一致性）。
* **负载均衡**：Kafka 的 Partition 已经分散在不同 Broker 上，读写 Leader 本身就已经实现了负载均衡。如果读 Follower，反而可能导致代码复杂且收益有限（不像 MySQL 读写分离是为了分担主库压力）。


### Kafka 面试题库大全 (补充精选)

#### 1. 基础与架构
1.  **Kafka 的 Zookeeper 作用？** 存元数据（Topic, Partition, Broker），选 Controller，早期存 Consumer Offset（现存 Broker）。
2.  **KRaft 模式（去 ZK）的好处？** 元数据存 Kafka 内部 Topic，扩展性更强（支持百万分区），运维更简单。
3.  **Broker 是什么？** Kafka 服务器节点，存储消息。
4.  **Topic 和 Partition 的关系？** Topic 是逻辑分类，Partition 是物理存储分片（并发单位）。
5.  **Partition 和 Segment 的关系？** Partition 包含多个 Segment 文件（.log, .index）。
6.  **Replica（副本）的作用？** 高可用。Leader 读写，Follower 同步。
7.  **Offset 是什么？** 消息在 Partition 中的唯一递增编号。
8.  **Consumer Group（消费者组）？** 组内竞争消费（一个 Partition 只能被组内一个 Consumer 消费），组间发布订阅（不同组都收到全量消息）。
9.  **Kafka 默认端口？** 9092。
10. **Kafka 为什么不支持读写分离？** 延迟问题；Leader 已分散负载；代码复杂度。

#### 2. 生产者 (Producer)
11. **Producer 发送流程？** 拦截器 -> 序列化 -> 分区器 -> 累加器 (Buffer) -> Sender 线程 -> Broker。
12. **`acks` 参数详解？**
    *   0：发后即忘（最快，易丢）。
    *   1：Leader 确认（折中）。
    *   all/-1：ISR 所有副本确认（最稳，慢）。
13. **`retries` 参数？** 发送失败重试次数。
14. **发送消息是同步还是异步？** 默认异步（`send()` 返回 Future，回调处理）。同步需调用 `get()`。
15. **如何保证消息顺序？** 单 Partition + 单 Producer 线程（或 `max.in.flight.requests.per.connection=1`）。
16. **消息路由策略（Partition 怎么选）？**
    *   指定 Partition：直接发。
    *   指定 Key：Hash(Key) % 分区数。
    *   无 Key：轮询 (Round-robin) 或 粘性分区 (Sticky Partitioning, 2.4+)。
17. **Buffer 满了怎么办？** `block.on.buffer.full` 阻塞或抛异常。
18. **压缩算法有哪些？** GZIP, Snappy, LZ4, Zstd (推荐)。
19. **`batch.size` 和 `linger.ms`？** 攒够大小或攒够时间发送（吞吐量 vs 延迟权衡）。
20. **自定义拦截器？** 实现 `ProducerInterceptor` 接口（加 Filter、统计）。

#### 3. 消费者 (Consumer)
21. **Consumer 消费模式？** Pull（拉）模式。长轮询（Long Polling）。
22. **为什么用 Pull 不用 Push？** 速率由消费者控制，避免 Broker 压垮消费者。
23. **`__consumer_offsets` Topic？** 存储消费者组的 Offset（默认 50 个分区）。
24. **Offset 提交方式？** 自动提交 (`enable.auto.commit`) vs 手动提交 (`commitSync`/`commitAsync`)。
25. **自动提交的坑？** 业务未处理完就提交（丢消息）或处理完未提交（重复消费）。
26. **重复消费场景？** 提交 Offset 失败；Rebalance 导致重复处理。
27. **消息丢失场景？** 自动提交了但业务报错。
28. **Consumer 线程模型？** 单线程处理（简单）或 拉取后多线程处理（需处理 Offset 提交顺序）。
29. **`fetch.min.bytes`？** 每次 Pull 最小数据量，不够就等。
30. **Lag（积压）监控？** 生产 Offset - 消费 Offset。工具：Kafka-Eagle, Prometheus。

#### 4. 高级特性
31. **ISR, OSR, AR？**
    *   ISR: In-Sync Replicas (同步中)。
    *   OSR: Out-of-Sync Replicas (落后)。
    *   AR: Assigned Replicas (所有)。
32. **HW (High Watermark)？** 消费者可见的最高 Offset（ISR 中最小的 LEO）。
33. **LEO (Log End Offset)？** 日志末端下一条写入位移。
34. **Leader Epoch？** 解决 HW 截断导致的数据丢失/不一致问题。
35. **事务消息 (Transaction)？** 保证多 Partition 写入原子性（`initTransactions`, `beginTransaction`, `commit`）。
36. **幂等性 (Idempotence)？** `enable.idempotence=true`，Broker 根据 PID+SeqNum 去重。
37. **延迟队列实现？** 创建多个 Topic (delay-1m, delay-5m)，死信队列消费者转发。
38. **死信队列 (DLQ)？** 处理失败的消息转发到 DLQ，人工干预。
39. **Kafka 为什么快？** 顺序写盘、Page Cache、零拷贝、批量发送、分区并发。
40. **Kafka 消息最大多大？** 默认 1MB (`message.max.bytes`)，可调大但影响吞吐。

#### 5. 运维与调优
41. **如何增加分区？** `kafka-topics.sh --alter --partitions`。
42. **可以减少分区吗？** 不可以，数据无法回退合并。
43. **Broker 宕机选举过程？** Controller 从 ISR 选第一个作为新 Leader。
44. **Controller 宕机？** ZK 临时节点消失，其他 Broker 竞选。
45. **Rebalance 触发条件？** 组成员变更、订阅 Topic 变更、订阅 Topic 分区数变更。
46. **Rebalance 策略？** Range, RoundRobin, Sticky, CooperativeSticky。
47. **数据保留策略？** 时间 (`log.retention.hours`) 或 大小 (`log.retention.bytes`)。
48. **日志压缩 (Log Compaction)？** 相同 Key 只保留最新 Value（用于状态存储）。
49. **Kafka 硬件选择？** 内存（Page Cache）、磁盘（IOPS）、网卡（带宽）。
50. **JVM 调优？** 堆内存不宜过大（6-8G），留给 Page Cache。G1 收集器。

#### 6. 故障排查
51. **消息发不出去？** 检查网络、ACL、Leader 是否存在。
52. **消费不动？** 检查 Consumer Group 状态 (Stable/Rebalancing)、Lag、业务逻辑阻塞。
53. **磁盘满了？** 缩短保留时间、扩容 Broker、迁移分区。
54. **CPU 飙高？** 压缩解压消耗、频繁 GC、连接数过多。
55. **网络带宽打满？** 限制副本同步限流。
56. **Zookeeper 连接超时？** ZK 压力大，GC 停顿。
57. **Unclean Leader Election？** `unclean.leader.election.enable`，允许非 ISR 选主（丢数据换可用性）。
58. **Broker 脑裂？** Controller Epoch 防止旧 Controller 指挥。
59. **Partition Leader Skew (倾斜)？** `kafka-reassign-partitions.sh` 均衡。
60. **消息乱序？** 只有单 Partition 有序。

#### 7. 实战场景
61. **日志收集？** ELK (Elasticsearch + Logstash + Kafka + Kibana)。
62. **流计算？** Flink/Spark/Storm + Kafka。
63. **削峰填谷？** 比如秒杀场景，请求暂存 Kafka，后端慢慢消费。
64. **用户行为追踪？** 埋点 -> Kafka -> Hadoop。
65. **CDC (Change Data Capture)？** Debezium 监听 Binlog -> Kafka -> Data Warehouse。
66. **Kafka Connect？** 连接器生态，导入导出数据（Source/Sink）。
67. **Kafka Streams？** 轻量级流处理库（Java Lib）。
68. **多机房同步？** MirrorMaker 2.0。
69. **限流？** 生产端限流，消费端 RateLimiter。
70. **exactly-once 在 Flink+Kafka 中怎么做？** Flink Checkpoint + Kafka 事务提交 (Two-Phase Commit)。

#### 8. 对比
71. **Kafka vs RabbitMQ？**
    *   Kafka: 吞吐高，Pull，持久化好，适合大数据/日志。
    *   RabbitMQ: 延迟低，Push，路由灵活 (Exchange)，适合业务消息。
72. **Kafka vs RocketMQ？**
    *   RocketMQ: Java 写，支持延时消息，NameServer 轻量，适合业务/交易。
73. **Kafka vs Pulsar？**
    *   Pulsar: 存算分离 (BookKeeper)，多租户，支持百万 Topic。
74. **Kafka vs ActiveMQ？** ActiveMQ 较老，吞吐较低。

#### 9. 细节考察
75. **Offset 存在哪里？** ZK (0.8前) -> Broker Topic (0.8后)。
76. **如何查找 Offset 对应的消息？** 索引文件 (.index) 二分查找 -> 物理位置 -> 扫描 Log。
77. **时间戳索引？** 0.10+ 支持，.timeindex 文件，按时间回溯消息。
78. **ISR 伸缩？** 副本落后超过 `replica.lag.time.max.ms` 剔除，追上后加入。
79. **Follower Fetch 请求？** Follower 也是 Consumer，向 Leader 拉取数据。
80. **LEO 更新时机？** 写入 Log 后。HW 更新时机？ISR 所有副本 LEO 上报后取最小值。

#### 10. 综合
81. **Kafka 安全？** SSL/TLS 加密，SASL 认证，ACL 鉴权。
82. **Kafka 客户端版本兼容性？** 双向兼容性较好，但建议匹配。
83. **如何清空 Topic 数据？** 调短 retention 时间等待删除，或 delete topic 重建。
84. **消费组 Rebalance 过程？** FindCoordinator -> JoinGroup -> SyncGroup (Leader 分配方案)。
85. **Sticky Partitioning 粘性分区？** 生产者批次发送到同一分区，减少碎片，提升吞吐。
86. **Cooperative Sticky Assignor？** 增量式 Rebalance，不停顿整个消费组。
87. **Rack Awareness (机架感知)？** 副本分配在不同机架，防机房故障。
88. **Log Cleaner 墓碑机制？** 标记删除的 Key。
89. **Kafka 依赖 Java 吗？** Broker 是 Scala/Java，客户端有各种语言。
90. **Kafka 的瓶颈？** 磁盘 IO 和网络 IO。


