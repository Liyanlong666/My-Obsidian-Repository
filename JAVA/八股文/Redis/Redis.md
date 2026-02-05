
---
## 基础
### 1.说说什么是 Redis?
[Redis](https://javabetter.cn/redis/rumen.html) 是 **Re**mote **Di**ctionary **S**ervice 三个单词中加粗字母的组合，是一种基于键值对的 NoSQL 数据库。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-96e079f9-49a3-4c55-b0a4-47d043732b62.png](redis-96e079f9-49a3-4c55-b0a4-47d043732b62.png)

但比一般的键值对，比如 [HashMap](https://javabetter.cn/collection/hashmap.html) 强大的多，Redis 中的 value 支持 string、hash、 list、set、zset、Bitmaps、 [HyperLogLog](https://www.cnblogs.com/54chensongxia/p/13803465.html)、GEO等多种数据结构。
而且因为 Redis 的所有数据都存放在内存当中，所以它的读写性能非常出色。
不仅如此，Redis 还可以将内存数据持久化到硬盘上，这样在发生类似断电或者机器故障的时候，内存中的数据并不会“丢失”。
除此之外，Redis 还提供了键过期、发布订阅、事务、流水线、Lua 脚本等附加功能，是互联网技术领域中使用最广泛的缓存中间件。
#### Redis 和 MySQL 的区别？

- Redis：数据存储在内存中的 NoSQL 数据库，读写性能非常好，是互联网技术领域中使用最广泛的缓存中间件。
- MySQL：数据存储在硬盘中的关系型数据库，适用于需要事务支持和复杂查询的场景。#### 项目里哪里用到了 Redis？
在[技术派实战项目](https://javabetter.cn/zhishixingqiu/paicoding.html)中，很多地方都用到了 Redis，比如说用户活跃排行榜、作者白名单、常用热点数据（文章标签、文章分类）、计数统计（文章点赞收藏评论数粉丝数）等等。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20240420093229.png](redis-20240420093229.png)

#### 部署过 Redis 吗？
我是直接在本地部署的单机版，只需要下载 Redis 的安装包，解压后运行 `redis-server` 命令即可。
也可以通过 Docker 拉取 Redis 镜像，然后运行容器。

```shell
docker run -d --name redis -p 6379:6379 redis
```

> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的华为一面原题：说下 Redis 和 HashMap 的区别
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的字节跳动商业化一面的原题：Redis 和 MySQL 的区别
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的农业银行面经同学 7 Java 后端面试原题：Redis 相关的基础知识
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的华为 OD 面经同学 1 一面面试原题：Redis 的了解, 部署方案?
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的农业银行面经同学 3 Java 后端面试原题：项目里哪里用到了 Redis
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的 360 面经同学 3 Java 后端技术一面面试原题：用过 redis 吗 用来干什么
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的招商银行面经同学 6 招银网络科技面试原题：了解 MySQL、Redis 吗？
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的百度面经同学 1 文心一言 25 实习 Java 后端面试原题：项目中什么地方使用了 redis 缓存，redis 为什么快？
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的国企零碎面经同学 9 面试原题：数据库用什么多（说了 Mysql 和 Redis）
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的荣耀面经同学 4 面试原题：Redis和MySQL的区别？
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的海康威视同学 4面试原题：Redis部署
### 2.Redis 可以用来干什么？
Redis 可以用来做缓存、排行榜、分布式锁等等。
①、缓存
缓存是 Redis 最常见的用途，由于 Redis 的数据存储在内存中，所以读写速度非常快，远超基于磁盘存储的数据库。使用 Redis 缓存可以极大地提高应用的响应速度和吞吐量。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-d44c2397-5994-452f-8b7b-eb85d2b87685.png](redis-d44c2397-5994-452f-8b7b-eb85d2b87685.png)

②、排行榜/计数器
Redis 的 ZSet 非常适合用来实现排行榜的功能，可以根据 score（分值）进行排序，实时展示用户的活跃度。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20240420100012.png](redis-20240420100012.png)

同时 Redis 的原子递增操作可以用来实现计数器功能。
③、分布式锁
Redis 可以实现分布式锁，用来控制跨多个进程的资源访问。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20240808101904.png](redis-20240808101904.png)

> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的农业银行面经同学 7 Java 后端面试原题：Redis 相关的基础知识
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的字节跳动同学 7 Java 后端实习一面的原题：讲一下为什么要用 Redis 去存权限列表？
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的字节跳动同学 20 测开一面的原题：redis 有什么好处，为什么用 redis
### 3.Redis 有哪些数据类型？
Redis 有五种基本数据类型，这五种数据类型分别是：string（字符串）、hash（哈希）、list（列表）、set（集合）、sorted set（有序集合，也叫 zset）。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-10434dc7-c7a3-4c1a-b484-de3fb37669ee.png](redis-10434dc7-c7a3-4c1a-b484-de3fb37669ee.png)

#### 简单介绍下 string？
字符串是最基础的数据类型，key 是一个字符串，不用多说，value 可以是：

- 字符串（简单的字符串、复杂的字符串（例如 JSON、XML））
- 数字 （整数、浮点数）
- 甚至是二进制（图片、音频、视频），但最大不能超过 512MB。字符串主要有以下几个典型的使用场景：

- 缓存功能
- 计数
- 共享 Session
- 限速#### 简单介绍下 hash？
键值对集合，key 是字符串，value 是一个 Map 集合，比如说 `value = {name: '沉默王二', age: 18}`，name 和 age 属于字段 field，沉默王二 和 18 属于值 value。
哈希主要有以下两个典型应用场景：

- 缓存用户信息
- 缓存对象#### 什么使用 hash 类型而不使用 string 类型序列化存储？
来感受一下，使用字符串类型存储用户信息和使用哈希类型存储用户信息的区别：
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20240315115713.png](redis-20240315115713.png)

可以看得出，使用 hash 比使用 string 更便于进行序列化，我们可以将一整个用户对象序列化，然后作为一个 value 存储在 Redis 中，存取更加便捷。
#### 简单介绍下 list？
list 是一个简单的字符串列表，按照插入顺序排序。可以添加一个元素到列表的头部（左边）或者尾部（右边）。
列表主要有以下两个使用场景：

- 消息队列
- 文章列表#### 简单介绍下 set？
Set 是一个无序集合，元素是唯一的，不允许重复。
#### 简单介绍下 zset？
Zset 是有序集合，比 set 多了一个排序属性 score。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20240315120652.png](redis-20240315120652.png)

可以用来实现排行榜，比如[技术派实战项目](https://javabetter.cn/zhishixingqiu/paicoding.html)中，我们就使用了 Zset 来实现用户活跃排行榜。
### 4.Redis 为什么快呢？
Redis 的速度⾮常快，单机的 Redis 就可以⽀撑每秒十几万的并发，性能是 MySQL 的⼏⼗倍。原因主要有⼏点：
①、**基于内存的数据存储**，Redis 将数据存储在内存当中，使得数据的读写操作避开了磁盘 I/O。而内存的访问速度远超硬盘，这是 Redis 读写速度快的根本原因。
②、**单线程模型**，Redis 使用单线程模型来处理客户端的请求，这意味着在任何时刻只有一个命令在执行。这样就避免了线程切换和锁竞争带来的消耗。
③、**IO 多路复⽤**，基于 Linux 的 select/epoll 机制。该机制允许内核中同时存在多个监听套接字和已连接套接字，内核会一直监听这些套接字上的连接请求或者数据请求，一旦有请求到达，就会交给 Redis 处理，就实现了所谓的 Redis 单个线程处理多个 IO 读写的请求。
④、**高效的数据结构**，Redis 提供了多种高效的数据结构，如字符串（String）、列表（List）、集合（Set）、有序集合（Sorted Set）等，这些数据结构经过了高度优化，能够支持快速的数据操作。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-e05bca61-4600-495c-b92a-25ac822e034e.png](redis-e05bca61-4600-495c-b92a-25ac822e034e.png)

### 5.能说一下 I/O 多路复用吗？
IO 多路复用是一种高效管理多个 IO 事件的技术，通过单线程监控多个文件描述符（fd），实现高并发的 IO 操作。
常见的 I/O 多路复用机制包括 select、poll 和 epoll 等。
 | 特性 | `select` | `poll` | `epoll` | 
|---|---|---|---|
 | 文件描述符限制 | 受 `FD_SETSIZE` 限制 | 无限制 | 无限制 | 
 | 时间复杂度 | O(n) | O(n) | O(1) | 
 | 数据复制 | 需要 | 需要 | 不需要 | 
 | 工作方式 | 线性扫描 | 线性扫描 | 事件通知 | 
 | 内核支持 | 所有 UNIX 系统 | 所有 UNIX 系统 | Linux 2.6 及以上版本 | 
 | 适用场景 | 少量连接 | 中等连接 | 大量并发连接 | 
比如说你是一名数学老师，上课时提出了一个问题：“今天谁来证明一下勾股定律？”
同学小王举手，你就让小王回答；小李举手，你就让小李回答；小张举手，你就让小张回答。
这种模式就是 IO 多路复用，你只需要在讲台上等，谁举手谁回答，不需要一个一个去问。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20240918114125.png](redis-20240918114125.png)

Redis 就是使用 epoll 这样的 I/O 多路复用机制，在单线程模型下实现高效的网络 I/O，从而支持高并发的请求处理。
#### 举例子说一下 I/O 多路复用？
假设你是一个老师，让 30 个学生解答一道题目，然后检查学生做的是否正确，你有下面几个选择：

