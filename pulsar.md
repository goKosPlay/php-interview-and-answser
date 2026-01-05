# Apache Pulsar 面试题库大全 (精选 200 题)

### 1. 基础架构与核心组件 (Architecture)
1.  **Apache Pulsar 是什么？**
    *   下一代云原生分布式消息流平台，集消息、存储、轻量化函数式计算于一体。由 Yahoo 开发并开源。
2.  **Pulsar 的核心架构特点是什么？**
    *   **存算分离**：Broker (计算) 无状态，BookKeeper (存储) 负责持久化。
    *   **分片存储**：基于 Segment (Ledger) 的存储架构，易于扩缩容。
3.  **Pulsar 的主要组件有哪些？**
    *   **Broker**：无状态服务，处理消息生产和消费，管理 Topic，但不存数据。
    *   **BookKeeper (Bookie)**：分布式日志存储系统，负责持久化消息。
    *   **ZooKeeper**：存储元数据 (Metadata) 和集群配置，协调集群。
4.  **Broker 和 Bookie 的关系？**
    *   Broker 接收客户端请求，将消息写入 Bookie。Broker 不落盘数据 (除了缓存)，数据都在 Bookie 中。
5.  **Pulsar 为什么要存算分离？**
    *   **独立扩展**：需要更高吞吐扩 Broker，需要更大容量扩 Bookie。
    *   **快速恢复**：Broker 挂掉无需搬迁数据，Topic 快速漂移到其他 Broker。
6.  **ZooKeeper 在 Pulsar 中的作用？**
    *   存储租户、命名空间、Topic 的配置信息。
    *   协调 Broker 和 Bookie 的服务发现和选主。
7.  **Pulsar 代理 (Proxy) 的作用？**
    *   可选组件。作为集群的网关，转发客户端连接到具体的 Broker。适合在 K8s 或有防火墙限制的场景使用。
8.  **Pulsar 实例 (Instance) 与集群 (Cluster) 的关系？**
    *   一个 Pulsar Instance 可以包含多个 Cluster (用于跨地域复制 Geo-replication)。
9.  **什么是 Multi-tenancy (多租户)？**
    *   Pulsar 天生支持多租户。层级结构：Tenant (租户) -> Namespace (命名空间) -> Topic。
    *   支持资源隔离、权限控制和存储配额。
10. **Namespace (命名空间) 的作用？**
    *   管理 Topic 的逻辑分组单位。很多策略 (Policy) 如 TTL、副本数、Backlog Quota 都是在 Namespace 级别配置的。

### 2. 消息模型与消费模式 (Messaging Model)
11. **Pulsar 支持哪些消息传递模型？**
    *   **Pub/Sub** (发布/订阅) 模型。
    *   **Queue** (队列) 模型 (通过 Shared 订阅实现)。
    *   **Streaming** (流) 模型 (通过 Exclusive/Failover 订阅实现)。
12. **Pulsar 的四种订阅模式 (Subscription Modes)？**
    *   **Exclusive** (独占)：一个 Subscription 只能有一个 Consumer，保证有序。
    *   **Failover** (灾备)：主 Consumer 挂了切备 Consumer，保证有序。
    *   **Shared** (共享)：多个 Consumer 轮询消费，无序，适合队列场景。
    *   **Key_Shared** (Key 共享)：按 Key Hash 分发给不同 Consumer，相同 Key 有序。
13. **什么是 Partitioned Topic (分区主题)？**
    *   一个 Topic 被分为多个 Partition，分布在不同的 Broker 上，提高吞吐量。
14. **非分区 Topic (Non-partitioned Topic)？**
    *   只由一个 Broker 服务，性能受限于单节点。
15. **Message ID 的结构？**
    *   `ledgerId:entryId:partitionIndex:batchIndex`。唯一标识一条消息。
16. **Producer 的发送模式？**
    *   **Sync** (同步)：发完等待 ACK。
    *   **Async** (异步)：发完回调，高吞吐。
17. **Consumer 的接收模式？**
    *   **Sync** (同步 receive)。
    *   **Async** (MessageListener)。
18. **Pulsar 如何保证消息顺序？**
    *   在 Partition 级别，Exclusive 或 Failover 订阅模式下，消息绝对有序。
    *   Key_Shared 模式下，相同 Key 的消息有序。
