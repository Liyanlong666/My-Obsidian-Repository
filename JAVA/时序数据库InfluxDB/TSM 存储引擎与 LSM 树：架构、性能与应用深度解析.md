# <font style="color:rgb(27, 28, 29);">TSM 存储引擎与 LSM 树：架构、性能与应用深度解析</font>
## <font style="color:rgb(27, 28, 29);">I. 执行摘要</font>
<font style="color:rgb(27, 28, 29);">日志结构合并树（Log-Structured Merge-tree，LSM 树）是一种基础数据结构，专为高写入负载场景设计，旨在通过批量处理和顺序写入数据来最大程度地减少随机磁盘 I/O。时间结构合并树（Time-Structured Merge Tree，TSM）则是在 LSM 树原则基础上发展而来的一个专用变体，由 InfluxDB 专门构建，以应对时间序列数据所固有的独特挑战和特性。TSM 继承并优化了 LSM 树的核心理念，使其更适合处理时间戳数据。</font>

<font style="color:rgb(27, 28, 29);">这两种存储引擎都采用内存-磁盘分层架构，并依赖预写日志（Write-Ahead Log，WAL）来确保数据持久性。它们的核心操作都涉及将内存中的数据刷新到不可变的文件（LSM 树中的 SSTables，TSM 中的 TSM 文件），并通过周期性的“合并”或“压缩”（compaction）过程来组织数据、回收空间并优化读取性能。</font>

<font style="color:rgb(27, 28, 29);">然而，TSM 通过引入专门的索引（如时间序列索引 TSI）、列式存储、高级压缩技术（如增量编码）以及时间感知的压缩策略，与通用 LSM 树有所区别。这些优化都针对时间序列数据追加写入、时间排序的特性进行了定制。通用 LSM 树通常以牺牲部分读取性能为代价来换取高写入吞吐量，而 TSM 则致力于在特定领域内同时优化写入和读取性能。</font>

<font style="color:rgb(27, 28, 29);">总而言之，LSM 树是通用 NoSQL 数据库和高插入/更新量事务日志的理想选择，而 TSM 则在物联网、监控和实时分析等时间序列数据库领域表现卓越，尤其适用于处理海量时间戳数据。</font>

## <font style="color:rgb(27, 28, 29);">II. 日志结构合并树（LSM 树）：写入优化存储的基石</font>
### <font style="color:rgb(27, 28, 29);">A. 起源与核心原理</font>
<font style="color:rgb(27, 28, 29);">日志结构合并树（LSM 树）的诞生是为了克服传统数据库访问方法（如 B 树）在处理高频率插入和删除操作时的局限性 </font><sup>**<font style="color:rgb(87, 91, 95);">1</font>**</sup><font style="color:rgb(27, 28, 29);">。B 树虽然在读写平衡和范围查询方面表现出色，但其原地更新的特性会导致大量的随机磁盘 I/O 和磁盘臂移动，这在长时间内高频率的记录插入和删除场景下会成为瓶颈 </font><sup>**<font style="color:rgb(87, 91, 95);">1</font>**</sup><font style="color:rgb(27, 28, 29);">。当传统访问方法的插入操作磁盘臂成本超过存储介质成本时，LSM 树的出现极大地降低了成本性能 </font><sup>**<font style="color:rgb(87, 91, 95);">1</font>**</sup><font style="color:rgb(27, 28, 29);">。这种设计不仅追求速度，更注重成本效益，因为它直接解决了随机 I/O 在延迟和资源利用（如磁盘臂移动、SSD 磨损）方面不成比例的高昂成本。LSM 树通过顺序写入的方法，直接应对了这一经济瓶颈。</font>

<font style="color:rgb(27, 28, 29);">LSM 树的核心思想是利用顺序写入来提升写入性能 </font><sup>**<font style="color:rgb(87, 91, 95);">2</font>**</sup><font style="color:rgb(27, 28, 29);">。它通过将传入的写入操作缓冲在内存中，然后以排序批处理的方式刷新到磁盘，从而最大程度地减少随机磁盘写入 </font><sup>**<font style="color:rgb(87, 91, 95);">2</font>**</sup><font style="color:rgb(27, 28, 29);">。这种方法推迟并批量处理索引更改，以类似于归并排序的效率，将更改从内存组件级联到一个或多个磁盘组件 </font><sup>**<font style="color:rgb(87, 91, 95);">1</font>**</sup><font style="color:rgb(27, 28, 29);">。非原地更新是其核心原则，即磁盘上的数据通过追加而非覆盖的方式进行修改 </font><sup>**<font style="color:rgb(87, 91, 95);">3</font>**</sup><font style="color:rgb(27, 28, 29);">。删除操作则通过写入“墓碑”（tombstone）标记来实现 </font><sup>**<font style="color:rgb(87, 91, 95);">4</font>**</sup><font style="color:rgb(27, 28, 29);">。归并排序的比喻并非表面化，它揭示了 LSM 树核心的算法效率。正如归并排序组合已排序的子数组一样，LSM 树持续合并已排序的数据运行，确保磁盘上的整体排序顺序，同时避免了单个更新操作带来的昂贵随机写入。这种持续的合并是实现高效写入和最终读取优化的关键。</font>

<font style="color:rgb(27, 28, 29);">LSM 树主要为写入密集型应用而设计 </font><sup>**<font style="color:rgb(87, 91, 95);">2</font>**</sup><font style="color:rgb(27, 28, 29);">。在写入吞吐量至关重要且可以容忍略高读取延迟的场景中，LSM 树表现出色 </font><sup>**<font style="color:rgb(87, 91, 95);">2</font>**</sup><font style="color:rgb(27, 28, 29);">。虽然插入操作效率很高，但需要即时响应的索引查找在某些情况下可能会降低 I/O 效率 </font><sup>**<font style="color:rgb(87, 91, 95);">1</font>**</sup><font style="color:rgb(27, 28, 29);">。如果数据不在 Memtable 中，读取操作可能会变慢，需要多次查找 SSTable </font><sup>**<font style="color:rgb(87, 91, 95);">4</font>**</sup><font style="color:rgb(27, 28, 29);">。这被称为“读取放大” </font><sup>**<font style="color:rgb(87, 91, 95);">9</font>**</sup><font style="color:rgb(27, 28, 29);">。写入优化（顺序追加、非原地更新、多层排序运行）的设计选择直接导致了读取放大，因为读取操作可能需要检查多个位置（内存和多个磁盘文件）才能找到键的最新版本 </font><sup>**<font style="color:rgb(87, 91, 95);">4</font>**</sup><font style="color:rgb(27, 28, 29);">。这是存储引擎设计中固有的矛盾，压缩策略试图缓解但无法完全消除。</font>

### <font style="color:rgb(27, 28, 29);">B. 架构组件与数据流</font>
#### <font style="color:rgb(27, 28, 29);">Memtable（C0/内存组件）</font>
<font style="color:rgb(27, 28, 29);">Memtable 是一个临时的内存数据结构，所有写入操作（插入、更新、删除）都首先存储在这里 </font><sup>**<font style="color:rgb(87, 91, 95);">2</font>**</sup><font style="color:rgb(27, 28, 29);">。这使得写入操作非常快速，并能快速访问最新数据 </font><sup>**<font style="color:rgb(87, 91, 95);">2</font>**</sup><font style="color:rgb(27, 28, 29);">。它通常是一个有序的树结构，如红黑树或 AVL 树 </font><sup>**<font style="color:rgb(87, 91, 95);">2</font>**</sup><font style="color:rgb(27, 28, 29);">，或跳表 </font><sup>**<font style="color:rgb(87, 91, 95);">5</font>**</sup><font style="color:rgb(27, 28, 29);">，这允许高效的内存排序和搜索 </font><sup>**<font style="color:rgb(87, 91, 95);">7</font>**</sup><font style="color:rgb(27, 28, 29);">。新数据会被插入到有序树结构的适当位置，这可能涉及树的拆分或旋转 </font><sup>**<font style="color:rgb(87, 91, 95);">3</font>**</sup><font style="color:rgb(27, 28, 29);">。Memtable 选择有序数据结构（如红黑树或跳表）至关重要。它确保 Memtable 的内容在刷新到磁盘时已经排序。这种预排序是 LSM 树“归并排序”效率的基础，因为它允许向磁盘进行顺序写入 </font><sup>**<font style="color:rgb(87, 91, 95);">3</font>**</sup><font style="color:rgb(27, 28, 29);">。如果没有这一点，磁盘刷新将涉及随机写入，从而抵消了 LSM 的核心优势。</font>

#### <font style="color:rgb(27, 28, 29);">不可变 Memtable（Immutable Memtable）</font>
<font style="color:rgb(27, 28, 29);">一旦 Memtable 达到预定义的大小或内存阈值，它就会变为不可变（只读）状态，并创建一个新的空 Memtable 用于接收传入的写入 </font><sup>**<font style="color:rgb(87, 91, 95);">3</font>**</sup><font style="color:rgb(27, 28, 29);">。然后，不可变 Memtable 会被安排刷新或合并到基于磁盘的结构中 </font><sup>**<font style="color:rgb(87, 91, 95);">3</font>**</sup><font style="color:rgb(27, 28, 29);">。如果数据未持久化，其中的数据存在丢失的风险 </font><sup>**<font style="color:rgb(87, 91, 95);">3</font>**</sup><font style="color:rgb(27, 28, 29);">。从可变 Memtable 到不可变 Memtable 的转换是显式的批处理机制。它允许新的 Memtable 持续接收写入，而旧的不可变 Memtable 则在后台异步刷新到磁盘。这种职责分离实现了高写入并发性，并防止磁盘 I/O 阻塞传入的写入请求。</font>

