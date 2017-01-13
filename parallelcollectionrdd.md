## ParallelCollectionRDD {#_parallelcollectionrdd}

ParallelCollectionRDD 是具有 numSlices 分区和可选 locationPrefs 的元素集合的 RDD。

ParallelCollectionRDD 是 SparkContext.parallelize 和 SparkContext.makeRDD 方法的结果。

数据收集被分割到 numSlices 切片。

它使用 ParallelCollectionPartition。



























