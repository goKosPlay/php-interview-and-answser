# TiDB 面试题库大全 (精选 100 题)

### 1. 基础架构与原理 (Architecture)
1.  **TiDB 是什么？**
    *   开源分布式关系型数据库，支持 HTAP (Hybrid Transactional/Analytical Processing)，兼容 MySQL 协议。
2.  **TiDB 的核心组件有哪些？**
    *   **TiDB Server**：SQL 解析、优化、计算，无状态。
    *   **TiKV**：分布式 Key-Value 存储，行存储，负责数据持久化和事务。
    *   **PD (Placement Driver)**：元数据管理、集群调度 (负载均衡)、TSO (全局时间戳) 分配。
    *   **TiFlash**：列式存储，支持实时分析 (AP)。
3.  **TiDB 此时兼容 MySQL 哪个版本？**
    *   主要是 MySQL 5.7 和 8.0 的大部分语法和协议。
4.  **TiDB 与 MySQL 的主要区别？**
    *   TiDB 是分布式的，存储和计算分离；MySQL (单机) 是紧耦合的。TiDB 天然支持高可用和水平扩展。
5.  **TiDB 的存储引擎是什么？**
    *   底层使用 RocksDB (LSM-Tree) 作为单机存储引擎，TiKV 封装了 RocksDB。
6.  **TiKV 如何保证数据一致性？**
    *   使用 Multi-Raft 协议，将数据分片为 Region，每个 Region 有多个副本 (Replica)，通过 Raft 保证一致性。
7.  **PD 的作用是什么？**
    *   **TSO 分配**：提供全局唯一递增时间戳，用于事务模型。
    *   **元数据存储**：存储 Region 的位置信息。
    *   **调度**：根据负载自动迁移 Region，实现负载均衡。
8.  **什么是 Region？**
    *   TiKV 数据划分的基本单位，一段连续 Key 的范围 (默认 96MB)。
9.  **TiDB 的计算存储分离架构有什么优势？**
    *   计算节点 (TiDB) 和存储节点 (TiKV) 可以独立扩缩容，互不影响。
10. **TiFlash 是如何同步数据的？**
    *   作为 Raft Group 的 Learner 角色，异步从 TiKV Leader 复制日志，不参与 Raft 投票，不阻塞写入。

### 2. 存储与 Raft 协议
11. **TiKV 的 Key-Value 映射规则？**
    *   TableID + RowID -> Key (用于存储行数据)。
    *   TableID + IndexID + IndexColumns -> Key (用于存储索引)。
12. **RocksDB 的 LSM-Tree 读写流程？**
    *   写：写入 MemTable (内存) -> Immutable MemTable -> Flush 到 SSTable (磁盘) -> Compaction。
    *   读：MemTable -> Immutable MemTable -> Block Cache -> SST files (Level 0 -> Level N)。
13. **Raft 协议在 TiKV 中的角色？**
    *   保证多副本之间的数据复制和一致性。Leader 负责读写，Follower 负责同步。
14. **Multi-Raft 是什么？**
    *   TiKV 将数据切分为成千上万个 Region，每个 Region 独立运行一个 Raft Group。
15. **Leader Lease (Leader 租约) 是什么？**
    *   Raft Leader 在租约期内，即使发生网络隔离，也能保证自己是唯一的 Leader，允许直接提供读服务 (Lease Read)。
16. **Raft 的 Follower Read？**
    *   允许 Follower 节点提供读服务，前提是 Follower 需要向 Leader 询问最新的 Commit Index (Read Index)，保证线性一致性。
17. **什么是 TiKV 的 MVCC (多版本并发控制)？**
    *   Key 中包含 Version (TSO)，存储多个版本的数据。格式：`Key_Version -> Value`。
18. **TiKV 如何处理 GC (垃圾回收)？**
    *   TiDB 后台定期运行 GC，清理过期版本 (小于 Safe Point) 的数据，释放空间。
19. **Compaction 是什么？**
    *   RocksDB 后台操作，合并 SST 文件，清理被删除或覆盖的数据，减少读放大和空间占用。
20. **Region Split 和 Merge？**
    *   Split: Region 过大 (96MB) 时分裂。
    *   Merge: Region 过小 (20MB) 且 Key 相邻时合并。

### 3. 事务 (Transaction)
21. **TiDB 使用什么事务模型？**
    *   Percolator 模型 (Google)，基于两阶段提交 (2PC) 的分布式事务。
