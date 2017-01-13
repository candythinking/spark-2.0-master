## ShuffledRDD {#__a_id_shuffledrdd_a_shuffledrdd}

ShuffledRDD 是表示 RDD 谱系血统中的 shuffle 步骤的键值对的 RDD。它使用自定义 ShuffledRDDPartition 分区。

ShuffledRDD 是为触发数据混洗的 RDD 转换创建的：

1. coalesce transformation \(with shuffle flag enabled\).（合并转换（启用 shuffle 标志）。）

2. PairRDDFunctions 的 combineByKeyWithClassTag 和 partitionBy（当父 RDD 和指定的分区器不同时）。

3. OrderedRDDFunctions 的 sortByKey 和 repartitionAndSortWithinPartitions 有序运算符。

```
scala> val rdd = sc.parallelize(0 to 9)
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[0] at parallelize at <console>:24

scala> rdd.getNumPartitions
res0: Int = 8

// ShuffledRDD and coalesce Example

scala> rdd.coalesce(numPartitions = 4, shuffle = true).toDebugString
res1: String =
(4) MapPartitionsRDD[4] at coalesce at <console>:27 []
 |  CoalescedRDD[3] at coalesce at <console>:27 []
 |  ShuffledRDD[2] at coalesce at <console>:27 []
 +-(8) MapPartitionsRDD[1] at coalesce at <console>:27 []
    |  ParallelCollectionRDD[0] at parallelize at <console>:24 []

// ShuffledRDD and sortByKey Example

scala> val grouped = rdd.groupBy(_ % 2)
grouped: org.apache.spark.rdd.RDD[(Int, Iterable[Int])] = ShuffledRDD[6] at groupBy at <console>:26

scala> grouped.sortByKey(numPartitions = 2).toDebugString
res2: String =
(2) ShuffledRDD[9] at sortByKey at <console>:29 []
 +-(8) ShuffledRDD[6] at groupBy at <console>:26 []
    +-(8) MapPartitionsRDD[5] at groupBy at <console>:26 []
       |  ParallelCollectionRDD[0] at parallelize at <console>:24 []
```

ShuffledRDD在 创建时采用父 RDD 和 Partitioner。

getDependencies 返回一个具有 ShuffleDependency 的 RDD 依赖关系的单元素集合（使用 Serializer 根据地图侧合并内部标志）。

### Map-Side Combine`mapSideCombine`Internal Flag {#__a_id_mapsidecombine_a_map_side_combine_code_mapsidecombine_code_internal_flag}

```
mapSideCombine: Boolean
```

mapSideCombine 内部标志用于在创建 ShuffleDependency（它是 ShuffledRDD 的唯一依赖项）时选择 Serializer（用于混排）。

| Note | mapSideCombine 仅在未明确指定 userSpecifiedSerializer 可选 Serializer（这是默认值）时使用。 |
| :---: | :--- |


| Note | mapSideCombine 使用 SparkEnv 访问当前的 SerializerManager。 |
| :---: | :--- |


如果启用（即 true），mapSideCombine 指示找到类型 K 和 C 的`Serializer`。否则，getDependencies 找到类型 K 和 V 的`Serializer`。

| Note | 创建 ShuffledRDD 时，指定类型 K，C 和 V. |
| :---: | :--- |


| Note | 当创建 ShuffledRDD 时，mapSideCombine 被禁用（即为 false），并且可以使用 setMapSideCombine 方法设置。setMapSideCombine 方法只用于实验 PairRDDFunctions.combineByKeyWithClassTag 转换。 |
| :---: | :--- |


### Computing Partition \(in`TaskContext`\) — `compute`Method {#__a_id_compute_a_computing_partition_in_code_taskcontext_code_code_compute_code_method}

```
compute(split: Partition, context: TaskContext): Iterator[(K, C)]
```

| Note | compute 是 RDD 约定的一部分，用于计算 TaskContext 中的给定分区。 |
| :---: | :--- |


在内部，compute 确保输入拆分是 ShuffleDependency。然后它请 ShuffleManager ShuffleReader 读取键值对（作为迭代器\[（K，C）\]）进行拆分。

| Note | compute 使用 SparkEnv 访问 ShuffleManager。 |
| :---: | :--- |


| Note | 分区具有 index 属性以指定 startPartition 和 endPartition 分区偏移。 |
| :---: | :--- |


### Finding Preferred Locations for Partition — `getPreferredLocations`Method {#__a_id_getpreferredlocations_a_finding_preferred_locations_for_partition_code_getpreferredlocations_code_method}

```
getPreferredLocations(partition: Partition): Seq[String]
```

| Note | getPreferredLocations 是 RDD 契约的一部分，用于指定放置首选项（也称为首选任务位置），即任务应尽可能靠近数据执行。 |
| :---: | :--- |


在内部，getPreferredLocations 请求 MapOutputTrackerMaster 用于输入分区（一个和唯一的 ShuffleDependency）的首选位置，即具有最多映射输出的 BlockManagers。

| Note | getPreferredLocations 使用 SparkEnv 访问 MapOutputTrackerMaster（在驱动程序上运行）。 |
| :---: | :--- |


### ShuffledRDDPartition {#__a_id_shuffledrddpartition_a_shuffledrddpartition}

ShuffledRDDPartition 在创建索引时获取索引（依次是 ShuffledRDD 的分区器计算的分区的索引）。

