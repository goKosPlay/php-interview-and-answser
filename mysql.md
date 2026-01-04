### mysql varchar最大字符串多少
* 在 MySQL 5.0.3 及以上版本，VARCHAR 最大长度是 65,535 字节。
* 在更老的版本（如 5.0.3 之前），最大长度是 255 字符。

### mysql 如何优化sql
* 优化查询语句 选择需要的列：避免使用 SELECT *，明确指定所需列，减少数据传输量。 避免在 WHERE 中使用函数或计算（例如 WHERE YEAR(date_column) = 2025），改用范围查询（WHERE date_column BETWEEN '2025-01-01' AND '2025-12-31'）。 使用 = 而不是 LIKE（当不需要模糊匹配时）。
* 使用高效的 WHERE 条件: 尽量用 JOIN 替代子查询，性能通常更好。
* 减少子查询：尽量用 JOIN 替代子查询，性能通常更好。
* 限制返回行数：使用 LIMIT 限制结果集大小，避免查询过多数据。
* 避免不必要的排序：只在需要时使用 ORDER BY，并确保排序字段有索引。
* 使用覆盖索引：设计查询使所有所需字段都在索引中，减少回表操作。
* 索引优化 创建合适的索引 在 WHERE、JOIN、GROUP BY、ORDER BY 常用的列上添加索引。 使用复合索引（多列索引）优化复杂查询，但注意索引顺序（高选择性字段放前）。 避免冗余索引（例如，(a) 和 (a, b) 同时存在）。
* 使用 EXPLAIN 分析查询 执行 EXPLAIN SELECT ... 查看查询计划，检查是否使用索引、扫描行数等。 关注 type（如 ALL 表示全表扫描，需优化）、rows（扫描行数）和 key（使用的索引）。
* 避免索引失效： 不要在索引列上使用函数或运算（例如 WHERE UPPER(column) = 'VALUE'）。 避免 OR 条件过多，改用 IN 或 UNION。对 LIKE 查询，尽量避免前置通配符（如 LIKE '%abc'）。
* 表结构优化 选择合适的数据类型： 使用最小的数据类型（如用 INT 而不是 BIGINT，VARCHAR(50) 而不是 TEXT）。 日期用 DATE 或 DATETIME，避免用字符串存储。
* 分区表：对大数据量表（如按时间或范围）进行分区，减少查询扫描范围。
* 规范化与反规范化： 规范化减少数据冗余，但查询复杂时可适当反规范化（如冗余字段）以减少 JOIN。
* 避免过多列：列数过多会增加存储和查询开销。
* 数据库配置优化
* 调整缓存： 增大 innodb_buffer_pool_size（InnoDB 缓存数据和索引），建议占服务器内存的 60-80%。
* 调整 query_cache_size（MySQL 8.0 之前）或使用其他缓存机制。
* 优化连接池：调整 max_connections 和 wait_timeout，避免连接过多或空闲连接占用资源。
* 启用慢查询日志： 设置 slow_query_log = 1 和 long_query_time（如 1 秒），记录慢查询。 使用 mysqldumpslow 或 pt-query-digest 分析慢查询日志，针对性优化。
* 其他技巧 批量操作：批量插入或更新（如 INSERT INTO ... VALUES (), (), ()）比单条操作快。 
* 避免锁竞争： 使用事务时尽量缩短事务范围，减少锁等待。 对高并发表，考虑使用 InnoDB（行级锁）而非 MyISAM（表级锁）。
* 缓存结果：对频繁查询的静态数据，使用 Redis 或 Memcached 缓存，减少数据库压力。
* 分库分表：数据量极大时，考虑水平分表（如按用户 ID）或分库。
* 监控与测试 性能监控：使用工具如 MySQL Workbench、Percona Toolkit 或 Zabbix 监控查询性能。
* 压力测试：用 sysbench 或 mysqlslap 测试优化后的查询性能。

### mysql有四种隔离级别
* READ UNCOMMITTED（读未提交）: 该隔离级别的事务会读到其它未提交事务的数据，此现象也称之为脏读。
* READ COMMITTED（读已提交）: 一个事务可以读取另一个已提交的事务，多次读取会造成不一样的结果，此现象称为不可重复读问题，Oracle 和 SQL Server 的默认隔离级别。
* REPEATABLE READ（可重复读）: 该隔离级别是 MySQL 默认的隔离级别，在同一个事务里，select 的结果是事务开始时时间点的状态，因此，同样的 select 操作读到的结果会是一致的，但是，会有幻读现象
* SERIALIZABLE（串行化）: 在该隔离级别下事务都是串行顺序执行的，MySQL 数据库的 InnoDB 引擎会给读操作隐式加一把读共享锁，从而避免了脏读、不可重读复读和幻读问题。