- 第一种选择：按顺序逐个检查，先检查 A，然后是 B，之后是 C、D。。。这中间如果有一个学生卡住，全班都会被耽误。这种模式就好比，你用循环挨个处理 socket，根本不具有并发能力。
- 第二种选择：你创建 30 个分身，每个分身检查一个学生的答案是否正确。 这种类似于为每一个用户创建一个进程或者线程处理连接。
- 第三种选择，你站在讲台上等，谁解答完谁举手。这时 C、D 举手，表示他们解答问题完毕，你下去依次检查 C、D 的答案，然后继续回到讲台上等。此时 E、A 又举手，然后去处理 E 和 A。第一种就是阻塞 IO 模型，第三种就是 I/O 复用模型。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-eb541432-d68a-4dd9-b427-96c4dd607d64.png](redis-eb541432-d68a-4dd9-b427-96c4dd607d64.png)

Linux 系统有三种方式实现 IO 多路复用：select、poll 和 epoll。
例如 epoll 方式是将用户 socket 对应的 fd 注册进 epoll，然后 epoll 帮你监听哪些 socket 上有消息到达，这样就避免了大量的无用操作。此时的 socket 应该采用非阻塞模式。
这样，整个过程只在进行 select、poll、epoll 这些调用的时候才会阻塞，收发客户消息是不会阻塞的，整个进程或者线程就被充分利用起来，这就是事件驱动，所谓的 reactor 模式。
#### select、poll 和 epoll 的实现原理？
select 使用位图管理 fd，每次调用都需要将 fd 集合从用户态复制到内核态。最大支持 1024 个文件描述符。
poll 使用动态数组管理 fd，突破了 select 的数量限制。
epoll 使用红黑树和链表管理 fd，每次调用只需要将 fd 集合从用户态复制到内核态一次，不需要重复复制。
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的字节跳动面经同学 21  抖音商城一面面试原题：io多路复用了解吗？
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的快手同学 4 一面原题：IO多路复用中select/poll/epoll各自的实现原理和区别？
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的字节跳动面经同学19番茄小说一面面试原题：Linux中的IO多路复用
### 6. Redis 为什么早期选择单线程？
官方解释：[https://redis.io/topics/faq](https://redis.io/topics/faq)
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-344b8461-98d4-495b-a697-70275b0abad6.png](redis-344b8461-98d4-495b-a697-70275b0abad6.png)
官方 FAQ 表示，因为 Redis 是基于内存的操作，CPU 成为 Redis 的瓶颈的情况很少见，Redis 的瓶颈最有可能是内存的大小或者网络限制。
如果想要最大程度利用 CPU，可以在一台机器上启动多个 Redis 实例。
PS：网上有这样的回答，吐槽官方的解释有些敷衍，其实就是历史原因，开发者嫌多线程麻烦，后来这个 CPU 的利用问题就被抛给了使用者。
同时 FAQ 里还提到了， Redis 4.0 之后开始变成多线程，除了主线程外，它也有后台线程在处理一些较为缓慢的操作，例如清理脏数据、无用连接的释放、大 Key 的删除等等。
### 7.Redis 6.0 使用多线程是怎么回事?
单线程模型意味着 Redis 在大量 IO 请求时，无法充分利用多核 CPU 的优势。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-b7b24e25-d2dc-4457-994f-95bdb3674b8e.png](redis-b7b24e25-d2dc-4457-994f-95bdb3674b8e.png)

在 Redis 6.0 中，多线程主要用来处理网络 IO 操作，命令解析和执行仍然是单线程完成，这样既可以发挥多核 CPU 的优势，又能避免锁和上下文切换带来的性能损耗。
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的同学 30 腾讯音乐面试原题：redis6.0引入的多线程用作什么地方
### 8.说说 Redis 常用命令（补充）
> 2024 年 04 月 11 日增补

①、操作字符串的命令有：

- `SET key value`：设置键 key 的值为 value。
- `GET key`：获取键 key 的值。
- `DEL key`：删除键 key。
- `INCR key`：将键 key 存储的数值增一。
- `DECR key`：将键 key 存储的数值减一。②、操作列表的命令有：

- `LPUSH key value`：将一个值插入到列表 key 的头部。
- `RPUSH key value`：将一个值插入到列表 key 的尾部。
- `LPOP key`：移除并返回列表 key 的头元素。
- `RPOP key`：移除并返回列表 key 的尾元素。
- `LRANGE key start stop`：获取列表 key 中指定范围内的元素。③、操作集合的命令有：

- `SADD key member`：向集合 key 添加一个元素。
- `SREM key member`：从集合 key 中移除一个元素。
- `SMEMBERS key`：返回集合 key 中的所有元素。④、操作有序集合的命令有：

- `ZADD key score member`：向有序集合 key 添加一个成员，或更新其分数。
- `ZRANGE key start stop [WITHSCORES]`：按照索引区间返回有序集合 key 中的成员，可选 WITHSCORES 参数返回分数。
- `ZREVRANGE key start stop [WITHSCORES]`：返回有序集合 key 中，指定区间内的成员，按分数递减。
- `ZREM key member`：移除有序集合 key 中的一个或多个成员。⑤、操作哈希的命令有：

- `HSET key field value`：向键为 key 的哈希表中设置字段 field 的值为 value。
- `HGET key field`：获取键为 key 的哈希表中字段 field 的值。
- `HGETALL key`：获取键为 key 的哈希表中所有的字段和值。
- `HDEL key field`：删除键为 key 的哈希表中的一个或多个字段。#### 详细说说 set 命令？
在 Redis 中，设置键值对的命令是 set。set 命令有几个常用的参数：
①、可以通过 EX 或 PX 为键设置过期时间（秒或毫秒）

```shell
redis-cli SET session_id "xyz" EX 3600  # 设置键 session_id，值为 "xyz"，过期时间为 3600 秒
```
②、NX 选项表示只有键不存在时才设置

```shell
redis-cli SET lock_key "locked" NX
```
③、XX 选项表示只有键存在时才设置

```shell
redis-cli SET config "new_config" XX
```
#### sadd 命令的时间复杂度是多少？
向指定 Set 中添加 1 个或多个 member，如果指定 Set 不存在，会自动创建一个。**时间复杂度 O(N)** ，N 为添加的 member 个数。
#### incr命令了解吗？
INCR 命令是 Redis 中的一个原子操作，用于将存储在 key 中的数值加 1。
Redis 的单线程模型确保了每个命令都是原子执行的，不会被其他命令打断。
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的京东面经同学 1 Java 技术一面面试原题：说说 Redis 常用命令
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的农业银行面经同学 3 Java 后端面试原题：说的那么好，Redis 设置 key value 的函数是啥
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的快手面经同学 1 部门主站技术部面试原题：Redis 的 sadd 命令时间复杂度是多少？
### 9.单线程 Redis 的 QPS 是多少？(补充)
> 2024 年 4 月 14 日增补

Redis 的 QPS（Queries Per Second，每秒查询率）受多种因素影响，包括硬件配置（如 CPU、内存、网络带宽）、数据模型、命令类型、网络延迟等。
根据官方的基准测试，一个普通服务器的 Redis 实例通常可以达到每秒数万到几十万的 QPS。
可以通过 `redis-benchmark` 命令进行基准测试：

```shell
redis-benchmark -h 127.0.0.1 -p 6379 -c 50 -n 10000
```

- `-h`：指定 Redis 服务器的地址，默认是 127.0.0.1。
- `-p`：指定 Redis 服务器的端口，默认是 6379。
- `-c`：并发连接数，即同时有多少个客户端在进行测试。
- `-n`：请求总数，即测试过程中总共要执行多少个请求。我本机是一台 macOS，4 GHz 四核 Intel Core i7，32 GB 1867 MHz DDR3，测试结果如下：
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20240408100900.png](redis-20240408100900.png)

可以看得出，每秒能处理超过 10 万次请求。
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的字节跳动面经同学 1 Java 后端技术一面面试原题：单线程 Redis 的 QPS 是多少？
GitHub 上标星 10000+ 的开源知识库《[二哥的 Java 进阶之路](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括 Java 基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM 等等，共计 32 万余字，500+张手绘图，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 10000+ 的 Java 教程](https://javabetter.cn/overview/)
微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/gongzhonghao.png](JAVA/八股文/Redis/Redis.assert/gongzhonghao.png)

## 高可用
Redis 除了单机部署外，还可以通过主从复制、哨兵模式和集群模式来实现高可用。
**主从复制**：允许一个 Redis 服务器（主节点）将数据复制到一个或多个 Redis 服务器（从节点）。这种方式可以实现读写分离，适合读多写少的场景。
**哨兵模式**：用于监控主节点和从节点的状态，实现自动故障转移。如果主节点发生故障，哨兵可以自动将一个从节点升级为新的主节点。
**集群模式**：Redis 集群通过分片的方式存储数据，每个节点存储数据的一部分，用户请求可以并行处理。集群模式支持自动分区、故障转移，并且可以在不停机的情况下进行节点增加或删除。
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的华为 OD 面经同学 1 一面面试原题：Redis 的了解, 部署方案?
2. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的同学 30 腾讯音乐面试原题：redis的部署方式都有哪些呢，各自有什么优缺点？
### 15.主从复制了解吗？
主从复制是指将一台 Redis 服务器的数据，复制到其他的 Redis 服务器。
前者称为主节点 master，后者称为从节点slave。且数据的复制是单向的，只能由主节点到从节点。
![image.png](1747710304601-28395ca5-9f0d-4bdf-823b-ec1845f750fb.png)

在 Redis 主从架构中，主节点负责处理所有的写操作，并将这些操作异步复制到从节点。从节点主要用于读取操作，以分担主节点的压力和提高读性能。
#### 主从复制主要的作用是什么?
①、**数据冗余：** 主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
②、**故障恢复：** 如果主节点挂掉了，可以将一个从节点提升为主节点，从而实现故障的快速恢复。
通常会使用 Sentinel 哨兵来实现自动故障转移，当主节点挂掉时，Sentinel 会自动将一个从节点升级为主节点，保证系统的可用性。