22. **TiDB 事务的隔离级别？**
    *   默认 **Snapshot Isolation (SI)** (快照隔离)，兼容 MySQL 的 Repeatable Read (RR)。也支持 Read Committed (RC)。
23. **TiDB 的 2PC 流程？**
    *   **Prewrite**：锁住所有涉及的 Key，写入数据 (Primary Lock + Secondary Locks)。
    *   **Commit**：先 Commit Primary Key (获得 Commit TS)，成功后异步 Commit Secondary Keys。
24. **什么是 TSO？**
    *   Timestamp Oracle，由 PD 分配的全局唯一递增时间戳 (物理时间 + 逻辑序号)。
25. **TiDB 事务的大小限制？**
    *   单行最大 6MB。
    *   总 KV Entry 大小限制 (默认 100MB，可配置)。
    *   事务时间限制 (默认 TTL)。
26. **TiDB 的锁机制？**
    *   乐观锁 (Optimistic)：提交时检测冲突。
    *   悲观锁 (Pessimistic，默认)：执行 SQL 时加锁 (Select for update)，行为更像 MySQL。
27. **乐观锁与悲观锁的区别？**
    *   乐观锁高并发冲突时重试代价大；悲观锁适合冲突多的场景，性能略低但稳定。
28. **什么是 Write Conflict (写冲突)？**
    *   两个事务同时修改同一个 Key，乐观锁下后提交的会失败重试。
29. **Async Commit (异步提交)？**
    *   优化 2PC，Prewrite 完成后，只要计算出 Commit TS 满足条件，即可返回成功，无需等待 Commit 阶段写入磁盘。
30. **1PC (一阶段提交)？**
    *   如果事务只涉及一个 Region，Prewrite 后直接提交，减少 RPC 开销。

### 4. SQL 与 优化 (SQL Tuning)
31. **TiDB 执行计划 (Explain) 怎么看？**
    *   关注 `TableReader`, `IndexReader`, `IndexLookUp` (回表)。
    *   `estRows` (预估行数) vs `actRows` (实际行数)。
32. **统计信息 (Statistics) 如何收集？**
    *   `ANALYZE TABLE` 手动收集；TiDB 也会自动收集。
33. **SQL 优化常见手段？**
    *   添加索引、强制索引 (`USE INDEX`)、SQL Binding (执行计划绑定)、调整系统变量。
34. **什么是 IndexLookUp Join？**
    *   先扫索引，再回表查数据，最后做 Join。
35. **Hash Join vs Merge Join？**
    *   Hash Join：适合大表 Join，并行度高。
    *   Merge Join：适合有序数据，内存占用少。
36. **TiDB 支持存储过程和触发器吗？**
    *   不支持触发器。存储过程支持有限 (不推荐使用)。
37. **TiDB 支持外键吗？**
    *   早期版本不支持，新版本 (v6.6+) 开始实验性支持，但不建议在分布式系统强依赖外键。
38. **AUTO_INCREMENT 的坑？**
    *   TiDB 的自增 ID 在单个 TiDB Server 内单调递增，但全局不保证连续和单调递增 (分段分配)。
39. **如何解决热点问题 (Hotspot)？**
    *   `SHARD_ROW_ID_BITS`：打散隐式 RowID。
    *   `AUTO_RANDOM`：替代自增主键，生成随机主键。
    *   分区表 (Partition Table)。
40. **TopSQL 是什么？**
    *   TiDB Dashboard 功能，用于定位消耗资源最多的 SQL。

### 5. 运维与生态 (Ops & Ecosystem)
41. **TiUP 是什么？**
    *   TiDB 的包管理器和集群管理工具 (部署、扩缩容、升级)。
42. **如何备份 TiDB？**
    *   **BR (Backup & Restore)**：分布式备份工具，直接从 TiKV 备份 SST 文件，速度快。
    *   **Dumpling**：逻辑备份，导出 SQL/CSV。
43. **如何恢复 TiDB？**
    *   BR Restore (物理恢复)。
    *   TiDB Lightning (逻辑导入)。
44. **TiDB Lightning 的两种模式？**
    *   **Local Backend**：生成 SST 文件直接 Ingest 到 TiKV，速度极快，影响在线业务。
    *   **TiDB Backend**：执行 INSERT SQL，速度较慢，支持在线。
45. **TiCDC 是什么？**
    *   Change Data Capture，订阅 TiKV 的变更日志，同步给下游 (MySQL, Kafka, TiDB)。
