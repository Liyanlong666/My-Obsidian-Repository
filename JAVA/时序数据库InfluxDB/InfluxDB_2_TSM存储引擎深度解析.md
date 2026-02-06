# InfluxDB TSM 存储引擎深度解析

TSM (Time-Structured Merge Tree) 是 InfluxDB 专门为时序数据优化的核心存储引擎。它在 LSM 树的基础上，通过列式存储和专用索引解决了时间序列数据的高吞吐和高基数问题。

## 1. TSM 架构组成

### WAL (Write Ahead Log)
*   **目的**: 确保数据持久化，崩溃后可恢复。
*   **特性**: 顺序追加写入，性能极高。文件大小达到 10MB 后会滚动产生新段。
*   **流程**: 数据先写入 WAL，并在 fsync 后才向客户端返回成功。

### Cache (缓存)
*   **目的**: 存储最近写入的热数据，支持即时查询。
*   **结构**: WAL 中数据的内存表示，按 SeriesKey 组织的 Map。
*   **阈值**: 超过 `cache-snapshot-memory-size` 会触发快照刷新到 TSM 文件。

### TSM Files
*   **目的**: 磁盘上的只读、不可变数据文件。
*   **特性**: 采用列式存储格式，高度压缩。
*   **加载方式**: 内存映射 (mmap)。

### TSI (Time Series Index)
*   **目的**: 解决“高基数”问题（大量 SeriesKey 导致的内存瓶颈）。
*   **特性**: 将索引持久化到磁盘，支持按需加载。

---

## 2. TSM 文件格式 (TSM File Structure)

TSM 文件由 Header, Blocks, Index, Footer 四部分组成：

```text
+--------+------------------------------------+-------------+--------------+
| Header |               Blocks               |    Index    |    Footer    |
|5 bytes |              N bytes               |   N bytes   |   4 bytes    |
+--------+------------------------------------+-------------+--------------+
```

### Header
*   Magic (4 bytes): 标识文件类型。
*   Version (1 byte): 引擎版本号。

### Blocks (Data Section)
存储实际的压缩数据。每个 Block 仅包含**一个 SeriesKey 在一段连续时间内的一个 Field** 的数据。
*   **Type (1 byte)**: 数据类型（Float, Integer, Boolean, String）。
*   **Length (VByte)**: Timestamps 的压缩长度。
*   **Timestamps**: 压缩的时间戳集合（通常使用 Delta-delta 编码）。
*   **Values**: 压缩的指标值集合（根据类型使用 XOR, Snappy 等编码）。

### Index (Index Section)
索引块按 SeriesKey 的字典序排列，支持快速定位。
*   **Series Key**: Measurement + Tags。
*   **Field Key**: 字段名。
*   **Type**: 数据类型。
*   **Count**: 该 Series 下数据块的数量。
*   **Block Entries**: 包含每个数据块的 `MinTime`, `MaxTime`, `Offset` (文件偏移) 和 `Size`。

### Footer
存储索引块开始的偏移量 (8 bytes)，文件读取时先从页脚开始。

---

## 3. 核心流程

### 写入流程
1.  追加写入 WAL。
2.  更新内存 Cache。
3.  当 Cache 满或达到冷时间阈值，后台进程将其 Snapshot 成 TSM 文件并清空 WAL。

### 查询流程
1.  **定位文件**: 根据查询的时间范围，通过 FileStore 过滤可能相关的 TSM 文件。
2.  **查找索引**: 在 TSM 文件的 Index 部分执行二分查找，定位 SeriesKey。
3.  **时间过滤**: 根据 `MinTime` 和 `MaxTime` 进一步筛选目标 Data Block。
4.  **读取解压**: 按照 `Offset` 读取 Data Block，解压并返回数据。
5.  **合并结果**: 将从磁盘读取的数据与内存 Cache 中的热数据进行合并（Cache 数据优先，以支持更新）。

---
*整理自：内存索引和时间结构合并树 (TSM)、TSM（Time Structured Merge Tree）、InfluxDB TSM存储引擎之TSMFile 相关文档。*
