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