### mysql为什么喜欢用 B+tree
```
为什么 MySQL 选择 B+树
MySQL（特别是 InnoDB 引擎）选择 B+树作为索引结构的理由如下：
(1) 高效的磁盘 I/O

数据库的特点：数据库数据通常存储在磁盘上，I/O 操作是性能瓶颈。B+树的节点可以存储大量键值（高扇出），减少树的高度，从而减少磁盘 I/O 次数。
对比 B 树：B 树的非叶子节点也存储数据，导致每个节点存储的键值较少，树高度较高，I/O 次数更多。B+树将数据集中到叶子节点，优化了磁盘读取。
示例：假设一个节点可以存储 100 个键值，B+树高度为 3 时可以支持约 100^3 = 1000 万条记录，而只需 3 次磁盘 I/O。

(2) 支持范围查询

叶子节点链表：B+树的叶子节点通过双向链表连接，范围查询（如 WHERE id BETWEEN 10 AND 20）只需定位到起始节点，然后顺序遍历链表，效率极高。
对比其他结构：红黑树或普通二叉树没有链表结构，范围查询需要多次遍历，效率较低。

(3) 高效的点查询

平衡性：B+树的平衡性质确保点查询（如 WHERE id = 100）的时间复杂度稳定，为 O(log N)，适合主键查找。
高扇出：节点存储大量键值，减少树的高度，加快查找速度。

(4) 插入和删除效率

平衡调整：B+树在插入和删除时通过节点分裂或合并保持平衡，操作复杂度为 O(log N)。虽然有一定开销，但对数据库的增删改操作友好。
批量操作：B+树支持批量插入和删除，适合数据库事务场景。

(5) 适合大块数据存储

节点大小：B+树的节点大小通常与磁盘块大小对齐（例如 4KB 或 16KB），充分利用磁盘预读机制，减少 I/O 开销。
数据局部性：叶子节点的链表结构提高了数据局部性，范围查询时可以顺序读取磁盘块。

3. 对比其他数据结构

二叉搜索树（BST）：可能退化为链表，查询效率不稳定（O(N) 最差情况）。
红黑树：虽然也是平衡树，但每个节点只有两个子节点，树高度较高，磁盘 I/O 次数多，适合内存操作而非磁盘。
B 树：非叶子节点存储数据，导致扇出较低，树高度较高，且范围查询效率不如 B+树（无链表结构）。
哈希索引：适合精确查找（O(1)），但不支持范围查询和排序，适用场景有限（MySQL 的 Memory 引擎支持哈希索引）。
```

### mysql 有哪些索引
```
主键索引 (PRIMARY KEY)

每个表只能有一个主键索引，用于唯一标识表中的每一行记录。
自动具有唯一性约束，不能包含 NULL 值。
通常在创建表时定义，例如 PRIMARY KEY (column_name)。
底层通常使用 B+树实现。


唯一索引 (UNIQUE INDEX)

确保索引列中的值是唯一的，允许 NULL 值（但 NULL 值本身可以重复，视 MySQL 版本而定）。
用于强制数据唯一性，例如邮箱地址或身份证号。
示例：CREATE UNIQUE INDEX idx_name ON table_name (column_name);


普通索引 (INDEX/KEY)

最基本的索引类型，用于加速查询，无唯一性约束。
允许重复值和 NULL 值。
示例：CREATE INDEX idx_name ON table_name (column_name);


全文索引 (FULLTEXT INDEX)

专门用于文本搜索，适用于 CHAR、VARCHAR 或 TEXT 类型的列。
常用于全文搜索场景，例如模糊查询（MATCH ... AGAINST 语法）。
仅在 MyISAM 和 InnoDB（MySQL 5.6 及以上版本）存储引擎中支持。
示例：CREATE FULLTEXT INDEX idx_fulltext ON table_name (column_name);


空间索引 (SPATIAL INDEX)

用于空间数据类型（如 GEOMETRY、POINT、LINESTRING 等），仅在 MyISAM 和部分 InnoDB 版本中支持。
常用于地理信息系统（GIS）相关查询。
示例：CREATE SPATIAL INDEX idx_spatial ON table_name (column_name);


组合索引 (Composite/Multi-column Index)

基于多个列创建的索引，适合多列条件查询。
遵循“最左前缀原则”，即查询条件需包含索引的最左列才能有效利用。
示例：CREATE INDEX idx_composite ON table_name (column1, column2);


前缀索引 (Prefix Index)

针对较长的字符串列（如 VARCHAR 或 TEXT），只对列的前 N 个字符创建索引，节省存储空间。
示例：CREATE INDEX idx_prefix ON table_name (column_name(10));


覆盖索引 (Covering Index)

一种逻辑概念而非具体索引类型，指索引包含查询所需的所有列数据，查询无需访问表数据。
常用于优化 SELECT 查询性能。
例如：查询仅涉及索引列，MySQL 可直接从索引中获取数据。

```