```shell
# sentinel.conf

port 26379
sentinel monitor mymaster 192.168.1.1 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1
```
假如是从节点挂掉了，主节点不受影响，但应该尽快修复并重启挂掉的从节点，使其重新加入集群并从主节点同步数据。
③、**负载均衡：** 在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务 *（即写 Redis 时连接主节点，读 Redis 时连接从节点）*，分担服务器负载。尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高 Redis 服务器的并发量。
④、**高可用基石：** 除了上述作用以外，主从复制还是哨兵和集群能够实施的 **基础**。
#### 主从复制出现数据不一致怎么办？
Redis 的主从复制是异步进行的，这意味着主节点在执行完写操作后，会立即返回给客户端，而不是等待从节点完成数据同步。
在主节点将数据同步到从节点的过程中，可能会出现网络延迟或中断，从而导致从节点的数据滞后于主节点。
为了解决数据不一致的问题，应该尽量保证主从节点之间的网络连接状况良好，比如说避免在不同机房之间部署主从节点，以减少网络延迟。但可能会带来新的问题，就是整个机房都挂掉的情况。
此外，Redis 本身也提供了一些机制来解决数据不一致的问题，比如说通过 Redis 的 `INFO replication` 命令监控主从节点的复制进度，及时发现和处理复制延迟。
具体做法是获取主节点的 master_repl_offset 和从节点的 slave_repl_offset，计算两者的差值。如果差值超过预设的阈值，采取措施（如停止从节点的数据读取）以减少读到不一致数据的情况。
![image.png](1747710306251-93f0e05b-11b3-4b4f-a5c7-fc98028b9588.png)

#### Redis解决单点故障主要靠什么？
主从复制，当主节点发生故障时，可以通过手动或自动方式将某个从节点提升为新的主节点，继续对外提供服务，从而避免单点故障。
Redis 的哨兵机制（Sentinel）可以实现自动化的故障转移，当主节点宕机时，哨兵会自动将一个从节点升级为新的主节点。
另外，集群模式下，当某个节点发生故障时，Redis Cluster 会自动将请求路由到其他节点，并通过从节点进行故障恢复。
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的得物面经同学 1 面试原题：Redis 分布式，主从，一个节点挂掉怎么办
2. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的小米面经同学 F 面试原题：redis 的主从架构和主从哨兵区别
3. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的收钱吧面经同学 1 Java 后端一面面试原题：Redis解决单点故障主要靠什么？主从模式用的是异步还是同步？
### 16.Redis 主从有几种常见的拓扑结构？
Redis 的复制拓扑结构可以支持单层或多层复制关系，根据拓扑复杂性可以分为以下三种：一主一从、一主多从、树状主从结构。
1.一主一从结构
一主一从结构是最简单的复制拓扑结构，用于主节点出现宕机时从节点提供故障转移支持。
![image.png](1747710304732-44b40aa3-e461-4ef6-8e29-9a7c9c87e317.png)
 2.一主多从结构
一主多从结构（又称为星形拓扑结构）使得应用端可以利用多个从节点实现读写分离（见图 6-5）。对于读占比较大的场景，可以把读命令发送到从节点来分担主节点压力。
![image.png](1747710304709-2b96397c-5ca5-400b-ae86-beb1ae92840f.png)
 3.树状主从结构
树状主从结构（又称为树状拓扑结构）使得从节点不但可以复制主节点数据，同时可以作为其他从节点的主节点继续向下层复制。通过引入复制中间层，可以有效降低主节点负载和需要传送给从节点的数据量。
![image.png](1747710304758-edcef650-f386-4c1b-9fa5-7c97e78bf991.png)

### 17.Redis 的主从复制原理了解吗？
Redis 主从复制的工作流程大概可以分为如下几步：![image.png](1747710305583-e2f51cee-1574-40f7-a313-af6b56c1d1d7.png)


1. 保存主节点（master）信息这一步只是保存主节点信息，保存主节点的 ip 和 port。
2. 主从建立连接从节点（slave）发现新的主节点后，会尝试和主节点建立网络连接。
3. 发送 ping 命令连接建立成功后从节点发送 ping 请求进行首次通信，主要是检测主从之间网络套接字是否可用、主节点当前是否可接受处理命令。
4. 权限验证如果主节点要求密码验证，从节点必须正确的密码才能通过验证。
5. 同步数据集主从复制连接正常通信后，主节点会把持有的数据全部发送给从节点。
6. 命令持续复制接下来主节点会持续地把写命令发送给从节点，保证主从数据一致性。### 18.说说主从数据同步的方式？
Redis 在 2.8 及以上版本使用 psync 命令完成主从数据同步，同步过程分为：全量复制和部分复制。
![image.png](1747710305511-82b43907-8f48-46c6-8eec-cb6ad29444da.png)

**全量复制**一般用于初次复制场景，Redis 早期支持的复制功能只有全量复制，它会把主节点全部数据一次性发送给从节点，当数据量较大时，会对主从节点和网络造成很大的开销。
全量复制的完整运行流程如下：![image.png](1747710305669-54799e93-267d-408a-880b-a5eb61fee79f.png)


1. 发送 psync 命令进行数据同步，由于是第一次进行复制，从节点没有复制偏移量和主节点的运行 ID，所以发送 psync-1。
2. 主节点根据 psync-1 解析出当前为全量复制，回复+FULLRESYNC 响应。
3. 从节点接收主节点的响应数据保存运行 ID 和偏移量 offset
4. 主节点执行 bgsave 保存 RDB 文件到本地
5. 主节点发送 RDB 文件给从节点，从节点把接收的 RDB 文件保存在本地并直接作为从节点的数据文件
6. 对于从节点开始接收 RDB 快照到接收完成期间，主节点仍然响应读写命令，因此主节点会把这期间写命令数据保存在复制客户端缓冲区内，当从节点加载完 RDB 文件后，主节点再把缓冲区内的数据发送给从节点，保证主从之间数据一致性。
7. 从节点接收完主节点传送来的全部数据后会清空自身旧数据
8. 从节点清空数据后开始加载 RDB 文件
9. 从节点成功加载完 RDB 后，如果当前节点开启了 AOF 持久化功能， 它会立刻做 bgrewriteaof 操作，为了保证全量复制后 AOF 持久化文件立刻可用。**部分复制**部分复制主要是 Redis 针对全量复制的过高开销做出的一种优化措施， 使用 psync{runId}{offset}命令实现。当从节点（slave）正在复制主节点 （master）时，如果出现网络闪断或者命令丢失等异常情况时，从节点会向 主节点要求补发丢失的命令数据，如果主节点的复制积压缓冲区内存在这部分数据则直接发送给从节点，这样就可以保持主从节点复制的一致性。
![image.png](1747710306413-923a1d38-cc0a-4cd5-a05f-4c9e4ea81890.png)


1. 当主从节点之间网络出现中断时，如果超过 repl-timeout 时间，主节点会认为从节点故障并中断复制连接
2. 主从连接中断期间主节点依然响应命令，但因复制连接中断命令无法发送给从节点，不过主节点内部存在的复制积压缓冲区，依然可以保存最近一段时间的写命令数据，默认最大缓存 1MB。
3. 当主从节点网络恢复后，从节点会再次连上主节点
4. 当主从连接恢复后，由于从节点之前保存了自身已复制的偏移量和主节点的运行 ID。因此会把它们当作 psync 参数发送给主节点，要求进行部分复制操作。
5. 主节点接到 psync 命令后首先核对参数 runId 是否与自身一致，如果一 致，说明之前复制的是当前主节点；之后根据参数 offset 在自身复制积压缓冲区查找，如果偏移量之后的数据存在缓冲区中，则对从节点发送+CONTINUE 响应，表示可以进行部分复制。
6. 主节点根据偏移量把复制积压缓冲区里的数据发送给从节点，保证主从复制进入正常状态。### 19.主从复制存在哪些问题呢？
Redis 主从复制虽然实现了读写分离和数据备份，但也存在一些明显的缺点：

- 由于主从复制是异步的，如果主节点在数据尚未完全同步到从节点时崩溃，会导致数据丢失。
- 写操作集中在主节点，从节点只能处理读操作，无法分担写入压力。
- 在网络分区的情况下，主节点和从节点可能无法相互通信，导致两个节点都被认为是主节点，形成多个主节点的情况，也就是脑裂。#### 脑裂问题了解吗？
Redis 的脑裂问题是指在主从模式或集群模式下，由于网络分区或节点故障，可能导致系统中出现多个主节点，从而引发数据不一致、数据丢失等问题。
可以通过 Sentinel 模式和 Cluster 模式中的投票机制和强制下线机制来解决。
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的同学 30 腾讯音乐面试原题：主从复制有什么缺点呢？redis的脑裂问题
### 20.Redis 哨兵了解吗？
哨兵（Sentinel）机制是 Redis 提供的一个高可用性解决方案，主要用来监控 Redis 主从架构中的实例，并在主节点出现故障时，自动进行故障转移。
![image.png](1747710306256-05829c35-5c15-4dce-a426-baca4d09754f.png)

> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的比亚迪面经同学 1 面试原题：Redis 的哨兵机制了解吗？
### 21.Redis 哨兵实现原理知道吗？
哨兵的工作流程包括定时监控、主观下线和客观下线、领导者 Sentinel 节点选举、故障转移等。
![image.png](1747710306581-56403972-83db-459c-852a-18d011b3179d.png)

每个 Sentinel 实例会定期通过 PING 命令向主节点和从节点发送心跳包。
![image.png](1747710306674-a024e311-cda9-46a2-aec9-ea7fe04afaba.png)


如果一个节点长时间没有响应 PING 命令，Sentinel 会将该节点标记为主观下线。当多个 Sentinel 同时认为一个节点不可用时，该节点被标记为客观下线。
![image.png](1747710307193-03512ee4-eff1-4ac6-ab28-e26c5a867bf0.png)

当主节点被确认下线后，Sentinel 之间会通过类似 Raft 的选举算法进行协商，选出一个领导者 Sentinel 来负责执行故障转移。
![image.png](1747710307358-38663206-f1ac-413d-a215-b34e56fb7da9.png)


