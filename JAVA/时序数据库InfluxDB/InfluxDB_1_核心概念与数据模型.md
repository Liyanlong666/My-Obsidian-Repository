# InfluxDB 核心概念与数据模型

## 1. 基础概念

### Measurement
Measurement 形似传统数据库中的表，但没有预定义的 Schema。它在写入时指定，用来标记一条数据的归属。Measurement 更像是一个字符串类型的元数据字段。

### DataBase
InfluxDB 中的 Database 是 Schema 的载体。通过 Database 可以指定副本数、保留策略（Retention Policy）等。
```sql
CREATE DATABASE <database_name> [WITH [DURATION <duration>] [REPLICATION <n>] [SHARD DURATION <duration>] [NAME <retention-policy-name>]]
```

### Tag (标签)
*   **定义**: Key-Value 对，描述数据点的特征。
*   **特性**: **会建立倒排索引**。使用 Tag 作为查询条件（谓词）时性能极高。
*   **示例**: `region=beijing, ip=192.168.1.2`。

### Field (字段)
*   **定义**: Key-Value 对，表示指标名和值。
*   **特性**: **不会创建索引**。使用 Field 作为查询条件会导致全表扫描（Scan）。
*   **示例**: `cpu_usage=0.9, mem_usage=0.5`。

### Point (数据点)
相当于关系数据库的一行记录。由以下部分构成：
*   1个 Measurement
*   N个 Tag
*   N个 Field
*   1个 时间戳 (Timestamp)

### Retention Policy (RP, 保留策略)
定义数据的保留周期、副本数以及 Shard Duration。
*   时序数据通常具有时效性，RP 允许自动淘汰过期的历史数据。
*   默认 RP 为 `autogen`，保留周期为无限。

### Series (时间线)
时间线是 InfluxDB 的核心。所有指标数据都按时间线组织。
*   **Series Key**: 由 `Measurement + Tags` 唯一确定。
*   同一 Series 的数据在物理上会聚集存储，以优化查询。

---

## 2. 数据模型对比

### 关系模型 vs. 时间线模型
*   **关系模型**: 需要提前定义固定的 Schema，写入时需严格遵守，字段不能有空洞。
*   **时间线模型 (Schemaless)**: 采用 Key-Value 结构。
    *   **Key**: `SeriesKey + FieldKey`
    *   **Value**: `<Timestamp, FieldValue>` 的集合。
    *   **优势**: 极其灵活，可以随意增减 Tag 和 Field，天然适应物联网等动态场景。

---

## 3. 存储逻辑架构

从高到低：
1.  **Database**: 包含多个 RP。
2.  **Retention Policy (RP)**: 下设多个 Shard Group。
3.  **Shard Group (SG)**: 按照时间范围划分。
4.  **Shard**: 数据的最小物理单元。
    *   每个 Shard 拥有独立的 **WAL**、**TSM** 和 **TSI**。
    *   单机版中，一个 Shard Group 通常只包含一个 Shard。

---
*整理自：InfluxDB 技术概述、内存索引和时间结构合并树 (TSM) 相关文档。*