### InnoDB 和 MyISAM 的区别
* InnoDB 支持事务、行级锁、外键，崩溃恢复能力强（redo/undo），适合高并发写。
* MyISAM 不支持事务，表级锁，读取速度快但写入并发差，崩溃后易损坏（无事务日志保障）。
* InnoDB 聚簇索引：数据与主键索引组织在一起；MyISAM 索引与数据分离。

### 什么是聚簇索引（clustered index）？
* InnoDB 的主键索引是聚簇索引：叶子节点存放整行数据。
* 优点：按主键查询/范围查询很快。
* 代价：二级索引叶子节点存主键值，回表需要再走一次主键索引。
* 主键建议：尽量短、单调递增（减少页分裂/移动），避免用长字符串做主键。

### 什么是回表？如何减少回表？
* 回表：二级索引命中后，拿到主键再去聚簇索引取其它列。
* 减少回表：
* 使用覆盖索引（查询字段都在同一个索引里）。
* 避免 SELECT *，只查需要的列。
* 合理设计联合索引，让常用查询成为覆盖索引。

### 联合索引的最左前缀原则是什么？
* 对索引 (a,b,c)，能利用索引的典型条件：
* a
* a,b
* a,b,c
* a + 范围(between/</>) 会导致 b 后面的列（如 c）通常无法继续用于定位（仍可能用于排序/回表优化，取决于优化器）。
* 常见坑：WHERE b=1 AND c=2 不能用到 (a,b,c) 的索引（缺少最左列 a）。

### 什么情况下索引会失效？
* 在索引列上做函数/表达式：WHERE DATE(create_time)=...（可改为范围查询）。
* 隐式类型转换：字符串列用数字比较、或反之。
* LIKE 以 % 开头：LIKE '%abc'。
* 低选择性字段单列索引价值有限（如性别），需要结合业务与联合索引。
* OR 两侧若不能都走索引，可能退化为全表扫描（可用 UNION ALL 改写）。

### explain 主要看哪些字段？
* type：访问类型（从好到差常见：const > ref > range > index > ALL）。
* key：实际使用的索引。
* rows：预估扫描行数。
* Extra：
* Using index：覆盖索引。
* Using where：回表后再过滤或在存储引擎层过滤。
* Using filesort：需要额外排序（不一定在磁盘，但意味着无法利用索引顺序）。
* Using temporary：需要临时表（GROUP BY/ORDER BY 组合不当常见）。

### MySQL 事务的 ACID 分别是什么？
* A 原子性：要么全成功要么全失败（undo log 支撑回滚）。
* C 一致性：事务前后满足约束/规则（由应用 + 约束 + 事务共同保证）。
* I 隔离性：并发事务互不干扰（锁 + MVCC）。
* D 持久性：提交后不丢（redo log + 刷盘策略）。

### InnoDB 的 redo log / undo log / binlog 有什么区别？
* redo log：物理日志，用于崩溃恢复与持久性（InnoDB 特有）。
* undo log：逻辑日志，用于回滚与 MVCC（InnoDB 特有）。
* binlog：逻辑日志，MySQL Server 层，用于主从复制与点时间恢复（PITR）。
* 两阶段提交：为保证 redo log 与 binlog 一致性，提交时会做 2PC。

### MySQL 默认隔离级别为什么是 RR？RR 如何解决幻读？
* MySQL/InnoDB 默认 RR（可重复读）。
* 普通 SELECT 依靠 MVCC（一致性读）避免不可重复读；
* 对“当前读”（SELECT ... FOR UPDATE / UPDATE / DELETE），通过 next-key lock（记录锁 + 间隙锁）在 RR 下避免幻读。
* 说明：如果是普通 SELECT，并不会加锁；看到的是快照读。

### 什么是 MVCC？
* 多版本并发控制：同一行可能存在多个版本。
* InnoDB 通过隐藏字段（事务 id、回滚指针等）+ undo log 生成历史版本。
* 读操作通常不加锁（快照读），提升并发。
* 可见性由 Read View 决定（哪些事务对当前读可见）。

