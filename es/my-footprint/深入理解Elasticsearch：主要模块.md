Elasticsearch中每个模块的设置可以是：

- 静态的。这些设置是节点级别的，要么在`elasticsearch.yml`、要么在环境变量、要么在命令行启动参数。
- 动态的。使用API更新。

# 集群路由和分片分配（Cluster-level routing and shard allocation）

> 控制分片什么时候、怎样分配给哪个节点。

集群相关的设置都是动态的，可以通过API调整。

## 1. 集群级别的分片分配

分片分配是将分片放到节点的过程，它可以发生在初始化回复、备份分配、rebalance、移除或新增节点时。

### 分片分配的设置

分片分配和恢复的常见设置：

- `cluster.routing.allocation.enable` 控制特定类型的分片分配。
- ​



# 发现（Discovery）

>  节点如何相互发现从而构成一个集群。

# 网关（Gateway）

> 恢复启动之前有多少个节点需要加入集群。

# HTTP

> 控制HTTP REST 接口

# 索引（Indices）



# 网络（Network）



# 节点客户端（Node client）

> Java 节点客户端加入集群，但不存储任何数据，也不能成为master节点。

# 无痛（Pailess）

> Elasticsearch内置的脚本语言，旨在安全。

# 插件（Plugins）

# 脚本（Scripting）

# 快照（Snapshot/Restore)

# 线程池（Thread pools）

# 传输（Transport）

> 配置网络传输层，Elasticsearch用来实现节点间的通信。

# Tribe 节点（Tribe nodes）

> 一个Tribe节点可以加入一个或多个集群，在多个集群的范围内扮演一个联邦客户端(federated client)。

