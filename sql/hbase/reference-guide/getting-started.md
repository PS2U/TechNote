# 2. 快速开始 - Standalone HBase

Standalone 实例包含所有的 HBase 守护进程：

- Master
- RegionServer
- ZooKeeper

## 2.1 JDK 版本

HBase 需要安装 JDK。

| HBase Version | JDK 7                                    | JDK 8                                    |
| ------------- | ---------------------------------------- | ---------------------------------------- |
| 2.0           | [Not Supported](http://search-hadoop.com/m/YGbbsPxZ723m3as) | yes                                      |
| 1.3           | yes                                      | yes                                      |
| 1.2           | yes                                      | yes                                      |
| 1.1           | yes                                      | Running with JDK 8 will work but is not well tested. |

## 2.2 开始

1. 前往[Apache Download Mirrors](http://www.apache.org/dyn/closer.cgi/hbase/)下载 HBase，选择 `stable`目录下带`.bin.tar.gz`后缀的二进制文件。

2. 解压文件

3. 设置 `JAVA_HOME` 环境变量。HBase 提用了一个配置中心：`conf/hbase-env.sh`，在这里设置环境变量。

4. 更改 `conf/hbase.site.xml` 配置，如数据存放路径等。

```xml
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>file:///home/testuser/hbase</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/home/testuser/zookeeper</value>
  </property>
</configuration>
```

5. 启动 HBase `bin/start-hbase.sh`。使用`jps`命令可以查看`HMaster`进程。

**首次使用 HBase**

1. 连接到 HBase：`hbase shell`。

2. 查看 `help` 选项。

3. 创建表：`create 'test', 'cf'`。

4. 查看表的信息：`list 'test'`。

5. `Put` 数据到表中：`put 'test', 'row1', 'cf:a', 'value1'`。

6. 扫描表中的所有数据：`scan 'test'`。

7. 取单行数据：`get 'test', 'row1'`。

8. Disable 一个表：`disable 'test'`。

9. Drop 表：`drop 'test'`。

10. 退出 HBase Shell：`quit`。

**关闭 HBase**

```shell
bin/stop-hbase.sh
```

脚本执行需要几分钟时间，在此之后使用 `jps` 命令确认进程是否终止。


## 2.3  伪分布模式

伪分布模式意味着 HBase 在一台机器上运行`HMaster`、`HRegionServer`、`ZooKeeper`等独立的进程，而 Standalone 模式下所有的后台服务都是运行在一个 JVM 进程中。默认的 `hbase.rootdir` 为 `/tmp/`。

1. 确保关闭了 HBase。

2. 配置 HBase，修改`hbase-site.xml`中的配置 `hbase.cluster.distributed` 为 `true`。

3. 启动 HBase，`bin/start-hbase.sh`。

4. 检查 HDFS 中 HBase 的目录。`hdfs dfs -ls /hbase`。

5. 创建表，填充数据。

6. 启动和终止一个备用 HMaster 服务器，`local-master-back.sh`，终止一一个备用 HMaster，只能 `kill -9`。HMaster 控制着整个 HBase 集群。

7. 启动和终止额外的 RegionServer，`local-regionservers.sh start 2 3 4 5` 和 `local-regionservers.sh stop 3`。 RegionServer 使用 StoreFile 管理数据，直接受 HMaster 领导。通常来说，一台 HRegionServer 运行在一个节点上。一个机器上运行多个 HRegionServer 是伪分布模式。

8. 关闭 HBase，`bin/stop-hbase.sh`。

# 2.4 进阶 - 全分布式