### MySQL/ InnoDB 常见锁有哪些？
* 记录锁（Record Lock）：锁住索引记录。
* 间隙锁（Gap Lock）：锁住索引记录之间的间隙，防止插入。
* 临键锁（Next-Key Lock）：记录锁 + 间隙锁（RR 下当前读常用）。
* 意向锁：表级标记，配合行锁提升判断效率。
* 共享锁/排它锁：S/X。

### 什么是死锁？如何排查与减少？
* 死锁：事务间互相等待对方持有的锁。
* InnoDB 会检测死锁并回滚其中一个事务。
* 排查：SHOW ENGINE INNODB STATUS 查看 LATEST DETECTED DEADLOCK。
* 减少：
* 统一访问顺序（按相同索引顺序更新）。
* 缩短事务时间，尽量少做交互/网络 IO。
* 合理索引，让锁更精准（避免锁范围扩大）。

### 为什么大分页（LIMIT offset, size）慢？如何优化？
* offset 很大时需要扫描/丢弃大量行。
* 优化方式：
* 基于“上一页最后一条记录”的游标翻页（seek method）：WHERE id > last_id ORDER BY id LIMIT size。
* 先用覆盖索引定位主键，再回表取数据：
```
SELECT * FROM t
WHERE id IN (
  SELECT id FROM t
  WHERE cond=... ORDER BY id LIMIT 100000, 20
)
ORDER BY id;
```

### 为什么不建议在高并发场景使用 SELECT *？
* 增加网络传输与磁盘/缓存命中压力。
* 可能导致无法使用覆盖索引，产生更多回表。
* 字段变更会影响更多代码与缓存键。

### count(*) / count(1) / count(列) 有什么区别？
* count(*)：统计行数，包含 NULL。
* count(1)：对每行取常量 1，效果与 count(*) 类似（优化器通常等价处理）。
* count(列)：统计该列非 NULL 的行数。
* InnoDB 没有像 MyISAM 那样的“行数元数据”，大表 count 需要扫描（可用近似、汇总表或业务计数）。

### 什么是覆盖索引？它为什么快？
* 覆盖索引：查询所需列都在索引里，无需回表。
* 快的原因：减少随机 IO、减少页读取、减少锁/访问开销。

### order by 为什么会慢？Using filesort 一定是坏事吗？
* ORDER BY 若不能利用索引顺序，需要额外排序（EXPLAIN Extra: Using filesort）。
* filesort 不一定是磁盘排序，但说明无法“按索引顺序直接得到结果”。
* 优化：
* 让排序字段走索引（联合索引顺序与 ORDER BY 一致）。
* 减少返回行数（LIMIT）。
* 避免在排序前产生巨大中间集。

### group by 的常见优化点
* GROUP BY 与 ORDER BY 尽量使用同一索引顺序，减少临时表与排序。
* 只选需要的列，避免额外回表。
* 大聚合考虑预聚合（汇总表）、按时间窗口分批。

### 慢查询如何定位与优化？
* 开启 slow_query_log，关注 long_query_time。
* 用 EXPLAIN 分析执行计划，先保证索引命中与扫描行数合理。
* 优化顺序建议：
* 先改 SQL（减少扫描、减少回表、避免子查询/重复计算）。
* 再补/调索引（联合索引、覆盖索引）。
* 再看表结构与数据分布（字段类型、冗余、分区/分表）。
* 最后调参数与架构（缓存、读写分离）。

### 主从复制原理是什么？为什么会延迟？
* 基本链路：主库写 binlog -> 从库 IO 线程拉取 relay log -> SQL 线程回放。
* 延迟原因：
* 从库单线程回放（旧版本/配置），或回放压力大。
* 大事务/长事务导致回放积压。
* 主从硬件差异、网络抖动。
* 热点表/DDL 影响。

### binlog 有哪些格式？各自优缺点？
* STATEMENT：记录 SQL，日志小；但可能因非确定性函数导致不一致。
* ROW：记录行变更，最安全；日志大。
* MIXED：混合，默认更均衡。

### 什么时候需要分库分表？要注意什么？
* 单表数据量过大、热点写入/查询成为瓶颈、单机资源到顶时。
* 注意点：
* 分片键选择（尽量均匀、可路由）。
* 跨分片事务与一致性方案（尽量避免跨分片强事务）。
* 全局唯一 ID（雪花/号段）。
* 跨分片查询、排序、分页的成本。

### 常见 SQL 优化面试题：为什么“索引建了但没用上”？
* 可能原因：
* 条件不满足最左前缀。
* 选择性太差，优化器认为全表扫描更快。
* 隐式转换/函数导致无法走索引。
* 统计信息不准（可 ANALYZE TABLE，或等待自动更新）。
* 使用了不等号/范围后，联合索引后续列无法继续用于定位。