1. 将某个从节点提升为新的主节点。
2. 通知其他从节点重新复制新的主节点的数据。> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的 OPPO 面经同学 1 面试原题：Redis的Sentinel和Cluster怎么理解？说一下大概原理
### 22.领导者 Sentinel 节点选举了解吗？
Redis 使用 Raft 算法实现领导者选举的：当主节点挂掉后，新的主节点是由剩余的从节点发起选举后晋升的。
![image.png](1747710307599-f1dbab91-a186-4899-b28c-a1095829c2a8.png)

①、每个在线的 Sentinel 节点都有资格成为领导者，当它确认主节点下线时候，会向其他哨兵节点发送命令，表明希望由自己来执行主从切换，并让所有其他哨兵进行投票。
这个投票过程称为“Leader 选举”。候选者会给自己先投 1 票，然后向其他 Sentinel 节点发送投票的请求。
②、收到请求的 Sentinel 节点会进行判断，如果候选者的日志与自己的日志一样新，任期号也小于自己，且之前没有投票过，就会同意投票，回复 Y。否则回复 N。
③、候选者收到投票后会统计支持自己的得票数，如果候选者获得了集群中超过半数节点的投票支持（即多数原则），它将成为新的主节点。
新的主节点在确立后，会向其他从节点发送心跳信号，告诉它们自己已经成为主节点，并将其他节点的状态重置为从节点。
④、如果多个节点同时成为候选者，并且都有可能获得足够的票数，这种情况下可能会出现选票分裂。也就是没有候选者获得超过半数的选票，那么这次选举就会失败，所有候选者都会再次发起选举。
为了防止无限制的选举失败，每个节点都会有一个选举超时时间，且是随机的。
> 超时时间指从节点在没有收到主节点的心跳信号或日志追加请求后，等待多长时间才会认为主节点已挂掉，从而进入候选状态并发起选举。

推荐阅读：[Raft算法的选主过程详解](https://hoverzheng.github.io/post/technology-blog/blockchain/raft%E7%AE%97%E6%B3%95%E8%AF%A6%E8%A7%A33--%E9%80%89%E4%B8%BB/)
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的8 后端开发秋招一面面试原题：raft主节点挂了怎么选从节点
### 23.新的主节点是怎样被挑选出来的？
选出新的主节点，大概分为这么几步：![image.png](1747710307623-cc13edb2-3895-4cc8-bc46-e97a3d191200.png)


1. 过滤：“不健康”（主观下线、断线）、5 秒内没有回复过 Sentinel 节 点 ping 响应、与主节点失联超过 down-after-milliseconds*10 秒。
2. 选择 slave-priority（从节点优先级）最高的从节点列表，如果存在则返回，不存在则继续。
3. 选择复制偏移量最大的从节点（复制的最完整），如果存在则返 回，不存在则继续。
4. 选择 runid 最小的从节点。### 24.Redis 集群了解吗？
前面说到了主从存在高可用和分布式的问题，哨兵解决了高可用的问题，而集群就是终极方案，一举解决高可用和分布式问题。
![image.png](1747710307945-670ac70e-082e-4a99-8805-e705184c1c86.png)


1. **数据分区：** 数据分区 *(或称数据分片)* 是集群最核心的功能。集群将数据分散到多个节点，一方面 突破了 Redis 单机内存大小的限制，**存储容量大大增加**；**另一方面** 每个主节点都可以对外提供读服务和写服务，**大大提高了集群的响应能力**。
2. **高可用：** 集群支持主从复制和主节点的 **自动故障转移** *（与哨兵类似）*，当任一节点发生故障时，集群仍然可以对外提供服务。### 25.Redis Cluster了解吗？（补充）
> 2024 年 04 月 26 日新增

切片集群是一种将数据分片存储在多个 Redis 实例上的集群架构，每个 Redis 实例负责存储部分数据。比如说把 25G 的数据平均分为 5 份，每份 5G，然后启动 5 个 Redis 实例，每个实例保存一份数据。
![image.png](1747710308278-3a226e93-1639-4c03-9edd-18868a113490.png)

在 Redis 3.0 之前，官方并没有针对切片集群提供具体的解决方案；但是在 Redis 3.0 之后，官方提供了 Redis Cluster，数据和实例之间的映射通过哈希槽（hash slot）来实现。
Redis Cluster 有 16384 个哈希槽，每个键根据其名字的 CRC16 值被映射到这些哈希槽上。然后，这些哈希槽会被均匀地分配到所有的 Redis 实例上。
> CRC16 是一种哈希算法，它可以将任意长度的输入数据映射为一个 16 位的哈希值。

![image.png](1747710308539-26318497-62a8-4621-bd45-fdc67893b562.png)

例如，如果我们有 3 个 Redis 实例，那么每个实例可能会负责大约 5461 个哈希槽。
当需要存储或检索一个键值对时，Redis Cluster 会先计算这个键的哈希槽，然后找到负责这个哈希槽的 Redis 实例，最后在这个实例上进行操作。
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的字节跳动面经同学 1 Java 后端技术一面面试原题：Redis 切片集群？数据和实例之间的如何进行映射？
2. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的快手面经同学 1 部门主站技术部面试原题：Redis 的 cluster 集群如何实现？
### 26.集群中数据如何分区？
在 Redis 集群中，数据分区是通过将数据分散到不同的节点来实现的，常见的数据分区规则有三种：节点取余分区、一致性哈希分区、虚拟槽分区。
![image.png](1747710308499-4956da7e-6f28-4766-9286-7501ce27c763.png)

#### 说说节点取余分区
节点取余分区是一种简单的分区策略，其中数据项通过对某个值（通常是键的哈希值）进行取余操作来分配到不同的节点。
类似 HashMap 中的取余操作，数据项的键经过哈希函数计算后，对节点数量取余，然后将数据项分配到余数对应的节点上。
缺点是扩缩容时，大多数数据需要重新分配，因为节点总数的改变会影响取余结果，这可能导致大量数据迁移。
![image.png](1747710308668-eafa92ed-0808-44b3-b6ae-298a830eac6f.png)

#### 说说一致性哈希分区
一致性哈希分区的原理是：将哈希值空间组织成一个环，数据项和节点都映射到这个环上。数据项由其哈希值直接映射到环上，然后顺时针分配到遇到的第一个节点。
从而来减少节点变动时数据迁移的量。
![image.png](1747710309130-545840ff-e27b-4c38-acb6-41d89ccb57e8.png)

Key 1 和 Key 2 会落入到 Node 1 中，Key 3、Key 4 会落入到 Node 2 中，Key 5 落入到 Node 3 中，Key 6 落入到 Node 4 中。
这种方式相比节点取余最大的好处在于加入和删除节点只影响哈希环中相邻的节点，对其他节点无影响。
但它还是存在问题：

- 节点在圆环上分布不平均，会造成部分缓存节点的压力较大
- 当某个节点故障时，这个节点所要承担的所有访问都会被顺移到另一个节点上，会对后面这个节点造成压力。#### 说说虚拟槽分区？
在虚拟槽（也叫哈希槽）分区中，槽位的数量是固定的（例如 Redis Cluster 有 16384 个槽），每个键通过哈希算法（比如 CRC16）映射到这些槽上，每个集群节点负责管理一定范围内的槽。
这种分区可以灵活地将槽（以及槽中的数据）从一个节点迁移到另一个节点，从而实现平滑扩容和缩容；数据分布也更加均匀，Redis Cluster 采用的正是这种分区方式。
![image.png](1747710309515-815b7c64-baa1-4ffc-805e-bdb28bbedd48.png)

假设系统中有 4 个实际节点，假设为其分配了 16 个槽(0-15)；

- 槽 0-3 位于节点 node1；
- 槽 4-7 位于节点 node2；
- 槽 8-11 位于节点 node3；
- 槽 12-15 位于节点 node4。如果此时删除 `node2`，只需要将槽 4-7 重新分配即可，例如将槽 4-5 分配给 `node1`，槽 6 分配给 `node3`，槽 7 分配给 `node4`，数据在节点上的分布仍然较为均衡。
如果此时增加 node5，也只需要将一部分槽分配给 node5 即可，比如说将槽 3、槽 7、槽 11、槽 15 迁移给 node5，节点上的其他槽位保留。
当然了，这取决于 `CRC16(key) % 槽的个数` 的具体结果。因为在 Redis Cluster 中，槽的个数刚好是 2 的 14 次方，这和 HashMap 中数组的长度必须是 2 的幂次方有着异曲同工之妙。
它能保证扩容后，大部分数据停留在扩容前的位置，只有少部分数据需要迁移到新的槽上。
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的小米暑期实习同学 E 一面面试原题：你知道 Redis 的一致性 hash 吗
2. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的字节跳动面经同学 1 Java 后端技术一面面试原题：Redis 扩容之后，哈希槽的位置是否发生变化？
3. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的字节跳动面经同学 8 Java 后端实习一面面试原题：redis 分片集群，如何分片的，有什么好处
### 27.能说说 Redis 集群的原理吗？
Redis 集群通过数据分区来实现数据的分布式存储，通过自动故障转移实现高可用。
##### 集群创建
数据分区是在集群创建的时候完成的。
![image.png](1747710309375-53f55711-ac8d-4aa3-8680-5595de518b72.png)

**设置节点**Redis 集群一般由多个节点组成，节点数量至少为 6 个才能保证组成完整高可用的集群。每个节点需要开启配置 cluster-enabled yes，让 Redis 运行在集群模式下。
![image.png](1747710309589-72cf16ed-bfdb-45bc-a57f-c29e6f0d73e2.png)

**节点握手**节点握手是指一批运行在集群模式下的节点通过 Gossip 协议彼此通信， 达到感知对方的过程。节点握手是集群彼此通信的第一步，由客户端发起命 令：cluster meet{ip}{port}。完成节点握手之后，一个个的 Redis 节点就组成了一个多节点的集群。
**分配槽（slot）**Redis 集群把所有的数据映射到 16384 个槽中。每个节点对应若干个槽，只有当节点分配了槽，才能响应和这些槽关联的键命令。通过 cluster addslots 命令为节点分配槽。
![image.png](1747710309815-33fc9076-d7a1-463b-83a3-4700445c8ef8.png)

##### 故障转移
Redis 集群的故障转移和哨兵的故障转移类似，但是 Redis 集群中所有的节点都要承担状态维护的任务。
**故障发现**Redis 集群内节点通过 ping/pong 消息实现节点通信，集群中每个节点都会定期向其他节点发送 ping 消息，接收节点回复 pong 消息作为响应。如果在 cluster-node-timeout 时间内通信一直失败，则发送节 点会认为接收节点存在故障，把接收节点标记为主观下线（pfail）状态。
![image.png](1747710310025-b344835a-d323-41eb-93f1-2f06b1a9f475.png)

当某个节点判断另一个节点主观下线后，相应的节点状态会跟随消息在集群内传播。通过 Gossip 消息传播，集群内节点不断收集到故障节点的下线报告。当 半数以上持有槽的主节点都标记某个节点是主观下线时。触发客观下线流程。
![image.png](1747710310361-06c114f0-ed58-4f20-ae20-f6251e483c09.png)

**故障恢复**
故障节点变为客观下线后，如果下线节点是持有槽的主节点则需要在它 的从节点中选出一个替换它，从而保证集群的高可用。
![image.png](1747710310404-26443a9c-2409-41f4-a757-7b44cbf5695b.png)


1. 资格检查每个从节点都要检查最后与主节点断线时间，判断是否有资格替换故障 的主节点。
2. 准备选举时间当从节点符合故障转移资格后，更新触发故障选举的时间，只有到达该 时间后才能执行后续流程。
3. 发起选举当从节点定时任务检测到达故障选举时间（failover_auth_time）到达后，发起选举流程。
4. 选举投票持有槽的主节点处理故障选举消息。投票过程其实是一个领导者选举的过程，如集群内有 N 个持有槽的主节 点代表有 N 张选票。由于在每个配置纪元内持有槽的主节点只能投票给一个 从节点，因此只能有一个从节点获得 N/2+1 的选票，保证能够找出唯一的从节点。![image.png](1747710310728-f7e9d4cf-85e0-40b6-84f4-258a9c1f2021.png)

5. 替换主节点当从节点收集到足够的选票之后，触发替换主节点操作。#### 部署 Redis 集群至少需要几个物理节点？
在投票选举的环节，故障主节点也算在投票数内，假设集群内节点规模是 3 主 3 从，其中有 2 个主节点部署在一台机器上，当这台机器宕机时，由于从节点无法收集到 3/2+1 个主节点选票将导致故障转移失败。这个问题也适用于故障发现环节。因此部署集群时所有主节点最少需要部署在 3 台物理机上才能避免单点问题。
### 28.说说集群的伸缩？
Redis 集群使用数据分片和哈希槽的机制将数据分布到不同的节点上。集群扩容和缩容的关键，在于槽和节点之间的对应关系。
![image.png](1747710311135-16dfd9c2-0a1f-421d-bc23-94615390b442.png)

当需要扩容时，新的节点被添加到集群中，集群会自动执行数据迁移，以重新分布哈希槽到新的节点。数据迁移的过程可以确保在扩容期间数据的正常访问和插入。
![image.png](1747710311088-dc26e18d-14b1-48eb-b21d-f9019bf0594d.png)

当数据正在迁移时，客户端请求可能被路由到原有节点或新节点。Redis Cluster 会根据哈希槽的映射关系判断请求应该被路由到哪个节点，并在必要时进行重定向。
如果请求被路由到正在迁移数据的哈希槽，Redis Cluster 会返回一个 MOVED 响应，指示客户端重新路由请求到正确的目标节点。这种机制也就保证了数据迁移过程中的最终一致性。
当需要缩容时，Redis 集群会将槽从要缩容的节点上迁移到其他节点上，然后将要缩容的节点从集群中移除。
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的字节跳动面经同学 21  抖音商城一面面试原题：redis如何保证扩容过程中数据正常访问插入
GitHub 上标星 10000+ 的开源知识库《[二哥的 Java 进阶之路](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括 Java 基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM 等等，共计 32 万余字，500+张手绘图，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 10000+ 的 Java 教程](https://javabetter.cn/overview/)
微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。
![image.png](1747710311131-94e2628d-f52a-4bfe-bff1-f29f723e6703.png)

## Redis 运维
### 37.Redis 报内存不足怎么处理？
Redis 内存不足有这么几种处理方式：

- 修改配置文件 redis.conf 的 maxmemory 参数，增加 Redis 可用内存
- 也可以通过命令 set maxmemory 动态设置内存上限
- 修改内存淘汰策略，及时释放内存空间
- 使用 Redis 集群模式，进行横向扩容。### 38.Redis key 过期策略有哪些？
Redis 的 key 过期回收策略主要有两种：惰性删除和定期删除。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20240326214119.png](redis-20240326214119.png)

当某个键被访问时，如果发现它已经过期，Redis 会立即删除该键，俗称惰性删除。但这也意味着如果一个已过期的键从未被访问，它就不会被删除，会占用额外的内存空间。
那还有一种定期删除策略，即每隔一段时间，Redis 就会随机检查一些键是否过期，如果过期就删除。这种策略可以保证过期键及时被删除，但也会增加 Redis 的 CPU 消耗。
可以通过 `config get hz` 命令查看 Redis 内部定时任务的频率。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20240326214800.png](redis-20240326214800.png)

