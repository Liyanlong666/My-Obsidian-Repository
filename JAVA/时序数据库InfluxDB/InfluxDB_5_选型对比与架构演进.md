# InfluxDB 选型对比与架构演进

## 1. 为什么选择 InfluxDB？ (选型对比)

在时序数据场景下，InfluxDB 与传统关系数据库及其他时序数据库的对比优势：

### InfluxDB vs. MySQL
*   **写入性能**: InfluxDB 采用 LSM 变体（TSM），顺序写入磁盘，避开了 B+ 树在大规模随机插入时的索引维护成本。实验显示其磁盘利用率可比 MySQL 提升数倍。
*   **数据管理**: 内置 Retention Policy，自动处理冷数据过期。MySQL 需要手动维护表分区或清理脚本。
*   **Schemaless**: 传感器等物联网数据结构多变，InfluxDB 无需提前建表。
*   **时序函数**: 内置 `DERIVATIVE`, `MOVING_AVERAGE`, `FILL` 等专用分析函数。

### InfluxDB vs. LSM (LevelDB/RocksDB)
*   **文件句柄问题**: 传统 LSM 产生大量小文件，高负载下易触发文件句柄限制。TSM 优化了文件合并逻辑，减少了碎片。
*   **删除效率**: LSM 通过墓碑 (Tombstone) 标记删除，大规模删除很慢。TSM 通过 Shard 机制，直接删除整个底层文件。
*   **压缩率**: TSM 针对时间戳和指标值使用专项压缩（Gorilla, Delta-of-delta），比通用压缩算法空间利用率更高（可达 45 倍提升）。

### InfluxDB vs. TimescaleDB / OpenTSDB
*   **部署复杂度**: InfluxDB 独立部署，开箱即用。TimescaleDB 依赖 PostgreSQL，OpenTSDB 依赖 HBase/Hadoop，维护成本极高。
*   **生态适配**: 与 Telegraf (数据采集) 原生绑定，配合 Grafana 形成完整闭环。

---

## 2. 存储引擎的演进历史

1.  **0.8 版本**: 支持 LevelDB, RocksDB, HyperLevelDB 等多种引擎，但受限于文件句柄和备份困难。
2.  **0.9.0 - 0.9.2**: 转向 **BoltDB** (B+ 树)。解决了备份和文件句柄问题，但在大数据量下 IOPS 爆炸，性能严重下滑。
3.  **0.9.5+ (1.x)**: 自研 **TSM** 引擎诞生，重新吸纳 LSM 的优点并针对时序深度优化，成为最终方案。

---

## 3. 架构前瞻：InfluxDB 3.0

InfluxDB 3.0 是对架构的完全重构，标志着从“专用存储”向“云原生大数据生态”的转型。

### 核心改进
*   **语言切换**: 核心引擎从 Go 转向 **Rust**。
*   **数据格式**: 引入 **Apache Arrow** (内存格式) 和 **Apache Parquet** (磁盘格式)。
*   **云原生**: 存算分离。数据存储在 **对象存储 (S3/OSS)** 中。
*   **无限基数**: 通过新引擎支持无限量级的 Tag 基数，解决 1.x 中 TSI 的内存瓶颈。
*   **互操作性**: 支持“零拷贝”数据共享，可直接对接大数据分析平台。

---
*整理自：时序数据库选型...、TSM 存储引擎与 LSM 树... 相关文档。*
