## ShuffleDependency — Shuffle Dependency {#__a_id_shuffledependency_a_shuffledependency_shuffle_dependency}

ShuffleDependency 是对于键值对 RDD 的 ShuffleMapStage 的输出的 RDD 依赖性。

ShuffleDependency 使得 RDD 知道（map-side/pre-shuffle）分区的数量，Partitioner 用于（reduce-size/post-shuffle）分区的数量。

但是只有当 partitioners \(of the RDD’s and after transformations\) 不同时，ShuffleDependency 是 ShuffledRDD 以及 CoGroupedRDD 和 SubtractedRDD 的依赖。

为键值对 RDD 创建 ShuffleDependency，即 RDD \[Product2 \[K，V\]\]，其中 K 和 V 分别是 keys 和 values 的类型。

| Tip | 对RDD使用 dependencies method 来了解依赖关系。 |
| :---: | :--- |


```
scala> val rdd = sc.parallelize(0 to 8).groupBy(_ % 3)
rdd: org.apache.spark.rdd.RDD[(Int, Iterable[Int])] = ShuffledRDD[2] at groupBy at <console>:24

scala> rdd.dependencies
res0: Seq[org.apache.spark.Dependency[_]] = List(org.apache.spark.ShuffleDependency@454f6cc5)
```

每个 ShuffleDependency 都有在创建 ShuffleDependency 时分配（和用于整个 spark 的代码来引用 ShuffleDependency）一个独特的应用范围 shuffleId number。

| Note | Shuffle ID 由 SparkContext 跟踪。 |
| :---: | :--- |


### `keyOrdering`Property {#__a_id_keyordering_a_code_keyordering_code_property}

| Caution | FIXME |
| :--- | :--- |


### `serializer`Property {#__a_id_serializer_a_code_serializer_code_property}

| Caution | FIXME |
| :--- | :--- |


### Creating ShuffleDependency Instance {#__a_id_creating_instance_a_creating_shuffledependency_instance}

ShuffleDependency 在创建时采用以下内容：

1. A single key-value pair RDD, i.e.`RDD[Product2[K, V]]`,

2. Partitioner\(available as`partitioner`property\),

3. Serializer,