结果显示 hz 的值为 "10"，意味着 Redis 服务器每秒执行定时任务的频率是 10 次。可以通过 `CONFIG SET hz 20` 进行调整。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20240326215240.png](redis-20240326215240.png)

> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的腾讯面经同学 22 暑期实习一面面试原题：Redis key 删除策略
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的去哪儿面经同学 1 技术 2 面面试原题：redis 内存淘汰和过期策略
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的京东面经同学 5 Java 后端技术一面面试原题：redis key过期策略
### 39.Redis 有哪些内存淘汰策略？
当 Redis 的内存使用达到最大值时，它会根据配置的内存淘汰策略来决定如何处理新的请求。
> 最大值通过 maxmemory 参数设置

![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-5be7405c-ee11-4d2b-bea4-9598f10a1b17.png](redis-5be7405c-ee11-4d2b-bea4-9598f10a1b17.png)

常见的策略有：

1. noeviction：默认策略，不进行任何数据淘汰，直接返回错误信息。
1. allkeys-lru：从所有键中，使用 LRU 算法淘汰最近最少使用的键。
1. allkeys-lfu：从所有键中，使用 LFU 算法淘汰最少使用的键。
1. volatile-lru：从设置了过期时间的键中淘汰最近最少使用的键。
1. volatile-ttl：从设置了过期时间的键中淘汰即将过期的键。> TTL，Time To Live，存活时间

#### LRU 和 LFU 的区别是什么？
LRU（Least Recently Used）：基于时间维度，淘汰最近最少访问的键。适合访问具有时间特性的场景。
LFU（Least Frequently Used）：基于次数维度，淘汰访问频率最低的键。更适合长期热点数据场景。
### 40.Redis 阻塞？怎么解决？
Redis 发生阻塞，可以从以下几个方面排查：![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-e6a35258-7a78-4489-90b7-e47a4190802b.png](redis-e6a35258-7a78-4489-90b7-e47a4190802b.png)


- **API 或数据结构使用不合理**通常 Redis 执行命令速度非常快，但是不合理地使用命令，可能会导致执行速度很慢，导致阻塞，对于高并发的场景，应该尽量避免在大对象上执行算法复杂 度超过 O（n）的命令。对慢查询的处理分为两步：
1. 发现慢查询： slowlog get{n}命令可以获取最近 的 n 条慢查询命令；
1. 发现慢查询后，可以从两个方向去优化慢查询：1）修改为低算法复杂度的命令，如 hgetall 改为 hmget 等，禁用 keys、sort 等命 令2）调整大对象：缩减大对象数据或把大对象拆分为多个小对象，防止一次命令操作过多的数据。
- **CPU 饱和的问题**单线程的 Redis 处理命令时只能使用一个 CPU。而 CPU 饱和是指 Redis 单核 CPU 使用率跑到接近 100%。针对这种情况，处理步骤一般如下：
1. 判断当前 Redis 并发量是否已经达到极限，可以使用统计命令 redis-cli-h{ip}-p{port}--stat 获取当前 Redis 使用情况
1. 如果 Redis 的请求几万+，那么大概就是 Redis 的 OPS 已经到了极限，应该做集群化水品扩展来分摊 OPS 压力
1. 如果只有几百几千，那么就得排查命令和内存的使用
- **持久化相关的阻塞**对于开启了持久化功能的 Redis 节点，需要排查是否是持久化导致的阻塞。
1. fork 阻塞fork 操作发生在 RDB 和 AOF 重写时，Redis 主线程调用 fork 操作产生共享 内存的子进程，由子进程完成持久化文件重写工作。如果 fork 操作本身耗时过长，必然会导致主线程的阻塞。
1. AOF 刷盘阻塞当我们开启 AOF 持久化功能时，文件刷盘的方式一般采用每秒一次，后台线程每秒对 AOF 文件做 fsync 操作。当硬盘压力过大时，fsync 操作需要等 待，直到写入完成。如果主线程发现距离上一次的 fsync 成功超过 2 秒，为了 数据安全性它会阻塞直到后台线程执行 fsync 操作完成。
1. HugePage 写操作阻塞对于开启 Transparent HugePages 的 操作系统，每次写命令引起的复制内存页单位由 4K 变为 2MB，放大了 512 倍，会拖慢写操作的执行时间，导致大量写操作慢查询。### 41.大 key 问题了解吗？
大 key 指的是存储了大量数据的键，比如：