### 如何设计唯一约束？用唯一索引还是业务校验？
* 数据一致性必须靠数据库约束兜底：唯一索引能从根源避免并发下重复写。
* 应用层校验只能提升体验，不能替代唯一约束（并发会穿透）。
* 处理冲突：INSERT IGNORE / INSERT ... ON DUPLICATE KEY UPDATE（视业务需求）。

### MySQL 内核与进阶面试题

#### 1. Buffer Pool 的 LRU 算法是如何设计的？（解决全表扫描污染问题）
* **传统 LRU 问题**：如果全表扫描（如备份或无索引查询），所有热数据会被新数据挤出，导致缓存命中率骤降。
* **MySQL 改进（冷热分离）**：
    * 将 LRU 链表分为 **New Sublist（热区，约 63%）** 和 **Old Sublist（冷区，约 37%）**。
    * 新读入的页默认加入 **Old 区头部**。
    * 只有在 Old 区停留超过一定时间（`innodb_old_blocks_time`，默认 1s）后被再次访问，才会被移入 New 区。
    * **效果**：大表扫描产生的临时数据只会在 Old 区流转并快速被淘汰，保护了 New 区的热点数据。

#### 2. 什么是 Change Buffer（写缓冲）？有什么用？
* **场景**：针对 **非唯一普通索引** 的插入/更新。
* **原理**：
    * 如果目标数据页在 Buffer Pool 中，直接更新。
    * 如果**不在**，且不影响唯一性，则不需立即从磁盘读入该页，而是将修改记录在 Change Buffer 中。
    * 等待该页被读取（Merge）或后台线程定期 Merge，减少随机磁盘读 I/O。
* **限制**：仅适用于非唯一索引（唯一索引必须读入页来校验唯一性）。
* **适用**：写多读少、索引多的业务。

#### 3. Double Write Buffer（双写缓冲）解决了什么问题？
* **问题（页断裂/Partial Page Write）**：InnoDB 页大小默认 16KB，文件系统（OS）页通常 4KB。极端宕机时，可能只写了前 4KB，导致页损坏，Redo Log 无法恢复（Redo Log 记录的是物理变更，依赖页结构完整）。
* **机制**：
    1. 脏页刷盘时，先 `memcpy` 到内存中的 Double Write Buffer。
    2. 顺序写入系统表空间的 Double Write 区域（磁盘，分两次写，每次 1MB）。
    3. `fsync` 确保落盘。
    4. 最后离散写入对应的数据文件。
* **恢复**：若发现页损坏，从 Double Write 区域找到完整的副本覆盖，再重放 Redo Log。

#### 4. 什么是自适应哈希索引（Adaptive Hash Index, AHI）？
* InnoDB 会监控索引搜索，如果发现某些热点页频繁被访问，会自动建立哈希索引（在内存中）。
* **优点**：将 B+树的 O(Depth) 查找降为 O(1)。
* **缺点**：维护哈希表有锁开销，高并发写场景下可能成为瓶颈（可关闭 `innodb_adaptive_hash_index`）。

#### 5. Online DDL 的原理是什么？（为什么加字段不再锁表？）
* **MySQL 5.6 之前**：Copy Table（新建临时表->复制数据->重命名），期间锁全表写。
* **In-Place 方式（5.6+）**：
    * **Prepare 阶段**：持有元数据锁（MDL）排他锁，极短。
    * **Execute 阶段**：降级为 MDL 共享锁（允许 DML 读写）。不仅修改数据字典，还会记录 DDL 期间产生的 DML 操作日志（Row Log）。
    * **Commit 阶段**：升级 MDL 排他锁，重放 Row Log，确保数据一致。

#### 6. 自增主键为什么可能不连续？
* **事务回滚**：申请的 ID 不会退还。
* **唯一键冲突**：插入报错，ID 已消耗。
* **批量插入优化**：InnoDB 为批量插入预申请 ID（可能有空洞）。
* **重启（8.0 之前）**：自增计数器存内存，重启后取 `max(id)+1`，如果之前删除了最大记录，重启后 ID 可能回溯（8.0 修复，存 redo log）。

#### 7. MySQL 8.0 有哪些重要新特性？
* **原子 DDL**：DDL 操作（如 `DROP TABLE`）要么全成功要么全回滚，不再有残留文件。
* **窗口函数 (Window Functions)**：`RANK()`, `ROW_NUMBER()` 等，方便复杂统计。
* **公用表表达式 (CTE)**：`WITH recursive_cte AS (...)`，支持递归查询（如树形结构）。
* **不可见索引 (Invisible Indexes)**：软删除索引，用于测试删除索引对性能的影响。


