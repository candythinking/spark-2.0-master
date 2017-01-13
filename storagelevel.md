## StorageLevel {#__a_id_storagelevel_a_storagelevel}

StorageLevel 描述 RDD 如何持久化（并解决以下问题）：

* RDD 是否使用磁盘？

* RDD 在内存中有多少？

* RDD 是否使用堆外存储器？

* RDD 是否应该被序列化（而持久）？

* 有多少个副本（默认：1）要使用（只能小于40）？

有以下 StorageLevel（名称中的数字\_2表示2个副本）：

* `NONE`\(default\)

* `DISK_ONLY`

* `DISK_ONLY_2`

* `MEMORY_ONLY`\(default for`cache`operation for RDDs\)

* `MEMORY_ONLY_2`

* `MEMORY_ONLY_SER`

* `MEMORY_ONLY_SER_2`

* `MEMORY_AND_DISK`

* `MEMORY_AND_DISK_2`

* `MEMORY_AND_DISK_SER`+

* `MEMORY_AND_DISK_SER_2`

* `OFF_HEAP`

您可以使用 getStorageLevel（）操作检出存储级别。

```
val lines = sc.textFile("README.md")

scala> lines.getStorageLevel
res0: org.apache.spark.storage.StorageLevel = StorageLevel(disk=false, memory=false, offheap=false, deserialized=false, replication=1)
```



