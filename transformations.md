## Transformations {#_transformations}

**Transformations **是对 RDD 的延迟操作，它创建一个或多个新的 RDD，例如。 map，filter，reduceByKey，join，cogroup，randomSplit。

```
transformation: RDD => RDD
transformation: RDD => Seq[RDD]
```

换句话说，转换是以 RDD 作为输入并产生一个或多个 RDD 作为输出的函数。它们不改变输入 RDD（因为 RDD 是不可变的，因此不能被修改），但总是通过应用它们表示的计算来产生一个或多个新的 RDD。

通过应用 transformations，您可以使用最终 RDD 的所有父 RDD 逐步构建 RDD 谱系血统。

transformations 是惰性的，即不立即执行。只有在调用动作后才执行转换。

在执行转换之后，结果 RDD（s）将始终与它们的父节点不同，并且可以更小（例如，filter, count, distinct, sample），更大（例如flatMap, union, cartesian（笛卡尔）） 或相同大小（例如，map）。

| Caution | 有可能触发 job 的 transformations，例如。 sortBy，zipWithIndex等。 |
| :---: | :--- |


![](/img/mastering-apache-spark/spark core-rdd/figure15.png)

某些 transformations 可以是 **pipelined\(**流水线\)的，这是 Spark 用于提高计算性能的优化。

```
scala> val file = sc.textFile("README.md")
file: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[54] at textFile at <console>:24

scala> val allWords = file.flatMap(_.split("\\W+"))
allWords: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[55] at flatMap at <console>:26

scala> val words = allWords.filter(!_.isEmpty)
words: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[56] at filter at <console>:28

scala> val pairs = words.map((_,1))
pairs: org.apache.spark.rdd.RDD[(String, Int)] = MapPartitionsRDD[57] at map at <console>:30

scala> val reducedByKey = pairs.reduceByKey(_ + _)
reducedByKey: org.apache.spark.rdd.RDD[(String, Int)] = ShuffledRDD[59] at reduceByKey at <console>:32

scala> val top10words = reducedByKey.takeOrdered(10)(Ordering[Int].reverse.on(_._2))
INFO SparkContext: Starting job: takeOrdered at <console>:34
...
INFO DAGScheduler: Job 18 finished: takeOrdered at <console>:34, took 0.074386 s
top10words: Array[(String, Int)] = Array((the,21), (to,14), (Spark,13), (for,11), (and,10), (##,8), (a,8), (run,7), (can,6), (is,6))
```

There are two kinds of transformations:

* narrow transformations

* wide transformations

### Narrow Transformations {#__a_id_narrow_transformations_a_narrow_transformations}

**Narrow transformations **是 map, filter 等的结果，其仅来自单个分区的数据，即它是自持的。

输出 RDD 具有具有源自父 RDD 中的单个分区的记录的分区。只有用于计算结果的分区的有限子集。

Spark groups narrow transformations as a stage which is called **pipelining**。

### Wide Transformations {#__a_id_wide_transformations_a_wide_transformations}

**Wide transformations **是 groupByKey 和 reduceByKey 的结果。计算单个分区中的记录所需的数据可以驻留在父 RDD 的许多分区中。

| Note |  Wide transformations 也称为 shuffle 变换，因为它们可以或不依赖于 shuffle。 |
| :---: | :--- |


具有相同键的所有元组必须在同一个分区中结束，由同一个任务处理。为了满足这些操作，Spark 必须执行 RDD shuffle，它通过集群传输数据，并产生一个具有一组新分区的新阶段。

### mapPartitions {#__a_id_mappartitions_a_mappartitions}

使用外部键值存储（如 HBase，Redis，Cassandra）并在映射器内部执行查找/更新（在 mapPartitions 代码块中创建连接，以避免连接建立/拆卸开销）可能是一个更好的解决方案。

如果 hbase 用作外部键值存储，则原子性得到保证

### zipWithIndex {#__a_id_zipwithindex_a_zipwithindex}

```
zipWithIndex(): RDD[(T, Long)]
```

zipWithIndex 将此 RDD \[T\] 与其元素索引绑定。

如果源 RDD 的分区数大于1，它将提交一个额外的作业来计算开始索引。

```
val onePartition = sc.parallelize(0 to 9, 1)

scala> onePartition.partitions.length
res0: Int = 1

// no job submitted
onePartition.zipWithIndex

val eightPartitions = sc.parallelize(0 to 9, 8)

scala> eightPartitions.partitions.length
res1: Int = 8

// submits a job
eightPartitions.zipWithIndex
```

![](/img/mastering-apache-spark/spark core-rdd/figure16.png)







