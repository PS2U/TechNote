# 3. 配置文件

所有的配置文件都在`conf/`目录下，它需要在所有节点间保持同步。

**backup-masters**

默认没有这个文件，它是一个纯文本，在 Master 上，列出了所有的备用 Master 进程。

一行一个 host。

**hadoop-metrics2-hbase.properties**

用来连接 HBase Hadoop Metrics 2 框架。默认里面都是注释掉的 example。

**hbase-env.sh**

UNIX 环境下设置工作环境的脚本，包括 JAVA 选项和其他环境变量。

**hbase-policy.xml**

RPC 服务器用来鉴权的配置，只有在 HBase security 开启的情况下生效。

**hbase-site.xml**

HBase 的主要配置文件，它会覆盖 HBase 的默认选项 `docs/hbase-default.xml`。从 Web UI 可以查看该配置文件的详情。

**log4j.properties**

`log4j`日志的配置。

**regionservers**

包含 host 列表的纯文本，每一行表示一台 RegionServer，不应该有`localhost`。



# 4. 基本前提

| HBase Version | JDK 7                                    | JDK 8                                    |
| ------------- | ---------------------------------------- | ---------------------------------------- |
| 2.0           | [Not Supported](http://search-hadoop.com/m/YGbbsPxZ723m3as) | yes                                      |
| 1.3           | yes                                      | yes                                      |
| 1.2           | yes                                      | yes                                      |
| 1.1           | yes                                      | Running with JDK 8 will work but is not well tested. |

**操作系统相关**

- ssh。HBase 使用 ssh 进行节点间的通信。每个 Server 上必须运行着 ssh。
- DNS。HBase 使用本地的 hostname 来自报 IP 它的 IP 地址。`hadoop-dns-checker`工具可用来检查集群的 DNS 是否正确。
- 回环 IP。HBase 0.96.0 以前，必须使用 `127.0.0.1` 指向 `localhost`。
- Network Time Protocol。节点的时钟必须同步，少许误差可以接受，但大的偏离会引发不确定的后果。 推荐在集群中运行 NTP 服务。
- ulimit，至少 10, 000 。估算需要打开的文件数：(StoreFiles per ColumnFamily) x (regions per RegionServer)
- Linux Shell，必须是 GNU Bash shell。

## 4.1 Hadoop

HBase 与 Hadoop 的版本对应关系：

|                     | HBase-1.1.x | HBase-1.2.x | HBase-1.3.x | HBase-2.0.x |
| ------------------- | ----------- | ----------- | ----------- | ----------- |
| Hadoop-2.0.x-alpha  | X           | X           | X           | X           |
| Hadoop-2.1.0-beta   | X           | X           | X           | X           |
| Hadoop-2.2.0        | NT          | X           | X           | X           |
| Hadoop-2.3.x        | NT          | X           | X           | X           |
| Hadoop-2.4.x        | S           | S           | S           | X           |
| Hadoop-2.5.x        | S           | S           | S           | X           |
| Hadoop-2.6.0        | X           | X           | X           | X           |
| Hadoop-2.6.1+       | NT          | S           | S           | S           |
| Hadoop-2.7.0        | X           | X           | X           | X           |
| Hadoop-2.7.1+       | NT          | S           | S           | S           |
| Hadoop-2.8.0        | X           | X           | X           | X           |
| Hadoop-3.0.0-alphax | NT          | NT          | NT          | NT          |

- "S" = supported
- "X" = not supported
- "NT" = Not tested

### `dfs.datanode.max.transfer.threads`

HDFS DataNode 对同时可用的文件有个上限，设置`dfs.datanode.max.transfer.threads` 然后重启 HDFS。

## 4.2 ZooKeeper 要求

要求ZooKeeper 3.4.x ，HBase 需要其 `multi` 功能，需要将 `hbase.zookeeper.useMulti`设置为 `true`。



# 5.HBase 运行模式：单机和分布式

## 5.1 单机 HBase

单机模式上，HBase 不使用 HDFS，而是使用本地文件系统。所有的 HBase 后台服务和本地 ZooKeeper 全部在一个 JVM 中。

单机模式的 HBase 也可以使用 HDFS：

```xml
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://namenode.example.org:8020/hbase</value>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>false</value>
  </property>
</configuration>
```

# 5.2 分布式

分布式又分为伪分布式和全分布式，前者所有的后台服务跑在单节点，后者是跑在各个节点。

伪分布式可以使用本地文件系统，也可以使用 HDFS。全分布式只能使 HDFS。

### 伪分布式

 伪分布式最好用来做 HBase 的测试和原型设计，不要用于估计生产环境的 HBase 性能。

 ## 5.3 全分布式

默认情况下，HBase 是单机模式。

在全分布式下，集群的节点被配置为 RegionServer、ZooKeeper QuorumPeers、备用 HMaster。RegionServer 运行在不同的节点上，包括主备 Master。

**HDFS 客户端配置**

如果你修改了 HDFS 的客户端配置，必须通知 HBase，以下方法可选其一：

- `hbase-env.sh` 修改 `HBASE_CLASSPATH`，添加一个指向 `HADOOP_CONF_DIR` 的引用。
- 软连接 `hdfs-site.xml` 到 HBase 的`conf`目录。
- 如果只改了小部分的 HDFS 客户端配置，将它们添加到 `hbase-site.xml`。


# 6. 运行 HBase

首先确保 HDFS 运行正常。启动和终止 HDFS 采用 `bin/start-hdfs.sh`，使用`put`或 `get` 来测试 HDFS

HBase 不需要 MapReduce 和 Yarn，没必要启动它们。

如果有独立的 ZooKeeper，启动它。没有的话，HBase 会代劳。

启动 HBase: `bin/start-hbase.sh`。

默认情况下 Master 的端口 16010 会暴露 Web UI。

终止 HBase：`bin/stop-hbase.sh`。


# 7. 默认配置

HBase 的自定义配置在`conf/hbase-site.xml`文件中，默认配置在`hbase-default.xml`中。配置文件的修改需要重启 HBase 集群。

## 7.2 HBase 默认配置

**hbase.tmp.dir**

本地文件系统上的临时目录。注：`/tmp`目录重启后清空。

默认：`${java.io.tmpdir}/hbase-${user.name}`

**hbase.rootdir**

RegionServer 共享的目录，URL 必须包括文件系统 Schema。

默认：${hbase.tmp.dir}/hbase

**hbase.fs.tmp.dir**

一个 staging 目录，保存临时数据。

默认：` /user/${user.name}/hbase-staging`

**hbase.cluster.distributed**

表明集群所处的模式，单机或分布式。

默认：`false`

**hbase.zookeeper.quorum**

ZooKeeper 服务器的列表，若果`hbase-env.sh`中设置了`HBASE_MANAGES_ZK`，那么 HBase 就负责启动、停止 ZooKeeper。从客户端看来，这个列表和`hbase.zookeeper.clientPort`一起传递给 ZooKeeper 构造器，用作`connectString`的参数。

默认：`localhost`

**zookeeper.recovery.retry.maxsleeptime**

ZooKeeper 重试操作的最大休眠时间。

默认：60000

**hbase.local.dir**

本地文件系统的目录，用来本地存储。

默认：`${hbase.tmp.dir}/local/`

**hbase.master.port**

Master 绑定的端口

默认：16000

**hbase.master.info.port**

Master 的 Web UI 端口

默认：16010

**hbase.master.info.bindAddress**

Master Web UI 的绑定地址

默认：0.0.0.0

**hbase.master.logcleaner.plugins**

`LogsCleaner`服务用到的` BaseLogCleanerDelegate`列表。这些 WAL cleaner 会顺序调用。

默认：`org.apache.hadoop.hbase.master.cleaner.TimeToLiveLogCleaner`

**hbase.master.logcleaner.ttl**

WAL 停留在 `.oldlogdir` 目录中的最大时间，之后就被 Master 线程清理掉。

默认：600000

**hbase.master.hfilecleaner.plugins**

`HFileCleaner`服务用到的`BaseHFileCleanerDelegate`列表。

默认：`org.apache.hadoop.hbase.master.cleaner.TimeToLiveHFileCleaner`

**hbase.master.infoserver.redirect**

Master 是否监听 Master Web UI 端口，并将请求重定向到 Master 和 RegionServer 共享的 Web UI 服务器。

默认：`true`

**hbase.regionserver.port**

RegionServer 绑定的端口。

默认：16020

**hbase.regionserver.info.port**

RegionServer 的 Web UI 端口

默认：16030

**hbase.regionserver.info.bindAddress**

RegionServer Web UI 的绑定地址

默认：0.0.0.0

**hbase.regionserver.info.port.auto**

Master 或 RegionServer 是否应该查找一个能够绑定的端口。生产环境一般不用，直接 `hbase.regionserver.info.port`。

默认：`false`

**hbase.regionserver.handler.count**

RegionServer 上启动的 RPC 处理实例的数量，同样的属性 Master 上也有。

默认：30

**hbase.ipc.server.callqueue.handler.factor**

调用队列数量的因子。0 意味着所有的 handlers 共享一个队列。1 意味着每个 handler 都有自己的队列。

默认：0.1

**hbase.ipc.server.callqueue.read.ratio**

将调用队列拆分为读和写两个队列。0 意味着不拆分调用队列，即读和写请求都被 push 到该队列。0.5 意味着有相同数量的读队列和写队列。1.0 意味着除了一个之外的所有队列，都用于分发读请求。

默认：0

**hbase.ipc.server.callqueue.scan.ratio**

给定读队列的数量，设定 `get` 和 `scan` 队列的比例。比如设置为0.1，表示1个队列用于scan请求，另外8个用于get请求。

默认：0

**hbase.regionserver.msginterval**

RegionServer 到 Master 消息的时间间隔。

默认：3000

**hbase.regionserver.logroll.period**

滚动日志的周期，不管日志中包含多少个修改。

默认：3600000

**hbase.regionserver.logroll.errors.tolerated**

触发服务器终止之前允许的 连续的WAL 关闭错误数量。0 意味着在log rolling 时关闭当前 WAL writer，服务器会终止。设置一个小数值，允许 RegionServer 无视一些 HDFS 错误。

默认：2

**hbase.regionserver.hlog.reader.impl**

WAL file reader 的实现。

默认：`org.apache.hadoop.hbase.regionserver.wal.ProtobufLogReader`

同样的 hbase.regionserver.hlog.writer.impl，默认是：`org.apache.hadoop.hbase.regionserver.wal.ProtobufLogWriter`。

**hbase.regionserver.global.memstore.size**

在更新和刷写之前，一台 RegionServer 上允许的所有 MemStore 的 最大大小。默认是 0.4 倍的堆大小。

如果总大小达到了 `hbase.regionserver.global.memstore.size.lower.limit`，更新会被阻塞、刷写会被强制执行。这个值默认为空，让位于`hbase.regionserver.global.memstore.size.lower.limit`。

默认：none。

**hbase.regionserver.global.memstore.size.lower.limit**

刷写被强制执行前，一台 RegionServer 上所有 MemStore 的最大大小。默认是 `hbase.regionserver.global.memstore.size` 的 95%。100% 意味着 MemStore 限制让更新阻塞的时候，只会引发最小的可能的刷写。默认选项留空，让位于`hbase.regionserver.global.memstore.lowerLimit`。

默认：none。

**hbase.regionserver.optionalcacheflushinterval**

在被自动刷写前，一个修改在内存中的最长存活时间。默认一小时，设置为0表示禁用自动刷写。

默认：3600000

**hbase.regionserver.dns.interface**

RegionServer 报告 IP 地址的网络接口名字。

默认：`default`。

**hbase.regionserver.dns.nameserver**

RegionServer 使用的 DNS 主机名或 IP 地址，RegionServer 据此与 Master 通信。

默认：`default`。

**hbase.regionserver.region.split.policy**

决定 region 何时被拆分的策略。可选项包括：BusyRegionSplitPolicy, ConstantSizeRegionSplitPolicy, DisabledRegionSplitPolicy, DelimitedKeyPrefixRegionSplitPolicy, and KeyPrefixRegionSplitPolicy。

默认：`org.apache.hadoop.hbase.regionserver.SteppingSplitPolicy`。

**hbase.regionserver.regionSplitLimit**

