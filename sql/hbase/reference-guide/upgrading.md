升级的时候不能跳过主版本号，如果你想从0.90.x 升级到 0.94.x，必须先升级到 0.92.x，再升级到 0.94.x。

# HBase 版本号和兼容性

## 11.1 1.0 之后的笨笨

1.0.0之后，HBase 朝着语义版本控制进发，对一个版本号 `MAJOR.MINOR.PATCH` 而言：

- 改动了 API，增加 MAJOR 号
- 增加某项功能，并向后兼容，增加 MINOR 号
- 修改了 bug，并向后兼容，增加 PATCH 号
- 额外的标签，作为 `MAJOR.MINOR.PATCH` 的扩展使用

Client-Server 线路协议的兼容性：

- 允许不同步的 client 和 server 升级
- 只允许先升级 server，因为 server 可以兼容老的 client。

Server-Server 协议的兼容性：

- 同一集群中，不同版本的 Server 不能并存
- Server 间的线路协议是兼容的
- 分布式任务的 Worker，如 replication 和日志拆分，在同一集群中可以共存
- 依赖的协议（如 ZK）也不能改变

文件格式兼容性：

- 支持的文件格式是向前和向后兼容的
- 文件、ZK 编码、目录结构随着 HBase 的升级一起升级。

Client API 兼容性：

- 允许改变或移除Client API。
- 在改变或移除一个 API 之前，要在 major 版本中先 deprecate 掉。
- patch 版本中可用的 API，会出现在其后所有的 patch 版本中，但新的 API 对于前面的 patch 版本就不可见。
- patch 版本中添加的新 API，必须以代码兼容的方式添加。

Client 二进制兼容性：

- 某个 patch 版本中增加的 API，在以后的 patch 版本 jar 中，无差别地运行。
- 某个 patch 版本中增加的 API，在之前的 patch 版本 jar 中，可能跑不起来。

Server 端有限的 API 兼容性：

- 内部 API 都被标记为 Stable、Envolving、Unstable。
- 这暗示着协处理和插件的二进制兼容性，只使用标记的接口和类。
- 老的协处理器、过滤器、插件代码，在新 jar 包里，也能完美运行。

依赖兼容性：

- HBase 升级不需要其依赖项目的升级，如 Java 运行环境。
- 例如：Hadoop 的升级不会使 HBase 的兼容性失效。

操作兼容性：

- Metric 改变
- 服务的行为改变
- `jmx/`端点暴露的JMX API

总结：

- patch 升级是插入式的替换，任何和 Java 二进制或源码不兼容的修改都不被允许。patch 降级到先前的 patch 是不兼容的。
- Minor 升级不需要应用或客户端的代码改动。理想的情况下它会是一个插入式的替换，但客户端源码、协处理器、过滤器，如果使用了新的 jar 包，要重新编译。
- Major 升级允许 HBase 做出重大的改变。

### HBase API Surface


