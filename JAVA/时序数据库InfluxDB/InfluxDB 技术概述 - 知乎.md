## 基础概念
### [Measurement](https://zhida.zhihu.com/search?content_id=238735790&content_type=Article&match_order=1&q=Measurement&zhida_source=entity)
Measurement 形似传统数据库中的表，但本质上有很大的不同，比如 [InfluxDB](https://zhida.zhihu.com/search?content_id=238735790&content_type=Article&match_order=1&q=InfluxDB&zhida_source=entity) 并没有建表语句，也没有 schema。只是在写入的时候指定 measurement，用来标记一条数据的归属，所以说 Measurement 更像是一个普通的字符串类型字段。

### DataBase
InfluxDB 中的 database 更像是传统数据库中的表，database 是 schema 的载体，建数据库语法如下：

```plain
CREATE DATABASE <database_name> [WITH [DURATION <duration>] [REPLICATION <n>] [SHARD DURATION <duration>] [NAME <retention-policy-name>]]
```

通过 database 可以指定副本数，[shard group](https://zhida.zhihu.com/search?content_id=238735790&content_type=Article&match_order=1&q=shard+group&zhida_source=entity) 划分时间范围，保留策略等。所以说，database 用起来更像是传统数据库中的表。

### tag
tag 是一个 key-value 对，用来描述数据点的特征，比如一条主机的监控数据，region=beijing,ip=192.168.1.2 这两个 key-value 对都为 tag，其中 region 和 ip 称做 tag key，beijing 和 192.168.1.2 称为 tag value。tag 会建立倒排索引，因此，如果使用 tag 作为谓词，那么可以走索引提高查询性能。

### [field](https://zhida.zhihu.com/search?content_id=238735790&content_type=Article&match_order=1&q=field&zhida_source=entity)
同 tag 类似，field 也是一个 key-value 对，用来表示数据点中的指标名和值，比如 CPU 使用率监控 region=beijing,ip=192.168.1.2 cpu_uage=0.9，cpu_uage 称为 field key，0.9 为 field value。field 不会创建索引，因此，如果使用 field 作为谓词，那么必须 scan 数据。

### [point](https://zhida.zhihu.com/search?content_id=238735790&content_type=Article&match_order=1&q=point&zhida_source=entity)
在 InfluxDB 中，一个数据点相当于传统数据库的一行记录，**一个数据点由一个 measurement，多个 tag，多个 field 和一个时间戳构成**。注意，虽然 InfluxDB 行协议一条数据可以带多个 field，但是 InfluxDB 内部会将其拆分为多条数据。

### retention policy (RP)
时序数据往往不需要永久保留，随着时间的流逝，历史数据的价值也慢慢淡化，从存储成本和价值双方因素考虑，一般会淘汰某个时间点前的历史数据。RP 在 InfluxDB 中是个相当重要的概念，不仅通过它可以定义数据的保留周期，还能定义副本数、shard duration。

一个 database 可以创建多个 RP，如果未指定默认 RP，那么会使用 autogen 作为默认 RP，默认保留周期为无限大。数据写入时，可以指定 database 内的任一 RP。

### Series
时间线是 InfluxDB 中非常重要的概念，时间线相当于数据源，所有指标数据都是根据时间线产生和组织。在 InfluxDB 中时间线由 [series key](https://zhida.zhihu.com/search?content_id=238735790&content_type=Article&match_order=1&q=series+key&zhida_source=entity) 来表示，series key 由 measurement + tags 唯一确定。

时间线其实也是一种数据模型，比如一台主机可以由 series key: host,region=beijing,rack=r-1,ip=192.168.1.2 定义，这台主机产生的指标数据在逻辑上与该 series key 关联，在物理上由 series key 组织。

## 数据模型
我们最常见的关系模型使用一张二维表来描述数据，如下：

| Timestamp | region | rack | ip | cpu_usage | mem_usage |
| --- | --- | --- | --- | --- | --- |
| 2024-01-15 00:00:00 | Beijing | r-1 | 192.168.1.2 | 0.2 | 0.1 |
| 2024-01-15 00:00:10 | Beijing | r-1 | 192.168.1.2 | 0.5 | 0.3 |
| 2024-01-15 00:00:20 | Beijing | r-1 | 192.168.1.2 | 0.6 | 0.5 |


关系模型需要根据业务需求提前定义表结构的 schema，数据写入时，需严格按照 schema 定义的结构及类型写入，所有字段都需填充，不能有空洞。

InfluxDB 针对时序数据的特点，使用时间线模型组织数据，实质上也就是 key-value 结构，key 为 seriesKey+fieldKey，value 为 <timestamp, fieldValue> 集合。比如对于上表数据，使用时间线模型表示如下：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/jpeg/49518801/1746106013693-2ce4bbe6-d211-4b80-8af4-c71654925ca3.jpg.jpeg)

时间线模型

这种数据模型采用的 schemaless，可以随意增减 tag 和 field，相对关系模型更加灵活。数据按照 series key 聚簇，查询时，必须知道 series key 才能获取相应指标的值，这种模型的优缺点后文再讨论。

## 存储架构
由于 InfluxDB 并未开源分布式版本，这里介绍下单机版本的存储架构：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/jpeg/49518801/1746106013687-0c52526a-fa50-433f-9eca-ba3595ebdf9f.jpg.jpeg)

存储架构

从层次结构来看，database 可以包含多个 retention policy，数据写入时可以指定 RP，也就意味着相同measurement 的数据可以写到不同 RP 下。

RP 的 SHARD DURATION 决定了 shard group 覆盖的时间范围，因此 RP 下会产生多个 sg。

sg 包含 1 个及以上的 shard，数据到达 sg 后会通过 hash 路由到对应的 shard。当然，开源单机版 InfluxDB 中的sg 只有 1 个 shard。

shard 相当于数据承载的最小单元，每个 shard 有自己对应的 [wal](https://zhida.zhihu.com/search?content_id=238735790&content_type=Article&match_order=1&q=wal&zhida_source=entity)、[tsm](https://zhida.zhihu.com/search?content_id=238735790&content_type=Article&match_order=1&q=tsm&zhida_source=entity)、[tsi](https://zhida.zhihu.com/search?content_id=238735790&content_type=Article&match_order=1&q=tsi&zhida_source=entity)。wal 即预写日志，提高数据可靠性。tsm 存储的是时间线数据模型中的数据。tsi 存储的是倒排索引，这个后面详细介绍。

series 存储的是 series key 相关信息，同一个 database 的时间线相关信息都存在 series 中。

## 总结
本文整体概括了 InfluxDB 相关技术概念，后文将展开讨论相关技术细节。



> 来自: [InfluxDB 技术概述 - 知乎](https://zhuanlan.zhihu.com/p/677725021)
>