19. **Pulsar 支持消息去重 (Deduplication) 吗？**
    *   支持。Broker 端根据 ProducerName + SequenceID 自动去重。
20. **Pulsar 支持延迟消息 (Delayed Delivery) 吗？**
    *   支持。`deliverAfter` 或 `deliverAt`。Broker 会将消息暂存在 `DelayedMessageTracker` (基于堆或时间轮)，到期后再推送。

### 3. 存储机制与 BookKeeper (Storage)
21. **什么是 Ledger？**
    *   BookKeeper 的基本存储单元。一个 Append-only 的日志文件段。
22. **什么是 Entry？**
    *   Ledger 中的一条记录，包含实际的数据 (Message)。
23. **什么是 Fragment？**
    *   Ledger 中的物理存储片段，是 Ensembles (副本组) 的最小单位。
24. **BookKeeper 的写一致性策略 (Qw, Qa, E)？**
    *   **E (Ensemble Size)**：选取多少个 Bookie 存储该 Ledger。
    *   **Qw (Write Quorum)**：每次写入并发发给多少个 Bookie。
    *   **Qa (Ack Quorum)**：收到多少个 ACK 才算写入成功。
    *   典型配置：E=3, Qw=3, Qa=2 (3副本，2个写成功即返回)。
25. **Ledger 的生命周期？**
    *   Open -> Write -> Close (变为 Immutable) -> Delete (当所有 Cursor 确认后)。
26. **Ledger Rollover (轮替)？**
    *   当 Ledger 写满一定时间或大小时，关闭当前 Ledger，开启新的 Ledger。这使得 Topic 数据在物理上分散在集群中，易于重新平衡。
27. **Bookie 的 Journal 和 Ledger Storage？**
    *   **Journal** (WAL)：同步写入，保证持久化 (HDD/SSD)。
    *   **Ledger Storage**：异步归档，构建索引 (Index)，方便读取。
28. **Bookie 的 IO 模型？**
    *   读写分离。写入走 Journal (顺序写)；读取走 Ledger Storage/Cache。
29. **Pulsar 的游标 (Cursor) 是什么？**
    *   用于跟踪 Subscription 的消费进度 (Ack 位置)。游标状态也持久化在 BookKeeper 的特定 Ledger 中。
30. **分层存储 (Tiered Storage)？**
    *   将旧的 Ledger (Cold Data) 卸载 (Offload) 到廉价对象存储 (S3, GCS, HDFS)，释放 Bookie 磁盘空间。

### 4. 关键特性 (Features)
31. **Geo-Replication (跨地域复制)？**
    *   原生支持。配置 Cluster 间的复制，Broker 自动在不同集群间异步复制消息。
32. **Pulsar Functions？**
    *   轻量级计算框架 (Serverless)，在 Broker 端运行简单的 ETL 逻辑 (如过滤、转换)，无需额外部署 Spark/Flink。
33. **Pulsar IO (Connectors)？**
    *   内置连接器框架 (Source/Sink)，连接 MySQL, Kafka, Cassandra 等外部系统。
34. **Pulsar SQL (Presto/Trino)？**
    *   使用 Presto 直接查询 Bookie 中的数据，支持 SQL 分析，绕过 Broker。
35. **Schema Registry？**
    *   内置 Schema 管理 (Avro, JSON, Protobuf)，保证 Producer 和 Consumer 数据格式一致，支持兼容性检查。
36. **Topic Compaction (主题压缩)？**
    *   类似 Kafka Log Compaction。保留每个 Key 的最新 Value，用于构建状态表 (Table)。
37. **System Topic？**
    *   Pulsar 内部使用的 Topic，如 `__change_events` 存储配置变更。
38. **Dead Letter Topic (死信队列)？**
    *   消费者重试多次失败后，消息自动进入 DLQ，便于后续人工处理。
39. **Retry Letter Topic (重试队列)？**
    *   消费失败后，消息进入重试队列，延迟一段时间后再次投递。
40. **Transaction (事务) 支持？**
    *   Pulsar 支持跨 Topic 的原子写 (Atomic Write) 和 Ack。支持 `Consume-Process-Produce` 语义的 Exactly-Once。

### 5. 性能与调优 (Performance)
41. **Pulsar 吞吐量高的原因？**
    *   顺序写磁盘 (Journal)。
    *   分层架构，水平扩展能力强。
    *   网络模型 (Netty 异步非阻塞)。
