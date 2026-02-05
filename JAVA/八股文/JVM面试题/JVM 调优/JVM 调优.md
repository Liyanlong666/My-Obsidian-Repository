## [四、JVM 调优](https://javabetter.cn/sidebar/sanfene/jvm.html#%E5%9B%9B%E3%80%81jvm-%E8%B0%83%E4%BC%98)
### [37.有哪些常用的命令行性能监控和故障处理工具？](https://javabetter.cn/sidebar/sanfene/jvm.html#_37-%E6%9C%89%E5%93%AA%E4%BA%9B%E5%B8%B8%E7%94%A8%E7%9A%84%E5%91%BD%E4%BB%A4%E8%A1%8C%E6%80%A7%E8%83%BD%E7%9B%91%E6%8E%A7%E5%92%8C%E6%95%85%E9%9A%9C%E5%A4%84%E7%90%86%E5%B7%A5%E5%85%B7)

- 操作系统工具
- top：显示系统整体资源使用情况
- vmstat：监控内存和 CPU
- iostat：监控 IO 使用
- netstat：监控网络使用
- JDK 性能监控工具
- jps：虚拟机进程查看
- jstat：虚拟机运行时信息查看
- jinfo：虚拟机配置查看
- jmap：内存映像（导出）
- jhat：堆转储快照分析
- jstack：Java 堆栈跟踪
- jcmd：实现上面除了 jstat 外所有命令的功能#### [jmap 的具体命令有哪些？](https://javabetter.cn/sidebar/sanfene/jvm.html#jmap-%E7%9A%84%E5%85%B7%E4%BD%93%E5%91%BD%E4%BB%A4%E6%9C%89%E5%93%AA%E4%BA%9B)
①、我一般会使用 `jmap -heap <pid>` 查看堆内存摘要，包括新生代、老年代、元空间等。
![image.jpg](./JVM 调优.assert/1738590564457-ffaea0a3-dfd3-4d28-bcb0-2b0fd937b66a.png)

二哥的Java 进阶之路：jmap -heap
②、或者使用 `jmap -histo <pid>` 查看对象分布。
![image.jpg](./JVM 调优.assert/1738590564886-c8d42bc1-b5e7-4ab9-9827-c2381d830370.png)

二哥的Java 进阶之路：jmap -histo
③、还有生成堆转储文件：`jmap -dump:format=b,file=<path> <pid>`。
![image.jpg](./JVM 调优.assert/1738590564629-f12e2ab4-cd14-4e98-b929-1627b75dbbd5.png)

二哥的Java 进阶之路：jmap -dump
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的哔哩哔哩同学 1 二面面试原题：你是如何使用jmap，你用过哪些命令？
### [38.了解哪些可视化的性能监控和故障处理工具？](https://javabetter.cn/sidebar/sanfene/jvm.html#_38-%E4%BA%86%E8%A7%A3%E5%93%AA%E4%BA%9B%E5%8F%AF%E8%A7%86%E5%8C%96%E7%9A%84%E6%80%A7%E8%83%BD%E7%9B%91%E6%8E%A7%E5%92%8C%E6%95%85%E9%9A%9C%E5%A4%84%E7%90%86%E5%B7%A5%E5%85%B7)
我自己用过的可视化工具主要有：
①、JConsole：JDK 自带的监控工具，可以用来监视 Java 应用程序的运行状态，包括内存使用、线程状态、类加载、GC 等，还可以进行一些基本的性能分析。
![image.jpg](./JVM 调优.assert/1738590564831-39700a07-5f1d-4bd7-95e9-ef7272ebd605.png)

三分恶面渣逆袭：JConsole概览
②、VisualVM：VisualVM 是一个基于 NetBeans 平台的可视化工具，在很长一段时间内，VisualVM 都是 Oracle 官方主推的故障处理工具。集成了多个 JDK 命令行工具的功能，提供了一个友好的图形界面，非常适用于开发和生产环境。
![image.jpg](./JVM 调优.assert/1738590564840-7729a358-3c73-48a6-9112-472095a25955.png)

三分恶面渣逆袭：VisualVM安装插件
③、Java Mission Control：JMC 最初是 JRockit VM 中的诊断工具，但在 Oracle JDK7 Update 40 以后，就绑定到了 HotSpot VM 中。不过后来又被 Oracle 开源出来作为一个单独的产品。
![image.jpg](./JVM 调优.assert/1738590564838-6c6e5126-9ee2-4087-9a2a-02ef9991753c.png)