### MySQL 面试题库大全 (补充精选)

#### 1. 基础与数据类型
1.  **MySQL 的默认端口号是多少？** 3306。
2.  **MyISAM 和 InnoDB 的主要区别？**
    *   InnoDB：支持事务、行级锁、外键、崩溃恢复、聚簇索引。
    *   MyISAM：不支持事务、表级锁、全文索引（早期）、读取快、无崩溃恢复。
3.  **CHAR 和 VARCHAR 的区别？**
    *   CHAR：定长，不足补空格，查询快，适合短且固定长度（如身份证、MD5）。
    *   VARCHAR：变长，存储实际长度+长度前缀，节省空间。
4.  **TEXT 和 BLOB 的区别？**
    *   TEXT：存储字符数据，不区分大小写（依赖校对集）。
    *   BLOB：存储二进制数据，区分大小写。
5.  **DATETIME 和 TIMESTAMP 的区别？**
    *   DATETIME：8 字节，与时区无关，范围 1000-9999 年。
    *   TIMESTAMP：4 字节，随时区变化，范围 1970-2038 年。
6.  **DECIMAL 和 FLOAT/DOUBLE 的区别？**
    *   DECIMAL：定点数，精确存储，适合金额。
    *   FLOAT/DOUBLE：浮点数，有精度损失。
7.  **NULL 和空字符串的区别？**
    *   NULL：占用额外空间标记，不参与 count() 统计，索引效率略低。
    *   空字符串：长度为 0，不占标记空间。
8.  **`int(11)` 中的 11 代表什么？** 显示宽度，不影响存储大小（int 始终 4 字节）。
9.  **MySQL 里面的 `ENUM` 类型有什么优缺点？**
    *   优：节省空间（内部存整数）。
    *   缺：修改枚举值需要 DDL 锁表，排序按整数索引而非字符串。
10. **`DELETE`, `TRUNCATE`, `DROP` 的区别？**
    *   DELETE：DML，逐行删除，可回滚，慢。
    *   TRUNCATE：DDL，清空表，重置自增 ID，不可回滚，快。
    *   DROP：DDL，删除表结构和数据。

#### 2. 索引与 B+树
11. **为什么 MySQL 用 B+ 树不用 B 树？**
    *   B+ 树非叶子节点不存数据，扇出更高，树更矮，I/O 次数更少。
    *   叶子节点由链表连接，范围查询更优。
12. **为什么不用 Hash 索引？** Hash 不支持范围查询、排序、模糊匹配，仅支持精确匹配。
13. **什么是聚簇索引？** 数据存储在主键索引的叶子节点上。
14. **什么是非聚簇索引（二级索引）？** 叶子节点存储主键值，需回表查询。
15. **什么是覆盖索引？** 查询的列完全包含在索引中，无需回表。
16. **最左前缀原则是什么？** 联合索引 `(a,b,c)`，只能利用 `a`, `ab`, `abc`，跳过 `a` 直接用 `b` 无效。
17. **索引下推（ICP）是什么？** MySQL 5.6+，在存储引擎层利用索引过滤数据，减少回表次数。
18. **什么情况索引失效？**
    *   对索引列做运算/函数。
    *   隐式类型转换（字符串不加引号）。
    *   `LIKE '%abc'` 前缀模糊。
    *   `OR` 连接非索引列。
    *   `!=` 或 `<>`（可能失效，看优化器）。
19. **如何查看索引使用情况？** `EXPLAIN` 或 `SHOW INDEX FROM table`。
20. **主键选择自增 ID 还是 UUID？**
    *   推荐自增 ID：INT/BIGINT 占用小，顺序写入减少页分裂。
    *   UUID：字符串长（占用大），随机写入导致大量页分裂和磁盘随机 I/O。

#### 3. 事务与锁
21. **事务 ACID 特性？** 原子性、一致性、隔离性、持久性。
22. **并发事务带来的问题？** 脏读、不可重复读、幻读。
23. **MySQL 的四种隔离级别？**
    *   Read Uncommitted (读未提交)
    *   Read Committed (RC, 读已提交)
    *   Repeatable Read (RR, 可重复读, 默认)
    *   Serializable (串行化)
24. **InnoDB 如何解决幻读？**
    *   快照读：MVCC。
    *   当前读：Next-Key Lock（间隙锁）。