42. **如何优化 Producer 性能？**
    *   开启 `batching` (批量发送)。
    *   使用异步发送 (`sendAsync`)。
    *   调整 `batchingMaxPublishDelay`。
43. **如何优化 Consumer 性能？**
    *   调整 `receiverQueueSize` (预取数量)。
    *   使用 `Listener` (Push 模式) 而非 `receive` (Pull 模式)。
44. **Backlog 是什么？**
    *   积压的消息量 (未被 Ack 的消息)。
45. **Backlog Quota？**
    *   限制 Topic 的积压大小/时间。超过限制可策略：`producer_request_hold`, `producer_exception`, `consumer_backlog_eviction`。
46. **Broker 负载均衡机制？**
    *   Leader Broker 监控所有 Broker 的 CPU/内存/流量，将过载 Broker 的 Bundle (Topic 组) 卸载，转移到空闲 Broker。
47. **Bundle 是什么？**
    *   Topic 分片的集合 (Namespace 下的一部分 Topic)。负载均衡的最小迁移单位。
48. **Managed Ledger？**
    *   Broker 内部对 BookKeeper Ledger 的封装组件，管理 Ledger 的创建、删除和游标移动。
49. **Cache Eviction 策略？**
    *   Broker 和 Bookie 都有缓存。Bookie 使用 LRU 缓存 Entry。
50. **Catch-up Read (追赶读)？**
    *   当 Consumer 落后很多时，直接从 Bookie 磁盘读取旧数据，不影响 Broker 内存中的实时读写 (Tail Read)，实现读写隔离。

### 6. 对比 Kafka 与 RabbitMQ
51. **Pulsar vs Kafka：架构？**
    *   Kafka：存算耦合，Broker 既负责计算也负责存储 (Partition log 在本地磁盘)。
    *   Pulsar：存算分离，无状态 Broker + 分布式 Bookie。
52. **Pulsar vs Kafka：扩缩容？**
    *   Kafka：扩容需迁移 Partition 数据 (Rebalance)，耗时且影响性能。
    *   Pulsar：扩容只需加 Bookie，新 Ledger 自动写到新节点，无需搬迁旧数据，秒级生效。
53. **Pulsar vs Kafka：分区数？**
    *   Kafka：单集群分区数过多会导致 Controller 瓶颈和随机 IO 问题。
    *   Pulsar：支持百万级 Topic，因为元数据存在 ZK/Bookie，且存储分散。
54. **Pulsar vs Kafka：消费模型？**
    *   Kafka：仅支持 Pull，以 Partition 为单位消费 (Consumer Group)。
    *   Pulsar：支持 Queue (Shared) 和 Stream (Exclusive)，更灵活。
55. **Pulsar vs RabbitMQ？**
    *   RabbitMQ：基于 AMQP，Exchange 路由灵活，但堆积能力弱 (内存队列)，吞吐量较低。
    *   Pulsar：兼具 RabbitMQ 的队列特性和 Kafka 的流特性，堆积能力强，吞吐高。
56. **为什么说 Pulsar 是云原生 (Cloud Native) 的？**
    *   存算分离架构契合 K8s (Broker = Deployment, Bookie = StatefulSet)。
    *   支持多租户。
    *   无缝的分层存储 (S3)。
57. **Pulsar 的缺点？**
    *   架构复杂，组件多 (Broker, Bookie, ZK)，运维成本高。
    *   生态相对 Kafka 略小 (但在快速增长)。
58. **Zero Copy (零拷贝)？**
    *   Kafka 依赖 Sendfile 零拷贝。Pulsar 由于存算分离，网络开销相对大，但通过 Netty Zero Copy 优化内存。
59. **Exactly-Once 语义？**
    *   Kafka 和 Pulsar 都支持。Pulsar 2.7+ 引入了事务支持。
60. **消息回溯 (Rewind)？**
    *   Kafka：Reset Offset。
    *   Pulsar：Reset Cursor。都可以回溯消费。

### 7. 运维与监控 (Operations)
61. **Pulsar Manager？**
    *   官方提供的 Web UI 管理工具。
62. **如何监控 Pulsar？**
    *   Pulsar 暴露 Prometheus Metrics。结合 Grafana 监控。