三分恶面渣逆袭：JMC主要界面
还有一些第三方的工具：
①、**MAT**：

- Java 堆内存分析工具，主要用于分析和查找 Java 堆中的内存泄漏和内存消耗问题。
- 可以从 Java 堆转储文件中分析内存使用情况，并提供丰富的报告，如内存泄漏疑点、最大对象和 GC 根信息。
- 支持通过图形界面查询对象，以及检查对象间的引用关系。②、**GChisto**：GC 日志分析工具，帮助开发者优化垃圾收集行为和调整 GC 性能。
③、**GCViewer**：类似于 GChisto，也是用来分析 GC 日志，帮助开发者优化 Java 应用的垃圾回收过程。
④、**JProfiler**：一个全功能的商业 Java 性能分析工具，提供 CPU、 内存和线程的实时分析。
⑤、**arthas**：

- 阿里巴巴开源的 Java 诊断工具，主要用于线上的应用诊断。
- 支持在不停机的情况下进行 Java 应用的诊断。
- 包括 JVM 信息查看、监控、Trace 命令、反编译等。⑥、**async-profiler**：一个低开销的性能分析工具，支持生成火焰图，适用于复杂性能问题的分析。
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的华为面经同学 9 Java 通用软件开发一面面试原题：如何查看当前 Java 程序里哪些对象正在使用，哪些对象已经被释放
### [39.JVM 的常见参数配置知道哪些？](https://javabetter.cn/sidebar/sanfene/jvm.html#_39-jvm-%E7%9A%84%E5%B8%B8%E8%A7%81%E5%8F%82%E6%95%B0%E9%85%8D%E7%BD%AE%E7%9F%A5%E9%81%93%E5%93%AA%E4%BA%9B)
一些常见的参数配置：
**堆配置：**

- -Xms:初始堆大小
- -Xmx:最大堆大小
- -XX:NewSize=n:设置年轻代大小
- -XX:NewRatio=n:设置年轻代和年老代的比值。如：为 3 表示年轻代和年老代比值为 1：3，年轻代占整个年轻代年老代和的 1/4
- -XX:SurvivorRatio=n:年轻代中 Eden 区与两个 Survivor 区的比值。注意 Survivor 区有两个。如 3 表示 Eden： 3 Survivor：2，一个 Survivor 区占整个年轻代的 1/5
- -XX:MaxPermSize=n:设置持久代大小**收集器设置：**

- -XX:+UseSerialGC:设置串行收集器
- -XX:+UseParallelGC:设置并行收集器
- -XX:+UseParalledlOldGC:设置并行年老代收集器
- -XX:+UseConcMarkSweepGC:设置并发收集器**并行收集器设置**

- -XX:ParallelGCThreads=n:设置并行收集器收集时使用的 CPU 数。并行收集线程数
- -XX:MaxGCPauseMillis=n:设置并行收集最大的暂停时间（如果到这个时间了，垃圾回收器依然没有回收完，也会停止回收）
- -XX:GCTimeRatio=n:设置垃圾回收时间占程序运行时间的百分比。公式为：1/(1+n)
- -XX:+CMSIncrementalMode:设置为增量模式。适用于单 CPU 情况
- -XX:ParallelGCThreads=n:设置并发收集器年轻代手机方式为并行收集时，使用的 CPU 数。并行收集线程数**打印 GC 回收的过程日志信息**

- -XX:+PrintGC
- -XX:+PrintGCDetails
- -XX:+PrintGCTimeStamps
- -Xloggc:filename### [40.有做过 JVM 调优吗？](https://javabetter.cn/sidebar/sanfene/jvm.html#_40-%E6%9C%89%E5%81%9A%E8%BF%87-jvm-%E8%B0%83%E4%BC%98%E5%90%97)
JVM 调优是一个复杂的过程，主要包括对堆内存、垃圾收集器、JVM 参数等进行调整和优化。
![image.jpg](./JVM 调优.assert/1738590564996-5aa59200-d3cf-4fb9-951e-399f9273e4b4.png)