- 单个简单的 key 存储的 value 很大，size 超过 10KB
- hash，set，zset，list 中存储过多的元素（以万为单位）推荐阅读：[阿里：发现并处理 Redis 的大 Key 和热 Key](https://help.aliyun.com/zh/redis/user-guide/identify-and-handle-large-keys-and-hotkeys)
**大 key 会造成什么问题呢？**

- 客户端耗时增加，甚至超时
- 对大 key 进行 IO 操作时，会严重占用带宽和 CPU
- 造成 Redis 集群中数据倾斜
- 主动删除、被动删等，可能会导致阻塞**如何找到大 key?**
①、bigkeys 参数：使用 bigkeys 命令以遍历的方式分析 Redis 实例中的所有 Key，并返回整体统计信息与每个数据类型中 Top1 的大 Key
> bigkeys 命令的使用：`redis-cli --bigkeys`

![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20240309091503.png](redis-20240309091503.png)

②、redis-rdb-tools：redis-rdb-tools 是由 Python 语言编写的用来分析 Redis 中 rdb 快照文件的工具。
源码地址：[https://github.com/sripathikrishnan/redis-rdb-tools/](https://github.com/sripathikrishnan/redis-rdb-tools/)
> rdb，全称 Redis DataBase，是 Redis 在内存中的数据格式的一种持久化存储方式。

![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20240309092121.png](redis-20240309092121.png)

推荐阅读：[RDB 详解](https://redisbook.readthedocs.io/en/latest/internal/rdb.html)
**如何处理大 key?**
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-e4aaafda-fce1-47f0-8b2b-7261d47b720b.png](redis-e4aaafda-fce1-47f0-8b2b-7261d47b720b.png)

①、**删除大 key**

- 当 Redis 版本大于 4.0 时，可使用 UNLINK 命令安全地删除大 Key，该命令能够以非阻塞的方式，逐步地清理传入的大 Key。
- 当 Redis 版本小于 4.0 时，建议通过 SCAN 命令执行增量迭代扫描 key，然后判断进行删除。②、**压缩和拆分 key**

- 当 vaule 是 string 时，比较难拆分，则使用序列化、压缩算法将 key 的大小控制在合理范围内，但是序列化和反序列化都会带来额外的性能消耗。
- 当 value 是 string，压缩之后仍然是大 key 时，则需要进行拆分，将一个大 key 分为不同的部分，记录每个部分的 key，使用 multiget 等操作实现事务读取。
- 当 value 是 list/set 等集合类型时，根据预估的数据规模来进行分片，不同的元素计算后分到不同的片。> 
1. 华为 OD 的面试中出现过该题：讲一讲 Redis 的热 Key 和大 Key
### 42.Redis 常见性能问题和解决方案？

1. Master 最好不要做任何持久化工作，包括内存快照和 AOF 日志文件，特别是不要启用内存快照做持久化。
1. 如果数据比较关键，某个 Slave 开启 AOF 备份数据，策略为每秒同步一次。
1. 为了主从复制的速度和连接的稳定性，Slave 和 Master 最好在同一个局域网内。
1. 尽量避免在压力较大的主库上增加从库。
1. Master 调用 BGREWRITEAOF 重写 AOF 文件，AOF 在重写的时候会占大量的 CPU 和内存资源，导致服务 load 过高，出现短暂服务暂停现象。
1. 为了 Master 的稳定性，主从复制不要用图状结构，用单向链表结构更稳定，即主从关为：Master<–Slave1<–Slave2<–Slave3…，这样的结构也方便解决单点故障问题，实现 Slave 对 Master 的替换，也即，如果 Master 挂了，可以立马启用 Slave1 做 Master，其他不变。GitHub 上标星 10000+ 的开源知识库《[二哥的 Java 进阶之路](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括 Java 基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM 等等，共计 32 万余字，500+张手绘图，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 10000+ 的 Java 教程](https://javabetter.cn/overview/)
微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/gongzhonghao.png](JAVA/八股文/Redis/Redis.assert/gongzhonghao.png)

## Redis 应用
### 43.使用 Redis 如何实现异步队列？
我们知道 redis 支持很多种结构的数据，那么如何使用 redis 作为异步队列使用呢？一般有以下几种方式：

- **使用 list 作为队列，lpush 生产消息，rpop 消费消息**这种方式，消费者死循环 rpop 从队列中消费消息。但是这样，即使队列里没有消息，也会进行 rpop，会导致 Redis CPU 的消耗。![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-e4b192a1-3ba7-4f4e-98de-e93f437cff7c.png](redis-e4b192a1-3ba7-4f4e-98de-e93f437cff7c.png)
可以通过让消费者休眠的方式的方式来处理，但是这样又会又消息的延迟问题。
-**使用 list 作为队列，lpush 生产消息，brpop 消费消息**
brpop 是 rpop 的阻塞版本，list 为空的时候，它会一直阻塞，直到 list 中有值或者超时。![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-e9581e51-ffc8-4326-9af4-07816743dc88.png](redis-e9581e51-ffc8-4326-9af4-07816743dc88.png)

这种方式只能实现一对一的消息队列。

- **使用 Redis 的 pub/sub 来进行消息的发布/订阅**发布/订阅模式可以 1：N 的消息发布/订阅。发布者将消息发布到指定的频道频道（channel），订阅相应频道的客户端都能收到消息。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-bc6d05be-3701-4e23-b4ca-6330c949f020.png](redis-bc6d05be-3701-4e23-b4ca-6330c949f020.png)
但是这种方式不是可靠的，它不保证订阅者一定能收到消息，也不进行消息的存储。
所以，一般的异步队列的实现还是交给专业的消息队列。
### 44.Redis 如何实现延时队列?
可以使用 Redis 的 zset（有序集合）来实现延时队列。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-54bbcc36-0b00-4142-a6eb-bf2ef48c2213.png](redis-54bbcc36-0b00-4142-a6eb-bf2ef48c2213.png)

第一步，将任务添加到 zset 中，score 为任务的执行时间戳，value 为任务的内容。

```bash
ZADD delay_queue 1617024000 task1
```
第二步，定期（例如每秒）从 zset 中获取 score 小于当前时间戳的任务，然后执行任务。

```bash
ZREMRANGEBYSCORE delay_queue -inf 1617024000
```
第三步，任务执行后，从 zset 中删除任务。

```bash
ZREM delay_queue task1
```
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的腾讯面经同学 23 QQ 后台技术一面面试原题：Redis 实现延迟队列
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的字节跳动面经同学 8 Java 后端实习一面面试原题：redis 数据结构，用什么结构实现延迟消息队列
### 45.Redis 支持事务吗？
Redis 支持简单的事务，可以将多个命令打包，然后一次性的，按照顺序执行。主要通过 multi、exec、discard、watch 等命令来实现：

- multi：标记一个事务块的开始
- exec：执行所有事务块内的命令
- discard：取消事务，放弃执行事务块内的所有命令
- watch：监视一个或多个 key，如果在事务执行之前这个 key 被其他命令所改动，那么事务将被打断![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20240314101439.png](redis-20240314101439.png)

#### 说一下 Redis 事务的原理？
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-2ed7ae21-16a6-4716-ac89-117a8c76d3db.png](redis-2ed7ae21-16a6-4716-ac89-117a8c76d3db.png)


