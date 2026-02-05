## TSM 文件的合并时机
TSM 文件的合并（Compaction）时机主要有以下几种情况：

1. **缓存或 WAL 达到阈值时**  
当写入的 WAL（Write Ahead Log）段文件达到最大大小（如 10MB），或者内存中的 Cache 达到设定的内存阈值时，会触发快照（Snapshot）操作，将 Cache 和 WAL 中的数据转换为新的 TSM 文件。这属于最基础的合并时机，用于释放内存和磁盘空间，并保证数据持久化【[InfluxDB v1 Storage Engine - Writes](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#writes)】。
2. **定期检查合并需求**  
InfluxDB 每秒会检查一次是否需要进行合并（Compaction）。如果在 `compact-full-write-cold-duration`（默认 4 小时）内没有写入，InfluxDB 会对所有 TSM 文件进行一次全量合并（Full Compaction），以优化存储和查询性能【[TSI Compaction](https://docs.influxdata.com/influxdb/v1/concepts/tsi-details/#compaction)】。
3. **分级合并（Level Compactions）**  
TSM 文件分为多个级别（Level 0~4）。新生成的 TSM 文件为 Level 1，多个 Level 1 文件会被合并为 Level 2，依此类推，直到 Level 4。合并的优先级由文件的级别和可合并文件的数量决定，低级别文件优先合并。如果高级别有更多可合并文件，也可能优先合并高级别文件【[Compactions](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#compactions)】【[TSI Compaction](https://docs.influxdata.com/influxdb/v1/concepts/tsi-details/#compaction)】。
4. **索引优化和全量合并**  
当 Level 4 文件过多时，会触发索引优化合并（Index Optimization），将同一 series 的数据尽量合并到同一个 TSM 文件中，减少索引冗余。长时间无写入或有大量删除操作时，会触发全量合并（Full Compaction），生成最优的 TSM 文件集【[Compactions](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#compactions)】。
5. **配置参数影响**  
合并的时机和频率还受多项配置参数影响，如 `cache-snapshot-memory-size`、`cache-snapshot-write-cold-duration`、`max-concurrent-compactions`、`compact-throughput`、`compact-throughput-burst` 等，这些参数可以根据系统负载进行调整【[TSI Compaction](https://docs.influxdata.com/influxdb/v1/concepts/tsi-details/#compaction)】。

总结：  
TSM 文件的合并时机主要由写入速率、缓存/WAL 阈值、定期检查、文件分级、索引优化、全量合并以及相关配置参数共同决定。合并的目的是提升读写性能、优化存储和释放资源。  


## TSM（Time Structured Merge Tree）VS  LSM（Log Structured Merge Tree）  

  
TSM（Time Structured Merge Tree）相较于传统的 LSM（Log Structured Merge Tree）在以下几个方面进行了优化，特别针对时序数据的特点：

1. **更高的压缩率与更优的磁盘空间利用**  
TSM 针对时序数据的结构进行了专门设计，实现了比 LSM（如 LevelDB、RocksDB）更高的压缩率。在实际应用中，TSM 能够比 B+Tree 和 LSM Tree 实现高达 45 倍的磁盘空间节省【[The new InfluxDB storage engine](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#the-new-influxdb-storage-engine-from-lsm-tree-to-btree-and-back-again-to-create-the-time-structured-merge-tree)】。
2. **更高的写入和读取吞吐量**  
TSM 针对时序数据的批量写入和顺序读取进行了优化，能够实现比传统 LSM Tree 更高的写入和读取性能【[TSM Glossary](https://docs.influxdata.com/influxdb/v1/concepts/glossary/#tsm-time-structured-merge-tree)】。
3. **专为时序数据设计的数据组织方式**  
TSM 文件以列式（columnar）方式存储压缩后的时序数据，并按 series key 分组、按时间排序，便于高效的范围查询和压缩【[InfluxDB v2 Storage Engine Internals](https://docs.influxdata.com/influxdb/v2/reference/internals/storage-engine/#time-structured-merge-tree-tsm)】。
4. **高效的数据删除与保留策略**  
LSM Tree 在大规模数据删除时效率较低（依赖 tombstone 和后续合并），而 TSM 通过分片（shard）机制，可以直接丢弃整块过期数据文件，极大提升了数据保留和删除的效率【[LevelDB and log structured merge trees](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#leveldb-and-log-structured-merge-trees)】。
5. **更适合冷热数据分离和分布式场景**  
TSM 的分片机制便于冷热数据分离和在集群中的迁移，而 LSM Tree 结构在大数据量场景下容易出现文件句柄过多等问题【[LevelDB and log structured merge trees](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#leveldb-and-log-structured-merge-trees)】。

综上，TSM 针对时序数据的特点，在压缩率、读写性能、数据组织、数据删除和分布式管理等方面都优于传统的 LSM Tree 实现，是 InfluxDB 为满足高性能时序数据需求而专门设计的存储引擎【[The new InfluxDB storage engine](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#the-new-influxdb-storage-engine-from-lsm-tree-to-btree-and-back-again-to-create-the-time-structured-merge-tree)】。





## <font style="color:rgb(0, 0, 0);">influxDB 对比 mysql 的优势</font>
InfluxDB 相比 MySQL（或其他传统关系型数据库）在处理时序数据方面有以下显著优势：

1. **专为时序数据设计**  
InfluxDB 是专门为时序数据（如监控、物联网、日志等）设计的数据库，能够高效地存储和分析带有时间戳的数据点。MySQL 虽然可以存储时序数据，但并未针对高频写入和大规模时序数据的特点进行优化【[<font style="color:rgb(1, 10, 67);">Compare InfluxDB to SQL databases</font>](https://docs.influxdata.com/influxdb/v1/concepts/crosswalk/)】。
2. **高效的写入和查询性能**  
InfluxDB 能够应对极高的数据写入速率和大规模数据量，支持实时数据的快速写入和查询，适合基础设施监控、IoT、金融等场景。MySQL 在高并发写入和大数据量时容易出现性能瓶颈【[<font style="color:rgb(1, 10, 67);">Using InfluxDB for infrastructure monitoring</font>](https://www.influxdata.com/blog/using-time-series-data-infrastructure-monitoring-advantages-limitations/#heading1)】。
3. **灵活的数据模型与无模式（Schema-less）设计**  
InfluxDB 不需要提前定义表结构，数据点可以动态添加字段，适应数据结构频繁变化的场景。MySQL 需要固定的表结构，灵活性较差【[<font style="color:rgb(1, 10, 67);">Compare InfluxDB to SQL databases</font>](https://docs.influxdata.com/influxdb/v1/concepts/crosswalk/)】。
4. **高效的数据压缩与存储优化**  
InfluxDB 采用专为时序数据优化的存储引擎（如 TSM、Parquet），支持高效的数据压缩和分级存储，极大降低存储成本。MySQL 的存储引擎并未针对时序数据压缩和冷热分层做专门优化【[<font style="color:rgb(1, 10, 67);">Using InfluxDB for infrastructure monitoring</font>](https://www.influxdata.com/blog/using-time-series-data-infrastructure-monitoring-advantages-limitations/#heading1)】。
5. **内置数据保留与自动清理机制**  
InfluxDB 支持灵活的数据保留策略（Retention Policy），可以自动清理过期数据，便于长期存储和合规管理。MySQL 需要手动管理历史数据的清理和归档【[<font style="color:rgb(1, 10, 67);">Using InfluxDB for infrastructure monitoring</font>](https://www.influxdata.com/blog/using-time-series-data-infrastructure-monitoring-advantages-limitations/#heading1)】。
6. **查询语言优化**  
InfluxDB 支持 SQL 及类 SQL 查询语言（InfluxQL），并针对时序聚合、窗口分析等场景做了优化，查询表达能力强且易用【[<font style="color:rgb(1, 10, 67);">Compare InfluxDB to SQL databases</font>](https://docs.influxdata.com/influxdb/v1/concepts/crosswalk/)】。

**总结**：  
InfluxDB 在高性能时序数据写入、灵活的数据模型、自动数据管理和高效存储等方面，相比 MySQL 更适合大规模、实时的时序数据场景。