#### <font style="color:rgb(27, 28, 29);">预写日志（Write-Ahead Log, WAL）</font>
<font style="color:rgb(27, 28, 29);">每个写入操作（插入、更新、删除）在应用于 Memtable 之前，都会首先追加到磁盘上的持久性预写日志（WAL）中 </font><sup>**<font style="color:rgb(87, 91, 95);">5</font>**</sup><font style="color:rgb(27, 28, 29);">。WAL 确保了数据持久性和崩溃恢复 </font><sup>**<font style="color:rgb(87, 91, 95);">5</font>**</sup><font style="color:rgb(27, 28, 29);">。如果系统崩溃，任何已确认但尚未从 Memtable 刷新到 SSTable 的写入都可以通过重放 WAL 来恢复 </font><sup>**<font style="color:rgb(87, 91, 95);">5</font>**</sup><font style="color:rgb(27, 28, 29);">。WAL 不仅作为持久性机制，还间接促进了写入性能。通过在内存中处理并最终刷新为 SSTables 之前，将写入操作顺序记录到磁盘（WAL 是顺序写入的），它确保即使内存中的 Memtable 丢失，数据也是可恢复的。这使得内存操作可以针对速度进行优化，而不会牺牲数据完整性，因为真正的持久化点是 WAL。</font>

#### <font style="color:rgb(27, 28, 29);">排序字符串表（Sorted String Tables, SSTables）</font>
<font style="color:rgb(27, 28, 29);">SSTables 是存储在磁盘上的不可变、已排序的数据文件 </font><sup>**<font style="color:rgb(87, 91, 95);">2</font>**</sup><font style="color:rgb(27, 28, 29);">。一旦不可变 Memtable 满载，其内容就会作为新的 SSTable 刷新到磁盘 </font><sup>**<font style="color:rgb(87, 91, 95);">3</font>**</sup><font style="color:rgb(27, 28, 29);">。这个过程有时被称为“次要压缩”（Minor Compaction） </font><sup>**<font style="color:rgb(87, 91, 95);">3</font>**</sup><font style="color:rgb(27, 28, 29);">。SSTables 被组织成多个级别（Level 1 到 Level n），更高级别的文件大小和数量呈指数级增长 </font><sup>**<font style="color:rgb(87, 91, 95);">2</font>**</sup><font style="color:rgb(27, 28, 29);">。每个 SSTable 都包含按索引键排序的数据 </font><sup>**<font style="color:rgb(87, 91, 95);">4</font>**</sup><font style="color:rgb(27, 28, 29);">。SSTables 的不可变性是 LSM 树设计的基石。它简化了并发读取（磁盘文件永不更改，无需加锁）并允许高效的追加写入。然而，这也意味着更新和删除会创建新版本或墓碑，从而需要复杂的后台压缩过程来清理过期数据并保持效率。</font>

### <font style="color:rgb(27, 28, 29);">C. 运行机制</font>
#### <font style="color:rgb(27, 28, 29);">写入操作</font>
<font style="color:rgb(27, 28, 29);">传入数据首先追加到预写日志（WAL）以确保持久性 </font><sup>**<font style="color:rgb(87, 91, 95);">5</font>**</sup><font style="color:rgb(27, 28, 29);">。同时，数据被插入到内存中的 Memtable 中 </font><sup>**<font style="color:rgb(87, 91, 95);">2</font>**</sup><font style="color:rgb(27, 28, 29);">。这是一个快速的、仅内存操作 </font><sup>**<font style="color:rgb(87, 91, 95);">7</font>**</sup><font style="color:rgb(27, 28, 29);">。当 Memtable 达到一定大小时，它会变成不可变 Memtable 并作为新的 SSTable 刷新到磁盘 </font><sup>**<font style="color:rgb(87, 91, 95);">3</font>**</sup><font style="color:rgb(27, 28, 29);">。这个刷新操作涉及将内存中排序的数据顺序写入磁盘 </font><sup>**<font style="color:rgb(87, 91, 95);">3</font>**</sup><font style="color:rgb(27, 28, 29);">。LSM 树中的写入路径是一个高度优化的异步管道。写入被快速摄取到内存并记录以确保持久性，从而使系统能够迅速确认。刷新到 SSTables 的重型磁盘 I/O 被延迟并批量处理，在后台进行。这种管道化是 LSM 树实现其高写入吞吐量的关键。</font>

#### <font style="color:rgb(27, 28, 29);">读取操作</font>
<font style="color:rgb(27, 28, 29);">读取操作必须同时检查内存中的 Memtable 和磁盘上的 SSTables，以确保检索到最新版本的数据 </font><sup>**<font style="color:rgb(87, 91, 95);">3</font>**</sup><font style="color:rgb(27, 28, 29);">。搜索通常从 Memtable（Level 0）开始，然后顺序遍历基于磁盘的 SSTable 级别（Level 1 到 Level n），直到找到键，确保返回最新数据 </font><sup>**<font style="color:rgb(87, 91, 95);">2</font>**</sup><font style="color:rgb(27, 28, 29);">。为了优化磁盘查找，使用了布隆过滤器（Bloom filter）等附加结构 </font><sup>**<font style="color:rgb(87, 91, 95);">2</font>**</sup><font style="color:rgb(27, 28, 29);">。布隆过滤器可以快速判断 SSTable 是否“可能”包含某个键，从而大幅减少不必要的磁盘寻道 </font><sup>**<font style="color:rgb(87, 91, 95);">5</font>**</sup><font style="color:rgb(27, 28, 29);">。如果布隆过滤器指示键不存在，则可以跳过该 SSTable </font><sup>**<font style="color:rgb(87, 91, 95);">2</font>**</sup><font style="color:rgb(27, 28, 29);">。布隆过滤器是利用概率数据结构优化性能的典型例子。它们提供了一种快速、内存高效的方式来检查键是否存在，具有少量误报（声称键存在但实际不存在）的可能性，但没有漏报（从不声称键不存在但实际存在） </font><sup>**<font style="color:rgb(87, 91, 95);">5</font>**</sup><font style="color:rgb(27, 28, 29);">。这种权衡是可以接受的，因为误报只会导致不必要的磁盘读取，而不会导致错误的结果。这是一种务实的方法，旨在缓解读取放大。</font>

#### <font style="color:rgb(27, 28, 29);">更新与删除操作</font>
<font style="color:rgb(27, 28, 29);">LSM 树执行非原地更新 </font><sup>**<font style="color:rgb(87, 91, 95);">3</font>**</sup><font style="color:rgb(27, 28, 29);">。更新被视为新的写入：一个包含更新值的新条目被插入到 Memtable 中 </font><sup>**<font style="color:rgb(87, 91, 95);">3</font>**</sup><font style="color:rgb(27, 28, 29);">。旧版本保留在磁盘上直到压缩发生 </font><sup>**<font style="color:rgb(87, 91, 95);">6</font>**</sup><font style="color:rgb(27, 28, 29);">。删除操作通过在 Memtable 中为相应键写入一个“墓碑”条目（一个表示删除的特殊标记）来处理 </font><sup>**<font style="color:rgb(87, 91, 95);">4</font>**</sup><font style="color:rgb(27, 28, 29);">。实际数据仅在压缩期间从磁盘中移除 </font><sup>**<font style="color:rgb(87, 91, 95);">4</font>**</sup><font style="color:rgb(27, 28, 29);">。使用墓碑和非原地更新意味着磁盘上的数据（SSTables）与数据库的最新逻辑状态并非立即“一致”。它包含过时的版本和墓碑。真正的“一致性”和空间回收最终通过后台压缩过程实现。这突显了存储引擎层面的最终一致性，这是为了写入性能而做出的权衡。</font>

### <font style="color:rgb(27, 28, 29);">D. 压缩策略：LSM 树性能的引擎</font>
#### <font style="color:rgb(27, 28, 29);">压缩目的</font>
<font style="color:rgb(27, 28, 29);">压缩是 LSM 树中的核心操作，它通过合并和重组 SSTables 中的数据来提高效率 </font><sup>**<font style="color:rgb(87, 91, 95);">2</font>**</sup><font style="color:rgb(27, 28, 29);">。其主要目标包括：数据整合、减少 SSTable 数量、通过移除过时数据（旧版本、墓碑）回收磁盘空间、减少碎片化以及提升读取性能 </font><sup>**<font style="color:rgb(87, 91, 95);">2</font>**</sup><font style="color:rgb(27, 28, 29);">。压缩直接影响写入放大、读取放大和空间放大 </font><sup>**<font style="color:rgb(87, 91, 95);">6</font>**</sup><font style="color:rgb(27, 28, 29);">。压缩不是一次性事件，而是一个持续的后台过程 </font><sup>**<font style="color:rgb(87, 91, 95);">2</font>**</sup><font style="color:rgb(27, 28, 29);">。它是使 LSM 树随着时间推移保持高性能的“引擎”，通过不断平衡写入速度、读取效率和存储空间之间的权衡。如果没有有效的压缩，LSM 树的读取性能将迅速下降，并消耗过多的磁盘空间。</font>

#### <font style="color:rgb(27, 28, 29);">分层压缩（Leveled Compaction）</font>
<font style="color:rgb(27, 28, 29);">在分层压缩中，每个级别（除了内存组件）最多可以有一个“运行”（或一组键范围不重叠的文件），并且每个级别的容量都呈指数级增长 </font><sup>**<font style="color:rgb(87, 91, 95);">4</font>**</sup><font style="color:rgb(27, 28, 29);">。当 Level </font>

