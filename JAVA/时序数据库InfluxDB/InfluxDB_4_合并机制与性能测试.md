# InfluxDB 合并机制与性能测试

## 1. 合并 (Compaction) 机制

压缩（Compaction）是 TSM 引擎将写入优化格式（WAL/Cache）迁移到读取优化格式（TSM File）的过程，并负责空间回收。

### 合并时机
1.  **内存阈值触发**: 当 Cache 达到 `cache-snapshot-memory-size` 时，触发 Snapshot。
2.  **定期检查**: 默认每秒检查一次。
3.  **冷数据触发**: 如果 `compact-full-write-cold-duration`（默认4h）内无写入，对 Shard 执行 Full Compaction。

### 合并级别
*   **Level 1-4**: 新文件为 L1。多个 L1 合并为 L2，直至 L4。
*   **低级别合并**: 策略倾向于快速合并，避免过度的解压/重组，减少 CPU 消耗。
*   **高级别合并 (Full Compaction)**: 会重新合并所有 Block，极大提高压缩率，优化索引。

### 索引优化 (Index Optimization)
当 L4 文件过多导致内部索引过大时，触发此过程。
*   **目标**: 将同一 Series 的所有点尽可能聚拢到同一个 TSM 文件中。
*   **效果**: 减少查询时的磁盘 I/O，并减小全局索引体积。

---

## 2. 性能测试 (Benchmark)

基于官方测试工具的实验数据总结：

### 写入性能测试 (influx-stress)
*   **测试环境**: 针对 `POST /write` API 的高并发模拟。
*   **实验数据**:
    *   **20万 pts/s**: CPU 利用率 ~33%
    *   **50万 pts/s**: CPU 利用率 ~80%
    *   **60万 pts/s**: 达到瓶颈，CPU 利用率 ~90%，吞吐量不再增加。
*   **结论**: 单机吞吐极限约为 **60万点/秒**。

### 查询性能测试 (influxdb-comparisons)
*   **测试场景**: 基于 1 小时时间窗口的聚合查询（如 `SELECT max(usage_user) GROUP BY time(1m)`）。
*   **实验数据**:
    *   **平均响应时间**: ~1.64ms (单次查询)。
    *   **并发查询能力**: 在标准数据集下，约为每秒执行 **600次** 复杂查询。

---

## 3. 运维与调优建议
*   **批量写入**: 建议批次大小在 5,000-10,000 点，以获得最佳吞吐。
*   **防止背压**: 监控 `cache-max-memory-size`。若写入速度持续超过 Compaction 速度，Cache 满会导致新写入失败。
*   **Shard 策略**: 合理设置 Shard Duration。对于冷数据，Full Compaction 后将不再消耗资源。

---
*整理自：TSM 文件的合并时机、influxDB性能测试 相关文档。*
