# 93. 操作系统

## 93.1 内存

RAM ! RAM ! RAM ! HBase 很吃内存！

## 93.2 64 位

使用 64 位操作系统，64 位 JVM。

## 93.3 交换空间

关闭交换空间，将 `swappiness` 置为 0。

## 93.4 CPU

Hadoop 使用 native 的硬件校验和：[hadoop.native.lib]。


# 94. 网络

避免网络问题降低 HBase 性能最关键的是使用的交换设备。

## 94.1 单交换机

所有的流量都要经过这台交换机，它必须能够抗住。

## 94.2 多交换机

多交换机可能是一个陷阱。低价硬件的最常见配置是从一个交换机到另一个交换机使用简单的1Gbps上行链路。被忽视的单点很容易成为集群通信的瓶颈。特别是对于MapReduce作业，读取和写入大量数据，这种上行链路的通信可能会饱和。

缓解这个问题也很简单：

- 根据集群的规模使用适当的硬件。
- 使用较大的单交换机配置，即单个48端口，而不是2x 24端口
- 配置上行链路的端口中继，利用多个接口增加交换机带宽。

## 94.3 多机架

多机架也可能遇到遇到多交换机同样的问题：

- 交换机性能较差
- 到另一机架的上行带宽不足

跨多个机架避免问题的最简单的方法是使用端口集群来创建到其他机架的绑定上行链路。然而，该方法的缺点在于可能使用的端口开销。比如，从机架A到机架B创建一个8Gbps的端口通道，使用24个端口中的8个在机架之间进行通信，使您的投资回报率很差，但使用的数量太少可能意味着您没有充分利用簇。

## 94.4 网络端口

是否所有的网络接口都工作正常？[Case Study #1 (Performance Issue On A Single Node)](http://hbase.apache.org/book.html#casestudies.slownode)

## 94.5 网络一致性和分区容错

分布式系统的 CAP 定理：

- Consistency。所有节点看到同样的数据。
- Availability。每个请求都会收到回复，不管是成功还是失败的回复。
- Partition Tolerance。即使其中一些组件对其他组件不可用，系统也继续运行。


HBase 在一致性和分区容错上做的非常好。参见：[blog post](https://rayokota.wordpress.com/2015/09/30/call-me-maybe-hbase/)。


# 95. Java

## 95.1 GC

### GC 暂停

[Avoiding Full GCs with MemStore-Local Allocation Buffers](http://www.slideshare.net/cloudera/hbase-hug-presentation) 这篇文章里，Todd Lipcon 描述了两种 GC 引起的 stop-the-world：

- CMS 故障模式
- 老生代堆碎片

定位第一个问题，要开启`-XX:CMSInitiatingOccupancyFraction`，并降低它的值。

第二个问题，MSLAB 可以派上用场：`hbase.hregion.memstore.mslab.enabled`置为 `true`。开启后，每个MemStore实例将占用至少一个MSLAB内存实例。如果有数以千计的 region，每个r egion 都有许多列族，那么 MSLAB 的这种分配对内存是个考验，甚至导致 OOM。

[HBASE-8163 MemStoreChunkPool: An improvement for JAVA GC when using MSLAB](https://issues.apache.org/jira/browse/HBASE-8163) 是对 GC 的改进，在写入大量负载期间降低年轻GC的量的配置。

[Eliminating Large JVM GC Pauses Caused by Background IO Traffic](https://engineering.linkedin.com/blog/2016/02/eliminating-large-jvm-gc-pauses-caused-by-background-io-traffic)


# 96. HBase 配置

[Recommended Configurations](http://hbase.apache.org/book.html#recommended_configurations)

## 96.1改善99分位 

hedged_reads

## 96.2 管理 Compaction

[compactions and splits

## 96.3 `hbase.regionserver.hadnler.count` 

[hbase.regionserver.handler.count](http://hbase.apache.org/book.html#hbase.regionserver.handler.count)

## 96.4 `hfile.block.cache.size`

[hfile.block.cache.size](http://hbase.apache.org/book.html#hfile.block.cache.size)

RegionServer 进程的内存设定。

## 96.5 BlockCache 预取

[HBASE-9857](https://issues.apache.org/jira/browse/HBASE-9857) 增加了 HFile 的预取选项。在缓存打开之后，使用内存中的表数据尽可能快地加热BlockCache，并且预取不计为高速缓存未命中。

## 96.6 `hbase.regionserver.global.memstore.size`

[[hbase.regionserver.global.memstore.size\]](http://hbase.apache.org/book.html#hbase.regionserver.global.memstore.size)

## 96.7 `hbase.regionserver.global.memstore.size.lower.limit`

[[hbase.regionserver.global.memstore.size.lower.limit\]](http://hbase.apache.org/book.html#hbase.regionserver.global.memstore.size.lower.limit)

## 96.8 `hbase.hstore.blockingStoreFiles`

[[hbase.hstore.blockingStoreFiles\]](http://hbase.apache.org/book.html#hbase.hstore.blockingStoreFiles)

## 96.9 `hbase.hregion.memstore.block.multiplier`

[[hbase.hregion.memstore.block.multiplier\]](http://hbase.apache.org/book.html#hbase.hregion.memstore.block.multiplier)

如果有足够的 RAM，增加这个选项能够有所助益。

## 96.10 `hbase.regionserver.checksum.verify`

HBase 将校验和存入数据块，无论何时读取数据都需查找检验和。

See [[hbase.regionserver.checksum.verify\]](http://hbase.apache.org/book.html#hbase.regionserver.checksum.verify), [[hbase.hstore.bytes.per.checksum\]](http://hbase.apache.org/book.html#hbase.hstore.bytes.per.checksum) and [[hbase.hstore.checksum.algorithm\]](http://hbase.apache.org/book.html#hbase.hstore.checksum.algorithm). For more information see the release note on [HBASE-5074 support checksums in HBase block cache](https://issues.apache.org/jira/browse/HBASE-5074).

## 96.11 `callQueue` 选项

[HBASE-11355](https://issues.apache.org/jira/browse/HBASE-11355) 引入了`callQueue`的调优选项。

`hbase.ipc.server.num.callqueue` 增加 `callQueue` 的数量。

`hbase.ipc.server.callqueue.read.ratio` 调整读和写队列的比重。

`hbase.ipc.server.callqueue.scan.ratio` 分配 Get 和 Scan 队列的比重。

`hbase.ipc.server.callqueue.handler.factor` 在程序中调节 Queue 的数量：

- 0 表示所有 handler 共享一个队列
- 1 表示每个 handler 一个队列
- 0.5 表示每两个 handler 共享一个队列


# 97. ZooKeeper

See [ZooKeeper](http://hbase.apache.org/book.html#zookeeper) for information on configuring ZooKeeper, and see the part about having a dedicated disk.

# 98. Schema 设计

## 98.1 列族的数量

[On the number of column families](http://hbase.apache.org/book.html#number.of.cfs)

## 98.2 键和属性长度

See [Try to minimize row and column sizes](http://hbase.apache.org/book.html#keysize). See also [However…](http://hbase.apache.org/book.html#perf.compression.however) for compression caveats.

## 98.3 表的 region 大小

每张表的 region 大小可调用`HTableDescriptor`上的`setFileSize`设置。

[Determining region count and size](http://hbase.apache.org/book.html#ops.capacity.regions)

## 98.4 布隆过滤器