`<font style="color:rgb(87, 91, 95);">i-1</font>`<font style="color:rgb(27, 28, 29);"> 的一个运行移动到 Level </font>`<font style="color:rgb(87, 91, 95);">i</font>`<font style="color:rgb(27, 28, 29);"> 时，它会贪婪地与 Level </font>`<font style="color:rgb(87, 91, 95);">i</font>`<font style="color:rgb(27, 28, 29);"> 中现有且键范围重叠的运行进行排序合并 </font><sup>**<font style="color:rgb(87, 91, 95);">14</font>**</sup><font style="color:rgb(27, 28, 29);">。这种策略积极地将较小的排序运行合并到较大的运行中 </font><sup>**<font style="color:rgb(87, 91, 95);">18</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

**<font style="color:rgb(27, 28, 29);">性能影响：</font>**

+ **<font style="color:rgb(27, 28, 29);">较低的读取放大：</font>**<font style="color:rgb(27, 28, 29);"> 由于数据组织紧密且键范围在同一级别内通常不重叠 </font><sup>**<font style="color:rgb(87, 91, 95);">4</font>**</sup><font style="color:rgb(27, 28, 29);">，读取操作通常只需检查每个级别最多一个文件 </font><sup>**<font style="color:rgb(87, 91, 95);">6</font>**</sup><font style="color:rgb(27, 28, 29);">，从而导致比分级压缩更可预测且通常更低的读取放大 </font><sup>**<font style="color:rgb(87, 91, 95);">17</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">较高的写入放大：</font>**<font style="color:rgb(27, 28, 29);"> 积极的合并意味着数据在级联通过不同级别时可能会被多次重写，从而可能增加写入放大 </font><sup>**<font style="color:rgb(87, 91, 95);">17</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">磁盘空间需求：</font>**<font style="color:rgb(27, 28, 29);"> 与分级压缩相比，所需的空闲磁盘空间更少（例如，10 * sstable_size_in_mb） </font><sup>**<font style="color:rgb(87, 91, 95);">19</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

<font style="color:rgb(27, 28, 29);">分层压缩通过在磁盘上维护高度组织化、不重叠的结构来优先考虑读取的确定性和效率。这代价是可能更高的写入放大，因为即使是很小的传入刷新也可能触发整个级别的大规模合并操作。这是一种为了可预测的读取延迟而牺牲更多后台写入活动的设计选择。</font>

#### <font style="color:rgb(27, 28, 29);">大小分层压缩（Size-Tiered Compaction）</font>
<font style="color:rgb(27, 28, 29);">在大小分层压缩中，大小相似的 SSTables 被分组到相同的“桶”或“层”中 </font><sup>**<font style="color:rgb(87, 91, 95);">17</font>**</sup><font style="color:rgb(27, 28, 29);">。当一个层中积累了最小数量的 SSTables（例如，</font>

`<font style="color:rgb(87, 91, 95);">min_threshold</font>`<font style="color:rgb(27, 28, 29);">）时，就会在该组内进行压缩 </font><sup>**<font style="color:rgb(87, 91, 95);">17</font>**</sup><font style="color:rgb(27, 28, 29);">。这种策略会等待几个大小相似的排序运行积累起来，然后将它们合并在一起 </font><sup>**<font style="color:rgb(87, 91, 95);">18</font>**</sup><font style="color:rgb(27, 28, 29);">。它会延迟压缩，这对于写入密集型进程可能是有益的 </font><sup>**<font style="color:rgb(87, 91, 95);">17</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

**<font style="color:rgb(27, 28, 29);">性能影响：</font>**

+ **<font style="color:rgb(27, 28, 29);">较低的写入放大：</font>**<font style="color:rgb(27, 28, 29);"> 通过合并大小相似的 SSTables 并延迟压缩，它避免了不必要的将非常大的 SSTables 与小得多的 SSTables 合并，从而减少了压缩次数，并最大程度地减少了每次操作期间写入磁盘的数据量 </font><sup>**<font style="color:rgb(87, 91, 95);">17</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">较高的读取放大：</font>**<font style="color:rgb(27, 28, 29);"> 数据可能分散在许多键范围重叠的 SSTables 中，这意味着读取操作可能需要检查更多文件，从而导致较高的读取放大 </font><sup>**<font style="color:rgb(87, 91, 95);">17</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">磁盘空间需求：</font>**<font style="color:rgb(27, 28, 29);"> 压缩所需的空闲磁盘空间至少与最大列族的大小相同 </font><sup>**<font style="color:rgb(87, 91, 95);">19</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

<font style="color:rgb(27, 28, 29);">大小分层压缩通过最小化压缩期间重写的数据量来优化原始写入吞吐量。它接受更碎片化的磁盘布局，这可能导致更高且更不稳定的读取延迟，因为读取可能需要探测更多 SSTables。这是一种适用于写入吞吐量至关重要且读取延迟要求不那么严格的应用的策略。</font>

#### <font style="color:rgb(27, 28, 29);">通用压缩（Universal Compaction，RocksDB 的方法）</font>
<font style="color:rgb(27, 28, 29);">RocksDB 是 Google LevelDB 的演进版本，它使用通用压缩作为替代策略 </font><sup>**<font style="color:rgb(87, 91, 95);">5</font>**</sup><font style="color:rgb(27, 28, 29);">。它设计得更灵活，根据启发式算法按需合并文件 </font><sup>**<font style="color:rgb(87, 91, 95);">5</font>**</sup><font style="color:rgb(27, 28, 29);">。它将所有 SST 文件组织成覆盖整个键范围的排序运行，不同的排序运行在时间范围上不重叠 </font><sup>**<font style="color:rgb(87, 91, 95);">18</font>**</sup><font style="color:rgb(27, 28, 29);">。压缩只发生在两个或更多相邻时间范围的排序运行之间，产生一个单一的排序运行 </font><sup>**<font style="color:rgb(87, 91, 95);">18</font>**</sup><font style="color:rgb(27, 28, 29);">。该策略假设并试图保持较新数据位于较小排序运行中，而较旧数据位于较大排序运行中 </font><sup>**<font style="color:rgb(87, 91, 95);">18</font>**</sup><font style="color:rgb(27, 28, 29);">。当排序运行的数量达到阈值 </font>

`<font style="color:rgb(87, 91, 95);">N</font>`<font style="color:rgb(27, 28, 29);"> 时，就会触发压缩 </font><sup>**<font style="color:rgb(87, 91, 95);">18</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

**<font style="color:rgb(27, 28, 29);">优点：</font>**<font style="color:rgb(27, 28, 29);"> 与分层压缩相比，提供了“好得多”的写入放大 </font><sup>**<font style="color:rgb(87, 91, 95);">18</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

局限性： 由于排序运行数量可能较多，读取期间可能导致更高的 I/O 和 CPU 成本 18。完全压缩可能暂时使磁盘空间使用量翻倍 18。

<font style="color:rgb(27, 28, 29);">通用压缩试图在分层和大小分层策略的极端之间取得平衡 </font><sup>**<font style="color:rgb(87, 91, 95);">5</font>**</sup><font style="color:rgb(27, 28, 29);">。通过关注时间范围和选择性合并，它旨在减少写入放大，同时仍保持合理的读取性能水平，使其适用于混合读写模式的工作负载。其“通用”之处在于其适应性。</font>

#### <font style="color:rgb(27, 28, 29);">时间窗口压缩策略（Time-Windowed Compaction Strategy, TWCS）</font>
<font style="color:rgb(27, 28, 29);">TWCS 是一种专门的压缩策略，常用于 Cassandra，并针对时间序列数据进行了优化 </font><sup>**<font style="color:rgb(87, 91, 95);">17</font>**</sup><font style="color:rgb(27, 28, 29);">。在一个定义好的“压缩窗口”（例如，1 天）内创建的 SSTables 会使用大小分层压缩策略（STCS）进行合并 </font><sup>**<font style="color:rgb(87, 91, 95);">17</font>**</sup><font style="color:rgb(27, 28, 29);">。一旦窗口关闭（即，当天结束），该天的 SSTables 通常不会再次被压缩，即使它们包含墓碑或旧数据 </font><sup>**<font style="color:rgb(87, 91, 95);">17</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

**<font style="color:rgb(27, 28, 29);">对时间序列数据的益处：</font>**<font style="color:rgb(27, 28, 29);"> 压缩重点放在活跃的最新数据上，减少了写入密集型工作负载的压缩频率和大小 </font><sup>**<font style="color:rgb(87, 91, 95);">17</font>**</sup><font style="color:rgb(27, 28, 29);">。避免了对旧的、很少修改的数据进行不必要的工作 </font><sup>**<font style="color:rgb(87, 91, 95);">17</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

局限性： 如果数据在其压缩窗口关闭后频繁更新，或者如果生存时间（TTL）长于压缩窗口，可能会增加磁盘使用量和读取放大 17。

<font style="color:rgb(27, 28, 29);">TWCS 将时间作为压缩的首要维度，这对于时间序列数据非常有效，因为时间序列数据主要是追加写入，并且具有自然的过期模式。该策略将压缩工作与数据生命周期对齐，将资源集中在“热”数据上，并最大程度地减少对“冷”数据的工作量，但这需要根据数据更新模式和保留策略仔细调整窗口大小。</font>

