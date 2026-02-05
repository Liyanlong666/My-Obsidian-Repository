TSM（Time Structured Merge Tree）是InfluxDB的核心存储引擎，专门为时间序列数据优化设计。

# TSM 引擎
## TSM架构组成
**WAL（Write Ahead Log）**

+ 新写入的数据首先进入WAL
+ 提供崩溃恢复保障
+ 顺序写入，性能高效
+ 定期刷新到TSM文件

**Cache**

+ 内存中的数据缓存层
+ 存储最近写入的热数据
+ 按时间序列分组组织
+ 达到阈值后刷新到磁盘

**TSM Files**

+ 磁盘上的不可变数据文件
+ 按时间范围分片存储
+ 内部使用列式存储格式
+ 每个文件包含索引和数据块

## 存储优化特性
**数据压缩**

+ 针对不同数据类型使用专门压缩算法
+ 整数：Delta编码 + 变长编码
+ 浮点数：Gorilla压缩算法
+ 字符串：Snappy压缩
+ 布尔值：位打包
+ 时间戳：Delta-of-delta编码

**索引结构**

+ 每个TSM文件包含索引块
+ 索引记录series key到数据块的映射
+ 支持快速定位特定时间范围的数据
+ 使用布隆过滤器加速查找

## 数据组织方式
**Series Key**

+ 由measurement、tag set、field key组成
+ 例：`cpu,host=server01,region=us-west usage_idle`
+ 相同series的数据存储在一起

**时间分片**

+ 数据按时间范围分割到不同TSM文件
+ 默认按天或小时分片
+ 便于数据过期和查询优化

**Compaction过程**

+ 定期合并小的TSM文件
+ 去除重复和删除的数据
+ 优化存储空间和查询性能
+ 分为多个级别的compaction

## 查询优化
**时间范围查询**

+ 利用文件级时间索引快速过滤
+ 只读取相关时间范围的TSM文件
+ 支持并行读取多个文件

**Series过滤**

+ 使用tag索引快速定位series
+ 支持正则表达式和通配符匹配
+ 布隆过滤器减少磁盘IO

**Field查询**

+ 列式存储只读取需要的field
+ 支持聚合下推到存储层
+ 利用统计信息优化计算

## 写入性能优化
**批量写入**

+ 支持批量提交减少WAL写入次数
+ 内存中聚合相同series的数据点
+ 延迟刷新提高写入吞吐量

**无锁设计**

+ 使用Copy-on-Write避免读写锁竞争
+ WAL使用单线程写入保证顺序
+ Cache使用分片减少锁粒度

## 数据生命周期
**Retention Policy**

+ 自动删除过期数据
+ 按时间范围清理整个TSM文件
+ 支持不同精度的数据保留策略

**Shard管理**

+ 数据按时间分组到不同shard
+ 每个shard对应一组TSM文件
+ 支持shard的热冷数据分离

TSM存储引擎通过这些优化，在时间序列数据的写入性能、存储效率和查询速度方面都有很好的表现，特别适合高频率写入、按时间范围查询的场景。

# TSM文件的索引块
## 索引块整体结构
**文件布局**

```plain
TSM File Structure:
┌─────────────────┐
│   Header        │
├─────────────────┤
│   Data Blocks   │
│   (compressed)  │
├─────────────────┤
│   Index Block   │
├─────────────────┤
│   Footer        │
└─────────────────┘
```

索引块位于TSM文件末尾，包含所有数据块的元数据信息。

## 索引项结构
**单个索引项包含：**

+ **Series Key**: 完整的series标识符
+ **Field Key**: 具体的字段名
+ **Data Type**: 字段数据类型（float64, int64, string, boolean）
+ **Block Entries**: 该series+field的所有数据块信息

**Block Entry结构：**

```go
type BlockEntry struct {
    MinTime    int64  // 块内最小时间戳
    MaxTime    int64  // 块内最大时间戳
    Offset     int64  // 数据块在文件中的偏移量
    Size       uint32 // 压缩后的块大小
}
```

## 索引组织方式
**层次化索引**

```plain
Index Block:
├─ Series Index
│  ├─ "cpu,host=server01" → Field Index
│  │  ├─ "usage_idle" → [Block1, Block2, ...]
│  │  └─ "usage_user" → [Block3, Block4, ...]
│  └─ "memory,host=server01" → Field Index
│     ├─ "used_percent" → [Block5, Block6, ...]
│     └─ "available" → [Block7, Block8, ...]
```

**Series Key排序**

+ 索引项按series key字典序排列
+ 支持二分查找快速定位
+ 相同measurement的series聚集存储

## 索引优化技术
**布隆过滤器**

```go
type IndexBlock struct {
    SeriesBloomFilter *BloomFilter  // Series级别布隆过滤器
    FieldBloomFilter  *BloomFilter  // Field级别布隆过滤器
    Entries          []IndexEntry
}
```

+ 快速判断series或field是否存在
+ 减少不必要的磁盘IO
+ 假阳性可接受，假阴性为零

**时间范围索引**

+ 每个Block Entry记录时间范围
+ 查询时快速跳过不相关的数据块
+ 支持时间范围的快速过滤

## 查询处理流程
**Series定位**

1. 使用布隆过滤器预检查
2. 二分查找定位series索引项
3. 获取该series的所有field信息

**时间范围过滤**

```go
func (idx *Index) FindBlocks(seriesKey, fieldKey string, minTime, maxTime int64) []BlockEntry {
    var blocks []BlockEntry
    for _, entry := range idx.GetFieldBlocks(seriesKey, fieldKey) {
        if entry.MaxTime >= minTime && entry.MinTime <= maxTime {
            blocks = append(blocks, entry)
        }
    }
    return blocks
}
```

**数据块读取**

+ 根据Block Entry的offset和size信息
+ 直接定位到文件中的数据块位置
+ 避免扫描整个文件

## 内存缓存策略
**索引缓存**

+ 热点索引项缓存在内存中
+ LRU策略管理缓存空间
+ 减少重复的磁盘读取

**预读策略**

+ 连续查询时预读相邻的索引项
+ 利用磁盘顺序读取的优势
+ 提高范围查询性能

## 索引压缩
**Series Key压缩**

+ 使用前缀压缩减少存储空间
+ 相同measurement和tag的series共享前缀
+ Delta编码存储时间戳差值

**统计信息**

```go
type BlockStats struct {
    Count    int64    // 数据点数量
    Min      float64  // 最小值
    Max      float64  // 最大值
    Sum      float64  // 总和（用于聚合优化）
}
```

+ 存储每个数据块的统计信息
+ 支持聚合查询的下推优化
+ 避免读取实际数据进行简单聚合

## 索引维护
**增量更新**

+ 新数据写入时更新索引信息
+ 保持索引的有序性
+ 支持并发读取

**Compaction影响**

+ 文件合并时重建索引块
+ 合并多个索引项
+ 优化索引结构和大小

索引块的精心设计使得InfluxDB能够在处理大量时间序列数据时保持高效的查询性能，特别是在按时间范围和series过滤的典型时序查询场景中。