4. Optional key ordering \(of Scala’s [scala.math.Ordering](http://www.scala-lang.org/api/current/scala/math/Ordering.html) type\),

5. Optional Aggregator,

6. mapSideCombine flag which is disabled \(i.e.`false`\) by default.

| Note | `ShuffleDependency`uses`SparkEnv`to access`Serializer`. |
| :---: | :--- |


When created,`ShuffleDependency`gets shuffle id\(as`shuffleId`\).

| Note | `ShuffleDependency`uses the input RDD to access`SparkContext`and so the`shuffleId`. |
| :---: | :--- |


ShuffleDependency 使用 ShuffleManager 注册自身并获取 ShuffleHandle（可用作 shuffleHandle 属性）。

| Note | `ShuffleDependency`accesses`ShuffleManager`using`SparkEnv`. |
| :--- | :--- |


In the end,`ShuffleDependency`registers itself for cleanup with`ContextCleaner`.

| Note | `ShuffleDependency`accesses the optional`ContextCleaner`through`SparkContext`. |
| :--- | :--- |


| Note | `ShuffleDependency`is created when ShuffledRDD,CoGroupedRDD, and SubtractedRDD return their RDD dependencies. |
| :--- | :--- |


### `rdd`Property {#__a_id_rdd_a_code_rdd_code_property}

```
rdd: RDD[Product2[K, V]]
```

rdd 返回为这个 ShuffleDependency 创建的键值对 RDD。

rdd 用于：

1. `MapOutputTrackerMaster`finds preferred`BlockManagers`with most map outputs for a`ShuffleDependency`,\(MapOutputTrackerMaster 针对 ShuffleDependency 查找具有大多数 map 输出的首选 BlockManagers\)

2. `DAGScheduler`finds or creates new`ShuffleMapStage`stages for a`ShuffleDependency`,\(DAGScheduler 为 ShuffleDependency 找到或创建新的 ShuffleMapStage 阶段\)

3. `DAGScheduler`creates a`ShuffleMapStage`for a`ShuffleDependency`and a`ActiveJob`,\(DAGScheduler 为 ShuffleDependency 和 ActiveJob 创建 ShuffleMapStage\)

4. `DAGScheduler`finds missing ShuffleDependencies for a RDD,

5. `DAGScheduler`submits a`ShuffleDependency`for execution.

### `partitioner`Property {#__a_id_partitioner_a_code_partitioner_code_property}

partitioner 属性是用于分区 shuffle 输出的分区器。

在创建 ShuffleDependency 时指定分区器。

partitioner 用于：

1. `MapOutputTracker`computes the statistics for a`ShuffleDependency`\(and is the size of the array with the total sizes of shuffle blocks\),（MapOutputTracker 计算 ShuffleDependency 的统计信息（是具有 shuffle 块总大小的数组大小））

2. `MapOutputTrackerMaster`finds preferred`BlockManagers`with most map outputs for a`ShuffleDependency`,（MapOutputTrackerMaster 针对 ShuffleDependency 查找具有大多数 map 输出的首选 BlockManagers）

3. ShuffledRowRDD.adoc\#numPreShufflePartitions,

4. `SortShuffleManager`checks if`SerializedShuffleHandle`can be used \(for a`ShuffleHandle`\).

5. FIXME

### `shuffleHandle`Property {#__a_id_shufflehandle_a_code_shufflehandle_code_property}

```
shuffleHandle: ShuffleHandle
```

shuffleHandle 是在创建 ShuffleDependency 时，随机分配的 ShuffleDependency 的 ShuffleHandle。

| Note | shuffleHandle 用于计算 CoGroupedRDDs，ShuffledRDD，SubtractedRDD 和 ShuffledRowRDD（以获得 ShuffleDependency 的 ShuffleReader）和 ShuffleMapTask 运行时（为 ShuffleDependency 获取 ShuffleWriter）。 |
| :---: | :--- |


### Map-Size Combine Flag — `mapSideCombine`Attribute {#__a_id_mapsidecombine_a_map_size_combine_flag_code_mapsidecombine_code_attribute}

mapSideCombine 是控制是否使用部分聚合（也称为 map-side combine）的标志。

默认情况下，mapSideCombine 在创建 ShuffleDependency 时禁用（即 false）。

启用时，SortShuffleWriter 和 BlockStoreShuffleReader 假定还定义了聚合器。

| Note | 当 ShuffledRDD 返回依赖性（这是一个 ShuffleDependency）时，mapSideCombine 是唯一设置的（因此可以被启用）。 |
| :---: | :--- |


### `aggregator`Property {#__a_id_aggregator_a_code_aggregator_code_property}

```
aggregator: Option[Aggregator[K, V, C]] = None
```

`aggregator`is a map/reduce-side Aggregator\(for a RDD’s shuffle\).

`aggregator`is by default undefined \(i.e.`None`\) when`ShuffleDependency`is created.

| Note | 当 SortShuffleWriter 写入记录时使用 aggregator，而 BlockStoreShuffleReader 读取 reduce 任务的 combined key-values。 |
| :---: | :--- |


### Usage {#_usage}

使用 ShuffleDependency 的地方：

* ShuffledRDD 和 ShuffledRowRDD 是来自 shuffle 的 RDD

RDD操作可能使用或可能不使用上述RDD，因此 shuffling：

* coalesce

  * repartition

* `cogroup`

  * `intersection`

* `subtractByKey`

  * `subtract`

* `sortByKey`

  * `sortBy`

* `repartitionAndSortWithinPartitions`

* combineByKeyWithClassTag

  * `combineByKey`

  * `aggregateByKey`

  * `foldByKey`

  * `reduceByKey`

  * `countApproxDistinctByKey`

  * `groupByKey`

* `partitionBy`

| Note | There may be other dependent methods that use the above. |
| :--- | :--- |