46. **DM (Data Migration) 的作用？**
    *   将上游 MySQL/MariaDB 的数据全量+增量迁移到 TiDB。
47. **Sync-diff-inspector？**
    *   数据校验工具，对比 MySQL 和 TiDB 数据一致性。
48. **TiDB 升级流程？**
    *   使用 TiUP 滚动升级，先 PD，再 TiKV，最后 TiDB。
49. **如何扩容 TiKV？**
    *   `tiup cluster scale-out`，新节点加入后 PD 会自动均衡数据。
50. **缩容 TiKV 需要注意什么？**
    *   `tiup cluster scale-in`，节点下线会将 Region 搬迁到其他节点，需等待搬迁完成 (Tombstone)。

### 6. 高可用与容灾 (HA & DR)
51. **TiDB 如何实现高可用？**
    *   PD、TiKV、TiDB 均可多节点部署。TiKV 数据多副本 (通常 3 副本)。
52. **某台 TiKV 挂了会怎样？**
    *   Leader 在该节点：触发 Raft 选举，选出新 Leader。
    *   Follower 在该节点：短时间无影响，长时间 PD 会补副本。
53. **PD 挂了会怎样？**
    *   PD 是 Cluster (通常 3 节点)，挂一个不影响。挂多数 (2个) 整个集群不可用 (无法分配 TSO)。
54. **TiDB Server 挂了会怎样？**
    *   Load Balancer (LVS/HAProxy) 切断流量，应用重连其他 TiDB 节点。
55. **两地三中心部署？**
    *   同城双中心 (同步) + 异地灾备 (异步/同步)。
56. **同城双中心自适应同步 (Dr-Auto-Sync)？**
    *   平时同步复制，主中心挂了切从中心，降级为异步复制 (或保持同步)。
57. **Raft 副本 3 副本能容忍几台挂掉？**
    *   容忍 1 台 (多数派存活：3 - 1 = 2)。
58. **5 副本能容忍几台？**
    *   容忍 2 台 (5 - 2 = 3)。
59. **Labels 标签调度的作用？**
    *   告诉 PD 节点的物理位置 (Zone/Rack/Host)，让 PD 把副本打散到不同机房/机架，实现容灾。
60. **TiFlash 挂了影响主库吗？**
    *   不影响，TiFlash 是 Learner，不参与 Raft 投票和提交。

### 7. 故障排查 (Troubleshooting)
61. **如何定位慢 SQL？**
    *   TiDB Dashboard -> 慢查询日志。
    *   Grafana -> TiDB -> Query Summary。
    *   `INFORMATION_SCHEMA.SLOW_QUERY` 表。
62. **TiKV 写入慢的原因？**
    *   Raftstore 线程繁忙 (CPU 瓶颈)。
    *   磁盘 IO 延迟高 (WAL 写入慢)。
    *   热点写入 (Hotspot)。
    *   RocksDB Compaction 压力大。
63. **TiDB OOM (Out of Memory) 原因？**
    *   SQL 涉及大量数据 Join/Sort 落盘不及时。
    *   `tidb_mem_quota_query` 设置过大。
64. **PD 无法选举 Leader？**
    *   网络分区。
    *   磁盘空间满。
    *   时钟不同步。
65. **Copr (Coprocessor) 也是什么？**
    *   TiDB 下推给 TiKV 的计算任务 (如过滤、聚合)。Grafana 中 `Coprocessor CPU` 高说明下推计算多。
66. **Write Stall 是什么？**
    *   RocksDB 写入过快，Compaction 来不及，触发限流或停写保护。
67. **如何查看 Region 健康状态？**
    *   `pd-ctl region check` (miss-peer, extra-peer, down-peer)。
68. **Grafana 监控面板最重要的几个？**
    *   **Overview**：集群整体 QPS, Latency, 资源使用。
    *   **TiDB**：Duration, Connection, Heap Memory。
    *   **TiKV-Details**：Raft IO, Coprocessor, RocksDB-KV。
    *   **PD**：Cluster Status, TSO, Region health。
69. **Pessimistic Lock Wait Timeout？**
    *   悲观锁等待超时，通常是因为死锁或事务持有锁时间过长。
    *   检查 `tidb_txn_mode` 和应用逻辑。
    *   查看 `data_lock_waits` 表。