#### <font style="color:rgb(27, 28, 29);">压缩对系统指标的影响</font>
+ **<font style="color:rgb(27, 28, 29);">写入放大（Write Amplification, WA）：</font>**<font style="color:rgb(27, 28, 29);"> 写入存储设备的数据量与写入数据库的数据量之比 </font><sup>**<font style="color:rgb(87, 91, 95);">9</font>**</sup><font style="color:rgb(27, 28, 29);">。高 WA 会缩短闪存寿命 </font><sup>**<font style="color:rgb(87, 91, 95);">6</font>**</sup><font style="color:rgb(27, 28, 29);">。非原地更新和压缩期间数据的重复写入是写入放大的直接原因 </font><sup>**<font style="color:rgb(87, 91, 95);">6</font>**</sup><font style="color:rgb(27, 28, 29);">。不同的压缩策略（分层与大小分层）直接影响 WA 比率 </font><sup>**<font style="color:rgb(87, 91, 95);">17</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">读取放大（Read Amplification, RA）：</font>**<font style="color:rgb(27, 28, 29);"> 每次查询的磁盘读取次数 </font><sup>**<font style="color:rgb(87, 91, 95);">9</font>**</sup><font style="color:rgb(27, 28, 29);">。数据分散在多个键范围重叠的 SSTables 中，或者需要搜索多个级别，直接导致读取放大 </font><sup>**<font style="color:rgb(87, 91, 95);">4</font>**</sup><font style="color:rgb(27, 28, 29);">。压缩策略和布隆过滤器的使用可以缓解 RA </font><sup>**<font style="color:rgb(87, 91, 95);">2</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">空间放大（Space Amplification, SA）：</font>**<font style="color:rgb(27, 28, 29);"> 物理存储使用量与逻辑数据大小之比。这发生在旧版本数据、墓碑和未压缩文件的情况下 </font><sup>**<font style="color:rgb(87, 91, 95);">6</font>**</sup><font style="color:rgb(27, 28, 29);">。追加写入性质和过时数据（墓碑、旧版本）的延迟删除直接导致空间放大 </font><sup>**<font style="color:rgb(87, 91, 95);">6</font>**</sup><font style="color:rgb(27, 28, 29);">。压缩对于回收这些空间至关重要 </font><sup>**<font style="color:rgb(87, 91, 95);">6</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

<font style="color:rgb(27, 28, 29);">WA、RA 和 SA 并非独立存在；它们通常是反向相关的，代表了 LSM 树设计中相同底层权衡的不同方面。优化其中一个通常会导致另一个的非优化。例如，减少写入放大（例如，使用大小分层）可能会增加读取放大和空间放大，反之亦然。这种复杂的相互作用需要根据具体的工作负载优先级进行仔细调整 </font><sup>**<font style="color:rgb(87, 91, 95);">5</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

**<font style="color:rgb(27, 28, 29);">表：LSM 树压缩策略权衡</font>**

| <font style="color:rgb(27, 28, 29);">策略</font> | <font style="color:rgb(27, 28, 29);">主要优化目标</font> | <font style="color:rgb(27, 28, 29);">写入放大</font> | <font style="color:rgb(27, 28, 29);">读取放大</font> | <font style="color:rgb(27, 28, 29);">空间放大</font> | <font style="color:rgb(27, 28, 29);">关键特性</font> | <font style="color:rgb(27, 28, 29);">理想工作负载</font> |
| --- | --- | --- | --- | --- | --- | --- |
| <font style="color:rgb(27, 28, 29);">分层压缩</font> | <font style="color:rgb(27, 28, 29);">读取性能、空间效率</font> | <font style="color:rgb(27, 28, 29);">高</font> | <font style="color:rgb(27, 28, 29);">低</font> | <font style="color:rgb(27, 28, 29);">低</font> | <font style="color:rgb(27, 28, 29);">积极合并；每层键范围不重叠</font> | <font style="color:rgb(27, 28, 29);">读取密集型</font> |
| <font style="color:rgb(27, 28, 29);">大小分层压缩</font> | <font style="color:rgb(27, 28, 29);">写入吞吐量</font> | <font style="color:rgb(27, 28, 29);">低</font> | <font style="color:rgb(27, 28, 29);">高</font> | <font style="color:rgb(27, 28, 29);">高</font> | <font style="color:rgb(27, 28, 29);">合并大小相似的 SSTables；延迟压缩</font> | <font style="color:rgb(27, 28, 29);">写入密集型</font> |
| <font style="color:rgb(27, 28, 29);">通用压缩</font> | <font style="color:rgb(27, 28, 29);">混合读写平衡</font> | <font style="color:rgb(27, 28, 29);">较低</font> | <font style="color:rgb(27, 28, 29);">较高</font> | <font style="color:rgb(27, 28, 29);">较高</font> | <font style="color:rgb(27, 28, 29);">基于启发式算法的灵活合并；时间范围不重叠</font> | <font style="color:rgb(27, 28, 29);">混合型</font> |
| <font style="color:rgb(27, 28, 29);">时间窗口压缩</font> | <font style="color:rgb(27, 28, 29);">时间序列数据效率</font> | <font style="color:rgb(27, 28, 29);">较低</font> | <font style="color:rgb(27, 28, 29);">较高</font> | <font style="color:rgb(27, 28, 29);">较高</font> | <font style="color:rgb(27, 28, 29);">基于时间窗口合并；针对时间序列数据生命周期</font> | <font style="color:rgb(27, 28, 29);">时间序列</font> |


## <font style="color:rgb(27, 28, 29);">III. 时间结构合并树（TSM）：时间序列数据专用化</font>
### <font style="color:rgb(27, 28, 29);">A. 时间序列优化势在必行</font>
<font style="color:rgb(27, 28, 29);">时间序列数据具有独特的特性，使得对其进行专门优化变得至关重要。</font>

#### <font style="color:rgb(27, 28, 29);">时间序列数据的独特特性</font>
+ **<font style="color:rgb(27, 28, 29);">高摄取率：</font>**<font style="color:rgb(27, 28, 29);"> 时间序列数据通常由传感器、物联网设备和监控系统持续生成，导致传入数据点数量巨大 </font><sup>**<font style="color:rgb(87, 91, 95);">7</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">追加写入性质：</font>**<font style="color:rgb(27, 28, 29);"> 特定时间序列数据记录几乎从不原地更新或删除；新数据只是简单地追加 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。这与日志文件类似 </font><sup>**<font style="color:rgb(87, 91, 95);">21</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">基于时间的查询：</font>**<font style="color:rgb(27, 28, 29);"> 查询主要基于时间范围（例如，“过去一小时的数据”、“一个月的日均值”），通常涉及聚合、降采样和趋势分析 </font><sup>**<font style="color:rgb(87, 91, 95);">20</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">数据不可变性（摄取后）：</font>**<font style="color:rgb(27, 28, 29);"> 一旦记录，时间序列数据点很少更改 </font><sup>**<font style="color:rgb(87, 91, 95);">21</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">数据生命周期管理：</font>**<font style="color:rgb(27, 28, 29);"> 时间序列数据通常具有明确的保留策略，较旧的数据会批量删除 </font><sup>**<font style="color:rgb(87, 91, 95);">20</font>**</sup><font style="color:rgb(27, 28, 29);">。随着数据老化，粒度可能变得不那么重要，从而导致数据汇总 </font><sup>**<font style="color:rgb(87, 91, 95);">20</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">趋势关注：</font>**<font style="color:rgb(27, 28, 29);"> 随时间变化的趋势通常比任何特定时间点的值更重要 </font><sup>**<font style="color:rgb(87, 91, 95);">21</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">高基数：</font>**<font style="color:rgb(27, 28, 29);"> 唯一时间序列的数量（例如，传感器 ID、标签集）可能非常高，对索引和查询性能构成挑战 </font><sup>**<font style="color:rgb(87, 91, 95);">21</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

<font style="color:rgb(27, 28, 29);">时间序列数据中，时间不仅仅是另一个属性，它更是主要的组织原则和最常见的查询维度。这种根本性差异允许进行通用 LSM 树不适用或效率不高的优化（如基于时间的分区、增量压缩和时间感知压缩）。</font>

#### <font style="color:rgb(27, 28, 29);">通用 LSM 树为何需要针对 TSDBs 进行专门化</font>
<font style="color:rgb(27, 28, 29);">尽管 LSM 树提供了高写入吞吐量，但其通用性质意味着它们并未固有地针对时间序列数据特有的读取模式、压缩机会和数据生命周期管理需求进行优化 </font><sup>**<font style="color:rgb(87, 91, 95);">20</font>**</sup><font style="color:rgb(27, 28, 29);">。通用 LSM 压缩策略可能无法与基于时间的过期或聚合要求完美对齐 </font><sup>**<font style="color:rgb(87, 91, 95);">17</font>**</sup><font style="color:rgb(27, 28, 29);">。通用 LSM 树虽然写入优化，但在处理 TSDB 中常见的高基数索引和复杂分析查询方面仍然面临挑战 </font><sup>**<font style="color:rgb(87, 91, 95);">21</font>**</sup><font style="color:rgb(27, 28, 29);">。TSM 的专业化是为了应对在特定领域中，“足够好”变得“不够好”的局面，因为在该领域中，特定操作的规模化性能至关重要。</font>

### <font style="color:rgb(27, 28, 29);">B. InfluxDB 的 TSM 存储引擎架构（1.x/2.x 版本）</font>
<font style="color:rgb(27, 28, 29);">InfluxDB 的 TSM（时间结构合并树）存储引擎受 LSM 树启发 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">，包含几个关键组件 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