- 使用 MULTI 命令开始一个事务。从这个命令执行之后开始，所有的后续命令都不会立即执行，而是被放入一个队列中。在这个阶段，Redis 只是记录下了这些命令。
- 使用 EXEC 命令触发事务的执行。一旦执行了 EXEC，之前 MULTI 后队列中的所有命令会被原子地（atomic）执行。这里的“原子”意味着这些命令要么全部执行，要么（在出现错误时）全部不执行。
- 如果在执行 EXEC 之前决定不执行事务，可以使用 DISCARD 命令来取消事务。这会清空事务队列并退出事务状态。
- WATCH 命令用于实现乐观锁。WATCH 命令可以监视一个或多个键，如果在执行事务的过程中（即在执行 MULTI 之后，执行 EXEC 之前），被监视的键被其他命令改变了，那么当执行 EXEC 时，事务将被取消，并且返回一个错误。#### Redis 事务的注意点有哪些？
Redis 事务是不支持回滚的，一旦 EXEC 命令被调用，所有命令都会被执行，即使有些命令可能执行失败。
#### Redis 事务为什么不支持回滚？
引入事务回滚机制会大大增加 Redis 的复杂性，因为需要跟踪事务中每个命令的状态，并在发生错误时逆向执行命令以恢复原始状态。
Redis 是一个基于内存的数据存储系统，其设计重点是实现高性能。事务回滚需要额外的资源和时间来管理和执行，这与 Redis 的设计目标相违背。因此，Redis 选择不支持事务回滚。
换句话说，**就是我 Redis 不想支持事务，也没有这个必要**。
#### Redis 事务的 ACID 特性如何体现？
ACID 一般指 MySQL 事务中的四个特性：原子性、一致性、隔离性、持久性。虽然 Redis 提供了事务的支持，但它在 ACID 上的表现与 MySQL 有所不同。
Redis 事务中，所有命令会依次执行，但并不支持部分失败后的自动回滚。因此 Redis 在事务层面并不能保证一致性，我们必须通过程序逻辑来进行优化。
Redis 事务在一定程度上提供了隔离性，事务中的命令会按顺序执行，不会被其他客户端的命令插入。
Redis 的持久性依赖于其持久化机制（如 RDB 和 AOF），而不是事务本身。
#### Redis事务满足原子性吗？要怎么改进？
不满足，Redis 事务不支持回滚，一旦 EXEC 命令被调用，所有命令都会被执行，即使有些命令可能执行失败。
可以通过 Lua 脚本来实现事务的原子性，Lua 脚本在 Redis 中是原子执行的，执行过程中间不会插入其他命令。
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的华为一面原题：说下 Redis 事务
1. [二哥编程星球](https://javabetter.cn/zhishixingqiu/)球友[枕云眠美团 AI 面试原题](https://t.zsxq.com/BaHOh)：什么是 redis 的事务，它的 ACID 属性如何体现
### 46.有 Lua 脚本操作 Redis 的经验吗？
Redis 的事务不具备强制性的原子性，但可以通过 Lua 脚本来增强 Redis 的原子能力。
在 Redis 中，Lua 脚本是以原子操作的方式执行的，也就是说，在脚本执行期间，不会插入其他命令，天然保证了事务性。
比如秒杀系统是一个经典场景，我们可以用 Lua 脚本来实现扣减 Redis 库存的功能。

```java
-- 库存未预热
if (redis.call('exists', KEYS[2]) == 1) then
    return -9;
end;
-- 秒杀商品库存存在
if (redis.call('exists', KEYS[1]) == 1) then
    local stock = tonumber(redis.call('get', KEYS[1]));
    local num = tonumber(ARGV[1]);
    -- 剩余库存少于请求数量
    if (stock < num) then
        return -3
    end;
    -- 扣减库存
    if (stock >= num) then
        redis.call('incrby', KEYS[1], 0 - num);
        -- 扣减成功
        return 1
    end;
    return -2;
end;
-- 秒杀商品库存不存在
return -1;
```
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的快手同学 4 一面原题：Redis事务满足原子性吗？要怎么改进？
### 47.Redis 的管道Pipeline了解吗？
Pipeline 是 Redis 提供的一种优化手段，允许客户端一次性向服务器发送多个命令，而不必等待每个命令的响应，从而减少网络延迟。它的工作原理类似于批量操作，即多个命令一次性打包发送，Redis 服务器依次执行后再将结果一次性返回给客户端。
通常在 Redis 中，每个请求都会遵循以下流程：

1. 客户端发送命令到服务器。
1. 服务器执行命令并将结果返回给客户端。
1. 客户端接收返回结果。每一个请求和响应之间存在一次网络通信的往返时间（RTT，Round-Trip Time），如果大量请求依次发送，网络延迟会显著增加请求的总执行时间。
有了 Pipeline 后，流程变为：
> 发送命令1、命令2、命令3…… -> 服务器处理 -> 一次性返回所有结果。

例如，批量写入大量数据或执行一系列查询时，可以将这些操作打包通过 Pipeline 执行。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-38aee4c1-efd2-495e-8a6d-164d21a129b1.png](redis-38aee4c1-efd2-495e-8a6d-164d21a129b1.png)

在 Pipeline 模式下，客户端不会在每条命令发送后立即等待 Redis 的响应，而是将多个命令依次写入 TCP 缓冲区，所有命令一起发送到 Redis 服务器。
Redis 服务器接收到批量命令后，依次执行每个命令。
Redis 服务器执行完所有命令后，将每条命令的结果一次性打包通过 TCP 返回给客户端。
客户端一次性接收所有返回结果，并解析每个命令的执行结果。
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的京东面经同学 8 面试原题：对pipeline的理解，什么场景适合使用pipeline？有了解过pipeline的底层？
### 48.Redis 实现分布式锁了解吗？
分布式锁是一种用于控制多个不同进程在分布式系统中访问共享资源的锁机制。它确保在同一时刻，只有一个节点可以对资源进行访问，从而避免并发问题。
**可以使用 Redis 的 SET 命令实现分布式锁**。同时添加过期时间，以防止死锁的发生。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-710cdd19-98ea-4e96-b579-ff1ebb0d5de9.png](redis-710cdd19-98ea-4e96-b579-ff1ebb0d5de9.png)


```plain
SET key value NX PX 30000
```

- `key` 是锁名。
- `value` 是锁的持有者标识，可以使用 UUID 作为 value。
- `NX` 只在 key 不存在时才创建（避免覆盖锁）。
- `PX 30000`：设置锁的过期时间为 30 秒（防止死锁）。用 Java 来实现就是：

```java
String lockKey = "lock:order:123";
String uniqueId = UUID.randomUUID().toString();
boolean isLocked = redisTemplate.opsForValue()
    .setIfAbsent(lockKey, uniqueId, 10, TimeUnit.SECONDS);
if (isLocked) {
    try {
        // 执行业务逻辑
    } finally {
        // 释放锁
    }
}
```
#### 什么是 setnx？
setnx 从 Redis 版本 2.6.12 开始被弃用，因为可以通过 set 命令的 NX 选项来实现相同的功能。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20241122182250.png](redis-20241122182250.png)

使用 setnx 创建分布式锁时，虽然设置过期时间可以避免死锁问题，但可能存在这样的问题：线程 A 获取锁后开始任务，如果任务执行时间超过锁的过期时间，锁会提前释放，导致线程 B 也获取了锁并开始执行任务。这会破坏锁的独占性，导致并发访问资源，进而造成数据不一致。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20241122191044.png](redis-20241122191044.png)

可以引入锁的自动续约机制，在任务执行过程中定期续期，确保锁在任务完成之前不会过期。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20241122192038.png](redis-20241122192038.png)

比如说 Redisson 的 RedissonLock 就支持自动续期，通过看门狗机制定期续期锁的有效期。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20241122192708.png](redis-20241122192708.png)

#### Redisson 了解吗？
开发中，我们可以使用专业的轮子——[Redisson](https://xie.infoq.cn/article/d8e897f768eb1a358a0fd6300)。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20240308174708.png](redis-20240308174708.png)

Redisson 是一个基于 Redis 的 Java 驻内存数据网格，提供了一系列 API 用来操作 Redis，其中最常用的功能就是分布式锁。

```java
RLock lock = redisson.getLock("lock");
lock.lock();
try {
    // do something
} finally {
    lock.unlock();
}
```
实现源码在 RedissonLock 类中，通过 Lua 脚本封装 Redis 命令来实现，比如说 tryLockInnerAsync 源码：
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20240425120229.png](redis-20240425120229.png)

其中 hincrby 命令用于对哈希表中的字段值执行自增操作，pexpire 命令用于设置键的过期时间。
#### PmHub 系统里面的分布式锁是怎么做的？
主要通过 Redisson 框架实现的 RedLock 来完成的。

```java
// 创建 Redisson 客户端配置
Config config = new Config();
config.useClusterServers()
        .addNodeAddress("redis://127.0.0.1:6379",
                "redis://127.0.0.1:6380",
                "redis://127.0.0.1:6381"); // 假设有三个 Redis 节点
// 创建 Redisson 客户端实例
RedissonClient redissonClient = Redisson.create(config);
// 创建 RedLock 对象
RLock redLock = redissonClient.getLock("lock_key");

try {
    // 尝试获取分布式锁，最多尝试 5 秒获取锁，并且锁的有效期为 5000 毫秒
    boolean lockAcquired = redLock.tryLock(5, 5000, TimeUnit.MILLISECONDS);
    if (lockAcquired) {
        // 加锁成功，执行业务代码...
    } else {
        System.out.println("Failed to acquire the lock!");
    }
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
    System.err.println("Interrupted while acquiring the lock");
} finally {
    // 无论是否成功获取到锁，在业务逻辑结束后都要释放锁
    if (redLock.isLocked()) {
        redLock.unlock();
    }
    // 关闭 Redisson 客户端连接
    redissonClient.shutdown();
}
```
#### 你提到了Redlock，那它机制是怎么样的？
Redlock 是 Redis 作者提出的一种分布式锁实现方案，用于确保在分布式环境下安全可靠地获取锁。它的目标是在分布式系统中提供一种高可用、高容错的锁机制，确保在同一时刻，只有一个客户端能够成功获得锁，从而实现对共享资源的互斥访问。
Redisson 中的 RedLock 是基于 RedissonMultiLock（联锁）实现的。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20240816113330.png](redis-20240816113330.png)

RedissonMultiLock 的 tryLock 方法会在指定的 Redis 实例上逐一尝试获取锁。
在获取锁的过程中，Redlock 会根据配置的 waitTime（最大等待时间）和 leaseTime（锁的持有时间）进行灵活控制。比如，如果获取锁的时间小于锁的有效期（通过TTL命令获取锁的剩余时间），则表示获取锁成功。
通常，至少需要多数（如 5 个实例中的 3 个）实例成功获取锁，才能认为整个锁获取成功。
如果指定了锁的持有时间（leaseTime），在成功获取锁后，Redlock 会为锁进行续期，以防止锁在操作完成之前意外失效。
#### 红锁能不能保证百分百上锁？
Redlock 不能保证百分百上锁，因为在分布式系统中，网络延迟、时钟漂移、Redis 实例宕机等因素都可能导致锁的获取失败。
#### 加分布式锁时Redis如何保证不会发生冲突？
①、使用 SET NX PX 或 SETNX 命令确保锁的获取是一个原子操作，同时设置锁的过期时间防止死锁。
比如说 `SET lock_key unique_value NX PX 5000` 命令，其中 `NX` 确保了原子操作，，如果 lock_key 已存在，SET 操作会返回 nil；`PX 5000` 设置过期时间为 5000 毫秒，避免死锁。
②、使用 Lua 脚本将锁的检查和释放操作封装为一个原子操作，确保安全地释放锁。

```plain
EVAL "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end" 1 lock_key unique_value
```
③、使用 Redlock 算法确保锁的正确获取和释放。

```java
RLock lock = redisson.getLock("lock_key");
try {
    // 500ms 等待时间，10000ms 锁过期时间
    boolean isLocked = lock.tryLock(500, 10000, TimeUnit.MILLISECONDS);
    if (isLocked) {
        // 执行需要同步的操作
    }
} finally {
    lock.unlock();
}
```
#### Redisson 中的看门狗机制了解吗？
Redisson 提供的分布式锁是支持锁自动续期的，也就是说，如果线程在锁到期之前还没有执行完，那么 Redisson 会自动给锁续期。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20240918110433.png](redis-20240918110433.png)

