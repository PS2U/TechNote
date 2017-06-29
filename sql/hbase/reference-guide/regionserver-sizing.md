这篇博文值得参考：[HBase region server memory sizing](http://hadoop-hbase.blogspot.com/2013/01/hbase-region-server-memory-sizing.html)。

# 34. 列族的数量

HBase 不能很好地处理两个以上的列族。flush 和 compaction 是以 region 为单位进行的，当一个列族需要flush，相邻的列族也会被刷新。

尽量使用一个列族，除非你不会同时访问两个列族。

## 34.1 列族的基数

如果有多个列族，注意列族的基数（即行的数量）。

假设有个列族 A 有个100w 行，列族 B 有10亿行。A 的数据可能分布在多个 region （RegionServer）上。这就上针对 A 的扫描操作异常低效。



# 35. RowKey 设计

## 35.1 热点

HBase 的 RowKey 按字母顺序存储。因此，对 RowKey 的设计要对扫描友好，即允许将相关的 row 存在一起。差劲的 RowKey 设计会引发热点，大部分的访问请求都涌向少数节点。

**加盐**

加盐是为 RowKey 添加一个随机前缀，确保 Rowkey 按照不同的方式来排序。

**哈希**

使用单向哈希将给定的列加上相同的前缀，将它们分发到不同的 RegionServer 上，且读的时候能够预见。

**翻转 Key**

翻转固定宽度或数字的 RowKey，让最长变化的部分放在 RowKey 的前面。

## 35.2 单向递增的 RowKey 或时间序列 数据

避免使用时间戳或时间序列作为 RowKey，这会将数据一股脑儿地扔向某个 region。

在 HBase 中存储时间序列的数据，参考[OpenTSDB](http://opentsdb.net/)。

## 35.3 最小化 row 和 列的大小

[HBASE-3551](https://issues.apache.org/jira/browse/HBASE-3551?page=com.atlassian.jira.plugin.system.issuetabpanels:comment-tabpanel&focusedCommentId=13005272#comment-13005272) 这种 case，就是 cell 的基数太大。

### 列族

保持列的名字尽可能短。推荐一个字符（如，"d" 代表 `data/default`）。

### 属性

建议用短的属性名。

### RowKey 长度

尽可能短，但也要对 Get 和 Scan 友好。

这是一种折中。

### 字节模式

```java
// long
//
long l = 1234567890L;
byte[] lb = Bytes.toBytes(l);
System.out.println("long bytes length: " + lb.length);   // returns 8

String s = String.valueOf(l);
byte[] sb = Bytes.toBytes(s);
System.out.println("long as string length: " + sb.length);    // returns 10

// hash
//
MessageDigest md = MessageDigest.getInstance("MD5");
byte[] digest = md.digest(Bytes.toBytes(s));
System.out.println("md5 digest bytes length: " + digest.length);    // returns 16

String sDigest = new String(digest);
byte[] sbDigest = Bytes.toBytes(sDigest);
System.out.println("md5 digest as string length: " + sbDigest.length);    // returns 26
```

采用 `long` 还是 `string`来表示数字，是一种权衡：

- 要么更少的存储空间
- 要么更好的可读性

## 35.4 反向时间戳

反向时间戳，能让你快速找到最近版本的值。它将(`Long.MAX_VALUE - timestamp)`作为后缀加到 key 的后面。

## 35.5 RowKey 和列族

RowKey 的范围限定为列族，因为不同列族的同一个 RowKey，可以共存。

## 35.6 RowKey 的不可变性

RowKey 不能更改，但可以删除重建。

## 35.7 RowKey 和 Region 拆分的关系

欲拆分表的前提，是对 RowKey 非常了解，切要确保 所有的 region 都落在 key 的空间里。

不建议使用十六进制，但它却能够保证所有的 region 都落在 key 的空间里。



# 36. 版本数量

## 36.1 版本的最大数量

每个列族都有自己的版本最大数量，通过`HColumnDescriptor`配置，默认是1。

HBase 不是重写行的值，而是添加一个不同时间戳的新值。在 Major Compaction 时，多余版本的值会被清除。

不建议将该值调得很大，因为它会增加 StoreFile 的大小。

## 36.2 版本的最小数量

也是通过`HColumnDescriptor`配置，默认是0 。

它常于 TTL 一起使用，意在只保留最近的几个数据。



# 37. 支持的数据类型

HBase 只支持字节的写入和读出。其他输入，如字符串、数字、复杂结构体、图片，都转成字节才能写。

## 37.11 计数器

HBase 还支持一个计数器，参见[Increment](http://hbase.apache.org/apidocs/org/apache/hadoop/hbase/client/Table.html#increment%28org.apache.hadoop.hbase.client.Increment%29)。

计数器的同步是在 RegionServer 做的，而不是 Client 。



# 38. Joins

如果你有多张表，可能在 Schema 设计的时候，就考虑到 Join 的问题，毕竟 HBase 不支持 Join。



# 39. TTL

