关于非关系型数据库的优劣，参考博文：[No Relation: The Mixed Blessings of Non-Relational Databases](http://ianvarley.com/UT/MR/Varley_MastersReport_Full_2009-08-07.pdf)。

[Designing Your Schema](https://cloud.google.com/bigtable/docs/schema-design) 这篇文章对 HBase 也有很好的指导意义。

# 32. 创建 Schema

```java
Configuration config = HBaseConfiguration.create();
Admin admin = new Admin(conf);
TableName table = TableName.valueOf("myTable");

admin.disableTable(table);

HColumnDescriptor cf1 = ...;
admin.addColumn(table, cf1);      // adding new ColumnFamily
HColumnDescriptor cf2 = ...;
admin.modifyColumn(table, cf2);    // modifying existing ColumnFamily

admin.enableTable(table);
```

## 32.1 更新 Schema

无论是针对 Table 还是列族的修改，生效都要等到下一次 Major Compaction 之后。


# 33. Schema 的经验法则

以下只是一些经验法则，具体的实施情况还是要看自己的数据：

- region 大小在 10G ~ 50G。
- cell 大小不超过 10MB。如果你有 MOB，这个上限是 50MB。
- 一张表最好 1 ~ 3 个列族，不要设计成关系型数据库。
- 一张表最好 50 ~ 100 个 region。PS. 一个 region 是一个列族的一块连续分段。
- 列族名尽可能短，因为要排序。
- 如果写请求都是发往某个列族，这个列族就会占据内存。分配资源的时候，就要注意写的方式。
- 如果存储的是日志或其他基于时间的数据，RowKey 设计成设备 ID + 时间。



# 导航

[目录](README.md)

上一章：[Data Model](data-model.md)

下一章：[RegionServer Sizing Rules of Thumb](regionserver-sizing.md)
