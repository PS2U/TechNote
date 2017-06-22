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

5. 启动 HBase `bin/start-hbase.sh`