二哥的 Java 进阶之路：JVM 调优
①、JVM 的堆内存主要用于存储对象实例，如果堆内存设置过小，可能会导致频繁的垃圾回收。所以，[技术派实战项目](https://javabetter.cn/zhishixingqiu/paicoding.html)是在启动 JVM 的时候就调整了一下 -Xms 和-Xmx 参数，让堆内存最大可用内存为 2G。
②、在项目运行期间，我会使用 JVisualVM 定期观察和分析 GC 日志，如果发现频繁的 Full GC，就需要特别关注老年代的使用情况。
接着，通过分析 Heap dump 寻找内存泄漏的源头，看看是否有未关闭的资源，长生命周期的大对象等。
之后，就要进行代码优化了，比如说减少大对象的创建、优化数据结构的使用方式、减少不必要的对象持有等。
### [41.CPU 占用过高怎么排查？](https://javabetter.cn/sidebar/sanfene/jvm.html#_41-cpu-%E5%8D%A0%E7%94%A8%E8%BF%87%E9%AB%98%E6%80%8E%E4%B9%88%E6%8E%92%E6%9F%A5)
![image.jpg](./JVM 调优.assert/1738590565062-ea4641f4-4ef6-431d-bbe6-972c65718901.png)

三分恶面渣逆袭：CPU飙高
首先，使用 top 命令查看 CPU 占用情况，找到占用 CPU 较高的进程 ID。
![image.jpg](./JVM 调优.assert/1738590565816-da54351e-1a52-4242-a84b-75f524a6adac.png)

haikuotiankongdong：top 命令结果
接着，使用 jstack 命令查看对应进程的线程堆栈信息。

```java
jstack -l <pid> > thread-dump.txt
```
> 上面 👆🏻 这个命令会将所有线程的堆栈信息输出到 thread-dump.txt 文件中。

然后再使用 top 命令查看进程中线程的占用情况，找到占用 CPU 较高的线程 ID。
![image.jpg](./JVM 调优.assert/1738590565433-1445e2fc-6a35-44dd-8cee-8da84d15a919.png)

haikuotiankongdong：Java 进程中的线程情况
注意，top 命令显示的线程 ID 是十进制的，而 jstack 输出的是十六进制的，所以需要将线程 ID 转换为十六进制。
在 jstack 的输出中搜索这个十六进制的线程 ID，找到对应的堆栈信息。

```java
"Thread-5" #21 prio=5 os_prio=0 tid=0x00007f812c018800 nid=0x1a85 runnable [0x00007f811c000000]
   java.lang.Thread.State: RUNNABLE
    at com.example.MyClass.myMethod(MyClass.java:123)
    at ...
```
最后，根据堆栈信息定位到具体的业务方法，查看是否有死循环、频繁的垃圾回收（GC）、资源竞争（如锁竞争）导致的上下文频繁切换等问题。
### [42.内存飙高问题怎么排查？](https://javabetter.cn/sidebar/sanfene/jvm.html#_42-%E5%86%85%E5%AD%98%E9%A3%99%E9%AB%98%E9%97%AE%E9%A2%98%E6%80%8E%E4%B9%88%E6%8E%92%E6%9F%A5)
内存飚高一般是因为创建了大量的 Java 对象所导致的，如果持续飙高则说明垃圾回收跟不上对象创建的速度，或者内存泄漏导致对象无法回收。
排查的方法主要分为以下几步：
第一，先观察垃圾回收的情况，可以通过 `jstat -gc PID 1000` 查看 GC 次数和时间。
或者 `jmap -histo PID | head -20` 查看堆内存占用空间最大的前 20 个对象类型。
第二步，通过 jmap 命令 dump 出堆内存信息。
![image.jpg](./JVM 调优.assert/1738590565288-9f26835d-65b9-4ae4-8751-ed9dcfa6daa0.png)

二哥的 Java 进阶之路：dump
第三步，使用可视化工具分析 dump 文件，比如说 VisualVM，找到占用内存高的对象，再找到创建该对象的业务代码位置，从代码和业务场景中定位具体问题。
![image.jpg](./JVM 调优.assert/1738590565667-c3844260-f7ea-42b9-8197-fad2f2b0de15.png)

二哥的 Java 进阶之路：分析
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的联想面经同学 7 面试原题：怎么定位线上的内存问题。
### [43.频繁 minor gc 怎么办？](https://javabetter.cn/sidebar/sanfene/jvm.html#_43-%E9%A2%91%E7%B9%81-minor-gc-%E6%80%8E%E4%B9%88%E5%8A%9E)
频繁的 Minor GC（也称为 Young GC）通常表示新生代中的对象频繁地被垃圾回收，可能是因为新生代空间设置过小，或者是因为程序中存在大量的短生命周期对象（如临时变量、方法调用中创建的对象等）。
可以使用 GC 日志进行分析，查看 GC 的频率和耗时，找到频繁 GC 的原因。

```java
-XX:+PrintGCDetails -Xloggc:gc.log
```
或者使用监控工具（如 VisualVM、jstat、jconsole 等）查看堆内存的使用情况，特别是新生代（Eden 和 Survivor 区）的使用情况。
如果是因为新生代空间不足，可以通过 `-Xmn` 增加新生代的大小，减少新生代的填满速度。

```java
java -Xmn256m your-app.jar
```
如果对象未能在 Survivor 区足够长时间存活，就会被晋升到老年代，可以通过 `-XX:SurvivorRatio` 参数调整 Eden 和 Survivor 的比例。默认比例是 8:1，表示 8 个空间用于 Eden，1 个空间用于 Survivor 区。
这将减少 Eden 区的大小，增加 Survivor 区的大小，以确保对象在 Survivor 区中存活的时间足够长，避免过早晋升到老年代。
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的京东面经同学 8 面试原题：young GC频繁如何排查？修改哪些参数？
### [44.频繁 Full GC 怎么办？](https://javabetter.cn/sidebar/sanfene/jvm.html#_44-%E9%A2%91%E7%B9%81-full-gc-%E6%80%8E%E4%B9%88%E5%8A%9E)
Full GC 是指对整个堆内存（包括新生代和老年代）进行垃圾回收操作。Full GC 频繁会导致应用程序的暂停时间增加，从而影响性能。
常见的原因有：

- 大对象（如大数组、大集合）直接分配到老年代，导致老年代空间快速被占用。
- 程序中存在内存泄漏，导致老年代的内存不断增加，无法被回收。比如 IO 资源未关闭。
- 一些长生命周期的对象进入到了老年代，导致老年代空间不足。
- 不合理的 GC 参数配置也导致 GC 频率过高。比如说新生代的空间设置过小。#### [该怎么排查 Full GC 频繁问题？](https://javabetter.cn/sidebar/sanfene/jvm.html#%E8%AF%A5%E6%80%8E%E4%B9%88%E6%8E%92%E6%9F%A5-full-gc-%E9%A2%91%E7%B9%81%E9%97%AE%E9%A2%98)
大厂一般都会有专门的性能监控系统，可以通过监控系统查看 GC 的频率和堆内存的使用情况。
否则可以使用 JDK 的一些自带工具，包括 jmap、jstat 等。

```java
# 查看堆内存各区域的使用率以及GC情况
jstat -gcutil -h20 pid 1000
# 查看堆内存中的存活对象，并按空间排序
jmap -histo pid | head -n20
# dump堆内存文件
jmap -dump:format=b,file=heap pid
```
或者使用一些可视化的工具，比如 VisualVM、JConsole 等。
#### [如何解决 Full GC 频繁问题？](https://javabetter.cn/sidebar/sanfene/jvm.html#%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3-full-gc-%E9%A2%91%E7%B9%81%E9%97%AE%E9%A2%98)
假如是因为大对象直接分配到老年代导致的 Full GC 频繁，可以通过 `-XX:PretenureSizeThreshold` 参数设置大对象直接进入老年代的阈值。
或者能不能将大对象拆分成小对象，减少大对象的创建。比如说分页。
假如是因为内存泄漏导致的 Full GC 频繁，可以通过分析堆内存 dump 文件找到内存泄漏的对象，再找到内存泄漏的代码位置。
假如是因为长生命周期的对象进入到了老年代，要及时释放资源，比如说 ThreadLocal、数据库连接、IO 资源等。
假如是因为 GC 参数配置不合理导致的 Full GC 频繁，可以通过调整 GC 参数来优化 GC 行为。或者直接更换更适合的 GC 收集器，如 G1、ZGC 等。