63. **关键监控指标？**
    *   `pulsar_rate_in`/`out` (吞吐)。
    *   `pulsar_storage_size` (存储)。
    *   `pulsar_producers_count` / `consumers_count`。
    *   `bookie_add_entry_latency` (写入延迟)。
64. **Pulsar 升级流程？**
    *   一般顺序：ZooKeeper -> BookKeeper -> Broker -> Proxy。支持滚动升级。
65. **如何清理积压数据？**
    *   `bin/pulsar-admin namespaces unload` (释放资源)。
    *   设置 TTL 自动过期。
    *   手动 `skip` 或 `clear-backlog`。
66. **Bookie Decommission (下线)？**
    *   使用 `bookkeeper-admin` 触发数据恢复 (Recovery)，将该 Bookie 上的 Ledger 副本复制到其他 Bookie。
67. **Auto Recovery (自动恢复)？**
    *   BookKeeper 的 AutoRecovery 进程 (Auditor 和 ReplicationWorker) 会检测副本缺失并自动补齐。
68. **如何查看 Topic 统计信息？**
    *   `bin/pulsar-admin topics stats <topic>`。
69. **Znode 存储了什么？**
    *   `managed-ledgers`：Topic 与 Ledger 的映射列表。
    *   `loadbalance`：负载信息。
70. **Pulsar Perf 工具？**
    *   自带的性能压测工具 `bin/pulsar-perf produce/consume`。

### 8. 客户端与 API (Client)
71. **Pulsar Client 的连接流程？**
    *   Client -> Service URL (DNS/LB) -> Broker -> Lookup Service (查询 Topic 所在 Broker) -> 直连目标 Broker。
72. **Service URL 格式？**
    *   `pulsar://broker:6650` (TCP)
    *   `pulsar+ssl://broker:6651` (TLS)
    *   `http://broker:8080` (WebSocket/Admin)
73. **Client 端的 Reader 接口？**
    *   区别于 Consumer，Reader 可以指定 StartMessageId 开始读取，不管理 Cursor，类似 Kafka 的低级 API。
74. **Batching (批量) 机制？**
    *   Producer 端将多条消息打包成一个 Batch (Entry) 发送，Consumer 端解包。提高吞吐，增加延迟。
75. **Compression (压缩)？**
    *   支持 LZ4, ZLIB, ZSTD, SNAPPY。通常在 Producer 端压缩。
76. **Message TTL (Time To Live)？**
    *   消息未被确认时的保留时间。超时后移至 DLQ 或删除 (取决于配置)。
77. **Message Retention (保留策略)？**
    *   即使消息已被 Ack，依然保留的时间/大小。用于回溯消费。
78. **Client 端的 `receiver_queue_size` 为 0 意味着什么？**
    *   禁用预取 (Prefetch)。Consumer 每次只拉取一条，处理完再拉。适合慢消费场景。
79. **Multi-topic Subscription？**
    *   Consumer 可以使用正则订阅多个 Topic (如 `topic-.*`)。
80. **TableView？**
    *   客户端提供的一种视图，将 Topic 数据当做 KV 表缓存在本地内存中 (类似 Kafka KTable)。

### 9. 安全性 (Security)
81. **Authentication (认证) 方法？**
    *   TLS 证书。
    *   JWT (Token)。
    *   OAuth2。
    *   Kerberos。
    *   Plain (Username/Password)。
82. **Authorization (授权)？**
    *   基于 Role 对 Tenant/Namespace/Topic 进行 `produce`, `consume`, `functions` 等权限控制。
83. **端到端加密 (End-to-End Encryption)？**
    *   Producer 加密，Consumer 解密。Broker 对内容不可见。
84. **TLS 作用？**
    *   传输层加密，防止中间人窃听。
85. **Token 认证机制？**
    *   使用 Secret Key 签发 JWT Token，Client 携带 Token 访问，Broker 验证签名。

### 10. 高级场景 (Advanced)
86. **Effectively-Once (E11) vs Exactly-Once (E11)？**
    *   Pulsar 官方称 Exactly-Once (通过事务)。
87. **WebSocket API？**
    *   Pulsar 提供 WebSocket 接口，方便浏览器或非 SDK 语言直接收发消息。
88. **Pulsar Adapter for Kafka (KoP)？**
    *   在 Pulsar Broker 上加载 KoP 插件，使其支持 Kafka 协议。Kafka 客户端可直接连 Pulsar。