#### <font style="color:rgb(27, 28, 29);">核心组件</font>
+ **<font style="color:rgb(27, 28, 29);">预写日志（WAL）：</font>**<font style="color:rgb(27, 28, 29);"> 一种写入优化的存储格式，用于持久写入，追加到固定大小的段中 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。它确保即时持久性，并用于在重启时重建内存中的缓存 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">缓存（Cache）：</font>**<font style="color:rgb(27, 28, 29);"> WAL 中当前存储的所有数据点的内存中未压缩副本 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。数据点按键（测量、标签集和唯一字段）组织，每个字段都保持其自身的时间有序范围 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。查询将缓存中的数据与 TSM 文件中的数据合并 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。内存阈值可防止内存不足情况并对写入速度过快的客户端施加背压 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。TSM 中明确的“缓存”组件（与通用 LSM 中的 Memtable 概念不同）突出了对“热”数据（最近写入的数据）进行实时查询的有意优化 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。通过将这些数据保持未压缩状态并驻留在内存中，TSM 确保了即时查询能力，这对于监控和实时分析用例至关重要。</font>
+ **<font style="color:rgb(27, 28, 29);">TSM 文件：</font>**<font style="color:rgb(27, 28, 29);"> 只读、内存映射文件 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">，以列式格式存储排序、压缩的时间序列数据 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。它们的结构与 LSM 树变体中的 SSTables 非常相似 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">时间序列索引（Time Series Index, TSI）：</font>**<font style="color:rgb(27, 28, 29);"> 一种专门的索引，旨在随着数据基数增长而保持查询速度 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。它存储按测量、标签和字段分组的时间序列键，从而实现高效的元查询和时间序列键查找 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

#### <font style="color:rgb(27, 28, 29);">TSM 文件内的数据组织</font>
<font style="color:rgb(27, 28, 29);">TSM 文件以列式格式存储压缩的时间序列数据 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。存储引擎按序列键（测量、标签集、字段键）对字段值进行分组，然后按时间对这些字段值进行排序 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。这种列式存储允许引擎按序列键读取数据并省略无关数据，从而提高查询效率 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。列式存储的选择是时间序列数据特性的直接结果。由于查询通常涉及特定时间范围内的特定字段，因此按列存储数据可以实现更好的压缩（因为列中的值通常相似）和更高效的读取（只读取相关列） </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。这是与行式存储的关键区别，也是分析查询的强大优化。</font>

#### <font style="color:rgb(27, 28, 29);">写入和读取流程</font>
+ **<font style="color:rgb(27, 28, 29);">写入流程：</font>**<font style="color:rgb(27, 28, 29);"> 批量数据点被发送到 InfluxDB，压缩后写入 WAL 以实现即时持久性 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。数据点还会写入内存中的缓存并立即变为可查询 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">读取流程：</font>**<font style="color:rgb(27, 28, 29);"> 存储引擎的查询将缓存中的数据与 TSM 文件中的数据合并 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。查询在查询处理时从缓存副本上执行，确保写入不会影响正在运行的查询 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。在查询处理时对缓存副本执行查询的做法，为读取提供了某种形式的快照隔离。这确保了长时间运行的查询不受并发写入的影响，提供了持续一致的结果，并防止了读写竞争，这在高吞吐量实时系统中至关重要。</font>

#### <font style="color:rgb(27, 28, 29);">TSM 中的多阶段压缩</font>
<font style="color:rgb(27, 28, 29);">TSM 采用复杂的多阶段压缩过程 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

+ **<font style="color:rgb(27, 28, 29);">快照：</font>**<font style="color:rgb(27, 28, 29);"> 缓存和 WAL 中的值转换为 TSM 文件，以释放内存和磁盘空间 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。这些操作根据缓存内存和时间阈值发生 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">级别压缩（Level 1-4）：</font>**<font style="color:rgb(27, 28, 29);"> TSM 文件逐步压缩 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。快照成为 Level 1 文件，多个 Level 1 文件压缩为 Level 2，依此类推，直到 Level 4（完全压缩） </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。较低级别的压缩避免了 CPU 密集型活动，而较高级别的压缩则重新组合块以实现更好的压缩 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">索引优化：</font>**<font style="color:rgb(27, 28, 29);"> 当许多 Level 4 TSM 文件累积时，内部索引可能变得庞大 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。此压缩将系列和索引拆分到新的 TSM 文件中，将给定系列的所有点排序到一个文件中，从而减小索引大小并减少重叠 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">完全压缩：</font>**<font style="color:rgb(27, 28, 29);"> 当分片长时间未进行写入或发生删除操作时运行 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。它们生成一组最优的 TSM 文件，并包含级别和索引优化压缩的所有优化 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

<font style="color:rgb(27, 28, 29);">TSM 的多阶段压缩具有高度专业性，因为它明确识别了时间序列数据“从热到冷”的生命周期。早期阶段侧重于快速持久化和内存回收，而后期阶段则优先考虑历史（冷）数据的深度压缩和索引优化，这与典型的查询模式（实时数据的最新数据，历史数据的分析数据）相符。索引优化是一个独特的阶段，反映了高基数索引的重要性。</font>

#### <font style="color:rgb(27, 28, 29);">TSM 中的压缩技术</font>
<font style="color:rgb(27, 28, 29);">TSM 文件存储压缩的时间序列数据 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。数据被压缩以节省存储空间并加快操作 </font><sup>**<font style="color:rgb(87, 91, 95);">16</font>**</sup><font style="color:rgb(27, 28, 29);">。存储引擎仅存储序列中值之间的差异（或增量），这显著提高了压缩率 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。时间戳和值使用不同的编码进行压缩和单独存储，具体取决于数据类型和形状 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。一些编码器是静态的，始终以相同的方式编码，而另一些则根据数据形状调整其策略 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。列式压缩 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);"> 对于时间序列数据非常有效，因为列内值的同质性以及应用特定类型压缩算法的能力 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。增量编码和类型特定的列式压缩之所以强大，是因为时间序列数据表现出很强的时间局部性——序列中的连续值通常相似或遵循可预测的模式。TSM 明确利用这一特性实现非常高的压缩比，这直接转化为更低的存储成本和更少的读取 I/O </font><sup>**<font style="color:rgb(87, 91, 95);">23</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

#### <font style="color:rgb(27, 28, 29);">TSM 中的索引技术</font>
<font style="color:rgb(27, 28, 29);">TSM 文件索引按键（测量、标签集、字段）的字典顺序，然后按时间排序 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。每个索引条目包含键长度、键、块类型、索引块条目计数、块的最小/最大时间、文件偏移量和块大小 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。这种结构允许高效访问块，并确定特定键/时间戳查找的成本 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。时间序列索引（TSI）旨在随着数据基数增长而保持查询速度 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。它存储按测量、标签和字段分组的时间序列键，从而实现高效的元查询 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。高基数（数十亿个独立数据点 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">；许多独特的系列 </font><sup>**<font style="color:rgb(87, 91, 95);">24</font>**</sup><font style="color:rgb(27, 28, 29);">）是时间序列数据库中臭名昭著的问题，因为它可能导致庞大且缓慢的索引 </font><sup>**<font style="color:rgb(87, 91, 95);">21</font>**</sup><font style="color:rgb(27, 28, 29);">。TSM 明确的时间序列索引（TSI）及其索引优化压缩阶段 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);"> 表明了专门管理这一挑战的努力，认识到高效的元数据查找与数据查找对于时间序列查询同样重要。</font>

### <font style="color:rgb(27, 28, 29);">C. InfluxDB 3.0 架构：范式转变</font>
#### <font style="color:rgb(27, 28, 29);">向基于 Rust、利用 Apache Arrow、DataFusion 和 Parquet 的列式数据库转型</font>
<font style="color:rgb(27, 28, 29);">InfluxDB 3.0 代表了架构上的重大演进，它基于 Rust 构建，并利用 Apache Arrow 和 DataFusion </font><sup>**<font style="color:rgb(87, 91, 95);">23</font>**</sup><font style="color:rgb(27, 28, 29);">。它将时间序列数据以 Apache Parquet 格式存储在对象存储中 </font><sup>**<font style="color:rgb(87, 91, 95);">23</font>**</sup><font style="color:rgb(27, 28, 29);">。Parquet 是一种开源的列式数据文件格式，专为复杂数据的快速处理而设计，支持各种编码和压缩方案 </font><sup>**<font style="color:rgb(87, 91, 95);">23</font>**</sup><font style="color:rgb(27, 28, 29);">。这种转型支持无限标签基数、实时查询，并优化以降低存储成本 </font><sup>**<font style="color:rgb(87, 91, 95);">27</font>**</sup><font style="color:rgb(27, 28, 29);">。转向 Apache Arrow 和 Parquet 是 InfluxDB 3.0 超越纯粹专有存储引擎的战略举措。它标志着对开放、列式数据标准的拥抱，这有助于与更广泛的数据生态系统（包括数据湖和数据仓库）无缝集成，实现“零拷贝、无 ETL 数据共享” </font><sup>**<font style="color:rgb(87, 91, 95);">23</font>**</sup><font style="color:rgb(27, 28, 29);">。这使得 InfluxDB 不仅仅是一个专业的 TSDB，而是更广泛分析数据栈中的一个组件。</font>

#### <font style="color:rgb(27, 28, 29);">分布式组件</font>
<font style="color:rgb(27, 28, 29);">InfluxDB 3.0 引入了分布式架构 </font><sup>**<font style="color:rgb(87, 91, 95);">27</font>**</sup><font style="color:rgb(27, 28, 29);">：</font>

