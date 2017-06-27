HBase 数据模型的名词：

- 表。一张表包含多个列。
- 列。一列包含 RowKey 以及一个或多个带值的列。行按照 RowKey 的字母序排列，这就凸显了 RowKey 设计的重要性。
- 列。一列由一个列族和一个列限定符组成，以冒号分隔。
- 列族。列族在物理上分开了不同的列和它们的值。每个列族都有一系列存储的属性，如cache、压缩、RowKey 编码等。表中的每一行都有相同的列族，尽管它可以不存任何数据。
- 列限定符。列限定符附在列族上，提供了特定分片的数据索引。列族在表创建的时候就固定了，但列限定符可以动态添加。不同的行，列限定符也可以大相径庭。
- Cell。Cell 是行、列族、列限定符的结合，包括一个值和一个时间戳。
- 时间戳。每个值都有一个时间戳，标识了值的版本。它是在 RegionServer 更新数据的同时写入的。


# 19. 概念视图

表 `webtable`：

| Row Key       | Time Stamp | ColumnFamily `contents`   | ColumnFamily `anchor`         | ColumnFamily `people` |
| ------------- | ---------- | ------------------------- | ----------------------------- | --------------------- |
| "com.cnn.www" | t9         |                           | anchor:cnnsi.com = "CNN"      |                       |
| "com.cnn.www" | t8         |                           | anchor:my.look.ca = "CNN.com" |                       |
| "com.cnn.www" | t6         | contents:html = "<html>…" |                               |                       |
| "com.cnn.www" | t5         | contents:html = "<html>…" |                               |                       |
| "com.cnn.www" | t3         | contents:html = "<html>…" |                               |                       |

表中空白的 Cell 实际上并不存在。

# 20. 物理视图

表在物理上是按照列族存储的。

列族 `anchor`：

| Row Key       | Time Stamp | Column Family `anchor`          |
| ------------- | ---------- | ------------------------------- |
| "com.cnn.www" | t9         | `anchor:cnnsi.com = "CNN"`      |
| "com.cnn.www" | t8         | `anchor:my.look.ca = "CNN.com"` |

列族 `contents`: 

| Row Key       | Time Stamp | ColumnFamily `contents:`  |
| ------------- | ---------- | ------------------------- |
| "com.cnn.www" | t6         | contents:html = "<html>…" |
| "com.cnn.www" | t5         | contents:html = "<html>…" |
| "com.cnn.www" | t3         | contents:html = "<html>…" |

概念视图中的空 Cell 压根不存储。当你请求`contents:html`在时间戳`t8`的值，不返回任何值。

不带时间戳的访问请求，默认访问的是最新的值，即时间戳最大的值。

# 21. 命名空间

命名空间是表的逻辑分组，类似于关系数据库中的 `database`。基于这个抽象，可以推导出多租户的相关特性：

- 配额管理。限制一个命名空间可消费的资源。
- 命名空间安全管理。为各租户提供不同级别的安全管理。
- RegionServer 分组。一个命名空间、表可以 pin 在几个 RegionServe 上，做到一定程度上是隔离。

## 21.1 命名空间管理

命名空间可以被创建、移除、修改。命名空间的成员在表创建的时候就确定了：

```
#Create a namespace
create_namespace 'my_ns'

#create my_table in my_ns namespace
create 'my_ns:my_table', 'fam'

#drop namespace
drop_namespace 'my_ns'

#alter namespace
alter_namespace 'my_ns', {METHOD => 'set', 'PROPERTY_NAME' => 'PROPERTY_VALUE'}
```

## 21.2 预定义的命名空间

有两个预定义的特殊命名空间：

- `hbase`。系统命名空间，用来容纳 HBase 的内部表。
- `default`。没有指定命名空间的表，默认都是这项。

```
#namespace=foo and table qualifier=bar
create 'foo:bar', 'fam'

#namespace=default and table qualifier=bar
create 'bar', 'fam'
```



# 22. 表

表在定义 Schema 的时候就声明了。



# 23. 列

RowKey 是无意义的字节数组。列按照字母顺序递增排列。

空的字节数组表示表的命名空间的起止。



# 24. 列族

HBase 的列按照列族分组，同一列族的列有相同的前缀。列族前缀必须是可打印的字符，限定符则可以是任意字节。

列族在 Scheme 定义的时候就确定了，限定符则可以动态添加。

调优和存储都是在列族这一级别做的，所以最好同一列族的成员有相同的访问模式和大小特征。



# 25. Cell

*{row, column, version}* 元组精确定位了一个 Cell。Cell 的内容是粗略的字节。



# 26. 数据模型操作

## 26.1 Get

Get 操作返回一个特定的列，执行的`Table.get`。

## 26.2 Put

Put 可以添加新行、修改已存在的行，执行的是`Table.put`或`Table.batch`（无写缓冲）。

## 26.3 扫描

```java
public static final byte[] CF = "cf".getBytes();
public static final byte[] ATTR = "attr".getBytes();
...

Table table = ...      // instantiate a Table instance

Scan scan = new Scan();
scan.addColumn(CF, ATTR);
scan.setRowPrefixFilter(Bytes.toBytes("row"));
ResultScanner rs = table.getScanner(scan);
try {
  for (Result r = rs.next(); r != null; r = rs.next()) {
    // process result...
  }
} finally {
  rs.close();  // always close the ResultScanner!
}
```

## 26.4 Delete 

Delete 删除一列，执行`Table.delete`。

HBase 不是原地修改数据，而是为它创建个墓碑标记。在 Major Compaction 的时候，带有该标记的值会被清理。


# 27. 版本