89. **Pulsar Adapter for AMQP (AoP)？**
    *   支持 RabbitMQ 协议。
90. **Pulsar Adapter for MQTT (MoP)？**
    *   支持 MQTT 协议 (IoT 场景)。
91. **Fencing 机制？**
    *   防止脑裂。BookKeeper 保证同一时刻只有一个 Writer 能写 Ledger (通过 LastAddConfirmed 指针和 Fencing bit)。
92. **Striping (条带化)？**
    *   数据根据 Entry ID 轮询写入不同的 Ledger/Fragment，提高并发读写速度。
93. **Rack Awareness (机架感知)？**
    *   BookKeeper 副本放置策略，尽量将副本放在不同机架，防止机架断电导致数据丢失。
94. **Non-Persistent Topic (非持久化 Topic)？**
    *   消息只在内存传输，不落 Bookie。性能极快，但 Broker 挂掉会丢数据。
95. **Compaction 的实现原理？**
    *   Broker 后台扫描 Ledger，构建 Hash Map 保存 Key 的最新位置，生成 Compaction Ledger。
96. **Pulsar SQL 的架构？**
    *   Presto Worker 通过 Split Manager 获取 Segment 信息，并行从 Bookie 读取数据。
97. **Schema Evolution (Schema 演进)？**
    *   支持 FORWARD, BACKWARD, FULL 兼容性检查。
98. **Pulsar Functions 的 Runtime？**
    *   Thread (Broker 进程内), Process (独立进程), Kubernetes (独立 Pod)。
99. **Window Function 在 Pulsar Functions 中？**
    *   支持基于时间或数量的窗口计算 (Tumbling/Sliding Window)。
100. **Pulsar 的未来发展？**
    *   Lakehouse 集成。
    *   更轻量级的存算分离 (K8s Native)。
    *   更强的 Kafka 兼容性。

### 11. 深入 BookKeeper (Deep Dive)
101. **BookKeeper 的 LAC (Last Add Confirmed) 是什么？**
    *   Ledger 中最后一个被确认写入成功的 Entry ID。Readers 只能读取 LAC 之前的数据。
102. **BookKeeper 的 Fencing 过程？**
    *   当 Ledger 发生切换 (Recovery) 时，新 Writer 会修改 Ledger Metadata 状态，旧 Writer 再次尝试写入时会收到 Fenced 异常，防止脑裂。
103. **Bookie 的 `DbLedgerStorage` (RocksDB)？**
    *   新版 Bookie 默认使用的存储引擎，基于 RocksDB 存储索引 (LedgerId/EntryId -> Offset)，数据存在 EntryLog 文件中。
104. **EntryLog 文件？**
    *   Bookie 将来自不同 Ledger 的 Entry 混合顺序写入同一个 EntryLog 文件，提高磁盘写入吞吐。
105. **Index 文件 (索引)？**
    *   用于记录 Ledger 中每个 Entry 在 EntryLog 中的偏移量。读取时先查 Index，再查 EntryLog。
106. **Garbage Collection (GC) 在 Bookie 中如何工作？**
    *   GC 线程定期扫描 EntryLog，如果某个 Ledger 已被删除，且 EntryLog 中大部分数据都是死数据，则重写有效数据到新文件，删除旧 EntryLog (Compact)。
107. **AutoRecovery 的 Auditor？**
    *   监控 Ledger 的健康状态，发现副本缺失 (Under-replicated) 的 Ledger，并发布任务到 ZK。
108. **AutoRecovery 的 ReplicationWorker？**
    *   从 ZK 领取修复任务，将数据从现有副本复制到新 Bookie。
109. **Bookie 的 Speculative Read (推测读)？**
    *   如果读取某个副本超时，Bookie Client 会并发读取其他副本，谁先返回用谁。
110. **Write Quorum (Qw) 和 Ack Quorum (Qa) 的权衡？**
    *   Qa 越小，写入延迟越低 (不用等慢节点)，但数据安全性稍低 (未确认副本可能丢失)。
111. **Ensemble Change？**
    *   当某个 Bookie 写入失败时，Client 会自动替换一个新的 Bookie 加入 Ensemble，保证后续写入的高可用。
112. **BookKeeper 是否支持更新 (Update) 操作？**
    *   不支持。Ledger 是 Append-only 的。