+ **<font style="color:rgb(27, 28, 29);">路由器（Router）：</font>**<font style="color:rgb(27, 28, 29);"> 解析传入数据并将其路由到摄取器，查询目录以获取模式兼容性和持久化位置，并复制数据以确保持久性 </font><sup>**<font style="color:rgb(87, 91, 95);">27</font>**</sup><font style="color:rgb(27, 28, 29);">。可垂直和水平扩展以提高写入吞吐量 </font><sup>**<font style="color:rgb(87, 91, 95);">27</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">摄取器（Ingester）：</font>**<font style="color:rgb(27, 28, 29);"> 处理行协议，将时间序列数据以 Parquet 格式持久化到对象存储中（每个文件一个分区），使尚未持久化的数据可供查询器使用，并维护短期 WAL </font><sup>**<font style="color:rgb(87, 91, 95);">27</font>**</sup><font style="color:rgb(27, 28, 29);">。可水平扩展以提高写入吞吐量 </font><sup>**<font style="color:rgb(87, 91, 95);">27</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">查询器（Querier）：</font>**<font style="color:rgb(27, 28, 29);"> 处理查询请求（SQL 和 InfluxQL），构建查询计划，查询摄取器以获取最新数据，查询目录以获取分区位置，读取 Parquet 文件，过滤并执行操作 </font><sup>**<font style="color:rgb(87, 91, 95);">27</font>**</sup><font style="color:rgb(27, 28, 29);">。可垂直扩展以处理计算密集型查询，水平扩展以处理并发查询 </font><sup>**<font style="color:rgb(87, 91, 95);">27</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">目录（Catalog）：</font>**<font style="color:rgb(27, 28, 29);"> 一个兼容 PostgreSQL 的关系数据库，存储与时间序列数据相关的元数据（模式、分区的物理位置） </font><sup>**<font style="color:rgb(87, 91, 95);">27</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">对象存储（Object Store）：</font>**<font style="color:rgb(27, 28, 29);"> 包含 Apache Parquet 格式的时间序列数据，分区（例如，按天）经过排序、编码和压缩 </font><sup>**<font style="color:rgb(87, 91, 95);">27</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">压缩器（Compactor）：</font>**<font style="color:rgb(27, 28, 29);"> 处理并压缩对象存储中的分区，以持续优化存储，然后更新目录 </font><sup>**<font style="color:rgb(87, 91, 95);">27</font>**</sup><font style="color:rgb(27, 28, 29);">。可垂直扩展以处理计算密集型压缩 </font><sup>**<font style="color:rgb(87, 91, 95);">27</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">垃圾回收器（Garbage Collector）：</font>**<font style="color:rgb(27, 28, 29);"> 运行后台作业以清除过期或删除的数据，移除过时的压缩文件，并回收目录和对象存储中的空间 </font><sup>**<font style="color:rgb(87, 91, 95);">27</font>**</sup><font style="color:rgb(27, 28, 29);">。不适合水平扩展 </font><sup>**<font style="color:rgb(87, 91, 95);">27</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

<font style="color:rgb(27, 28, 29);">InfluxDB 3.0 架构明确将计算（路由器、摄取器、查询器、压缩器）与存储（对象存储、目录）分离。这允许根据工作负载独立扩展组件，提高容错性，并降低成本 </font><sup>**<font style="color:rgb(87, 91, 95);">23</font>**</sup><font style="color:rgb(27, 28, 29);">。这是现代云原生数据库设计的标志，超越了单体存储引擎。</font>

#### <font style="color:rgb(27, 28, 29);">对可伸缩性、实时查询性能和存储成本降低的影响</font>
+ **<font style="color:rgb(27, 28, 29);">可伸缩性：</font>**<font style="color:rgb(27, 28, 29);"> 路由器、摄取器、查询器和压缩器组件的水平扩展 </font><sup>**<font style="color:rgb(87, 91, 95);">27</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">实时查询性能：</font>**<font style="color:rgb(27, 28, 29);"> 摄取器使尚未持久化的数据可供查询器使用，确保查询结果包含最新数据 </font><sup>**<font style="color:rgb(87, 91, 95);">27</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">存储成本降低：</font>**<font style="color:rgb(27, 28, 29);"> 高数据压缩（Parquet </font><sup>**<font style="color:rgb(87, 91, 95);">23</font>**</sup><font style="color:rgb(27, 28, 29);">）和利用对象存储 </font><sup>**<font style="color:rgb(87, 91, 95);">23</font>**</sup><font style="color:rgb(27, 28, 29);"> 显著减少存储占用（高达 90% 或 4.5 倍 </font><sup>**<font style="color:rgb(87, 91, 95);">23</font>**</sup><font style="color:rgb(27, 28, 29);">）。</font>
+ **<font style="color:rgb(27, 28, 29);">互操作性：</font>**<font style="color:rgb(27, 28, 29);"> Apache Iceberg 和 Parquet 实现了与数据湖和数据仓库的零拷贝、无 ETL 数据共享 </font><sup>**<font style="color:rgb(87, 91, 95);">23</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

<font style="color:rgb(27, 28, 29);">InfluxDB 3.0 的架构反映了数据库设计中向云原生原则的更广泛趋势：弹性、通过对象存储实现的成本效益和互操作性。这种演进是由时间序列数据巨大的规模以及将其无缝集成到更大企业数据战略中的需求所驱动的，从而超越了专业化的数据孤岛。</font>

### <font style="color:rgb(27, 28, 29);">D. TSM 针对时间序列工作负载的特定优化</font>
#### <font style="color:rgb(27, 28, 29);">时间戳数据的高写入和查询吞吐量</font>
<font style="color:rgb(27, 28, 29);">TSM 从头开始设计，旨在处理时间戳数据的高写入和查询负载 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。它通过在内存中缓冲写入（缓存）并将其顺序刷新到磁盘上的 TSM 文件来实现高写入吞吐量 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。通过将缓存数据与 TSM 文件合并并使用 TSI，优化了实时查询 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

#### <font style="color:rgb(27, 28, 29);">高效的数据压缩和存储成本管理</font>
<font style="color:rgb(27, 28, 29);">TSM 文件包含排序、压缩的时间序列数据 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。通过高数据压缩（增量编码、列式格式）和利用对象存储，实现了显著的存储成本降低（在 InfluxDB 3.0 中高达 90% 或 4.5 倍） </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

#### <font style="color:rgb(27, 28, 29);">支持时间序列特定操作</font>
+ **<font style="color:rgb(27, 28, 29);">数据生命周期管理：</font>**<font style="color:rgb(27, 28, 29);"> 支持批量删除数据的保留策略 </font><sup>**<font style="color:rgb(87, 91, 95);">20</font>**</sup><font style="color:rgb(27, 28, 29);">。InfluxDB 按保留策略和时间间隔将数据组织到分片组中，从而实现高效的批量删除操作 </font><sup>**<font style="color:rgb(87, 91, 95);">20</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">汇总和降采样：</font>**<font style="color:rgb(27, 28, 29);"> 提供根据指定时间窗口汇总数据并保存到新表的功能，原始数据和汇总数据具有不同的生命周期 </font><sup>**<font style="color:rgb(87, 91, 95);">20</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">专业分析功能：</font>**<font style="color:rgb(27, 28, 29);"> 原生支持时间序列特定功能，如插值、时间加权平均、移动平均和累积和，这些对于时间序列分析至关重要 </font><sup>**<font style="color:rgb(87, 91, 95);">21</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">连续查询：</font>**<font style="color:rgb(27, 28, 29);"> 时间序列应用程序通常会定期在滑动时间窗口上运行查询，以填充仪表盘、生成报告和降采样数据 </font><sup>**<font style="color:rgb(87, 91, 95);">21</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

<font style="color:rgb(27, 28, 29);">TSM 的优化超越了原始的读写性能；它们解决了时间序列数据的整个生命周期和分析工作流。这不仅包括存储和检索，还包括数据老化、聚合和专门的计算。这种整体方法才是将专用 TSDB 与通用数据库（即使是使用 LSM 树的数据库）真正区分开来的关键。</font>

#### <font style="color:rgb(27, 28, 29);">应对高基数挑战</font>
<font style="color:rgb(27, 28, 29);">时间序列索引（TSI）专门设计用于即使数据基数（唯一序列的数量）增长也能保持查询速度 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。InfluxDB 3.0 支持“无限标签基数” </font><sup>**<font style="color:rgb(87, 91, 95);">27</font>**</sup><font style="color:rgb(27, 28, 29);">，这是处理多样化元数据的重大进步。</font>

**<font style="color:rgb(27, 28, 29);">表：TSM 的时间序列特定优化</font>**

| <font style="color:rgb(27, 28, 29);">优化特性</font> | <font style="color:rgb(27, 28, 29);">描述</font> | <font style="color:rgb(27, 28, 29);">对时间序列数据的益处</font> |
| --- | --- | --- |
| <font style="color:rgb(27, 28, 29);">列式存储</font> | <font style="color:rgb(27, 28, 29);">按列存储数据，按系列键分组，按时间排序</font> | <font style="color:rgb(27, 28, 29);">提高压缩率；查询时只读取相关字段，提升效率</font> |
| <font style="color:rgb(27, 28, 29);">增量压缩</font> | <font style="color:rgb(27, 28, 29);">存储值之间的差异（增量）</font> | <font style="color:rgb(27, 28, 29);">极大地利用时间局部性，实现高压缩比</font> |
| <font style="color:rgb(27, 28, 29);">时间序列索引（TSI）</font> | <font style="color:rgb(27, 28, 29);">专门索引系列元数据（测量、标签、字段）</font> | <font style="color:rgb(27, 28, 29);">即使基数高也能保持查询速度；高效的元数据查找</font> |
| <font style="color:rgb(27, 28, 29);">多阶段压缩</font> | <font style="color:rgb(27, 28, 29);">快照、分级、索引优化、完全压缩等不同阶段</font> | <font style="color:rgb(27, 28, 29);">与数据生命周期对齐；针对不同数据“冷热”程度进行优化</font> |
| <font style="color:rgb(27, 28, 29);">数据生命周期管理</font> | <font style="color:rgb(27, 28, 29);">支持保留策略、批量删除和数据汇总</font> | <font style="color:rgb(27, 28, 29);">降低存储成本；简化数据管理；支持不同粒度数据</font> |
| <font style="color:rgb(27, 28, 29);">基于时间的查询优化</font> | <font style="color:rgb(27, 28, 29);">针对时间范围、聚合、降采样等查询进行优化</font> | <font style="color:rgb(27, 28, 29);">提高时间序列分析效率和响应速度</font> |
| <font style="color:rgb(27, 28, 29);">高基数处理</font> | <font style="color:rgb(27, 28, 29);">通过 TSI 和架构设计支持大量唯一时间序列</font> | <font style="color:rgb(27, 28, 29);">确保数据库在面对复杂、多样化数据时仍能保持高性能</font> |


