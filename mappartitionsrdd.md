## MapPartitionsRDD {#_mappartitionsrdd}

MapPartitionsRDD 是将提供的函数 f 应用于父 RDD 的每个分区的 RDD。

默认情况下，它不保留分区 - 最后一个输入参数 preservevesPartitioning 为 false。如果为真，它保留原始 RDD 的分区。

`MapPartitionsRDD`is the result of the following transformations:

* `map`

* `flatMap`

* `filter`

* `glom`

* mapPartitions

* `mapPartitionsWithIndex`+

* PairRDDFunctions.mapValues

* PairRDDFunctions.flatMapValues