这被称为“看门狗”机制。

```java
class RedissonWatchdogExample {
    public static void main(String[] args) {
        // 配置 Redisson 客户端
        Config config = new Config();
        config.useSingleServer().setAddress("redis://127.0.0.1:6379");
        RedissonClient redisson = Redisson.create(config);

        // 获取锁对象
        RLock lock = redisson.getLock("myLock");

        try {
            // 获取锁，默认看门狗机制会启动
            lock.lock();

            // 模拟任务执行
            System.out.println("Task is running...");
            Thread.sleep(40000); // 模拟长时间任务（40秒）

            System.out.println("Task completed.");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            // 释放锁
            lock.unlock();
        }

        // 关闭 Redisson 客户端
        redisson.shutdown();
    }
}
```
看门狗启动后，每隔 10 秒会刷新锁的过期时间，将其延长到 30 秒，确保在锁持有期间不会因为过期而释放。
当任务执行完成时，客户端调用 `unlock()` 方法释放锁，看门狗也随之停止。
#### 检查锁的过程是原子操作吗？
在 Redis 的看门狗机制中，检查锁的过程并不是单独的一个步骤，而是与锁的续期操作绑定在一起，通过 Lua 脚本完成的。因此，检查与续期是一个整体的原子操作，以确保只有持有锁的客户端才能成功续期。

```java
if redis.call('get', KEYS[1]) == ARGV[1] then
    return redis.call('expire', KEYS[1], ARGV[2])
else
    return 0
end
```

> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的腾讯 Java 后端实习一面原题：分布式锁用了 Redis 的什么数据结构
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的小公司面经合集同学 1 Java 后端面试原题：Redisson 的底层原理？以及与 SETNX 的区别？
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的百度面经同学 1 文心一言 25 实习 Java 后端面试原题：redis 分布式锁的实现原理？setnx？
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的小米同学 F 面试原题：自己实现 redis 分布式锁的坑（主动提了 Redission）
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的腾讯云智面经同学 20 二面面试原题：redission 的原理是什么？ setnx + lua 脚本？
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的收钱吧面经同学 1 Java 后端一面面试原题：系统里面分布式锁是怎么做的？你提到了redlock，那它机制是怎么样的？红锁能不能保证百分百上锁？
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的字节跳动面经同学 21  抖音商城一面面试原题：加分布式锁时redis如何保证不会发生冲突？分布式锁过期怎么办？
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的拼多多面经同学 8 一面面试原题：Redis分布式锁如何实现的
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的百度同学 4 面试原题：Setnx,知道吗? 用这个加锁有什么问题吗?怎么解决?
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的阿里系面经同学 19 饿了么面试原题：分布式锁用redis实现思路
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的京东面经同学 9 面试原题：redis的分布式锁有了解过吗
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的同学 30 腾讯音乐面试原题：redis锁有几种实现方式
GitHub 上标星 10000+ 的开源知识库《[二哥的 Java 进阶之路](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括 Java 基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM 等等，共计 32 万余字，500+张手绘图，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 10000+ 的 Java 教程](https://javabetter.cn/overview/)
微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/gongzhonghao.png](JAVA/八股文/Redis/Redis.assert/gongzhonghao.png)

## 补充
### 55.假如 Redis 里面有 1 亿个 key，其中有 10w 个 key 是以某个固定的已知的前缀开头的，如何将它们全部找出来？
使用 `keys` 指令可以扫出指定模式的 key 列表。但是要注意 keys 指令会导致线程阻塞一段时间，线上服务会停顿，直到指令执行完毕，服务才能恢复。这个时候可以使用 `scan` 指令，`scan` 指令可以无阻塞的提取出指定模式的 `key` 列表，但是会有一定的重复概率，在客户端做一次去重就可以了，但是整体所花费的时间会比直接用 `keys` 指令长。
### 56.Redis 的秒杀场景下扮演了什么角色？（补充）
秒杀主要是指大量用户集中在短时间内对服务器进行访问，从而导致服务器负载剧增，可能出现系统响应缓慢甚至崩溃的情况。
针对秒杀的场景来说，最终抢到商品的用户是固定的，也就是说 100 个人和 10000 个人来抢一个商品，最终都只能有 100 个人抢到。
但是对于秒杀活动的初心来说，肯定是希望参与的用户越多越好，但真正开始下单时，最好能把请求控制在服务器能够承受的范围之内（😂）。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20240420102552.png](redis-20240420102552.png)

解决这一问题的关键就在于错峰削峰和限流。当然了，前端页面的静态化、按钮防抖也能够有效的减轻服务器的压力。

- 页面静态化：将商品详情等页面静态化，使用 CDN 分发。
- 按钮防抖：避免用户因频繁点击造成的额外请求，比如设定间隔时间后才能再次点击。#### 如何实现错峰削峰呢？
针对车流量的晚高峰和早高峰，最强有力的办法就是限行，但限行不是无损的，毕竟限行的牌号无法出行。
无损的方式就是有的车辆早出发，有的车辆晚出发，这样就能够实现错峰出行。
在秒杀场景下，可以通过以下几种方式实现错峰削峰：
①、**预热缓存**：提前将热点数据加载到 Redis 缓存中，减少对数据库的访问压力。
②、**消息队列**：引入消息队列，将请求异步处理，减少瞬时请求压力。消息队列就像一个水库，可以削减上游的洪峰流量。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20240420104633.png](redis-20240420104633.png)

③、**多阶段多时间窗口**：将秒杀活动分为多个阶段，每个阶段设置不同的时间窗口，让用户在不同的时间段内参与秒杀活动。
④、**插入答题系统**：在秒杀活动中加入答题环节，只有答对题目的用户才能参与秒杀活动，这样可以减少无效请求。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20240420104921.png](redis-20240420104921.png)

#### 如何限流呢？
采用令牌桶算法，它就像在帝都买车，摇到号才有资格，没摇到就只能等下一次（😁）。
在实际开发中，我们需要维护一个容器，按照固定的速率往容器中放令牌（token），当请求到来时，从容器中取出一个令牌，如果容器中没有令牌，则拒绝请求。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\Redis/Redis.assert/redis-20240420114025.png](redis-20240420114025.png)

第一步，使用 Redis 初始化令牌桶：

```shell
redis-cli SET "token_bucket" "100"
```
第二步，使用 Lua 脚本实现令牌桶算法；假设每秒向桶中添加 10 个令牌，但不超过桶的最大容量。

```lua
-- Lua 脚本来添加令牌，并确保不超过最大容量
local bucket = KEYS[1]
local add_count = tonumber(ARGV[1])
local max_tokens = tonumber(ARGV[2])
local current = tonumber(redis.call('GET', bucket) or 0)
local new_count = math.min(current + add_count, max_tokens)
redis.call('SET', bucket, tostring(new_count))
return new_count
```
第三步，使用 Shell 脚本调用 Lua 脚本：

```shell
#!/bin/bash
while true; do
    redis-cli EVAL "$(cat add_tokens.lua)" 1 token_bucket 10 100
    sleep 1
done
```
第四步，当请求到达时，需要检查并消耗一个令牌。

```lua
-- Lua 脚本来消耗一个令牌
local bucket = KEYS[1]
local tokens = tonumber(redis.call('GET', bucket) or 0)
if tokens > 0 then
    redis.call('DECR', bucket)
    return 1  -- 成功消耗令牌
else
    return 0  -- 令牌不足
end
```
调用 Lua 脚本：

```shell
redis-cli EVAL "$(cat consume_token.lua)" 1 token_bucket
```
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的农业银行面经同学 3 Java 后端面试原题：秒杀问题（错峰、削峰、前端、流量控制）
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的滴滴面经同学 3 网约车后端开发一面原题：限流算法
### 57. 客户端宕机后 Redis 服务端如何感知到？
每个客户端在 Redis 中维护一个特定的键（称为心跳键），用于表示客户端的健康状态。该键具有一个设置的超时时间，例如 10 秒。
客户端定期（如每 5 秒）更新这个心跳键的超时时间，保持它的存活状态，通常通过 SET 命令重设键的过期时间。

```java
import redis.clients.jedis.Jedis;

public class ClientHeartbeat {
    private static final String HEARTBEAT_KEY = "client:heartbeat";
    private static final int EXPIRE_TIME = 10; // 10秒

    public static void main(String[] args) {
        // 创建 Redis 连接
        Jedis jedis = new Jedis("localhost");

        // 定时更新心跳键
        while (true) {
            try {
                // 设置心跳键并设置过期时间
                jedis.setex(HEARTBEAT_KEY, EXPIRE_TIME, "alive");

                // 打印心跳日志
                System.out.println("Heartbeat sent.");

                // 等待一段时间后再次发送心跳
                Thread.sleep(5000); // 每5秒发送一次心跳
            } catch (InterruptedException e) {
                e.printStackTrace();
                break;
            }
        }
    }
}
```
Redis 服务端定期检查这个心跳键。如果发现该键已超时并被 Redis 自动删除，说明客户端可能已宕机。

```java
import redis.clients.jedis.Jedis;

public class ServerMonitor {
    private static final String HEARTBEAT_KEY = "client:heartbeat";

    public static void main(String[] args) {
        // 创建 Redis 连接
        Jedis jedis = new Jedis("localhost");

        // 定期检查心跳键
        while (true) {
            try {
                // 检查心跳键是否存在
                if (jedis.exists(HEARTBEAT_KEY)) {
                    System.out.println("Client is alive.");
                } else {
                    System.out.println("Client is down or disconnected.");
                }

                // 每隔一段时间检查一次
                Thread.sleep(10000); // 每10秒检查一次
            } catch (InterruptedException e) {
                e.printStackTrace();
                break;
            }
        }
    }
}
```

---