25. **什么是 MVCC？** 多版本并发控制，通过 Undo Log 实现历史版本读取，避免读写阻塞。
26. **什么是行锁、表锁、页锁？** InnoDB 支持行锁（开销大，并发高），MyISAM 仅支持表锁。
27. **共享锁（S）与排他锁（X）？**
    *   S 锁：读锁，允许其他事务读，阻止写。
    *   X 锁：写锁，阻止其他事务读写。
28. **意向锁（IS/IX）的作用？** 表级锁，用于快速判断表中是否有行锁，避免遍历检查。
29. **什么是死锁？如何解决？**
    *   两个事务互相等待对方持有的锁。
    *   解决：设置超时、开启死锁检测（自动回滚小事务）、固定顺序访问资源。
30. **`SELECT ... FOR UPDATE` 是什么锁？** 排他锁（X 锁）。

#### 4. 日志系统 (Logs)
31. **Redo Log（重做日志）的作用？** 保证持久性（Crash Safe），物理日志，记录页的修改。循环写。
32. **Undo Log（回滚日志）的作用？** 保证原子性和 MVCC，逻辑日志，记录反向操作。
33. **Binlog（归档日志）的作用？** 主从复制、数据恢复。逻辑日志，追加写。
34. **Redo Log 和 Binlog 的区别？**
    *   Redo：InnoDB 层，物理日志，循环写，用于恢复。
    *   Binlog：Server 层，逻辑日志，追加写，用于复制。
35. **什么是两阶段提交（2PC）？** 保证 Redo Log 和 Binlog 的一致性。Prepare -> 写 Binlog -> Commit。
36. **Binlog 的格式？**
    *   Statement：记录 SQL（少，可能有不一致）。
    *   Row：记录行变更（多，安全）。
    *   Mixed：混合模式。
37. **Relay Log（中继日志）的作用？** 从库 I/O 线程读取主库 Binlog 写入 Relay Log，SQL 线程读取执行。
38. **Slow Query Log（慢查询日志）？** 记录执行时间超过 `long_query_time` 的 SQL。
39. **General Log？** 记录所有 SQL，一般关闭。
40. **Error Log？** 记录启动、运行错误。

#### 5. 性能优化
41. **SQL 优化的一般步骤？**
    *   看慢日志。
    *   `EXPLAIN` 分析执行计划。
    *   优化索引、改写 SQL。
42. **`EXPLAIN` 关键字段？**
    *   `type`：system > const > eq_ref > ref > range > index > ALL。
    *   `key`：实际用到的索引。
    *   `rows`：扫描行数。
    *   `Extra`：Using filesort (需优化), Using temporary (需优化), Using index (好)。
43. **如何优化 `LIMIT 1000000, 10`？**
    *   子查询 ID：`SELECT * FROM t WHERE id >= (SELECT id FROM t LIMIT 1000000, 1) LIMIT 10`。
    *   `JOIN` 延迟关联。
44. **如何优化 `COUNT(*)`？**
    *   MyISAM 自带计数器快。
    *   InnoDB 需扫描，可用 Redis 缓存计数或近似值 (`SHOW TABLE STATUS`)。
45. **如何优化 `ORDER BY`？** 尽量利用索引排序，避免 `Using filesort`。
46. **如何优化 `GROUP BY`？** 默认会排序，若无需排序加 `ORDER BY NULL`。
47. **大表如何加索引？** 
    *   业务低峰期。
    *   使用 `pt-online-schema-change` 或 `gh-ost`。
    *   MySQL 5.6+ 支持 Online DDL。
48. **`JOIN` 优化？** 小表驱动大表，连接字段加索引。
49. **`IN` 和 `EXISTS` 的区别？**
    *   `IN`：适合子查询表小。
    *   `EXISTS`：适合外表小。
50. **垂直拆分和水平拆分？**
    *   垂直：按列拆分（大字段独立）。
    *   水平：按行拆分（分库分表）。

#### 6. 架构与高可用
51. **MySQL 主从复制原理？**
    *   主库写 Binlog。
    *   从库 I/O 线程拉取 Binlog 写入 Relay Log。
    *   从库 SQL 线程重放 Relay Log。
52. **主从延迟的原因及解决？**
    *   原因：从库单线程（5.6 前）、大事务、网络延迟。
    *   解决：升级 MySQL（MTS 多线程复制）、拆分大事务、半同步复制。
53. **GTID 是什么？** 全局事务 ID，简化主从切换和故障恢复。
54. **什么是半同步复制？** 主库等待至少一个从库接收 Binlog 后才返回 Commit 成功。
55. **常见的 MySQL 高可用方案？**
    *   MHA (Master High Availability)。
    *   MGR (MySQL Group Replication)。
    *   Orchestrator。
    *   PXC (Percona XtraDB Cluster)。