70. **TiCDC 同步延迟高？**
    *   上游写入 QPS 过高。
    *   网络带宽不足。
    *   下游数据库写入瓶颈。

### 8. 进阶概念 (Advanced)
71. **聚簇索引 (Clustered Index)？**
    *   v5.0+ 默认开启。主键即 Key，行数据存储在 Value 中。减少一次回表，读写效率更高。
72. **非聚簇索引？**
    *   TiDB 早期默认。主键是唯一的二级索引，RowID 是 Key。
73. **Stale Read (陈旧读)？**
    *   允许读取指定时间点 (TSO) 的历史数据。`SELECT * FROM t AS OF TIMESTAMP ...`。
    *   可用于读写分离，降低 Leader 压力。
74. **Point Get？**
    *   基于主键或唯一索引的精确查找，直接走 KV 接口，不走复杂的执行计划，速度最快。
75. **Batched Coprocessor？**
    *   TiFlash 的特性，向量化执行计算。
76. **MPP (Massively Parallel Processing) 模式？**
    *   TiDB v5.0+ 引入 TiFlash MPP，计算在 TiFlash 节点间交换数据，无需汇聚到 TiDB，加速复杂分析查询。
77. **TTL (Time To Live) 表？**
    *   v6.5+ 支持，自动删除过期数据。
78. **Resource Control (资源管控)？**
    *   v7.0+ 引入，基于 Request Unit (RU) 对不同用户/任务进行资源限流 (IO/CPU)。
79. **Global Sort？**
    *   分布式排序，用于 `DISTINCT`, `ORDER BY`，数据量大时落盘。
80. **TiDB 能够替代 Redis 吗？**
    *   部分场景可以 (Point Get 性能很高)，但 TiDB 重在强一致和持久化，延迟略高于纯内存 Redis。

### 9. 场景与选型 (Scenario)
81. **什么场景适合用 TiDB？**
    *   数据量大 (TB/PB 级)。
    *   单机 MySQL 无法支撑 (分库分表痛点)。
    *   需要强一致性、高可用。
    *   HTAP 混合负载 (既有交易又有报表)。
82. **什么场景不适合用 TiDB？**
    *   数据量很小 (几 GB)。
    *   极低延迟要求 (微秒级，如高频交易)。
    *   操作系统资源受限。
83. **迁移 MySQL 到 TiDB 需要改代码吗？**
    *   大部分不需要，但需注意自增 ID 不连续、不支持外键 (旧版)、大事务限制等差异。
84. **TiDB 分页查询慢怎么优化？**
    *   `LIMIT 1000000, 10` 会很慢。
    *   优化：`WHERE id > last_id LIMIT 10` (Keyset Pagination)。
85. **如何处理大批量删除？**
    *   不要 `DELETE FROM t` (大事务)。
    *   使用 `DELETE ... LIMIT 1000` 循环删除。
    *   或者 `GC` 设置短时间，直接 `DROP partition/table`。
86. **TiDB 如何实现读写分离？**
    *   使用 Follower Read。
    *   配置 TiDB Server 为 `readonly`。
    *   使用 Stale Read。
87. **TiDB 云原生 (Kubernetes) 部署？**
    *   TiDB Operator，自动化管理 K8s 上的 TiDB 集群。
88. **TiFlash 与 ClickHouse 的区别？**
    *   TiFlash 强一致实时更新，与 TiDB 融合；ClickHouse 侧重离线分析，更新能力弱。
89. **TiDB 在金融场景的应用？**
    *   核心交易系统、账务系统 (两地三中心、强一致、ACID)。
90. **TiKV 的 Block Cache 默认大小？**
    *   默认占用系统内存的 45%。

### 10. 综合测试 (Mixed)
91. **pd-ctl 的常用命令？**
    *   `member`, `store`, `region`, `scheduler`。
92. **TiKV 节点下线流程？**
    *   PD 标记为 Offline -> 迁移 Region -> 迁移完成 -> Tombstone。
93. **什么是 Region Overlap？**
    *   Region 范围重叠，通常是元数据异常，需要通过 `pd-ctl` 修复。
94. **TiDB 支持全文索引吗？**
    *   不支持 (类似 MySQL 的 Fulltext)。建议对接 Elasticsearch。
95. **TiDB 支持 Spatial (GIS) 索引吗？**
    *   不支持。
96. **CTE (Common Table Expressions) 支持吗？**
    *   支持 (v5.0+)，`WITH RECURSIVE` 递归查询。