113. **BookKeeper 的数据一致性模型？**
    *   强一致性。写入成功后，所有 Reader 都能读到。
114. **Ledger Metadata 存储在哪里？**
    *   ZooKeeper (或 Etcd)。包含 Ensemble 列表、状态、LAC 等。
115. **Bookie 的 Force Sync？**
    *   写入 Journal 后强制刷盘 (fsync)，保证断电不丢数据。可配置 `journalSyncData=false` 牺牲安全性换性能。

### 12. 深入 Broker 机制
116. **Broker 的 Lookup Service？**
    *   负责处理客户端的 `Connect` 和 `Lookup` 请求，根据 Bundle 映射关系，重定向 Client 到 Owner Broker。
117. **Load Manager (负载管理器)？**
    *   Broker 内部组件，负责收集负载信息，写入 ZK (LoadReport)，并决策 Bundle 的拆分与迁移。
118. **Modular Load Manager？**
    *   新版负载管理器，更灵活，支持插件化策略 (LeastLongTermMessageRate 等)。
119. **Broker 的 Cache？**
    *   Broker 也会维护 ManagedLedger 的 Cache (EntryCache)，减少对 Bookie 的读取。
120. **Broker 重启对 Client 的影响？**
    *   Client 连接断开，自动重连。Lookup Service 指向新 Owner。短暂延迟，无数据丢失。
121. **Subscription 的状态存储？**
    *   存储在 Cursor Ledger 中。Broker 定期更新 Cursor 位置到 BookKeeper。
122. **Dispatcher (分发器)？**
    *   Broker 内部组件，负责从 Managed Ledger 读取消息并分发给 Connected Consumers。
123. **Redelivery (重投递) 机制？**
    *   Consumer 没 Ack 或调用 `negativeAcknowledge`，Dispatcher 会将消息加入 Redelivery Tracker，稍后重发。
124. **Broker 端的限流 (Throttling)？**
    *   可以配置 Publisher/Consumer 的速率限制 (QPS/ByteRate)。
125. **Topic Ownership？**
    *   每个 Bundle (包含多个 Topic) 在同一时刻只能由一个 Broker 拥有 (Owner)。ZK 中利用临时节点锁实现。

### 13. Pulsar Functions 进阶
126. **Functions 的 Delivery Semantics？**
    *   At-most-once, At-least-once, Effectively-once (通过处理状态去重)。
127. **State Store (状态存储)？**
    *   Functions 可以读写状态 (KV)，状态存储在 BookKeeper (Table Service) 中。
128. **Function Mesh？**
    *   基于 K8s CRD 的 Serverless 编排框架，声明式管理 Pulsar Functions。
129. **Functions 的日志？**
    *   可以输出到 stdout (K8s logs) 或指定的 Log Topic。
130. **Functions 支持哪些语言？**
    *   Java, Python, Go。

### 14. 消息协议与客户端进阶
131. **Protobuf 在 Pulsar 中的应用？**
    *   Pulsar 的通信协议 (Pulsar Api) 基于 Google Protobuf 定义。
132. **Consumer 的 `AckGroupingTracker`？**
    *   客户端为了性能，不会每条消息发 Ack，而是分组/批量发送 Ack (Cumulative Ack 或 Individual Ack Batch)。
133. **Consumer 的 `UnackedMessageTracker`？**
    *   客户端追踪已接收但未 Ack 的消息。超时后触发 Redelivery。
134. **Producer 的 `BatchingBuilder`？**
    *   `DefaultBatchingBuilder` (基于大小/时间) vs `KeyBasedBatchingBuilder` (按 Key 打包)。
135. **Client 端的重试机制？**
    *   连接断开会自动重试 (Exponential Backoff)。
136. **Reader 如何实现 `tail -f` 效果？**
    *   Reader 读取到最新消息后，会阻塞等待新消息 (`hasMessageAvailable` + `readNext`)。
137. **Pulsar Client 的线程模型？**
    *   基于 Netty 的 EventLoop 线程池处理 IO，回调在专用线程或外部 Executor 中执行。

### 15. 集群管理与多机房
138. **Global Configuration Store (Global ZK)？**
    *   跨集群复制时，需要一个全局 ZK 存储集群间的配置信息。
139. **Replication Set？**
    *   指定消息需要复制到哪些集群。
