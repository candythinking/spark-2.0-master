## PairRDDFunctions {#__a_id_pairrddfunctions_a_pairrddfunctions}

| Tip | 读取 PairRDDFunctions 的 scaladoc。 |
| :---: | :--- |


PairRDDFunctions 通过 Scala 的隐式转换在键值对的 RDD 中可用。

| Note | 分区是一种高级功能，直接链接到（或推断）使用 PairRDDFunctions。在分区和分区中阅读它。 |
| :---: | :--- |


### `countApproxDistinctByKey`Transformation {#__a_id_countapproxdistinctbykey_a_code_countapproxdistinctbykey_code_transformation}

| Caution | FIXME |
| :--- | :--- |


### `foldByKey`Transformation {#__a_id_foldbykey_a_code_foldbykey_code_transformation}

| Caution | FIXME |
| :--- | :--- |


### `aggregateByKey`Transformation {#__a_id_aggregatebykey_a_code_aggregatebykey_code_transformation}

| Caution | FIXME |
| :--- | :--- |


### `combineByKey`Transformation {#__a_id_combinebykey_a_code_combinebykey_code_transformation}

| Caution | FIXME |
| :--- | :--- |


### `partitionBy`Operator {#__a_id_partitionby_a_code_partitionby_code_operator}

```
partitionBy(partitioner: Partitioner): RDD[(K, V)]
```

| Caution | FIXME |
| :--- | :--- |


### `groupByKey`and`reduceByKey`Transformations {#__a_id_reducebykey_a_a_id_groupbykey_a_code_groupbykey_code_and_code_reducebykey_code_transformations}

reduceByKey 是 aggregateByKey 的一种特殊情况。

您可能想从另一个角度查看分区数。

通常，预先给定数量的分区（在从数据源加载数据时在 RDD 创建时）可能不重要，因此在它是 RDD 之后，仅通过键“重新分组”数据可能是...关键意图）。

您可以使用 groupByKey 或另一个 PairRDDFunctions 方法在一个处理流程中拥有键。

您可以使用可用于 RDD 的 partitionBy 作为元组的 RDD，即 PairRDD：

```
rdd.keyBy(_.kind)
  .partitionBy(new HashPartitioner(PARTITIONS))
  .foreachPartition(...)
```

想想种类具有低基数或高度偏斜分布的情况，并且使用分区技术可能不是最佳解决方案。

您可以执行以下操作：

```
rdd.keyBy(_.kind).reduceByKey(....)
```

### mapValues, flatMapValues {#__a_id_mapvalues_a_a_id_flatmapvalues_a_mapvalues_flatmapvalues}

### `combineByKeyWithClassTag`Transformations {#__a_id_combinebykeywithclasstag_a_code_combinebykeywithclasstag_code_transformations}

```
combineByKeyWithClassTag[C](
  createCombiner: V => C,
  mergeValue: (C, V) => C,
  mergeCombiners: (C, C) => C)(implicit ct: ClassTag[C]): RDD[(K, C)] (1)
combineByKeyWithClassTag[C](
  createCombiner: V => C,
  mergeValue: (C, V) => C,
  mergeCombiners: (C, C) => C,
  numPartitions: Int)(implicit ct: ClassTag[C]): RDD[(K, C)] (2)
combineByKeyWithClassTag[C](
  createCombiner: V => C,
  mergeValue: (C, V) => C,
  mergeCombiners: (C, C) => C,
  partitioner: Partitioner,
  mapSideCombine: Boolean = true,
  serializer: Serializer = null)(implicit ct: ClassTag[C]): RDD[(K, C)]
```

combineByKeyWithClassTag 转换默认情况下启用 mapSideCombine（即 true）。当输入分区程序与 RDD 中的当前分区程序不同时，它们使用 mapSideCombine 的值创建 ShuffledRDD。

| Note | combineByKeyWithClassTag 是基于 combineByKey 的 transformation，aggregateByKey，foldByKey，reduceByKey，countApproxDistinctByKey 和 groupByKey 的基本 transformations。 |
| :---: | :--- |