97. **Window Function (窗口函数) 支持吗？**
    *   支持 (MySQL 8.0 语法)。
98. **TiDB 的配置热加载？**
    *   部分配置支持在线修改 (`set global ...`) 或 `tiup reload` (不重启)。
99. **TiDB 社区版和企业版的区别？**
    *   企业版提供更多工具 (审计、加密、官方支持) 和企业级特性。
100. **未来 TiDB 的发展方向？**
    *   Serverless (TiDB Cloud)。
    *   更强的 HTAP (TiFlash 增强)。
    *   AI 结合 (Chat2Query)。

### 11. 高级 SQL 特性 (Advanced SQL)
101. **TiDB 支持 JSON 类型吗？**
    *   支持，与 MySQL 5.7 兼容。支持 `JSON_EXTRACT`, `JSON_OBJECT` 等函数，且支持对 JSON 生成虚拟列建立索引。
102. **TiDB 支持全文检索 (Fulltext Search) 吗？**
    *   目前不支持原生 MySQL 语法的 FULLTEXT 索引。建议使用 TiFlash 或集成 Elasticsearch。
103. **TiDB 支持 Sequence (序列) 吗？**
    *   支持 (`CREATE SEQUENCE`)，用于生成全局唯一的递增数字，替代自增 ID。
104. **TiDB 支持临时表 (Temporary Table) 吗？**
    *   支持本地临时表 (Local Temporary Table) 和全局临时表 (Global Temporary Table)。
105. **TiDB 支持视图 (View) 吗？**
    *   支持普通视图，不支持物化视图 (Materialized View)。
106. **如何模拟物化视图？**
    *   使用 TiFlash (列存副本) 加速聚合查询，或者使用 TiCDC 同步数据到下游汇总表。
107. **TiDB 对字符集 (Charset) 和排序规则 (Collation) 的支持？**
    *   默认 `utf8mb4`。v6.0+ 支持 `GBK`。支持 `utf8mb4_general_ci` (大小写不敏感) 等排序规则 (需要在集群初始化时开启 `new_collations_enabled_on_first_bootstrap`)。
108. **`INSERT IGNORE` 和 `REPLACE INTO` 的区别？**
    *   `INSERT IGNORE`: 主键冲突时忽略当前写入，保留原数据。
    *   `REPLACE INTO`: 主键冲突时删除原数据，写入新数据 (会触发 Delete + Insert)。
109. **`ON DUPLICATE KEY UPDATE` 的行为？**
    *   主键冲突时，执行 Update 操作。
110. **TiDB 的 `LOAD DATA` 命令？**
    *   支持从本地文件或 S3 导入数据，但性能不如 Lightning。
111. **TiDB 是否支持存储过程？**
    *   支持，但建议尽量少用，逻辑放在应用层。
112. **TiDB 是否支持 Trigger (触发器)？**
    *   不支持。需在应用层实现或通过 TiCDC 捕获变更。
113. **TiDB 是否支持 Event Scheduler (事件调度器)？**
    *   不支持。建议使用 Linux Crontab 或应用层定时任务。
114. **什么是 SQL Binding (SPM)？**
    *   SQL Plan Management，用于固定特定 SQL 的执行计划，防止统计信息更新导致选错索引。
115. **如何创建 SQL Binding？**
    *   `CREATE GLOBAL BINDING FOR ... USING ...`。
116. **什么是 Invisible Index (不可见索引)？**
    *   将索引设为不可见，优化器不会使用它。用于测试删除索引对性能的影响。
117. **什么是 Expression Index (表达式索引)？**
    *   基于表达式创建索引，例如 `CREATE INDEX idx ON t ((col1 + col2))`。
118. **TiDB 对 `FOREIGN KEY` 的约束检查？**
    *   v6.6.0 之前只支持语法解析但不检查约束。v6.6.0+ 开始支持外键约束。
119. **`SELECT FOR UPDATE` 在 TiDB 中的行为？**
    *   悲观事务模式下，会加悲观锁，阻塞其他事务修改。乐观事务模式下，仅在提交时检查冲突。
120. **TiDB 的 `AUTO_RANDOM` 使用限制？**
    *   必须是 `BIGINT` 类型，必须是主键 (`PRIMARY KEY`)。

### 12. 深入 RocksDB 与 存储引擎
121. **RocksDB 的 Write Stall 触发条件？**
    *   MemTable 数量达到上限。
    *   L0 层 SST 文件数量过多。
    *   待 Compaction 的字节数过多。