56. **读写分离的实现？** ShardingSphere, MyCat, MySQL Router, 代码层封装。
57. **分库分表带来的问题？** 分布式事务、跨库 JOIN、跨库排序分页、全局 ID。
58. **如何生成全局唯一 ID？**
    *   Snowflake 算法。
    *   UUID。
    *   Redis 自增。
    *   数据库号段模式。
59. **MySQL 8.0 新特性？** 窗口函数、CTE、原子 DDL、JSON 增强、降序索引。
60. **Buffer Pool 也就是缓冲池，作用？** 缓存数据页和索引页，减少磁盘 I/O。

#### 7. 常用命令与运维
61. **如何连接 MySQL？** `mysql -u user -p -h host`。
62. **如何备份数据库？** `mysqldump`。
63. **如何导入数据？** `source file.sql` 或 `mysql < file.sql`。
64. **查看当前连接？** `SHOW PROCESSLIST`。
65. **杀掉连接？** `KILL connection_id`。
66. **查看变量？** `SHOW VARIABLES LIKE '%xxx%'`。
67. **查看状态？** `SHOW STATUS LIKE '%xxx%'`。
68. **查看表结构？** `DESC table_name` 或 `SHOW CREATE TABLE table_name`。
69. **修改密码？** `ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_pass'`。
70. **授权用户？** `GRANT ALL PRIVILEGES ON db.* TO 'user'@'%'`。

#### 8. 进阶与坑
71. **自增主键用完了怎么办？** 改用 BIGINT，若 BIGINT 也能用完（几乎不可能）。
72. **为什么 `SELECT *` 不好？** 无法覆盖索引，增加网络传输，解析开销大。
73. **MySQL 默认隔离级别？** RR（可重复读）。Oracle 是 RC。
74. **什么是幻读？** 同一事务两次查询，结果集条数不一致（多了或少了）。
75. **Gap Lock（间隙锁）的作用？** 防止插入，解决幻读。
76. **什么情况会死锁？** 逆序更新、间隙锁冲突。
77. **Online DDL 的原理？** `In-Place` 模式，不锁表，通过 Row Log 记录 DDL 期间的 DML。
78. **Change Buffer 适用场景？** 写多读少，非唯一索引。
79. **Double Write 的作用？** 防止页断裂（Partial Page Write）。
80. **AHI (自适应哈希索引)？** 监控热点页自动建立哈希索引。

#### 9. SQL 语句专项
81. **去重查询？** `DISTINCT`。
82. **模糊查询？** `LIKE`。
83. **排序？** `ORDER BY`。
84. **分组？** `GROUP BY`。
85. **分组后过滤？** `HAVING`。
86. **连接查询有哪些？** `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, `FULL JOIN` (MySQL 不支持，用 UNION)。
87. **子查询分类？** 标量子查询、列子查询、行子查询、表子查询。
88. **`UNION` 和 `UNION ALL`？** `UNION` 去重（慢），`UNION ALL` 不去重（快）。
89. **插入或更新？** `INSERT INTO ... ON DUPLICATE KEY UPDATE`。
90. **替换插入？** `REPLACE INTO` (先删后插)。

#### 10. 综合考察
91. **一条 SQL 执行很慢的原因？**
    *   偶尔慢：锁等待、Redo Log 刷盘。
    *   一直慢：没索引、索引失效、数据量太大。
92. **MySQL 大表数据归档策略？** `pt-archiver`，按时间分批导出删除。
93. **如何清洗数据？** 存储过程或脚本。
94. **MySQL 占用 CPU 过高怎么排查？** `top` 定位，`show processlist` 找慢 SQL。
95. **MySQL 内存占用过高？** 检查 `innodb_buffer_pool_size` 及线程 buffer。
96. **如何彻底删除 MySQL？** `yum remove` 或 `apt remove` 并清理数据目录 /var/lib/mysql。
97. **MySQL 5.7 和 8.0 区别？** 8.0 性能更好，NoSQL 支持，窗口函数，默认字符集 utf8mb4。
98. **UTF-8 和 UTF8MB4？** UTF-8 是 MySQL 的阉割版（3 字节），UTF8MB4 是完整版（4 字节，支持 Emoji）。
99. **MySQL 是行式存储还是列式存储？** 行式存储（InnoDB/MyISAM）。列式存储如 ClickHouse。
100. **如何实现乐观锁？** 增加 `version` 字段，更新时 `WHERE version = old_version`。