140. **Async Replication vs Sync Replication？**
    *   Pulsar 默认 Geo-replication 是异步的。同步复制需要特殊配置 (跨机房写 Bookie)。
141. **Replication Cursor？**
    *   用于记录复制进度的特殊 Cursor。
142. **集群间网络不通怎么办？**
    *   消息会在本地集群堆积，直到网络恢复后继续复制。
143. **Active-Active vs Active-Passive？**
    *   Pulsar 原生支持双活 (Active-Active)，两个集群同时读写，双向复制。
144. **Replication 的 Loop 问题？**
    *   Pulsar 会在消息元数据中记录经过的集群 ID，防止环路复制。

### 16. 高级调优 (Tuning)
145. **JVM 调优建议？**
    *   Broker 和 Bookie 主要使用 Direct Memory (Netty/RocksDB)，Heap 此时不需要设太大。注意 GC 停顿。
146. **Bookie 磁盘配置？**
    *   Journal 必须用 SSD (低延迟顺序写)。Ledger 可以用 HDD (大容量)。
147. **Netty 线程数配置？**
    *   `numIOThreads`。通常设置为 CPU 核数。
148. **ManagedLedger 的 Cache 配置？**
    *   `managedLedgerCacheSizeMB`。Broker 缓存大小，越大读取越快 (Cache Hit)，但也越耗内存。
149. **Bookie 的 Page Cache？**
    *   利用 OS Page Cache 加速读取。
150. **调整 Ensemble Size 的影响？**
    *   E 越大，数据越分散，读取吞吐越高 (Striping)，但元数据越复杂。

### 17. 故障排查与恢复
151. **ZooKeeper 丢失数据？**
    *   灾难性故障。需重置集群元数据，Bookie 数据可能变为孤儿 (Orphan)，需扫描恢复。
152. **Bookie 磁盘满？**
    *   Bookie 会变为 Read-only 模式。需扩容或清理旧 Ledger。
153. **Broker OOM？**
    *   检查 `managedLedgerCache` 是否过大，或积压消息过多导致内存元数据膨胀。
154. **消息无法消费 (Stuck)？**
    *   检查 Cursor 状态 (`stats-internal`)。可能是 MarkDeletePosition 未推进。
    *   尝试 Unload Topic 或 Reset Cursor。
155. **Bookie 启动失败 (Cookie Mismatch)？**
    *   通常是 IP 变了或数据目录被清空但 ZK 还有旧元数据。需手动清理 ZK 中的 Cookie。

### 18. 生态系统集成
156. **Pulsar + Flink？**
    *   使用 Pulsar Flink Connector。支持 Source/Sink，支持 Exactly-Once (利用 Pulsar 事务)。
157. **Pulsar + Spark？**
    *   Pulsar Spark Connector。支持 Structured Streaming。
158. **Pulsar + SkyWalking？**
    *   使用 SkyWalking 监控 Pulsar 链路追踪。
159. **StreamNative 是什么？**
    *   Pulsar 创始团队成立的公司，提供 Pulsar 商业化服务和云平台。
160. **Kafka-on-Pulsar (KoP) 的原理？**
    *   协议转换器。解析 Kafka 请求，映射到 Pulsar Topic/Ledger。支持 Kafka 0.10+ 协议。

### 19. 场景与最佳实践
161. **日志收集场景？**
    *   高吞吐，允许少量延迟。使用 Async Send, Batching, Non-persistent topic (可选)。
162. **金融交易场景？**
    *   强一致，低延迟。Sync Send, E=3/Qw=3/Qa=2, 开启 Fsync。使用事务。
163. **IoT 场景？**
    *   百万级 Topic，海量连接。Pulsar 优势场景。使用 MoP 或 MQTT Proxy。
164. **大文件传输？**
    *   Chunking (分块) 机制。Producer 将大消息切分，Consumer 组装。
165. **延时队列场景？**
    *   使用 `deliverAfter`。注意大量延时消息会占用 Bookie 存储和 Broker 内存索引。
166. **如何实现优先级队列？**
    *   Pulsar 原生不支持。通常创建多个 Topic (High/Mid/Low)，Consumer 优先拉取 High Topic。

### 20. 综合知识
167. **Pulsar 端口号？**
    *   Broker: 6650 (Binary), 8080 (HTTP)。Bookie: 3181。