122. **RocksDB 的 Bloom Filter 作用？**
    *   快速判断一个 Key 是否**不**在某个 SST 文件中，减少磁盘读取。
123. **TiKV 使用了几个 RocksDB 实例？**
    *   两个：`raftdb` (存储 Raft Log) 和 `kvdb` (存储实际数据)。
124. **kvdb 中的 CF (Column Family) 有哪些？**
    *   `default`: 存储大 Value (数据部分)。
    *   `write`: 存储 MVCC 版本信息 (Commit TS, 指向 default CF)。
    *   `lock`: 存储悲观锁和 Prewrite 锁信息。
125. **为什么要把 Raft Log 单独存 (raftdb)？**
    *   Raft Log 是顺序写入，数据是随机写入，隔离以减少相互影响。
126. **LSM-Tree 相比 B+Tree 的优势？**
    *   写性能更高 (Append Only)，适合高吞吐写入场景。
127. **LSM-Tree 的读放大 (Read Amplification)？**
    *   读取一个 Key 可能需要查找 MemTable 和多层 SST 文件。
128. **LSM-Tree 的写放大 (Write Amplification)？**
    *   Compaction 过程会多次重写数据。
129. **LSM-Tree 的空间放大 (Space Amplification)？**
    *   存储多版本数据和已删除数据的标记，直到 Compaction 才清理。
130. **Block Cache 的置换算法？**
    *   LRU (Least Recently Used)。
131. **Titan 是什么？**
    *   RocksDB 的插件，用于 KV 分离存储。将大 Value 存到 Blob 文件，LSM-Tree 只存 Key 和 Blob 指针，减少写放大。
132. **如何调整 RocksDB 压缩算法？**
    *   通常 L0-L2 不压缩 (NoCompression)，L3+ 使用 LZ4 或 Zstd。
133. **TiKV 的 `raftstore` 线程池作用？**
    *   处理 Raft 消息 (Propose, Append, Apply)。
134. **TiKV 的 `apply` 线程池作用？**
    *   将 Raft Log 应用到状态机 (写入 kvdb)。
135. **TiKV 的 `scheduler` 线程池作用？**
    *   负责事务的 latch (内存锁) 控制和读取快照。

### 13. Raft 深度解析
136. **Raft 的 Log Replication 流程？**
    *   Leader 接收写请求 -> Append Log (本地) -> 广播 AppendEntries -> Follower 接收并 Append -> 回复 Leader -> Leader 收到多数派响应 -> Commit -> Apply -> 回复 Client。
137. **Raft PreVote 机制？**
    *   预投票。Candidate 在发起正式投票前先询问其他节点，防止网络分区的节点不断发起选举干扰集群。
138. **Raft Joint Consensus (联合共识)？**
    *   成员变更算法。TiDB 目前主要使用单步成员变更 (每次只变一个节点)。
139. **Raft Snapshot (快照) 的作用？**
    *   当 Log 过大时，将当前状态生成快照，截断旧日志，释放空间，并用于落后过多的 Follower 快速同步。
140. **Learner 角色有什么用？**
    *   只同步日志，不投票。用于 TiFlash 和新加节点的数据同步 (Add Peer)。
141. **Leader Transfer (领导权转移)？**
    *   命令 `transfer leader`。Leader 主动将 Leadership 移交给指定 Peer，用于负载均衡或停机维护。
142. **Raft Group 的 Hibernate (休眠) 特性？**
    *   如果 Region 长时间无读写，Raft Group 进入静默状态，减少心跳开销 (Raft tick)。
143. **Split Region 的原子性？**
    *   Split 也是一条 Raft Log，应用该 Log 时原子地修改元数据。
144. **Merge Region 的条件？**
    *   两个 Region 必须相邻，且都在同一个 Store 上 (通常由 PD 调度先搬迁到一起)。

### 14. 性能调优实战 (Performance Tuning)
145. **如何处理索引热点？**
    *   如果索引是递增的 (如时间)，会造成写热点。可以使用 `SHARD_ROW_ID_BITS` 或者建表时打散数据。
146. **如何识别写瓶颈在 Raftstore 还是 RocksDB？**
    *   看 `Propose Wait Duration` 高 -> Raftstore 忙。
    *   看 `RocksDB Append Duration` 高 -> RocksDB 写慢。
