# InfluxDB TSM 存储引擎深度解析

TSM (Time-Structured Merge Tree) 是 InfluxDB 针对时序数据优化的核心。

## 1. 核心组件详解

### WAL (Write Ahead Log)
*   **存储格式**: 采用 **TLV (Type-Length-Value)** 标准。
    *   `Type` (1 byte): 条目类型（写入或删除）。
    *   `Length` (4 bytes, uint32): 压缩块的长度。
    *   `Compressed Block`: 使用 Snappy 压缩的数据。
*   **持久化**: 采用 `fsync` 确保落盘。文件编号单调递增（如 `_00001.wal`）。

### Cache (内存缓存)
*   **本质**: WAL 的内存副本，数据未压缩。
*   **查询合并**: 查询时会从 Cache 中读取副本，与 TSM 文件数据合并，确保能读到刚写入的数据。
*   **控制参数**:
    *   `cache-snapshot-memory-size`: 达到此阈值（默认25MB）触发 Snapshot，将数据存入 TSM 文件。
    *   `cache-max-memory-size`: 达到此上限（默认512MB）拒绝新写入，产生“背压”。
    *   `cache-snapshot-write-cold-duration`: 若指定时间内（默认10min）没有新写入，强制执行 Snapshot。

### TSM Files
磁盘上的不可变列式存储文件。

---

## 2. TSM 文件详细结构

一个 TSM 文件逻辑上分为四块，物理结构如下：

### 2.1 整体布局
```text
+--------+------------------------------------+-------------+--------------+
| Header |               Blocks               |    Index    |    Footer    |
|5 bytes |              N bytes               |   N bytes   |   4 bytes    |
+--------+------------------------------------+-------------+--------------+
```

### 2.2 Blocks (数据块)
每个 Block 包含 CRC32 校验和以及压缩数据。
```text
+---------------------+-----------------------+----------------------+
|       Block 1       |        Block 2        |       Block N        |
+---------------------+-----------------------+----------------------+
|   CRC    |  Data    |    CRC    |   Data    |   CRC    |   Data    |
| 4 bytes  | N bytes  |  4 bytes  | N bytes   | 4 bytes  |  N bytes  |
+---------------------+-----------------------+----------------------+
```
**Data 内部**: `Type(1B) | Len(VByte) | Compressed Timestamps | Compressed Values`

### 2.3 Index (索引块)
索引是按 Series Key 的字典序排列的。
```text
+-----------------------------------------------------------------------------+
| Key Len |   Key   | Type | Count |Min Time |Max Time | Offset |  Size  |...|
| 2 bytes | N bytes |1 byte|2 bytes| 8 bytes | 8 bytes |8 bytes |4 bytes |   |
+-----------------------------------------------------------------------------+
```
*   **Key**: `measurement + tagset + field`。
*   **Min/Max Time**: 方便查询时根据时间范围快速跳过无关 Block。

### 2.4 Footer (页脚)
仅 8 字节，存储 **Index 部分的起始偏移量**。
读取逻辑：`Open File -> Seek to Footer -> Read Index Offset -> Load Index -> Binary Search Key`。

---

## 3. 查询与删除细节

### 查询处理
1.  **二分查找**: 在内存映射的 Index 部分查找 Series Key。
2.  **定位 Block**: 获取 `Offset` 和 `Size`，直接跳到 Blocks 区域读取。
3.  **解压**: 仅解压目标数据块，IO 效率极高。

### 删除操作 (Tombstones)
由于 TSM 是不可变的，删除并不直接修改文件。
1.  在内存 Cache 逐出数据。
2.  写入一个 `.tombstone` 文件记录删除的 Series 和时间段。
3.  在下一次 **Compaction (合并)** 时，才会真正移除这些数据。

---
*整理自：内存索引和时间结构合并树 (TSM)、TSM（Time Structured Merge Tree）、InfluxDB TSM存储引擎之TSMFile 相关文档。*