168. **Pulsar Admin CLI？**
    *   `pulsar-admin`：管理租户、Topic、Function 等核心工具。
169. **Pulsar Perf？**
    *   基准测试工具，测试集群极限吞吐和延迟。
170. **Managed Ledger 的 Offload 触发条件？**
    *   手动触发，或配置大小/时间阈值。
171. **Compaction 后的数据存在哪？**
    *   存在特殊的 Compacted Ledger 中。
172. **Pulsar 是否支持 XA 事务？**
    *   不支持标准 XA，但有自己的 TC (Transaction Coordinator) 实现跨 Topic 事务。
173. **如何查看 Bookie 的 Ledger 分布？**
    *   `bookkeeper-admin bookies list-ledgers`。
174. **Bundle Split 策略？**
    *   当 Bundle 流量过大时，Broker 会将其 Split 为两个子 Bundle。
175. **Schema Validation？**
    *   Broker 在 Producer 连接或发送时校验 Schema 兼容性。
176. **Proxy 模式下的 Client IP？**
    *   Proxy 会通过协议扩展透传真实 Client IP 给 Broker。
177. **Pulsar 能完全替代 Kafka 吗？**
    *   功能上覆盖且更强，但运维复杂度和成熟度 (早期) 是门槛。KoP 降低了迁移成本。
178. **Tiered Storage 支持哪些后端？**
    *   AWS S3, GCS, Azure Blob, HDFS, Aliyun OSS。
179. **Pulsar 社区版本发布周期？**
    *   通常 3-6 个月一个大版本 (2.x -> 3.x)。
180. **Pulsar 的命名来源？**
    *   Pulsar (脉冲星)，象征高频、精准的消息流。

### 21. 补充细节
181. **Batch Index Acknowledgement？**
    *   Pulsar 支持对 Batch 中的单条消息进行 Ack (需开启配置)，避免整个 Batch 重发。
182. **Key_Shared 的 Hash 模式？**
    *   `AutoSplit` (自动均分) vs `Sticky` (固定范围)。
183. **Exclusive Producer？**
    *   防止多个 Producer 写入同一 Topic。
184. **Topic 级别的 Policy？**
    *   可以覆盖 Namespace 级别的配置 (如 Retention, TTL)。
185. **Broker 的 Health Check？**
    *   `/admin/v2/brokers/health` 接口。
186. **Bookie 的 Read-only Mode？**
    *   磁盘满或显式设置时进入，只读不写。
187. **Pulsar Function 的 `log-topic`？**
    *   函数日志会自动发送到该 Topic，方便调试。
188. **State Storage 的 API？**
    *   `context.putState(key, value)` / `context.getState(key)`。
189. **KoP 的 Group Coordinator？**
    *   KoP 实现了 Kafka 的 Group Coordinator 逻辑，管理 Consumer Group Offset (存在 __consumer_offsets Topic)。
190. **Pulsar 的 3 层 API？**
    *   Client API (Pub/Sub), Admin API (管理), Function API (计算)。
191. **Entry Log 预分配 (Pre-allocation)？**
    *   Bookie 预先分配磁盘空间，减少文件系统碎片。
192. **Journal 的 Group Commit？**
    *   批量提交 Journal 写入，减少 fsync 次数。
193. **Broker 端的 Deduplication 快照？**
    *   去重状态定期做快照，防止重启后丢失去重信息。
194. **Namespace Isolation Policy？**
    *   指定某些 Namespace 只能在特定的 Broker 组上加载 (物理隔离)。
195. **Bookie Affinity Group？**
    *   指定 Ledger 优先写在某些 Bookie 上。
196. **Pulsar IO 的 Debezium Source？**
    *   捕获 DB (MySQL/PG) 变更，发送到 Pulsar。
197. **Shadow Topic？**
    *   复制生产流量到影子 Topic 进行测试。
198. **Pulsar 的 `advertisedAddress`？**
    *   Broker 注册到 ZK 供 Client 连接的 IP/Host。NAT 环境下需正确配置。
199. **Pulsar 3.0 的 LTS (长期支持)？**
    *   引入了新的版本发布模型，LTS 版本支持周期更长。
200. **如何开始贡献 Pulsar？**
    *   阅读 PIP (Pulsar Improvement Proposal)，参与 GitHub Issue 讨论。