147. **Tidb 侧的 `CopCache` (Coprocessor Cache)？**
    *   缓存 Coprocessor 的计算结果。适合 TPC-H 等分析型查询。
148. **`tidb_distsql_scan_concurrency`？**
    *   控制向 TiKV 发送 Coprocessor 请求的并发度。
149. **`tidb_index_lookup_concurrency`？**
    *   控制 IndexLookUp 操作中回表的并发度。
    *   过高会占用大量内存，过低会增加延迟。
150. **如何优化 `COUNT(*)` 性能？**
    *   TiDB 会下推 `COUNT` 到 TiKV/TiFlash。TiFlash 列存通常更快。
151. **Batch Insert 的最佳实践？**
    *   单批次大小建议 100-1000 行，总大小 < 10MB。
    *   显式开启事务。
152. **TiKV 内存占用过高怎么办？**
    *   检查 `block-cache-size` 设置。
    *   检查 `write-buffer-size` (MemTable)。
    *   检查 Coprocessor 是否在处理大查询。
153. **PD 调度慢怎么办？**
    *   调整 `leader-schedule-limit` 和 `region-schedule-limit`。
154. **网络延迟对 TiDB 的影响？**
    *   Raft 协议对延迟敏感。跨机房延迟高会直接增加写入 Latency。

### 15. TiDB 生态工具进阶
155. **TiDB Binlog 组件 (Pump & Drainer)？**
    *   旧版同步工具。Pump 生成 binlog，Drainer 消费并同步到下游。v7.0+ 已被 TiCDC 取代。
156. **TiSpark 是什么？**
    *   基于 Spark 的 TiDB 连接器。直接从 TiKV 读取数据，绕过 TiDB Server，适合大规模离线计算。
157. **TiDB Operator 的作用？**
    *   在 K8s 上自动运维 TiDB。支持自动故障转移、滚动更新。
158. **Chaos Mesh？**
    *   混沌工程平台，用于在 TiDB 集群中注入故障 (网络延迟、Pod 杀掉)，测试系统稳定性。
159. **TiDB Dashboard 的 Key Visualizer (热力图)？**
    *   可视化查看集群的流量分布。明亮的对角线通常代表顺序写热点。
160. **TiDB Cloud？**
    *   PingCAP 提供的全托管 DBaaS 服务。
161. **OssInsight？**
    *   基于 TiDB Cloud 的 GitHub 数据分析洞察平台。

### 16. 安全 (Security)
162. **TiDB 支持 TLS/SSL 加密吗？**
    *   支持客户端与 TiDB、组件内部通信 (TiDB-PD-TiKV) 的加密。
163. **TDE (Transparent Data Encryption)？**
    *   静态数据加密。TiKV 支持落盘加密，保护磁盘数据安全。
164. **TiDB 的 RBAC (基于角色的访问控制)？**
    *   兼容 MySQL 8.0 的 Role 机制。
165. **TiDB 审计日志 (Audit Log)？**
    *   记录所有用户操作。可配置记录特定用户的行为。
166. **如何防止 SQL 注入？**
    *   TiDB 使用预处理语句 (Prepare Statement) 参数化查询。
167. **TiDB 的密码策略？**
    *   支持 `validate_password` 组件，强制密码复杂度。

### 17. 故障恢复案例
168. **误删表 (`DROP TABLE`) 如何恢复？**
    *   如果 GC Safe Point 还没过，可以使用 `FLASHBACK TABLE` (v6.4+)。
    *   或者使用 `RECOVER TABLE` (基于 GC)。
169. **误执行 `DELETE` / `UPDATE` 如何恢复？**
    *   使用 `FLASHBACK CLUSTER TO TIMESTAMP` (需 v6.4+)，将整个集群回滚到指定时间点。
    *   或者利用 TiCDC/Binlog 回放逆向操作。
170. **TiKV 数据损坏 (Corruption) 怎么办？**
    *   使用 `tikv-ctl` 工具检查和修复 SST 文件。
    *   极端情况下标记该 Store 为 Tombstone，让 PD 补副本。
171. **集群 ID (Cluster ID) 不匹配？**
    *   通常发生在错误地将旧数据节点加入新集群。需清理数据目录重新启动。
172. **`region is unavailable` 错误？**
    *   多数副本挂掉，Region 失去 Leader。需检查 TiKV 状态。

### 18. 版本特性 (Version Features)
173. **TiDB v4.0 的重大特性？**
    *   TiUP, TiFlash, 悲观事务 GA, 大事务支持 (10GB)。
