### 压测程序：
从github上找的influxdata公司提供的两款测试工具

influx-stress 用于写入测试

influxdb-comparisons用于查询测试

### 测试场景：
| <font style="color:rgb(38, 38, 38);">写入测试</font> | |
| --- | --- |
| <font style="color:rgb(38, 38, 38);">工具名称</font> | <font style="color:rgb(38, 38, 38);">influx-stress   </font> |
| <font style="color:rgb(38, 38, 38);">工具github地址</font> | [<font style="color:rgb(19, 102, 236);">https://github.com/influxdata/influx-stress</font>](https://github.com/influxdata/influx-stress?spm=a2c6h.12873639.article-detail.4.19e8b578wp5b7c) |
| <font style="color:rgb(38, 38, 38);">测试原理</font> | <font style="color:rgb(38, 38, 38);">该工具是通过go语言的fasthttp库编写的。</font>   <font style="color:rgb(38, 38, 38);">1.     会在服务器上创建一个数据库stress</font>   <font style="color:rgb(38, 38, 38);">2.     然后创建一个MEASUREMENT（类似关系数据库的表）名为ctr</font>   <font style="color:rgb(38, 38, 38);">该表有time,n.some三个字段</font>   <font style="color:rgb(38, 38, 38);">3.     不断的向stress数据库的ctr表插入数据，每次插入的数据都包含三个字段。每一条数据称为一个points。</font>   <font style="color:rgb(38, 38, 38);">插入数据的方法是通过influxDB的HTTP API 发送请求（POST /write?db=stress</font> |
| <font style="color:rgb(38, 38, 38);">测试命令</font> | <font style="color:rgb(38, 38, 38);">influx-stress insert -r 60s --strict --pps 200000 --host</font><font style="color:rgb(38, 38, 38);"> </font>[<font style="color:rgb(19, 102, 236);">http://10.XX.XX.XX:8086</font>](http://10.0.3.83:8086/) |




| <font style="color:rgb(38, 38, 38);">测试程序运行结果</font> | | |
| --- | --- | --- |
| <font style="color:rgb(38, 38, 38);">Points Per Second（发起请求）</font> | <font style="color:rgb(38, 38, 38);">Write Throughput(points/s)   </font><font style="color:rgb(38, 38, 38);">（数据库实际处理结果）</font> | <font style="color:rgb(38, 38, 38);">CPU平均利用率</font> |
| <font style="color:rgb(38, 38, 38);">200000</font> | <font style="color:rgb(38, 38, 38);">199713</font> | <font style="color:rgb(38, 38, 38);">33%</font> |
| <font style="color:rgb(38, 38, 38);">300000</font> | <font style="color:rgb(38, 38, 38);">299280</font> | <font style="color:rgb(38, 38, 38);">45%</font> |
| <font style="color:rgb(38, 38, 38);">400000</font> | <font style="color:rgb(38, 38, 38);">392873</font> | <font style="color:rgb(38, 38, 38);">62%</font> |
| <font style="color:rgb(38, 38, 38);">500000</font> | <font style="color:rgb(38, 38, 38);">491135</font> | <font style="color:rgb(38, 38, 38);">80%</font> |
| <font style="color:rgb(38, 38, 38);">600000</font> | <font style="color:rgb(38, 38, 38);">593542</font> | <font style="color:rgb(38, 38, 38);">90%</font> |
| <font style="color:rgb(38, 38, 38);">650000</font> | <font style="color:rgb(38, 38, 38);">606036</font> | <font style="color:rgb(38, 38, 38);">93%</font> |
| <font style="color:rgb(38, 38, 38);">700000</font> | <font style="color:rgb(38, 38, 38);">613791</font> | <font style="color:rgb(38, 38, 38);">95%</font> |


  


测试结论：最大的吞吐量为每秒写入60万条数据。这之后，每秒发送的points再多，吞吐量也不会增加，同时CPU利用率已达90%。

| 查询测试 | |
| --- | --- |
| 工具名称 | influxdb-comparisons    |
| 工具github地址 | [https://github.com/influxdata/influxdb-comparisons](https://github.com/influxdata/influxdb-comparisons) |
| 测试原理 | 该工具是通过go语言的fasthttp库编写的。   1.     会在服务器上创建一个数据库benchmark_db   2.     然后创建9个MEASUREMENT ：cpu,disk,diskio,kernel,mem,net,nginx,postgresl   每个measurement 有2160行数据。   3.     通过http GET请求"GET /query?db=benchmark_db“查询cpu这张表。   查询语句为：SELECT max(usage_user) from cpu where (hostname = 'host_0') and time >= '2016-01-01T01:16:32Z' and time < '2016-01-01T02:16:32Z' group by time(1m)   可以取出61条数据。    |
| 测试命令 | ./bulk_query_gen -query-type "1-host-1-hr" | ./query_benchmarker_influxdb -urls [http://10.XX.XX.XX:8086](http://10.XX.XX.XX:8086)<br/> -limit 1000    |


| 测试程序运行结果 | | | | |
| --- | --- | --- | --- | --- |
| 查询命令执行次数   （-limit） | 命令最短执行时间 | 每条命令平均执行时间 | 命令最大执行时间 | 总耗时 |
| 100 | 1.20ms | 1.69ms | 4.36ms | 0.2sec |
| 200 | 1.20ms | 1.71ms | 7.40ms | 0.3sec |
| 300 | 1.25ms | 1.73ms | 7.54ms | 0.5sec |
| 400 | 1.21ms | 1.71ms | 7.54ms | 0.7sec |
| 500 | 1.20ms | 1.70ms | 7.54ms | 0.8sec |
| 600 | 1.17ms | 1.67ms | 7.54ms | 1.0sec |
| 700 | 1.14ms | 1.66ms | 8.33ms | 1.2sec |
| 800 | 1.14ms | 1.65ms | 8.33ms | 1.3sec |
| 900 | 1.14ms | 1.63ms | 8.33ms | 1.5sec |
| 1000 | 1.14ms | 1.64ms | 8.33ms | 1.6sec |


测试结论：因为该工具最大只能测到读取1000条数据，所以没有继续加大压力测试。查询操作的消耗时间因为受到被查询表的数据量和查询语句的复杂性影响，所以在influxDate官方给出的被查表和查询语句下，算出来是平均每秒执行600次查询。