## <font style="color:rgb(27, 28, 29);">IV. 对比分析：LSM 树与 TSM</font>
### <font style="color:rgb(27, 28, 29);">A. 共享架构原理与相似性</font>
<font style="color:rgb(27, 28, 29);">LSM 树和 TSM 都采用分层架构，包含一个内存组件（Memtable/Cache）用于热数据，以及基于磁盘的组件（SSTables/TSM 文件）用于持久存储 </font><sup>**<font style="color:rgb(87, 91, 95);">2</font>**</sup><font style="color:rgb(27, 28, 29);">。这种分离允许每个层级针对其各自的存储介质进行优化。两者的核心原则都是将随机写入操作转换为磁盘上的顺序追加 </font><sup>**<font style="color:rgb(87, 91, 95);">1</font>**</sup><font style="color:rgb(27, 28, 29);">。这通过在内存中缓冲写入并以排序批处理的方式刷新来实现。两种架构都严重依赖周期性后台压缩过程来合并数据、删除过时记录（墓碑、旧版本）、回收空间并保持读取效率 </font><sup>**<font style="color:rgb(87, 91, 95);">2</font>**</sup><font style="color:rgb(27, 28, 29);">。两者都利用预写日志（WAL）来确保已确认的写入是持久的，并且在系统故障时可以恢复，即使内存缓冲区丢失 </font><sup>**<font style="color:rgb(87, 91, 95);">5</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

<font style="color:rgb(27, 28, 29);">TSM 并非一个根本不同的概念，而是 LSM 树范式的高度演进和专业化适应。它的核心优势在于将 LSM 原理应用于特定的数据领域（时间序列），其特性允许独特而强大的优化。这种共享的基础使得 LSM 成为理解 TSM 的先决条件。</font>

**<font style="color:rgb(27, 28, 29);">表：核心组件比较（LSM 树 vs. TSM）</font>**

| <font style="color:rgb(27, 28, 29);">组件类别</font> | <font style="color:rgb(27, 28, 29);">通用 LSM 树术语</font> | <font style="color:rgb(27, 28, 29);">TSM 术语（InfluxDB 1.x/2.x/3.0）</font> | <font style="color:rgb(27, 28, 29);">关键相似性/差异</font> |
| --- | --- | --- | --- |
| <font style="color:rgb(27, 28, 29);">内存写入缓冲区</font> | <font style="color:rgb(27, 28, 29);">Memtable (C0)</font> | <font style="color:rgb(27, 28, 29);">Memtable (InfluxDB 3.0 Ingester)</font> | <font style="color:rgb(27, 28, 29);">两者都用于缓冲写入</font> |
| <font style="color:rgb(27, 28, 29);">内存读取缓冲区</font> | <font style="color:rgb(27, 28, 29);">（隐式 Memtable）</font> | <font style="color:rgb(27, 28, 29);">Cache (InfluxDB 1.x/2.x)</font> | <font style="color:rgb(27, 28, 29);">TSM Cache 是显式的热数据读取缓冲区</font> |
| <font style="color:rgb(27, 28, 29);">持久日志</font> | <font style="color:rgb(27, 28, 29);">WAL</font> | <font style="color:rgb(27, 28, 29);">WAL</font> | <font style="color:rgb(27, 28, 29);">两者都确保写入持久性</font> |
| <font style="color:rgb(27, 28, 29);">磁盘不可变文件</font> | <font style="color:rgb(27, 28, 29);">SSTables</font> | <font style="color:rgb(27, 28, 29);">TSM Files / Parquet Files (InfluxDB 3.0)</font> | <font style="color:rgb(27, 28, 29);">TSM 文件是列式且针对时间序列优化</font> |
| <font style="color:rgb(27, 28, 29);">索引</font> | <font style="color:rgb(27, 28, 29);">通用索引 / 布隆过滤器</font> | <font style="color:rgb(27, 28, 29);">时间序列索引 (TSI)</font> | <font style="color:rgb(27, 28, 29);">TSM 具有针对高基数的专业索引</font> |


### <font style="color:rgb(27, 28, 29);">B. 关键区别因素与专业化</font>
#### <font style="color:rgb(27, 28, 29);">数据模型与索引</font>
+ **<font style="color:rgb(27, 28, 29);">LSM 树（通用）：</font>**<font style="color:rgb(27, 28, 29);"> 通常处理通用键值对 </font><sup>**<font style="color:rgb(87, 91, 95);">2</font>**</sup><font style="color:rgb(27, 28, 29);">。索引通常基于键，并使用布隆过滤器提高效率 </font><sup>**<font style="color:rgb(87, 91, 95);">2</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">TSM（时间序列专用）：</font>**<font style="color:rgb(27, 28, 29);"> 采用时间序列专用数据模型，包含时间戳、序列键（测量、标签集、字段）和字段值 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。具有专门的时间序列索引（TSI），用于高效查找序列元数据和处理高基数 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。TSM 文件内的索引按键的字典顺序，然后按时间排序 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

<font style="color:rgb(27, 28, 29);">TSM 的索引是“语义感知”时间序列数据的。它不仅仅索引一个通用键；它将时间序列的组件（测量、标签、字段）和时间本身作为独立的、可查询的维度进行索引。这使得高度优化的时间范围查询和高效处理“高基数”问题成为可能，而这对于 TSDB 中的通用索引来说是一个重大挑战。</font>

#### <font style="color:rgb(27, 28, 29);">压缩策略</font>
+ **<font style="color:rgb(27, 28, 29);">LSM 树（通用）：</font>**<font style="color:rgb(27, 28, 29);"> 使用通用压缩策略，如分层、大小分层或通用压缩，这些策略针对一般的写入/读取/空间权衡进行了优化 </font><sup>**<font style="color:rgb(87, 91, 95);">5</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">TSM（时间感知和索引优化）：</font>**<font style="color:rgb(27, 28, 29);"> 采用多阶段压缩（快照、级别压缩、索引优化、完全压缩），专门为时间序列数据的生命周期和查询模式而设计 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。包含明确的“索引优化”压缩 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

<font style="color:rgb(27, 28, 29);">TSM 的压缩策略与时间序列数据生命周期（热数据与冷数据、数据过期）深度集成。这允许更具针对性和高效的资源分配：对最新数据进行频繁、轻量级的压缩，对较旧的归档数据进行不那么频繁、更密集的压缩，包括索引重构。</font>

#### <font style="color:rgb(27, 28, 29);">压缩</font>
+ **<font style="color:rgb(27, 28, 29);">LSM 树（通用）：</font>**<font style="color:rgb(27, 28, 29);"> 可能使用通用数据压缩技术 </font><sup>**<font style="color:rgb(87, 91, 95);">30</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">TSM（高度优化，时间序列列式）：</font>**<font style="color:rgb(27, 28, 29);"> 利用专门的列式压缩技术，如增量编码和类型特定编码器，这些技术对于时间序列数据中可预测的模式非常有效 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。InfluxDB 3.0 通过 Apache Parquet 进一步增强了这一点 </font><sup>**<font style="color:rgb(87, 91, 95);">23</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

<font style="color:rgb(27, 28, 29);">TSM 的压缩策略是利用时间序列数据固有模式（例如，随时间推移的小变化，字段列中相似的值）的典范。这导致显著更高的压缩比和更低的存储成本，与对任意键值数据进行通用压缩相比。</font>

#### <font style="color:rgb(27, 28, 29);">读取模式</font>
+ **<font style="color:rgb(27, 28, 29);">LSM 树（通用）：</font>**<font style="color:rgb(27, 28, 29);"> 针对通用点查找和范围扫描进行优化，通常需要多级别搜索 </font><sup>**<font style="color:rgb(87, 91, 95);">1</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">TSM（时间范围、聚合、分析查询）：</font>**<font style="color:rgb(27, 28, 29);"> 专门针对时间范围查询、聚合（例如，时间窗口内的平均值、总和）、降采样和其他时间序列专用分析功能进行优化 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

<font style="color:rgb(27, 28, 29);">时间序列数据上主要执行的查询类型与通用键值查找根本不同。TSM 的设计通过优化基于时间的范围扫描和聚合来反映这一点，这些对于从时间序列数据中获取洞察至关重要，而不仅仅是检索单个点。</font>

### <font style="color:rgb(27, 28, 29);">C. 性能权衡与工作负载适用性</font>
#### <font style="color:rgb(27, 28, 29);">写入放大</font>
+ **<font style="color:rgb(27, 28, 29);">LSM 树（通用）：</font>**<font style="color:rgb(27, 28, 29);"> 写入放大因压缩策略而异 </font><sup>**<font style="color:rgb(87, 91, 95);">17</font>**</sup><font style="color:rgb(27, 28, 29);">。分层压缩通常具有较高的 WA，而大小分层压缩具有较低的 WA </font><sup>**<font style="color:rgb(87, 91, 95);">17</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">TSM：</font>**<font style="color:rgb(27, 28, 29);"> 由于其追加写入性质和时间窗口压缩（类似于 TWCS </font><sup>**<font style="color:rgb(87, 91, 95);">17</font>**</sup><font style="color:rgb(27, 28, 29);">），TSM 对于典型的时间序列工作负载可以实现相对较低的写入放大，因为旧数据很少更新，并且通常在没有大量重写的情况下过期。然而，激进的压缩阶段（如完全压缩）仍然会产生 WA </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

