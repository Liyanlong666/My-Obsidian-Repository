## In-memory indexing and the Time-Structured Merge Tree (TSM)  
内存索引和时间结构合并树（TSM）
This page documents an earlier version of InfluxDB OSS. [InfluxDB 3 Core](https://docs.influxdata.com/influxdb3/core/) is the latest stable version.  
本页记录了 InfluxDB OSS 的早期版本。 [InfluxDB 3 Core](https://docs.influxdata.com/influxdb3/core/) 是最新稳定版本。

## [The InfluxDB storage engine and the Time-Structured Merge Tree (TSM)InfluxDB 存储引擎和时间结构合并树 (TSM)](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#the-influxdb-storage-engine-and-the-time-structured-merge-tree-tsm)
The InfluxDB storage engine looks very similar to a LSM Tree. It has a write ahead log and a collection of read-only data files which are similar in concept to SSTables in an LSM Tree. TSM files contain sorted, compressed series data.  
InfluxDB 存储引擎看起来与 LSM 树非常相似。它有一个预写日志和一组只读数据文件，这些文件的概念类似于 LSM 树中的 SSTable。TSM 文件包含已排序且压缩的序列数据。

InfluxDB will create a [shard](https://docs.influxdata.com/influxdb/v1/concepts/glossary/#shard) for each block of time. For example, if you have a [retention policy](https://docs.influxdata.com/influxdb/v1/concepts/glossary/#retention-policy-rp) with an unlimited duration, shards will be created for each 7 day block of time. Each of these shards maps to an underlying storage engine database. Each of these databases has its own [WAL](https://docs.influxdata.com/influxdb/v1/concepts/glossary/#wal-write-ahead-log) and TSM files.  
InfluxDB 会为每个时间段创建一个[分片](https://docs.influxdata.com/influxdb/v1/concepts/glossary/#shard)。例如，如果您的[保留策略](https://docs.influxdata.com/influxdb/v1/concepts/glossary/#retention-policy-rp)为无限期，则每 7 天的时间段都会创建分片。每个分片都映射到一个底层存储引擎数据库。每个数据库都有自己的 [WAL 文件](https://docs.influxdata.com/influxdb/v1/concepts/glossary/#wal-write-ahead-log)和 TSM 文件。

We’ll dig into each of these parts of the storage engine.  
我们将深入研究存储引擎的每个部分。

## [Storage engine存储引擎](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#storage-engine)
The storage engine ties a number of components together and provides the external interface for storing and querying series data. It is composed of a number of components that each serve a particular role:  
存储引擎将多个组件绑定在一起，并提供用于存储和查询系列数据的外部接口。它由多个组件组成，每个组件都发挥着特定的作用：

+ In-Memory Index - The in-memory index is a shared index across shards that provides the quick access to [measurements](https://docs.influxdata.com/influxdb/v1/concepts/glossary/#measurement), [tags](https://docs.influxdata.com/influxdb/v1/concepts/glossary/#tag), and [series](https://docs.influxdata.com/influxdb/v1/concepts/glossary/#series). The index is used by the engine, but is not specific to the storage engine itself.  
**内存索引 - 内存索引是跨分片的共享索引，提供对**[**测量值**](https://docs.influxdata.com/influxdb/v1/concepts/glossary/#measurement)**、**[**标签**](https://docs.influxdata.com/influxdb/v1/concepts/glossary/#tag)**和**[**序列**](https://docs.influxdata.com/influxdb/v1/concepts/glossary/#series)**的快速访问。该索引由引擎使用，但并不特定于存储引擎本身。**
+ WAL - The WAL is a write-optimized storage format that allows for writes to be durable, but not easily queryable. Writes to the WAL are appended to segments of a fixed size.  
**WAL - WAL 是一种针对写入优化的存储格式，它允许写入持久化，但不易查询。对 WAL 的写入会被附加到固定大小的段中。**
+ Cache - The Cache is an in-memory representation of the data stored in the WAL. It is queried at runtime and merged with the data stored in TSM files.  
**缓存 - 缓存是 WAL 中存储数据的内存表示。它在运行时被查询，并与存储在 TSM 文件中的数据合并。**
+ TSM Files - TSM files store compressed series data in a columnar format.  
**TSM 文件 - TSM 文件以列式格式存储压缩系列数据。**
+ FileStore - The FileStore mediates access to all TSM files on disk. It ensures that TSM files are installed atomically when existing ones are replaced as well as removing TSM files that are no longer used.  
**FileStore - FileStore 负责协调对磁盘上所有 TSM 文件的访问。它确保在替换现有 TSM 文件时自动安装 TSM 文件，并删除不再使用的 TSM 文件。**
+ Compactor - The Compactor is responsible for converting less optimized Cache and TSM data into more read-optimized formats. It does this by compressing series, removing deleted data, optimizing indices and combining smaller files into larger ones.  
**压缩器 - 压缩器负责将优化程度较低的缓存和 TSM 数据转换为更适合读取的格式。它通过压缩序列、移除已删除的数据、优化索引以及将较小的文件合并为较大的文件来实现此目的。**
+ Compaction Planner - The Compaction Planner determines which TSM files are ready for a compaction and ensures that multiple concurrent compactions do not interfere with each other.  
**压缩规划器 - 压缩规划器确定哪些 TSM 文件已准备好进行压缩，并确保多个并发压缩不会相互干扰。**
+ Compression - Compression is handled by various Encoders and Decoders for specific data types. Some encoders are fairly static and always encode the same type the same way; others switch their compression strategy based on the shape of the data.  
**压缩 - 压缩由各种编码器和解码器针对特定数据类型进行处理。有些编码器相当静态，始终以相同的方式对同一类型进行编码；另一些编码器则根据数据的形状切换其压缩策略。**
+ Writers/Readers - Each file type (WAL segment, TSM files, tombstones, etc..) has Writers and Readers for working with the formats.  
**写入器/读取器——每种文件类型（WAL 段、TSM 文件、墓碑等）都有用于处理格式的写入器和读取器。**

### [Write Ahead Log (WAL)预写日志（WAL）](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#write-ahead-log-wal)
The WAL is organized as a bunch of files that look like `_000001.wal`. The file numbers are monotonically increasing and referred to as WAL segments. When a segment reaches 10MB in size, it is closed and a new one is opened. Each WAL segment stores multiple compressed blocks of writes and deletes.  
WAL 被组织成一堆类似 `_000001.wal` 的文件。这些文件编号单调递增，被称为 WAL 段。当一个段的大小达到 10MB 时，它会被关闭，并打开一个新的段。**<font style="background-color:#FBDE28;">每个 WAL 段存储多个压缩的写入和删除块。</font>**

<font style="background-color:#FBDE28;"></font>When a write comes in the new points are serialized, compressed using Snappy, and written to a WAL file. The file is `fsync`’d and the data is added to an in-memory index before a success is returned. This means that batching points together is required to achieve high throughput performance. (Optimal batch size seems to be 5,000-10,000 points per batch for many use cases.)  
当写入操作到来时，新的数据点会被序列化，使用 Snappy 压缩，并写入 WAL 文件。该文件会进行 `fsync` 操作，并将数据添加到内存索引中，然后返回成功结果。这意味着需要将数据点分批处理才能实现高吞吐量性能。（在许多用例中，最佳批处理大小似乎是每批 5,000-10,000 个数据点。）

Each entry in the WAL follows a [TLV standard](https://en.wikipedia.org/wiki/Type-length-value) with a single byte representing the type of entry (write or delete), a 4 byte `uint32` for the length of the compressed block, and then the compressed block.  
WAL 中的每个条目都遵循 [TLV 标准](https://en.wikipedia.org/wiki/Type-length-value)，其中单个字节表示条目的类型（写入或删除），4 字节 `uint32` 表示压缩块的长度，然后是压缩块。

### [Cache缓存](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#cache)
The Cache is an in-memory copy of all data points current stored in the WAL. The points are organized by the key, which is the measurement, [tag set](https://docs.influxdata.com/influxdb/v1/concepts/glossary/#tag-set), and unique [field](https://docs.influxdata.com/influxdb/v1/concepts/glossary/#field). Each field is kept as its own time-ordered range. The Cache data is not compressed while in memory.  
Cache 是 WAL 中当前所有数据点的内存副本。这些数据点按键（即测量值、[标签集](https://docs.influxdata.com/influxdb/v1/concepts/glossary/#tag-set)和唯一[字段）](https://docs.influxdata.com/influxdb/v1/concepts/glossary/#field) 进行组织。每个字段都按其自身的时间顺序排列。Cache 数据在内存中时不会被压缩。

Queries to the storage engine will merge data from the Cache with data from the TSM files. Queries execute on a copy of the data that is made from the cache at query processing time. This way writes that come in while a query is running won’t affect the result.  
对存储引擎的查询会将缓存中的数据与 TSM 文件中的数据合并。查询在查询处理时对缓存中的数据副本执行。这样，查询运行时的写入操作不会影响结果。

Deletes sent to the Cache will clear out the given key or the specific time range for the given key.  
发送到缓存的删除将清除给定的键或给定键的特定时间范围。

The Cache exposes a few controls for snapshotting behavior. The two most important controls are the memory limits. There is a lower bound, [cache-snapshot-memory-size](https://docs.influxdata.com/influxdb/v1/administration/config#cache-snapshot-memory-size), which when exceeded will trigger a snapshot to TSM files and remove the corresponding WAL segments. There is also an upper bound, [cache-max-memory-size](https://docs.influxdata.com/influxdb/v1/administration/config#cache-max-memory-size), which when exceeded will cause the Cache to reject new writes. These configurations are useful to prevent out of memory situations and to apply back pressure to clients writing data faster than the instance can persist it. The checks for memory thresholds occur on every write.  
Cache 公开了一些快照行为控制选项。其中最重要的两个控制选项是内存限制。有一个下限，即 [cache-snapshot-memory-size](https://docs.influxdata.com/influxdb/v1/administration/config#cache-snapshot-memory-size) ，超过该值将触发 TSM 文件快照并删除相应的 WAL 段。还有一个上限，即 [cache-max-memory-size](https://docs.influxdata.com/influxdb/v1/administration/config#cache-max-memory-size) ，超过该值将导致 Cache 拒绝新的写入。这些配置有助于防止内存不足的情况，并对写入速度超过实例持久化速度的客户端施加背压。每次写入时都会检查内存阈值。

The other snapshot controls are time based. The idle threshold, [cache-snapshot-write-cold-duration](https://docs.influxdata.com/influxdb/v1/administration/config#cache-snapshot-write-cold-duration), forces the Cache to snapshot to TSM files if it hasn’t received a write within the specified interval.  
其他快照控制基于时间。空闲阈值 [cache-snapshot-write-cold-duration](https://docs.influxdata.com/influxdb/v1/administration/config#cache-snapshot-write-cold-duration) 强制缓存在指定时间间隔内未收到写入操作时，将快照创建到 TSM 文件。

The in-memory Cache is recreated on restart by re-reading the WAL files on disk.  
通过重新读取磁盘上的 WAL 文件，在重新启动时重新创建内存缓存。

### [TSM filesTSM 文件](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#tsm-files)
TSM files are a collection of read-only files that are memory mapped. The structure of these files looks very similar to an SSTable in LevelDB or other LSM Tree variants.  
TSM 文件是内存映射的只读文件集合。这些文件的结构与 LevelDB 或其他 LSM Tree 变体中的 SSTable 非常相似。

A TSM file is composed of four sections: header, blocks, index, and footer.  
TSM 文件由四个部分组成：标题、块、索引和页脚。

```plain
+--------+------------------------------------+-------------+--------------+
| Header |               Blocks               |    Index    |    Footer    |
|5 bytes |              N bytes               |   N bytes   |   4 bytes    |
+--------+------------------------------------+-------------+--------------+
```

The Header is a magic number to identify the file type and a version number.  
标头是一个用于识别文件类型和版本号的神奇数字。

```plain
+-------------------+
|      Header       |
+-------------------+
|  Magic  │ Version |
| 4 bytes │ 1 byte  |
+-------------------+
```

Blocks are sequences of pairs of CRC32 checksums and data. The block data is opaque to the file. The CRC32 is used for block level error detection. The length of the blocks is stored in the index.  
块是 CRC32 校验和与数据对的序列。块数据对文件不透明。CRC32 用于块级错误检测。块的长度存储在索引中。

```plain
+--------------------------------------------------------------------+
│                           Blocks                                   │
+---------------------+-----------------------+----------------------+
|       Block 1       |        Block 2        |       Block N        |
+---------------------+-----------------------+----------------------+
|   CRC    |  Data    |    CRC    |   Data    |   CRC    |   Data    |
| 4 bytes  | N bytes  |  4 bytes  | N bytes   | 4 bytes  |  N bytes  |
+---------------------+-----------------------+----------------------+
```

+ Copy
+  Fill window

  
块之后是文件中块的索引。索引由按键的字典顺序排列然后按时间排序的索引条目序列组成。键包括<font style="background-color:#FBDE28;">测量名称、标签集和一个字段</font>。每个点的多个字段在 TSM 文件中创建多个索引条目。  
**每个索引条目都以键长度和键开头，后跟块类型（浮点型、整型、布尔型、字符串）以及该键后面的索引块条目数。每个索引块条目由块的最小时间和最大时间、块所在文件的偏移量以及块的大小组成。TSM 文件中每个包含键的块都有一个索引块条目。**

  
索引结构可以高效地访问所有数据块，并能够确定访问给定键的相关成本。给定一个键和时间戳，我们可以确定文件是否包含该时间戳对应的数据块。我们还可以确定该数据块的位置以及检索该数据块需要读取多少数据。了解数据块的大小后，我们就可以高效地配置 IO 语句。

```plain
+-----------------------------------------------------------------------------+
│                                   Index                                     │
+-----------------------------------------------------------------------------+
│ Key Len │   Key   │ Type │ Count │Min Time │Max Time │ Offset │  Size  │...│
│ 2 bytes │ N bytes │1 byte│2 bytes│ 8 bytes │ 8 bytes │8 bytes │4 bytes │   │
+-----------------------------------------------------------------------------+
```

+ Copy
+  Fill window

The last section is the footer that stores the offset of the start of the index.  
最后一部分是页脚，用于存储索引开始的偏移量。

```plain
+---------+
│ Footer  │
+---------+
│Index Ofs│
│ 8 bytes │
+---------+
```

+ Copy
+  Fill window

### [Compression压缩](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#compression)
Each block is compressed to reduce storage space and disk IO when querying. A block contains the timestamps and values for a given series and field. Each block has one byte header, followed by the compressed timestamps and then the compressed values.  
每个块都经过压缩，以减少查询时的存储空间和磁盘 IO。每个块包含给定系列和字段的时间戳和值。每个块都有一个字节的头，后面是压缩后的时间戳，然后是压缩后的值。

```plain
+--------------------------------------------------+
| Type  |  Len  |   Timestamps    |      Values    |
|1 Byte | VByte |     N Bytes     |    N Bytes     │
+--------------------------------------------------+
```

+ Copy
+  Fill window

The timestamps and values are compressed and stored separately using encodings dependent on the data type and its shape. Storing them independently allows timestamp encoding to be used for all timestamps, while allowing different encodings for different field types. For example, some points may be able to use run-length encoding whereas other may not.  
时间戳和值会被分别压缩和存储，并使用依赖于数据类型及其形状的编码。独立存储允许所有时间戳使用相同的编码，同时允许不同字段类型使用不同的编码。例如，某些点可能可以使用游程编码，而其他点则不能。

Each value type also contains a 1 byte header indicating the type of compression for the remaining bytes. The four high bits store the compression type and the four low bits are used by the encoder if needed.  
每种值类型还包含一个 1 字节的标头，指示剩余字节的压缩类型。高四位存储压缩类型，低四位供编码器在需要时使用。

#### [Timestamps时间戳](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#timestamps)
Timestamp encoding is adaptive and based on the structure of the timestamps that are encoded. It uses a combination of delta encoding, scaling, and compression using simple8b run-length encoding, as well as falling back to no compression if needed.  
**时间戳编码是自适应的，基于被编码时间戳的结构。它结合了增量编码、缩放和使用 simple8b 游程编码的压缩，并且可以根据需要回退到无压缩。**

Timestamp resolution is variable but can be as granular as a nanosecond, requiring up to 8 bytes to store uncompressed. During encoding, the values are first delta-encoded. The first value is the starting timestamp and subsequent values are the differences from the prior value. This usually converts the values into much smaller integers that are easier to compress. Many timestamps are also monotonically increasing and fall on even boundaries of time such as every 10s. When timestamps have this structure, they are scaled by the largest common divisor that is also a factor of 10. This has the effect of converting very large integer deltas into smaller ones that compress even better.  
时间戳的分辨率是可变的，但可以精确到纳秒，需要最多 8 个字节来存储未压缩的数据。在编码过程中，首先对值进行增量编码。第一个值是起始时间戳，后续值是与前一个值的差值。这通常会将值转换为更小的整数，从而更容易压缩。许多时间戳也是单调递增的，并且落在偶数时间边界上，例如每 10 秒一次。当时间戳具有这种结构时，它们会按最大公约数（也是 10 的倍数）进行缩放。这可以将非常大的整数增量转换为更小的增量，从而获得更好的压缩效果。

Using these adjusted values, if all the deltas are the same, the time range is stored using run-length encoding. If run-length encoding is not possible and all values are less than (1 « 60) - 1 ([~36.5 years](https://www.wolframalpha.com/input/?i=(1+%3C%3C+60)+-+1+nanoseconds+to+years) at nanosecond resolution), then the timestamps are encoded using [simple8b encoding](https://github.com/jwilder/encoding/tree/master/simple8b). Simple8b encoding is a 64bit word-aligned integer encoding that packs multiple integers into a single 64bit word. If any value exceeds the maximum the deltas are stored uncompressed using 8 bytes each for the block. Future encodings may use a patched scheme such as Patched Frame-Of-Reference (PFOR) to handle outliers more effectively.  
使用这些调整后的值，如果所有增量均相同，则使用游程编码存储时间范围。如果无法进行游程编码，且所有值均小于 (1 « 60) - 1（纳秒分辨率下[约为 36.5 年](https://www.wolframalpha.com/input/?i=(1+%3C%3C+60)+-+1+nanoseconds+to+years)），则使用 [simple8b 编码](https://github.com/jwilder/encoding/tree/master/simple8b)对时间戳进行编码。Simple8b 编码是一种 64 位字对齐的整数编码，它将多个整数打包成一个 64 位字。如果任何值超过最大值，则增量将以未压缩的形式存储，每个块使用 8 个字节。未来的编码可能会使用诸如补丁参考帧 (PFOR) 之类的补丁方案来更有效地处理异常值。

#### [Floats浮点数](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#floats)
Floats are encoded using an implementation of the [Facebook Gorilla paper](http://www.vldb.org/pvldb/vol8/p1816-teller.pdf). The encoding XORs consecutive values together to produce a small result when the values are close together. The delta is then stored using control bits to indicate how many leading and trailing zeroes are in the XOR value. Our implementation removes the timestamp encoding described in paper and only encodes the float values.  
浮点数采用 [Facebook Gorilla 论文](http://www.vldb.org/pvldb/vol8/p1816-teller.pdf)中的实现进行编码。该编码将连续的值进行异或运算，当这些值接近时，会产生较小的结果。然后，使用控制位存储差值，以指示异或值中有多少个前导零和尾随零。我们的实现删除了论文中描述的时间戳编码，仅对浮点值进行编码。

#### [Integers整数](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#integers)
Integer encoding uses two different strategies depending on the range of values in the uncompressed data. Encoded values are first encoded using [ZigZag encoding](https://developers.google.com/protocol-buffers/docs/encoding#signed-integers). This interleaves positive and negative integers across a range of positive integers.  
整数编码根据未压缩数据中值的范围使用两种不同的策略。编码值首先使用 [ZigZag 编码](https://developers.google.com/protocol-buffers/docs/encoding#signed-integers)进行编码。这会将正整数和负整数交织在正整数范围内。

For example, [-2,-1,0,1] becomes [3,1,0,2]. See Google’s [Protocol Buffers documentation](https://developers.google.com/protocol-buffers/docs/encoding#signed-integers) for more information.  
例如，[-2,-1,0,1] 变为 [3,1,0,2]。有关更多信息，请参阅 Google 的 [Protocol Buffers 文档](https://developers.google.com/protocol-buffers/docs/encoding#signed-integers)。

If all ZigZag encoded values are less than (1 « 60) - 1, they are compressed using simple8b encoding. If any values are larger than the maximum then all values are stored uncompressed in the block. If all values are identical, run-length encoding is used. This works very well for values that are frequently constant.  
如果所有 ZigZag 编码值均小于 (1 « 60) - 1，则使用 simple8b 编码进行压缩。如果任何值大于最大值，则所有值均以未压缩形式存储在块中。如果所有值均相同，则使用游程编码。这种方法对于经常为常数的值非常有效。

#### [Booleans布尔值](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#booleans)
Booleans are encoded using a simple bit packing strategy where each Boolean uses 1 bit. The number of Booleans encoded is stored using variable-byte encoding at the beginning of the block.  
布尔值采用简单的位打包策略进行编码，每个布尔值占用 1 位。已编码的布尔值数量以可变字节编码存储在区块的开头。

#### [Strings字符串](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#strings)
Strings are encoding using [Snappy](http://google.github.io/snappy/) compression. Each string is packed consecutively and they are compressed as one larger block.  
字符串使用 [Snappy](http://google.github.io/snappy/) 压缩进行编码。每个字符串被连续打包，并压缩为一个更大的块。

### [Compactions压缩](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#compactions)
Compactions are recurring processes that migrate data stored in a write-optimized format into a more read-optimized format. There are a number of stages of compaction that take place while a shard is hot for writes:  
压缩是将以写入优化格式存储的数据迁移到读取优化格式的重复过程。当分片处于写入热状态时，压缩会执行多个阶段：

+ **Snapshots** - Values in the Cache and WAL must be converted to TSM files to free memory and disk space used by the WAL segments. These compactions occur based on the cache memory and time thresholds.  
**快照 **- 必须将缓存和 WAL 中的值转换为 TSM 文件，以释放 WAL 段占用的内存和磁盘空间。这些压缩操作基于缓存内存和时间阈值进行。
+ **Level Compactions** - Level compactions (levels 1-4) occur as the TSM files grow. TSM files are compacted from snapshots to level 1 files. Multiple level 1 files are compacted to produce level 2 files. The process continues until files reach level 4 (full compaction) and the max size for a TSM file. They will not be compacted further unless deletes, index optimization compactions, or full compactions need to run. Lower level compactions use strategies that avoid CPU-intensive activities like decompressing and combining blocks. Higher level (and thus less frequent) compactions will re-combine blocks to fully compact them and increase the compression ratio.  
**级别压缩 **- 随着 TSM 文件的增长，级别压缩（1-4 级）会进行。TSM 文件会从快照压缩为 1 级文件。多个 1 级文件会被压缩生成 2 级文件。该过程持续进行，直到文件达到 4 级（完全压缩）并达到 TSM 文件的最大大小。除非需要执行删除、索引优化压缩或完全压缩，否则不会进一步压缩。较低级别的压缩会使用避免 CPU 密集型操作（例如解压缩和合并块）的策略。较高级别（因此频率较低）的压缩会重新合并块以完全压缩它们，从而提高压缩率。
+ **Index Optimization** - When many level 4 TSM files accumulate, the internal indexes become larger and more costly to access. An index optimization compaction splits the series and indices across a new set of TSM files, sorting all points for a given series into one TSM file. Before an index optimization, each TSM file contained points for most or all series, and thus each contains the same series index. After an index optimization, each TSM file contains points from a minimum of series and there is little series overlap between files. Each TSM file thus has a smaller unique series index, instead of a duplicate of the full series list. In addition, all points from a particular series are contiguous in a TSM file rather than spread across multiple TSM files.  
**索引优化 **- 当许多 4 级 TSM 文件累积起来时，内部索引会变得更大，访问成本也会更高。索引优化压缩会将系列和索引拆分到一组新的 TSM 文件中，并将给定系列的所有数据点排序到一个 TSM 文件中。索引优化之前，每个 TSM 文件包含大多数或所有系列的数据点，因此每个 TSM 文件包含相同的系列索引。索引优化之后，每个 TSM 文件包含来自最少系列的数据点，并且文件之间的系列重叠很小。因此，每个 TSM 文件都有一个较小的唯一系列索引，而不是完整系列列表的副本。此外，来自特定系列的所有数据点在 TSM 文件中是连续的，而不是分散在多个 TSM 文件中。
+ **Full Compactions** - Full compactions (level 4 compactions) run when a shard has become cold for writes for long time, or when deletes have occurred on the shard. Full compactions produce an optimal set of TSM files and include all optimizations from Level and Index Optimization compactions. Once a shard is fully compacted, no other compactions will run on it unless new writes or deletes are stored.  
**完全压缩 **- 当分片长时间处于写入冷状态，或分片上发生删除操作时，将运行完全压缩（级别 4 压缩）。完全压缩会生成一组优化的 TSM 文件，并包含级别优化和索引优化压缩的所有优化。分片完全压缩后，除非有新的写入或删除操作被存储，否则不会再对其进行其他压缩。

### [Writes写入](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#writes)
Writes are appended to the current WAL segment and are also added to the Cache. Each WAL segment has a maximum size. Writes roll over to a new file once the current file fills up. The cache is also size bounded; snapshots are taken and WAL compactions are initiated when the cache becomes too full. If the inbound write rate exceeds the WAL compaction rate for a sustained period, the cache may become too full, in which case new writes will fail until the snapshot process catches up.  
写入操作会附加到当前 WAL 段，同时也会添加到缓存中。每个 WAL 段都有最大大小限制。当前文件写满后，写入操作会转移到新文件。缓存也有大小限制；当缓存过满时，会创建快照并启动 WAL 压缩。如果入站写入速率持续超过 WAL 压缩速率，缓存可能会过满，在这种情况下，新的写入操作将失败，直到快照进程赶上来。

When WAL segments fill up and are closed, the Compactor snapshots the Cache and writes the data to a new TSM file. When the TSM file is successfully written and `fsync`’d, it is loaded and referenced by the FileStore.  
当 WAL 段写满并关闭时，Compactor 会对 Cache 进行快照，并将数据写入新的 TSM 文件。TSM 文件写入成功并 `fsync` 后，会被 FileStore 加载并引用。

### [Updates更新](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#updates)
Updates (writing a newer value for a point that already exists) occur as normal writes. Since cached values overwrite existing values, newer writes take precedence. If a write would overwrite a point in a prior TSM file, the points are merged at query runtime and the newer write takes precedence.  
更新（为已存在的点写入较新的值）与正常写入操作相同。由于缓存值会覆盖现有值，因此较新的写入操作优先。如果写入操作会覆盖先前 TSM 文件中的点，则这些点会在查询运行时合并，并以较新的写入操作优先。

### [Deletes删除](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#deletes)
Deletes occur by writing a delete entry to the WAL for the measurement or series and then updating the Cache and FileStore. The Cache evicts all relevant entries. The FileStore writes a tombstone file for each TSM file that contains relevant data. These tombstone files are used at startup time to ignore blocks as well as during compactions to remove deleted entries.  
删除操作通过将删除条目写入测量或系列的 WAL 来实现，然后更新缓存和文件存储。缓存会逐出所有相关条目。文件存储会为每个包含相关数据的 TSM 文件写入一个墓碑文件。这些墓碑文件在启动时用于忽略块，并在压缩期间用于删除已删除的条目。

Queries against partially deleted series are handled at query time until a compaction removes the data fully from the TSM files.  
针对部分删除系列的查询在查询时处理，直到压缩将数据从 TSM 文件完全删除。

### [Queries查询](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#queries)
When a query is executed by the storage engine, it is essentially a seek to a given time associated with a specific series key and field. First, we do a search on the data files to find the files that contain a time range matching the query as well containing matching series.  
当存储引擎执行查询时，它本质上是查找与特定系列键和字段相关联的给定时间。首先，我们对数据文件进行搜索，以找到包含与查询匹配的时间范围以及匹配系列的文件。

Once we have the data files selected, we next need to find the position in the file of the series key index entries. We run a binary search against each TSM index to find the location of its index blocks.  
选定数据文件后，接下来需要找到序列键索引项在文件中的位置。我们对每个 TSM 索引进行二分查找，以找到其索引块的位置。

In common cases the blocks will not overlap across multiple TSM files and we can search the index entries linearly to find the start block from which to read. If there are overlapping blocks of time, the index entries are sorted to ensure newer writes will take precedence and that blocks can be processed in order during query execution.  
通常情况下，多个 TSM 文件中的数据块不会重叠，我们可以线性搜索索引条目，找到要读取的起始块。如果存在重叠的时间块，则对索引条目进行排序，以确保较新的写入优先，并且在查询执行期间可以按顺序处理数据块。

When iterating over the index entries the blocks are read sequentially from the blocks section. The block is decompressed and we seek to the specific point.  
当迭代索引条目时，会从块部分按顺序读取块。块被解压缩后，我们定位到特定的点。

##   
新的 InfluxDB 存储引擎：从 LSM 树到 B+ 树再返回以创建时间结构化合并树 The new InfluxDB storage engine: from LSM Tree to B+Tree and back again to create the Time Structured Merge Tree
Writing a new storage format should be a last resort. So how did InfluxData end up writing our own engine? InfluxData has experimented with many storage formats and found each lacking in some fundamental way. The performance requirements for InfluxDB are significant, and eventually overwhelm other storage systems. The 0.8 line of InfluxDB allowed multiple storage engines, including LevelDB, RocksDB, HyperLevelDB, and LMDB. The 0.9 line of InfluxDB used BoltDB as the underlying storage engine. This writeup is about the Time Structured Merge Tree storage engine that was released in 0.9.5 and is the only storage engine supported in InfluxDB 0.11+, including the entire 1.x family.  
编写新的存储格式应该是不得已而为之。那么，InfluxData 最终是如何决定自己开发引擎的呢？InfluxData 尝试过多种存储格式，发现每种格式都存在一些根本性的缺陷。InfluxDB 的性能要求非常高，最终会压垮其他存储系统。InfluxDB 0.8 版本支持多种存储引擎，包括 LevelDB、RocksDB、HyperLevelDB 和 LMDB。InfluxDB 0.9 版本使用 BoltDB 作为底层存储引擎。本文主要介绍 0.9.5 版本中发布的 Time Structured Merge Tree 存储引擎，它是 InfluxDB 0.11+ 版本（包括整个 1.x 系列）唯一支持的存储引擎。

The properties of the time series data use case make it challenging for many existing storage engines. Over the course of InfluxDB development, InfluxData tried a few of the more popular options. We started with LevelDB, an engine based on LSM Trees, which are optimized for write throughput. After that we tried BoltDB, an engine based on a memory mapped B+Tree, which is optimized for reads. Finally, we ended up building our own storage engine that is similar in many ways to LSM Trees.  
时间序列数据用例的特性对许多现有的存储引擎来说都极具挑战性。在 InfluxDB 的开发过程中，InfluxData 尝试了一些较为流行的方案。我们首先尝试了 LevelDB，这是一个基于 LSM 树的引擎，针对写入吞吐量进行了优化。之后，我们尝试了 BoltDB，这是一个基于内存映射 B+树的引擎，针对读取进行了优化。最终，我们构建了自己的存储引擎，它在很多方面与 LSM 树类似。

With our new storage engine we were able to achieve up to a 45x reduction in disk space usage from our B+Tree setup with even greater write throughput and compression than what we saw with LevelDB and its variants. This post will cover the details of that evolution and end with an in-depth look at our new storage engine and its inner workings.  
使用新的存储引擎，我们能够将 B+Tree 架构的磁盘空间占用量减少高达 45 倍，并且写入吞吐量和压缩率甚至比 LevelDB 及其变体更高。本文将详细介绍这一演进过程，并深入探讨我们的新存储引擎及其内部工作原理。

## [时间序列数据的属性](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#properties-of-time-series-data)
The workload of time series data is quite different from normal database workloads. There are a number of factors that conspire to make it very difficult to scale and remain performant:  
时间序列数据的工作负载与普通数据库工作负载截然不同。有许多因素共同导致其难以扩展并保持高性能：

+ Billions of individual data points  
数十亿个单独的数据点
+ High write throughput  高写入吞吐量
+ High read throughput  高读取吞吐量
+ Large deletes (data expiration)  
大量删除（数据过期）
+ Mostly an insert/append workload, very few updates  
主要是插入/附加工作负载，很少更新

The first and most obvious problem is one of scale. In DevOps, IoT, or APM it is easy to collect hundreds of millions or billions of unique data points every day.  
第一个也是最明显的问题是规模问题。在 DevOps、IoT 或 APM 中，每天很容易收集数亿甚至数十亿个唯一数据点。

For example, let’s say we have 200 VMs or servers running, with each server collecting an average of 100 measurements every 10 seconds. Given there are 86,400 seconds in a day, a single measurement will generate 8,640 points in a day per server. That gives us a total of 172,800,000 (`200 * 100 * 8,640`) individual data points per day. We find similar or larger numbers in sensor data use cases.  
例如，假设我们有 200 台虚拟机或服务器正在运行，每台服务器平均每 10 秒收集 100 个测量数据。假设一天有 86,400 秒，那么每台服务器每天单次测量将生成 8,640 个数据点。这样一来，我们每天总共会产生 172,800,000 个（ `200 * 100 * 8,640` ）个单独的数据点。在传感器数据用例中，我们发现的数据点数量与此类似或更大。

The volume of data means that the write throughput can be very high. We regularly get requests for setups than can handle hundreds of thousands of writes per second. Some larger companies will only consider systems that can handle millions of writes per second.  
数据量巨大意味着写入吞吐量可能非常高。我们<font style="background-color:#FBDE28;">经常收到要求设置每秒可处理数十万次写入的系统的请求</font>。一些大型公司只会考虑每秒可处理数百万次写入的系统。

At the same time, time series data can be a high read throughput use case. It’s true that if you’re tracking 700,000 unique metrics or time series you can’t hope to visualize all of them. That leads many people to think that you don’t actually read most of the data that goes into the database. However, other than dashboards that people have up on their screens, there are automated systems for monitoring or combining the large volume of time series data with other types of data.  
同时，时间序列数据可以成为高读取吞吐量的用例。诚然，如果您要跟踪 70 万个唯一指标或时间序列，您不可能指望将它们全部可视化。这导致许多人认为您实际上并没有读取进入数据库的大部分数据。然而，除了人们屏幕上显示的仪表板之外，还有一些自动化系统可以监控或将大量时间序列数据与其他类型的数据合并。

Inside InfluxDB, aggregate functions calculated on the fly may combine tens of thousands of distinct time series into a single view. Each one of those queries must read each aggregated data point, so for InfluxDB the read throughput is often many times higher than the write throughput.  
在 InfluxDB 内部，动态计算的聚合函数可能会将数万个不同的时间序列组合成一个视图。每个查询都必须读取每个聚合数据点，因此对于 InfluxDB 来说，读取吞吐量通常比写入吞吐量高出许多倍。

Given that time series is mostly an append-only workload, you might think that it’s possible to get great performance on a B+Tree. Appends in the keyspace are efficient and you can achieve greater than 100,000 per second. However, we have those appends happening in individual time series. So the inserts end up looking more like random inserts than append only inserts.  
鉴于时间序列主要属于仅追加型工作负载，您可能认为在 B+Tree 上可以获得出色的性能。键空间中的追加操作非常高效，每秒可以达到 100,000 次以上。然而，这些追加操作是分阶段进行的，因此最终的插入操作看起来更像是随机插入，而不是仅追加插入。

One of the biggest problems we found with time series data is that it’s very common to delete all data after it gets past a certain age. The common pattern here is that users have high precision data that is kept for a short period of time like a few days or months. Users then downsample and aggregate that data into lower precision rollups that are kept around much longer.  
我们发现时间序列数据最大的问题之一是，数据超过一定时间后，通常会被全部删除。这种情况的常见模式是，用户拥有高精度数据，但只保存几天或几个月。然后，用户会将这些数据降采样并聚合成精度较低的汇总数据，并保存更长时间。

The naive implementation would be to simply delete each record once it passes its expiration time. However, that means that once the first points written reach their expiration date, the system is processing just as many deletes as writes, which is something most storage engines aren’t designed for.  
最简单的实现方式是，一旦记录超过其过期时间，就直接删除。然而，这意味着，一旦第一个写入的数据点到达其过期日期，系统处理的删除操作和写入操作的数量就一样多，而大多数存储引擎的设计并非如此。

Let’s dig into the details of the two types of storage engines we tried and how these properties had a significant impact on our performance.  
让我们深入了解我们尝试过的两种存储引擎的细节，以及这些属性如何对我们的性能产生重大影响。

## [LevelDB 和日志结构合并树  (LevelDB and log structured merge trees) ](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#leveldb-and-log-structured-merge-trees)
When the InfluxDB project began, we picked LevelDB as the storage engine because we had used it for time series data storage in the product that was the precursor to InfluxDB. We knew that it had great properties for write throughput and everything seemed to “just work”.  
InfluxDB 项目启动时，我们选择 LevelDB 作为存储引擎，因为我们在 InfluxDB 的前身产品中曾使用它来存储时间序列数据。我们知道它具有出色的写入吞吐量，一切看起来都“运行良好”。

LevelDB is an implementation of a log structured merge tree (LSM tree) that was built as an open source project at Google. It exposes an API for a key-value store where the key space is sorted. This last part is important for time series data as it allowed us to quickly scan ranges of time as long as the timestamp was in the key.  
LevelDB 是 Google 开源项目“日志结构化合并树”（LSM 树）的实现。它公开了一个键值存储 API，其中键空间是经过排序的。这部分对于时间序列数据非常重要，因为它允许我们快速扫描时间范围，只要时间戳包含在键中即可。

LSM Trees are based on a log that takes writes and two structures known as Mem Tables and SSTables. These tables represent the sorted keyspace. SSTables are read only files that are continuously replaced by other SSTables that merge inserts and updates into the keyspace.  
LSM 树基于一个用于写入的日志，以及两个称为内存表 (Mem Table) 和 SSTable 的结构。这些表表示已排序的键空间。SSTable 是只读文件，它们会不断被其他 SSTable 替换，这些 SSTable 将插入和更新合并到键空间中。

The two biggest advantages that LevelDB had for us were high write throughput and built in compression. However, as we learned more about what people needed with time series data, we encountered a few insurmountable challenges.  
LevelDB 对我们来说最大的两个优势是高写入吞吐量和内置压缩功能。然而，随着我们越来越了解人们对时间序列数据的需求，我们遇到了一些难以克服的挑战。

The first problem we had was that LevelDB doesn’t support hot backups. If you want to do a safe backup of the database, you have to close it and then copy it. The LevelDB variants RocksDB and HyperLevelDB fix this problem, but there was another more pressing problem that we didn’t think they could solve.  
我们遇到的第一个问题是 LevelDB 不支持热备份。如果要对数据库进行安全备份，必须先关闭数据库，然后再进行复制。LevelDB 的变体 RocksDB 和 HyperLevelDB 解决了这个问题，但还有一个更紧迫的问题，我们认为它们无法解决。

Our users needed a way to automatically manage data retention. That meant we needed deletes on a very large scale. In LSM Trees, a delete is as expensive, if not more so, than a write. A delete writes a new record known as a tombstone. After that queries merge the result set with any tombstones to purge the deleted data from the query return. Later, a compaction runs that removes the tombstone record and the underlying deleted record in the SSTable file.  
我们的用户需要一种自动管理数据保留的方法。这意味着我们需要大规模的删除操作。在 LSM 树中，删除操作的开销与写入操作一样高，甚至更高。删除操作会写入一条新记录，称为“墓碑”。之后，查询会将结果集与所有“墓碑”记录合并，以从查询返回的数据中清除已删除的数据。之后，会运行压缩操作，从 SSTable 文件中移除“墓碑”记录和底层已删除的记录。

To get around doing deletes, we split data across what we call shards, which are contiguous blocks of time. Shards would typically hold either one day or seven days worth of data. Each shard mapped to an underlying LevelDB. This meant that we could drop an entire day of data by just closing out the database and removing the underlying files.  
为了避免删除操作，我们将数据拆分到所谓的“分片”中，这些分片是连续的时间段。分片通常保存一天或七天的数据。每个分片都映射到底层的 LevelDB。这意味着我们只需关闭数据库并删除底层文件，就可以删除一整天的数据。

Users of RocksDB may at this point bring up a feature called ColumnFamilies. When putting time series data into Rocks, it’s common to split blocks of time into column families and then drop those when their time is up. It’s the same general idea: create a separate area where you can just drop files instead of updating indexes when you delete a large block of data. Dropping a column family is a very efficient operation. However, column families are a fairly new feature and we had another use case for shards.  
RocksDB 的用户此时可能会想到一个叫做 ColumnFamilies 的功能。将时间序列数据放入 Rocks 时，通常会将时间块拆分成列族，然后在时间到时删除这些列族。这和一般思路是一样的：创建一个单独的区域，删除大量数据时只需删除文件即可，而无需更新索引。删除列族是一个非常高效的操作。然而，列族是一个相当新的功能，我们还有另一个分片用例。

Organizing data into shards meant that it could be moved within a cluster without having to examine billions of keys. At the time of this writing, it was not possible to move a column family in one RocksDB to another. Old shards are typically cold for writes so moving them around would be cheap and easy. We would have the added benefit of having a spot in the keyspace that is cold for writes so it would be easier to do consistency checks later.  
将数据组织到分片中意味着它可以在集群内移动，而无需检查数十亿个键。在撰写本文时，无法将一个 RocksDB 中的列族移动到另一个。旧分片通常对于写入来说是冷的，因此移动它们会既便宜又简单。我们还有一个额外的好处，那就是在键空间中有一个对于写入来说是冷的位置，这样以后进行一致性检查会更容易。

The organization of data into shards worked great for a while, until a large amount of data went into InfluxDB. LevelDB splits the data out over many small files. Having dozens or hundreds of these databases open in a single process ended up creating a big problem. Users that had six months or a year of data would run out of file handles. It’s not something we found with the majority of users, but anyone pushing the database to its limits would hit this problem and we had no fix for it. There were simply too many file handles open.  
将数据组织到分片中一度效果很好，直到大量数据涌入 InfluxDB。LevelDB 将数据拆分到许多小文件中。在单个进程中打开数十或数百个这样的数据库最终会造成大问题。拥有六个月或一年数据的用户会耗尽文件句柄。我们发现大多数用户不会遇到这种情况，但任何将数据库推向极限的用户都会遇到这个问题，而我们对此束手无策。原因很简单，打开的文件句柄太多了。

## [BoltDB and mmap B+Trees BoltDB 和 mmap B+Trees](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/#boltdb-and-mmap-btrees)
After struggling with LevelDB and its variants for a year we decided to move over to BoltDB, a pure Golang database heavily inspired by LMDB, a mmap B+Tree database written in C. It has the same API semantics as LevelDB: a key value store where the keyspace is ordered. Many of our users were surprised. Our own posted tests of the LevelDB variants vs. LMDB (a mmap B+Tree) showed RocksDB as the best performer.  
在与 LevelDB 及其变体斗争了一年之后，我们决定迁移到 BoltDB，这是一个纯 Golang 数据库，其灵感源自 LMDB（一个用 C 语言编写的 mmap B+Tree 数据库）。它拥有与 LevelDB 相同的 API 语义：一个键值存储，其中键空间是有序的。许多用户对此感到惊讶。我们自己发布的 LevelDB 变体与 LMDB（一个 mmap B+Tree 数据库）的测试结果显示，RocksDB 的性能最佳。

However, there were other considerations that went into this decision outside of the pure write performance. At this point our most important goal was to get to something stable that could be run in production and backed up. BoltDB also had the advantage of being written in pure Go, which simplified our build chain immensely and made it easy to build for other OSes and platforms.  
然而，除了纯粹的写入性能之外，我们还有其他考虑因素促成了这一决定。目前，我们最重要的目标是开发出一个能够在生产环境中运行并备份的稳定版本。BoltDB 还具有纯 Go 编写的优势，这极大地简化了我们的构建链，并使其易于为其他操作系统和平台构建。

The biggest win for us was that BoltDB used a single file as the database. At this point our most common source of bug reports were from people running out of file handles. Bolt solved the hot backup problem and the file limit problems all at the same time.  
对我们来说，最大的优势在于 BoltDB 使用单个文件作为数据库。目前，我们最常见的错误报告来源是文件句柄用完的问题。Bolt 同时解决了热备份问题和文件限制问题。

We were willing to take a hit on write throughput if it meant that we’d have a system that was more reliable and stable that we could build on. Our reasoning was that for anyone pushing really big write loads, they’d be running a cluster anyway.  
如果这意味着我们能拥有一个更可靠、更稳定的系统，我们愿意承受写入吞吐量的损失。我们的理由是，对于任何需要处理大量写入负载的人来说，他们无论如何都会运行集群。

We released versions 0.9.0 to 0.9.2 based on BoltDB. From a development perspective it was delightful. Clean API, fast and easy to build in our Go project, and reliable. However, after running for a while we found a big problem with write throughput. After the database got over a few GB, writes would start spiking IOPS.  
我们发布了基于 BoltDB 的 0.9.0 到 0.9.2 版本。从开发角度来看，这令人欣喜。API 简洁，在我们的 Go 项目中构建快速便捷，而且可靠。然而，运行一段时间后，我们发现写入吞吐量存在很大问题。当数据库超过几 GB 时，写入操作就会开始飙升 IOPS。

Some users were able to get past this by putting InfluxDB on big hardware with near unlimited IOPS. However, most users are on VMs with limited resources in the cloud. We had to figure out a way to reduce the impact of writing a bunch of points into hundreds of thousands of series at a time.  
一些用户可以通过将 InfluxDB 部署到拥有近乎无限 IOPS 的大型硬件上来解决这个问题。然而，大多数用户使用的是云端资源有限的虚拟机。我们必须找到一种方法来降低一次性将大量数据点写入数十万个系列所带来的影响。

With the 0.9.3 and 0.9.4 releases our plan was to put a write ahead log (WAL) in front of Bolt. That way we could reduce the number of random insertions into the keyspace. Instead, we’d buffer up multiple writes that were next to each other and then flush them at once. However, that only served to delay the problem. High IOPS still became an issue and it showed up very quickly for anyone operating at even moderate work loads.  
在 0.9.3 和 0.9.4 版本中，我们计划在 Bolt 前面添加一个预写日志 (WAL)。这样可以减少随机插入键空间的次数。取而代之的是，我们会缓冲相邻的多个写入操作，然后一次性刷新它们。然而，这只能延缓问题的发生。高 IOPS 仍然是一个问题，即使是在中等负载下，这个问题也会很快显现出来。

However, our experience building the first WAL implementation in front of Bolt gave us the confidence we needed that the write problem could be solved. The performance of the WAL itself was fantastic, the index simply could not keep up. At this point we started thinking again about how we could create something similar to an LSM Tree that could keep up with our write load.  
然而，我们在 Bolt 之前构建第一个 WAL 实现的经验给了我们足够的信心，相信写入问题能够得到解决。WAL 本身的性能非常出色，但索引根本跟不上。这时，我们又开始思考如何创建类似于 LSM 树的东西来应对写入负载。

Thus was born the Time Structured Merge Tree.  
因此，时间结构合并树诞生了。

---



> 来自: [内存索引和时间结构合并树 (TSM) | InfluxDB OSS v1 文档 --- In-memory indexing and the Time-Structured Merge Tree (TSM) | InfluxDB OSS v1 Documentation](https://docs.influxdata.com/influxdb/v1/concepts/storage_engine/)
>





