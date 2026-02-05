言归正传，本篇文章主要介绍 InfluxDB TSM [存储引擎](https://zhida.zhihu.com/search?content_id=233245422&content_type=Article&match_order=3&q=%E5%AD%98%E5%82%A8%E5%BC%95%E6%93%8E&zhida_source=entity)之 TSMFile。为了保证时序数据写入的高效，InfluxDB 采用 LSM 结构，数据先写入内存以及 WAL，当内存容量达到一定阈值之后 flush 成文件，文件数超过一定阈值执行合并。这个套路与其他 LSM 系统（比如 HBase）大同小异。不过，InfluxDB 在 LSM [体系架构](https://zhida.zhihu.com/search?content_id=233245422&content_type=Article&match_order=2&q=%E4%BD%93%E7%B3%BB%E6%9E%B6%E6%9E%84&zhida_source=entity)的基础上针对时序数据做了针对性的存储改进，官方称改进后的存储引擎为 TSM（Time-Structured Merge Tree）结构引擎。本篇文章主要集中介绍 TSM 引擎中文件格式针对时序数据做了哪些针对性的改进，才使得 InfluxDB 在处理时序数据存储、读写方面表现的如此优秀。  
**TSM 引擎核心基石－时间线**  
《时序系列文章之时序数据存储模型设计》介绍了 InfluxDB 时间线的概念以及相比其他时序数据库在[数据模型](https://zhida.zhihu.com/search?content_id=233245422&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E6%A8%A1%E5%9E%8B&zhida_source=entity)设计上的优势，有兴趣的童鞋可以详细阅读上篇文章。这里做一个简单的概括，InfluxDB 在时序数据模型设计方面提出了一个非常重要的概念：SeriesKey。SeriesKey 实际上就是 measurement+[datasource](https://zhida.zhihu.com/search?content_id=233245422&content_type=Article&match_order=1&q=datasource&zhida_source=entity)(tags)。时序数据写入内存之后按照 SeriesKey 进行组织：

<!-- 这是一张图片，ocr 内容为：FIELDL TIMESTAMPN'VALUCN TIMESTAMP LLVALUE L TIMESTAMP2.VALUC2 FICLD2 SERIESKEY FIELD3 -->
![](https://cdn.nlark.com/yuque/0/2025/png/49518801/1745064433312-300f1910-9f04-45e4-9cec-ab612528fa7c.png)

  
我们可以认为 SeriesKey 就是一个[数据源](https://zhida.zhihu.com/search?content_id=233245422&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E6%BA%90&zhida_source=entity)，源源不断的产生时序数据，只要数据源还在，时序数据就会一直产生。举个简单的例子，SeriesKey 可以认为是智能手环，智能手环可以有多个采集组件，比如说心跳采集器、脉搏采集器等，每个采集器好比上图中 [field1](https://zhida.zhihu.com/search?content_id=233245422&content_type=Article&match_order=1&q=field1&zhida_source=entity)、field2 和 field3。心跳采集器可以不间断的采集用户的[心跳信息](https://zhida.zhihu.com/search?content_id=233245422&content_type=Article&match_order=1&q=%E5%BF%83%E8%B7%B3%E4%BF%A1%E6%81%AF&zhida_source=entity)，心跳信息就是紫色方框表示的时序数据序列。当然，智能手环非常之多，使用 SeriesKey 要唯一表示某个智能手环，就必须使用标签唯一刻画出该智能手环，所以 SeriesKey 设计为 measurement+tags，其中 measurement 表示智能手环（类似于通常意义上的表），tags 用多组维度来唯一表示该智能手环，比如用智能手环用户 + 智能手环型号两个标签来表示。  
**TSM 引擎工作原理－时序数据写入**  
<font style="background-color:#FBDE28;">InfluxDB 在内存中使用一个 Map 来存储时间线数据，这个 Map 可以表示为 >。其中 Key 表示为 seriesKey+fieldKey，Map 中一个 Key 对应一个 List，List 中存储时间线数据。其实这是个非常自然的想法，并没有什么高深的难点。基于 Map 这样的</font>[<font style="background-color:#FBDE28;">数据结构</font>](https://zhida.zhihu.com/search?content_id=233245422&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84&zhida_source=entity)<font style="background-color:#FBDE28;">，时序数据写入内存流程可以表示为如下三步：</font>

:::danger
1. [时间序列数据](https://zhida.zhihu.com/search?content_id=233245422&content_type=Article&match_order=1&q=%E6%97%B6%E9%97%B4%E5%BA%8F%E5%88%97%E6%95%B0%E6%8D%AE&zhida_source=entity)进入系统之后首先根据 measurement + datasource(tags)拼成 seriesKey
2. 根据这个 seriesKey 以及待查 fieldKey 拼成 Key，再在 Map 中根据 Key 找到对应的时间序列集合，如果没有的话就新建一个新的 List
3. 找到之后将 Timestamp|Value 组合值追加写入时间线[数据链表](https://zhida.zhihu.com/search?content_id=233245422&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E9%93%BE%E8%A1%A8&zhida_source=entity)中

:::

**TSM 文件结构**  
每隔一段时间，内存中的时序数据就会执行 flush 操作将数据写入到文件（称为 TSM 文件），整个文件的组织和 HBase 中 HFile 基本相同，对 HFile 文件比较了解的童鞋会很容易理解。相同点主要在于两个方面：

1. 数据都是以 Block 为最小读取单元存储在文件中
2. 文件[数据块](https://zhida.zhihu.com/search?content_id=233245422&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E5%9D%97&zhida_source=entity)都有相应的类 B+ 树索引，而且数据块和索引结构存储在同一个文件中

笔者参考 InfluxDB 最新代码按照自己的理解将文件结构表示为：

<!-- 这是一张图片，ocr 内容为：SERIES DATA BLOCK SERIES DATA BLOCK SERIES DATA SECTION SERIES DATA BLOCK SERIES DATA BLOCK  SERIES INDEX BLOCK SERIES INDEX SECTION  SERIES INDEX BLOCK SERIES INDEX BLOCK FOOTER TSM文件 -->
![](https://cdn.nlark.com/yuque/0/2025/png/49518801/1745064475362-df597965-add0-4512-8878-0fd100008d1c.png)

  
TSM 文件最核心的由 Series Data Section 以及 Series Index Section 两个部分组成，其中前者表示存储时序数据的 Block，而后者存储文件级别 B+ 树索引 Block，用于在文件中快速查询时间序列数据块。  
**Series Data Block**  
上文说到时序数据在内存中表示为一个 Map：>， 其中 Key = seriesKey + fieldKey。这个 Map 执行 flush 操作形成 TSM 文件。  
Map 中一个 Key 对应一系列时序数据，因此能想到的最简单的 flush 策略是将这一系列时序数据在内存中构建成一个 Block 并持久化到文件。然而，有可能一个 Key 对应的时序数据非常之多，导致一个 Block 非常之大，超过 Block 大小阈值，因此在实际实现中有可能会将同一个 Key 对应的时序数据构建成多个连续的 Block。但是，在任何时候，同一个 Block 中只会存储同一种 Key 的数据。  
另一个需要关注的点在于，Map 会按照 Key 顺序排列并执行 flush，这是构建索引的需求。Series Data Block 文件结构如下图所示：

<!-- 这是一张图片，ocr 内容为：TIMESTAMP TYPE DELTA-DELTA编码 LENGTH TIMESTAMP SERIES DATA BLOCK TIMESTAMPS VALUES VALUE 数值编码 VALUE -->
![](https://cdn.nlark.com/yuque/0/2025/png/49518801/1745064546334-2668a5d6-f45c-4814-a7a5-99ff1f0732ac.png)

<font style="color:rgb(25, 27, 31);">Series Data Block 由四部分构成：Type、Length、Timestamps 以及 Values，分别表示意义如下：</font>

1. <font style="color:rgb(25, 27, 31);">Type：表示该 seriesKey 对应的时间序列的数据类型，数值数据类型通常为 int、long、float 以及 double 等。不同的数据类型对应不同的</font>[<font style="color:rgb(9, 64, 142);">编码方式</font>](https://zhida.zhihu.com/search?content_id=233245422&content_type=Article&match_order=1&q=%E7%BC%96%E7%A0%81%E6%96%B9%E5%BC%8F&zhida_source=entity)<font style="color:rgb(25, 27, 31);">。</font>
2. <font style="color:rgb(25, 27, 31);">Length：len(Timestamps)，用于读取 Timestamps</font><font style="color:rgb(25, 27, 31);"> </font>[<font style="color:rgb(9, 64, 142);">区域数据</font>](https://zhida.zhihu.com/search?content_id=233245422&content_type=Article&match_order=1&q=%E5%8C%BA%E5%9F%9F%E6%95%B0%E6%8D%AE&zhida_source=entity)<font style="color:rgb(25, 27, 31);">，解析 Block。</font>

<font style="color:rgb(25, 27, 31);">时序数据的时间值以及指标值在一个 Block 内部是按照列式存储的：所有的时间值存储在一起，所有的指标值存储在一起。使用</font>[<font style="color:rgb(9, 64, 142);">列式存储</font>](https://zhida.zhihu.com/search?content_id=233245422&content_type=Article&match_order=2&q=%E5%88%97%E5%BC%8F%E5%AD%98%E5%82%A8&zhida_source=entity)<font style="color:rgb(25, 27, 31);">可以极大提高系统的压缩效率。  
</font>

1. <font style="color:rgb(25, 27, 31);">Timestamps：时间值存储在一起形成的数据集，通常来说，时间序列中时间值的间隔都是比较固定的，比如每隔一秒钟采集一次的时间值间隔都是 1s，这种具有固定间隔值的时间序列压缩非常高效，TSM 采用了 Facebook 开源的 Geringei 系统中对时序时间的</font>[<font style="color:rgb(9, 64, 142);">压缩算法</font>](https://zhida.zhihu.com/search?content_id=233245422&content_type=Article&match_order=1&q=%E5%8E%8B%E7%BC%A9%E7%AE%97%E6%B3%95&zhida_source=entity)<font style="color:rgb(25, 27, 31);">：delta-delta 编码。</font>
2. <font style="color:rgb(25, 27, 31);">Values：指标值存储在一起形成的数据集，同一种 Key 对应的指标值数据类型都是相同的，由 Type 字段表征，相同类型的数据值可以很好的压缩，而且时序数据的特点决定了这些相邻时间序列的数据值基本都相差不大，因此也可以非常高效的压缩。需要注意的是，不同数据类型对应不同的编码算法。</font>

**<font style="color:rgb(25, 27, 31);">Series Index Block</font>**<font style="color:rgb(25, 27, 31);">  
</font><font style="color:rgb(25, 27, 31);">很多时候用户需要根据 Key 查询某段时间（比如最近一小时）的时序数据，如果没有索引，就会需要将整个 TSM 文件加载到内存中才能一个 Data Block 一个 Data Block 查找，这样一方面非常占用内存，另一方面查询效率非常之低。为了在不占用太多内存的前提下提高查询效率，TSM 文件引入了索引，其实 TSM 文件索引和 HFile 文件索引基本相同。TSM 文件索引数据由一系列索引 Block 组成，每个索引 Block 的结构如下图所示：</font>

<!-- 这是一张图片，ocr 内容为：MIN TIME MAX TIME INDEXENTRY ONSET INDEXENTRY SCRIES INDEX BLOCK SIZE INDEXENTRY KEYLENGTH INDEX BLOCK META KEY SCRICSKEY+FICLDKCY TYPC COUNT -->
![](https://cdn.nlark.com/yuque/0/2025/png/49518801/1745064589449-d123f1cb-70dd-4e96-aaf7-1b1ef5501ec5.png)

<font style="color:rgb(25, 27, 31);">Series Index Block 由 Index Block Meta 以及一系列 Index Entry 构成：  
</font>

1. <font style="color:rgb(25, 27, 31);">Index Block Meta 最核心的字段是 Key，表示这个索引 Block 内所有 IndexEntry 所索引的时序数据块都是该 Key 对应的时序数据。</font>
2. <font style="color:rgb(25, 27, 31);">Index Entry 表示一个索引字段，指向对应的 Series Data Block。指向的 Data Block 由 Offset 唯一确定，Offset 表示该 Data Block 在文件中的</font>[<font style="color:rgb(9, 64, 142);">偏移量</font>](https://zhida.zhihu.com/search?content_id=233245422&content_type=Article&match_order=1&q=%E5%81%8F%E7%A7%BB%E9%87%8F&zhida_source=entity)<font style="color:rgb(25, 27, 31);">，Size 表示指向的 Data Block 大小。Min Time 和 Max Time 表示指向的 Data Block 中时序数据集合的最小时间以及最大时间，用户在根据时间范围查找时可以根据这两个字段进行过滤。</font>

**<font style="color:rgb(25, 27, 31);">TSM 文件总体结构</font>**<font style="color:rgb(25, 27, 31);">  
</font><font style="color:rgb(25, 27, 31);">上文我们分别就 Series Data Block 以及 Series Index Block 进行了微观的分析，回过头我们再从宏观的角度将整个 TSM 文件表示为下图：</font>

<!-- 这是一张图片，ocr 内容为：TIMESTAMP TYPE DELTA-DELTA编码 SERIES DATA BLOCK TIMESTAMP LENGTH SERIES DATA BLOCK TIMESTAMPS VALUES VALUE 数值编码 SERIES DATA BLOCK VALUE SERIES DATA SECTION SERIES  DATA BLOCK MIN TIME MAX TIME INDEXENTRY OFFSET SERIES INDEX BLOCK INDEXENTRY SIZE SERIES INDEX SECTION SERIES INDEXBLOCK INDEXENTRY FOOTER KEYLENGTH INDEX BLOCK META SERIESKEY+FIELDKEY TSM文件 KEY SERIES INDEX BLOCK TYPE 日 COUNT -->
![](https://cdn.nlark.com/yuque/0/2025/png/49518801/1745064615784-d917d326-bd18-4d67-bfe1-8d03fd506f23.png)

**<font style="color:rgb(25, 27, 31);">TSM 引擎工作原理－时序数据读取</font>**  
<font style="color:rgb(25, 27, 31);">基于对 TSM 文件的了解，在一个文件内部根据 Key 查找一个某个时间范围的时序数据就会变得很简单，整个过程如下图所示：</font>

<!-- 这是一张图片，ocr 内容为：ROOT SERIES INDEX BLOCK SERIES INDEX BLOCK SERIES INDEX BLOCK KEY1(SERIESKEY+FIELDKEY) KEYN KEY2 MINTIME MINTIME MINTIME MINTIME 索引层:先根据KEY(SERIESKEY+FIELDKEY)过滤! MAXTIME MAXTIME MAX TIME MAXTIME 一次,再根据[MINTIME,MAXTIME]过滤一次 SERIES  DATA BLOCK SERIES DATA BLOCK SERIES DATA BLOCK SERIES DATA BLOCK -->
![](https://cdn.nlark.com/yuque/0/2025/png/49518801/1745064636563-d4689a8d-67bf-4550-a640-285a32db723a.png)

<font style="color:rgb(25, 27, 31);">上图中中间部分为</font>[<font style="color:rgb(9, 64, 142);">索引层</font>](https://zhida.zhihu.com/search?content_id=233245422&content_type=Article&match_order=1&q=%E7%B4%A2%E5%BC%95%E5%B1%82&zhida_source=entity)<font style="color:rgb(25, 27, 31);">，TSM 在启动之后就会将 TSM 文件的索引部分加载到内存，数据部分因为太大并不会直接加载到内存。用户查询可以分为三步：  
</font>

1. <font style="color:rgb(25, 27, 31);">首先根据 Key 找到对应的 SeriesIndex Block，因为 Key 是有序的，所以可以使用</font>[<font style="color:rgb(9, 64, 142);">二分查找</font>](https://zhida.zhihu.com/search?content_id=233245422&content_type=Article&match_order=1&q=%E4%BA%8C%E5%88%86%E6%9F%A5%E6%89%BE&zhida_source=entity)<font style="color:rgb(25, 27, 31);">来具体实现</font>
2. <font style="color:rgb(25, 27, 31);">找到 SeriesIndex Block 之后再根据查找的时间范围，使用[MinTime, MaxTime]索引定位到可能的 Series Data Block 列表</font>
3. <font style="color:rgb(25, 27, 31);">将满足条件的 Series Data Block 加载到内存中解压进一步使用</font>[<font style="color:rgb(9, 64, 142);">二分查找算法</font>](https://zhida.zhihu.com/search?content_id=233245422&content_type=Article&match_order=1&q=%E4%BA%8C%E5%88%86%E6%9F%A5%E6%89%BE%E7%AE%97%E6%B3%95&zhida_source=entity)<font style="color:rgb(25, 27, 31);">查找即可找到</font>

<font style="color:rgb(25, 27, 31);">文章总结  
</font><font style="color:rgb(25, 27, 31);">本文对 InfluxDB 的存储引擎 TSM 进行分析，主要介绍了 TSM 针对时序数据如何在内存中存储、在文件中存储，又如何根据文件索引实现在文件中查找时序数据。TSM 存储引擎基于 LSM 存储引擎针对时序数据做了相应的优化，确实是一款相当专业的时序数据库！</font>

  