<font style="color:rgb(27, 28, 29);">虽然 LSM 树固有地面临写入放大问题，但 TSM 的设计利用了时间序列数据追加写入的性质和时间感知压缩，以缓解其领域中常见情况下的写入放大。这并非一个通用解决方案，而是一种领域特定的优化。</font>

#### <font style="color:rgb(27, 28, 29);">读取放大与延迟</font>
+ **<font style="color:rgb(27, 28, 29);">LSM 树（通用）：</font>**<font style="color:rgb(27, 28, 29);"> 由于多级别搜索和压缩频繁导致缓存数据失效，可能会导致较高的读取延迟和较低的吞吐量 </font><sup>**<font style="color:rgb(87, 91, 95);">1</font>**</sup><font style="color:rgb(27, 28, 29);">。布隆过滤器有助于缓解此问题 </font><sup>**<font style="color:rgb(87, 91, 95);">2</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">TSM：</font>**<font style="color:rgb(27, 28, 29);"> 旨在优化时间序列数据的读取 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。其列式格式、基于时间的索引和专门的多阶段压缩（包括索引优化 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">）减少了磁盘读取并改进了时间范围查询的数据组织。虽然 InfluxDB (TSM) 在简单的单指标汇总方面可能更快，但 TimescaleDB (基于 B 树) 在高基数和复杂查询方面通常表现更优 </font><sup>**<font style="color:rgb(87, 91, 95);">24</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

<font style="color:rgb(27, 28, 29);">TSM 的读取优化高度针对时间序列特定的查询模式（时间范围、聚合、特定字段）。虽然这使其在其利基市场中非常高效，但与其它优化结构（例如 TimescaleDB 中的 B 树 </font><sup>**<font style="color:rgb(87, 91, 95);">24</font>**</sup><font style="color:rgb(27, 28, 29);">）相比，它不一定能为通用点查找或非常高基数场景带来卓越性能。“多级别搜索”是 LSM 派生结构固有的，仍然存在。</font>

#### <font style="color:rgb(27, 28, 29);">存储效率</font>
+ **<font style="color:rgb(27, 28, 29);">LSM 树（通用）：</font>**<font style="color:rgb(27, 28, 29);"> 压缩通过消除冗余和压缩数据来优化存储 </font><sup>**<font style="color:rgb(87, 91, 95);">6</font>**</sup><font style="color:rgb(27, 28, 29);">。然而，由于旧版本和压缩前的墓碑，空间放大可能是一个问题 </font><sup>**<font style="color:rgb(87, 91, 95);">6</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">TSM：</font>**<font style="color:rgb(27, 28, 29);"> 展示了卓越的时间序列数据压缩能力 </font><sup>**<font style="color:rgb(87, 91, 95);">16</font>**</sup><font style="color:rgb(27, 28, 29);">。其列式格式、增量编码和自适应压缩策略非常有效，可显著降低存储成本（高达 90% </font><sup>**<font style="color:rgb(87, 91, 95);">23</font>**</sup><font style="color:rgb(27, 28, 29);">）。</font>

<font style="color:rgb(27, 28, 29);">对于通常数据量巨大且长期存在的时间序列数据，存储成本是一个主要问题。TSM 的高级压缩不仅是性能调整，更是核心价值主张，直接解决了存储海量时间序列数据的经济可行性问题。</font>

#### <font style="color:rgb(27, 28, 29);">可伸缩性</font>
+ **<font style="color:rgb(27, 28, 29);">LSM 树（通用）：</font>**<font style="color:rgb(27, 28, 29);"> 由于能够管理多级别数据并通过压缩优化存储，因此可以有效扩展 </font><sup>**<font style="color:rgb(87, 91, 95);">2</font>**</sup><font style="color:rgb(27, 28, 29);">。分布式 LSM 树实现（例如，Cassandra、HBase）实现了水平可伸缩性 </font><sup>**<font style="color:rgb(87, 91, 95);">2</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>
+ **<font style="color:rgb(27, 28, 29);">TSM：</font>**<font style="color:rgb(27, 28, 29);"> 专为时间戳数据的高写入和查询负载而设计 </font><sup>**<font style="color:rgb(87, 91, 95);">13</font>**</sup><font style="color:rgb(27, 28, 29);">。通过 TSI 解决了高基数挑战 </font><sup>**<font style="color:rgb(87, 91, 95);">11</font>**</sup><font style="color:rgb(27, 28, 29);">。InfluxDB 3.0 的分布式、解耦架构进一步增强了可伸缩性 </font><sup>**<font style="color:rgb(87, 91, 95);">23</font>**</sup><font style="color:rgb(27, 28, 29);">。</font>

<font style="color:rgb(27, 28, 29);">虽然核心的 LSM/TSM 结构可以有效地处理单个节点上的数据，但对于海量数据集的真正可伸缩性需要分布式系统设计。InfluxDB 3.0 的演进 </font><sup>**<font style="color:rgb(87, 91, 95);">27</font>**</sup><font style="color:rgb(27, 28, 29);"> 表明了从优化单节点引擎到设计完全分布式系统的转变，这反映了现代数据挑战日益增长的规模。</font>

**<font style="color:rgb(27, 28, 29);">表：性能特性与工作负载适用性（LSM 树 vs. TSM）</font>**

| <font style="color:rgb(27, 28, 29);">特性</font> | <font style="color:rgb(27, 28, 29);">通用 LSM 树</font> | <font style="color:rgb(27, 28, 29);">TSM</font> |
| --- | --- | --- |
| <font style="color:rgb(27, 28, 29);">主要优化目标</font> | <font style="color:rgb(27, 28, 29);">写入密集型工作负载</font> | <font style="color:rgb(27, 28, 29);">时间序列数据的高写入和查询吞吐量</font> |
| <font style="color:rgb(27, 28, 29);">写入吞吐量</font> | <font style="color:rgb(27, 28, 29);">非常高（取决于压缩策略）</font> | <font style="color:rgb(27, 28, 29);">非常高，尤其适用于追加写入的工作负载</font> |
| <font style="color:rgb(27, 28, 29);">读取延迟（点查询）</font> | <font style="color:rgb(27, 28, 29);">较高（多级别搜索），但布隆过滤器可缓解</font> | <font style="color:rgb(27, 28, 29);">针对时间序列数据优化，但高基数下可能受限</font> |
| <font style="color:rgb(27, 28, 29);">读取延迟（范围查询）</font> | <font style="color:rgb(27, 28, 29);">良好（通过排序运行和布隆过滤器）</font> | <font style="color:rgb(27, 28, 29);">卓越（列式存储、时间索引、压缩）</font> |
| <font style="color:rgb(27, 28, 29);">存储效率</font> | <font style="color:rgb(27, 28, 29);">良好（通过压缩），但存在空间放大</font> | <font style="color:rgb(27, 28, 29);">卓越（增量编码、列式压缩、对象存储）</font> |
| <font style="color:rgb(27, 28, 29);">更新/删除处理</font> | <font style="color:rgb(27, 28, 29);">非原地更新，通过墓碑和压缩清理</font> | <font style="color:rgb(27, 28, 29);">非原地更新，通过墓碑和多阶段压缩清理；数据通常不可变</font> |
| <font style="color:rgb(27, 28, 29);">高基数处理</font> | <font style="color:rgb(27, 28, 29);">挑战（依赖通用索引和布隆过滤器）</font> | <font style="color:rgb(27, 28, 29);">专门优化（时间序列索引 TSI，InfluxDB 3.0 支持无限标签基数）</font> |


## <font style="color:rgb(27, 28, 29);">V. 结论</font>
<font style="color:rgb(27, 28, 29);">LSM 树作为一种写入优化型数据结构，通过其内存-磁盘分层架构、对顺序写入的依赖以及批处理和压缩机制，为处理高写入负载提供了坚实的基础。它在通用键值存储和事务日志等领域表现出色，通过将随机 I/O 转化为顺序 I/O，显著降低了成本性能。然而，其固有的读取放大和空间放大问题，需要通过精细的压缩策略来权衡写入、读取和存储效率。</font>

<font style="color:rgb(27, 28, 29);">时间结构合并树（TSM）并非 LSM 树的替代品，而是其在时间序列数据领域的专业化和高度演进。TSM 继承了 LSM 树的核心优势，并针对时间序列数据独有的高摄取率、追加写入性质、时间依赖性查询、数据生命周期管理和高基数等特性进行了深度定制。通过引入时间序列索引（TSI）、列式存储、增量编码等高级压缩技术以及时间感知多阶段压缩策略，TSM 能够实现卓越的存储效率和针对时间序列特定查询模式的优化读取性能。</font>

<font style="color:rgb(27, 28, 29);">InfluxDB 3.0 架构的演进，特别是其向基于 Rust、利用 Apache Arrow、DataFusion 和 Parquet 的云原生、分布式架构的转变，进一步凸显了时间序列数据库领域的发展方向。这种架构将计算与存储分离，提升了可伸缩性、实时查询能力和存储成本效益，并实现了与更广泛数据生态系统的无缝互操作性。</font>

<font style="color:rgb(27, 28, 29);">在选择存储引擎时，核心在于理解工作负载的特性和优先级。对于通用、写入密集型应用，LSM 树及其各种压缩策略提供了灵活且高性能的解决方案。而对于需要处理海量时间戳数据、执行复杂时间范围分析、并关注存储成本和高基数挑战的应用，TSM 及其专门优化则提供了更具针对性和高效的解决方案。未来的发展将继续在云原生、开放标准和领域特定优化方向上深化，以应对日益增长的数据规模和多样化的分析需求。</font>

