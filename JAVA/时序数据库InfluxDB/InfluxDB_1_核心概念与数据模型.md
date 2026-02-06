# InfluxDB 核心概念与数据模型

## 1. 基础概念详解

### Measurement (测量值)
Measurement 形似传统数据库中的表，但本质上是数据的逻辑容器。InfluxDB 没有建表语句，也没有预定义的 Schema。它更像是一个普通的字符串字段，在写入时自动创建。

### DataBase (数据库)
Database 是 Schema 的载体。它可以指定：
*   **DURATION**: 数据保留时长。
*   **REPLICATION**: 副本数。
*   **SHARD DURATION**: Shard Group 覆盖的时间范围。
```sql
CREATE DATABASE <database_name> WITH DURATION 30d REPLICATION 1 SHARD DURATION 1d NAME "monthly_rp"
```

### Tag (标签)
*   **定义**: Key-Value 对，用于描述数据点的元数据（如 `host`, `region`, `app_id`）。
*   **核心特性**: **索引化 (Indexed)**。InfluxDB 会为 Tag 建立倒排索引，因此它是查询谓词的首选。
*   **注意**: Tag Value 必须是字符串。

### Field (字段)
*   **定义**: 指标名和实测值（如 `cpu_usage`, `temperature`）。
*   **核心特性**: **非索引化 (Non-indexed)**。如果对 Field 进行过滤，会触发全表扫描。
*   **类型**: 支持 Float, Integer, String, Boolean。

### Series (时间线) —— 核心概念
时间线相当于数据源。在物理上，InfluxDB 是按时间线来组织和压缩数据的。
*   **Series Key**: `Measurement + Tags`。
*   **生动案例（智能手环）**: 
    我们可以认为 `SeriesKey` 就是一个智能手环（数据源）。
    *   **Measurement**: 智能手环。
    *   **Tags**: `user=ZhangSan, model=V2`（唯一标识这个手环）。
    *   **Fields**: `heart_rate`（心跳采集器）、`step_count`（计步器）。每个采集器产生的数据流就是一个时间序列。

---

## 2. 数据模型对比

### 关系模型 vs. 时间线模型 (Schemaless)
关系模型（MySQL等）需要严格定义 Schema，而 InfluxDB 更加灵活。

**关系模型示例：**

| Timestamp | region | host | cpu_usage | mem_usage |
| --- | --- | --- | --- | --- |
| 2024-01-15 00:00:00 | Beijing | server01 | 0.2 | 0.1 |
| 2024-01-15 00:00:10 | Beijing | server01 | 0.5 | 0.3 |

**时间线模型（InfluxDB）逻辑表示：**
InfluxDB 实质上是 Key-Value 结构：
*   **Key**: `seriesKey + fieldKey`
*   **Value**: `List<Timestamp, Value>`
这种结构允许随意增减 Tag 和 Field，无需修改表结构。

---

## 3. 存储逻辑架构层级

InfluxDB 的数据管理是层层嵌套的：
1.  **Database**: 顶层容器。
2.  **Retention Policy (RP)**: 决定数据存多久、存几份。一个 DB 可有多个 RP。
3.  **Shard Group (SG)**: RP 下按时间段划分的逻辑组（如按天划分）。
    *   解决大量删除问题：删除过期数据时，直接删除整个 Shard Group 对应的目录，效率极高。
4.  **Shard**: 物理存储单元。
    *   每个 Shard 包含自己的 **WAL**、**TSM** 和 **TSI**。
    *   数据根据 SeriesKey 的 Hash 值路由到不同的 Shard（单机版通常只有一个 Shard）。

---
*整理自：InfluxDB 技术概述、TSM 存储引擎与 LSM 树相关文档。*