174. **TiDB v5.0 的重大特性？**
    *   MPP 模式, 聚簇索引默认开启, Async Commit, 1PC。
175. **TiDB v6.0 的重大特性？**
    *   Placement Rules in SQL (SQL 控制数据放置), TiFlash 数据落 S3, 内存悲观锁。
176. **TiDB v7.0 的重大特性？**
    *   Resource Control (资源管控), TTL 表, Reorg DDL 并行。
177. **TiDB Serverless 的技术？**
    *   存储计算分离, 自动扩缩容, 按量计费, 共享存储池。

### 19. 对比其他数据库
178. **TiDB vs CockroachDB (CRDB)？**
    *   都基于 Raft。CRDB 兼容 PostgreSQL 协议，TiDB 兼容 MySQL 协议。CRDB 是 range-based + geo-partitioning 强项。
179. **TiDB vs HBase？**
    *   HBase 是 NoSQL，无事务 (仅单行)，API 访问。TiDB 是 NewSQL，支持 SQL 和 ACID 事务。
180. **TiDB vs ElasticSearch (ES)？**
    *   ES 擅长全文检索和非结构化日志分析。TiDB 擅长结构化数据和事务。
181. **TiDB vs MySQL 分库分表 (ShardingSphere/MyCat)？**
    *   TiDB 对业务透明，无需处理分片键、聚合查询、分布式事务等复杂问题。
    *   分库分表方案运维复杂度随规模指数上升。
182. **TiDB vs Oracle？**
    *   Oracle 单机性能极强，功能最全。TiDB 胜在水平扩展和开源成本。

### 20. 杂项与补充
183. **TiDB 的 `information_schema`？**
    *   存储元数据。`TIKV_REGION_STATUS`, `SLOW_QUERY`, `CLUSTER_INFO` 等特有表很有用。
184. **如何查看当前事务的 ID (StartTS)？**
    *   `SELECT tidb_current_tso()`.
185. **TiDB 支持 UDF (用户自定义函数) 吗？**
    *   不支持。
186. **TiDB 的系统变量作用域？**
    *   `SESSION` (当前连接) 和 `GLOBAL` (全局，新连接生效)。
187. **如何强制走 TiFlash 查询？**
    *   `set @@session.tidb_isolation_read_engines = "tiflash";`
    *   或者 SQL Hint: `/*+ READ_FROM_STORAGE(TIFLASH[t]) */`。
188. **`oom-use-tmp-storage`？**
    *   当单条 SQL 内存超限时，溢写到磁盘临时目录，防止 OOM。
189. **TiDB 的 `kill` 命令？**
    *   `KILL TIDB [ConnectionID]`：杀掉连接。
    *   `KILL QUERY`：取消正在执行的语句。
190. **DDL 是异步的吗？**
    *   是。TiDB 采用在线 DDL (Online DDL) 算法 (Google F1)，不锁表。
191. **Add Index 慢怎么加速？**
    *   调整 `tidb_ddl_reorg_worker_cnt` 和 `tidb_ddl_reorg_batch_size`。
192. **大表 `TRUNCATE` 速度？**
    *   极快，只需逻辑更新元数据和清理 Region 范围。
193. **PD 调度策略中的 `hot-region-scheduler`？**
    *   自动检测读写热点 Region，并将其 Leader 或 Peer 调度到负载低的 Store。
194. **TiDB 默认端口？**
    *   TiDB: 4000, TiKV: 20160, PD: 2379。
195. **如何从 mydumper 迁移到 Dumpling？**
    *   Dumpling 是 mydumper 的 Go 重写版，针对 TiDB 优化，参数基本兼容。
196. **TiDB 中的 `Null` 处理？**
    *   与 MySQL 一致。索引中 Null 值不重复。
197. **Vector Search (向量搜索)？**
    *   TiDB 正在探索集成向量搜索能力，以支持 AI 业务 (Beta 阶段)。
198. **TiKV API (RawKV)？**
    *   不经过 TiDB，直接访问 TiKV 的 KV 接口。性能更高，无 SQL 和事务开销。
199. **TiDB 社区活跃度？**
    *   CNCF 毕业项目，拥有庞大的开源社区和 Contributor。
200. **学习 TiDB 的最佳路径？**
    *   官方文档 -> TiDB 101 课程 -> 部署测试集群 -> 阅读源码 -> 参与社区。
