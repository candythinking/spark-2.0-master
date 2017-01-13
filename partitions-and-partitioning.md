## Partitions and Partitioning {#_partitions_and_partitioning}

### Introduction {#_introduction}

根据你对 Spark（程序员，devop，管理员）的看法，RDD 是关于内容（开发人员和数据科学家的观点）或如何在一个集群（性能）上扩展的，即 RDD 表示多少个分区。

分区（也称为分割）是大分布式数据集的逻辑块。

| Caution | 1.如何将分区数映射到任务数量？如何验证？                                 2.分区和任务之间的映射如何对应于数据位置（如果有）？ |
| :---: | :--- |


Spark 使用分区管理数据，这有助于以最小的网络流量并行执行分布式数据处理，从而在执行者之间发送数据。

默认情况下，Spark 尝试从靠近 RDD 的节点读取数据到 RDD 中。由于 Spark 通常访问分布式分区数据，为了优化 transformation 操作，它创建分区以保存数据块。

在如何将数据布置在数据存储器（如 HDFS 或 Cassandra）（它由于相同的原因而被分割）之间存在一对一的对应关系。

Features\(特征\):

* size

* number

* partitioning scheme

* node distribution

* repartitioning

Read the following documentations to learn what experts say on the topic:

* [How Many Partitions Does An RDD Have?](https://databricks.gitbooks.io/databricks-spark-knowledge-base/content/performance_optimization/how_many_partitions_does_an_rdd_have.html)

* [Tuning Spark](https://spark.apache.org/docs/latest/tuning.html)\(the official documentation of Spark\)

默认情况下，为每个 HDFS 分区创建一个分区，默认情况下是 128MB（来自 [Spark’s Programming Guide](http://spark.apache.org/docs/latest/programming-guide.html#external-datasets)）。

RDDs 自动分区，而无需程序员干预。但是，有时候您想根据应用程序的需要调整分区的大小和数量或分区方案。

您可以在 RDD 上使用 def getPartitions：Array \[Partition\] 方法来了解此 RDD 中的分区集。

当阶段执行时，您可以在 Spark UI 中查看给定阶段的分区数。

![](/img/mastering-apache-spark/spark core-rdd/figure17.png)

Start`spark-shell`and see it yourself!

```
scala> sc.parallelize(1 to 100).count
res0: Long = 100
```

当您执行 Spark 作业，即  sc.parallelize（1 to 100）.count 时，您应该在 Spark shell 应用程序 UI 中看到以下内容。

![](/img/mastering-apache-spark/spark core-rdd/figure18.png)

总共8个任务的原因是我在一个8核笔记本电脑上，默认情况下，分区数是所有可用内核的数量。

```
$ sysctl -n hw.ncpu
8
```

您可以请求最小分区数，使用第二个输入参数进行许多 transformations。

```
scala> sc.parallelize(1 to 100, 2).count
res1: Long = 100
```

![](/img/mastering-apache-spark/spark core-rdd/figure19.png)

您可以随时使用 RDD 的分区方法请求分区数：

```
scala> val ints = sc.parallelize(1 to 100, 4)
ints: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[1] at parallelize at <console>:24

scala> ints.partitions.size
res2: Int = 4
```

通常，更小/更多的分区允许工作分布在更多的 worker 中，但是更大/更少的分区允许以更大的块来完成工作，这可以导致工作更快完成，只要所有 worker 都忙，由于减少开销。

增加分区计数将使每个分区有更少的数据（或根本不是！）

Spark 只能对 RDD 的每个分区运行1个并发任务，直到集群中的核心数。所以如果你有一个拥有50个核心的集群，你希望 RDD 至少有50个分区（可能是2-3倍）。

至于选择“好的”分区数，通常至少需要与用于并行性的执行器数量一样多。您可以通过调用 sc.defaultParallelism 获取此计算值。

此外，分区数决定了将 RDD 保存到文件的操作生成多少文件。

分区的最大大小最终受到执行程序的可用内存的限制。

在第一 RDD transformation 中，使用 sc.textFile（path，partition）从文件读取，分区参数将应用于此 RDD 上的所有进一步的 transformations 和 actions。

每当 shuffle 发生时，分区在节点之间重新分配。重新分区可能会在某些情况下导致 shuffle 发生，但不能保证在所有情况下都会发生。它通常发生在行动阶段。

当通过使用 rdd = SparkContext（）.textFile（“hdfs：// ... / file.txt”）读取文件来创建 RDD 时，分区数可能会更小。理想情况下，您将获得与您在 HDFS 中看到的相同数量的块，但是如果文件中的行太长（长于块大小），则会有较少的分区。

设置 RDD 分区数的首选方法是直接将其作为调用中的第二个输入参数传递，如 rdd = sc.textFile（“hdfs：// ... /file.txt”，400），其中400是分区的数量。在这种情况下，分区使得400分裂将由 Hadoop 的 TextInputFormat 完成，而不是 Spark，它会工作得更快。它也是代码产生400个并发任务，尝试加载 file.txt 直接到400分区。

它将只对未压缩文件工作。

当使用带有压缩文件的文本文件（file.txt.gz 而不是 file.txt 或类似文件）时，Spark 会禁用对只有一个分区的 RDD 进行分割（因为对 gzip 压缩文件的读取不能并行化）。在这种情况下，要更改分区的数量，应该重新分区。

一些操作，例如。 map，flatMap，filter，不保留分区。

map，flatMap，filter 操作对每个分区应用一个函数。

### Repartitioning RDD — `repartition`Transformation {#__a_id_repartitioning_a_a_id_repartition_a_repartitioning_rdd_code_repartition_code_transformation}

```
repartition(numPartitions: Int)(implicit ord: Ordering[T] = null): RDD[T]
```

重新分区与启用 numPartition 和 shuffle 的 coalesce\(合并\)。

通过以下计算，您可以看到 repartition\(5\) 导致使用 NODE\_LOCAL 数据 locality 启动5个任务。

```
scala> lines.repartition(5).count
...
15/10/07 08:10:00 INFO DAGScheduler: Submitting 5 missing tasks from ResultStage 7 (MapPartitionsRDD[19] at repartition at <console>:27)
15/10/07 08:10:00 INFO TaskSchedulerImpl: Adding task set 7.0 with 5 tasks
15/10/07 08:10:00 INFO TaskSetManager: Starting task 0.0 in stage 7.0 (TID 17, localhost, partition 0,NODE_LOCAL, 2089 bytes)
15/10/07 08:10:00 INFO TaskSetManager: Starting task 1.0 in stage 7.0 (TID 18, localhost, partition 1,NODE_LOCAL, 2089 bytes)
15/10/07 08:10:00 INFO TaskSetManager: Starting task 2.0 in stage 7.0 (TID 19, localhost, partition 2,NODE_LOCAL, 2089 bytes)
15/10/07 08:10:00 INFO TaskSetManager: Starting task 3.0 in stage 7.0 (TID 20, localhost, partition 3,NODE_LOCAL, 2089 bytes)
15/10/07 08:10:00 INFO TaskSetManager: Starting task 4.0 in stage 7.0 (TID 21, localhost, partition 4,NODE_LOCAL, 2089 bytes)
...
```

执行 repartition\(1\) 后可以看到更改，导致使用 PROCESS\_LOCAL 数据位置启动2个任务。

```
scala> lines.repartition(1).count
...
15/10/07 08:14:09 INFO DAGScheduler: Submitting 2 missing tasks from ShuffleMapStage 8 (MapPartitionsRDD[20] at repartition at <console>:27)
15/10/07 08:14:09 INFO TaskSchedulerImpl: Adding task set 8.0 with 2 tasks
15/10/07 08:14:09 INFO TaskSetManager: Starting task 0.0 in stage 8.0 (TID 22, localhost, partition 0,PROCESS_LOCAL, 2058 bytes)
15/10/07 08:14:09 INFO TaskSetManager: Starting task 1.0 in stage 8.0 (TID 23, localhost, partition 1,PROCESS_LOCAL, 2058 bytes)
...
```

请注意，Spark 禁用拆分压缩文件，并创建只有1个分区的 RDD。在这种情况下，使用 sc.textFile（'demo.gz'）和使用 rdd.repartition（100）重新分区如下：

```
rdd = sc.textFile('demo.gz')
rdd = rdd.repartition(100)
```

使用 Line，你最终得到 rdd 正好100大小大致相等的分区。

rdd.repartition（N）执行 shuffle 以拆分数据以匹配 N

分区是在循环的基础上完成的

| Tip | 如果分区方案不适用于您，您可以编写自己的自定义分区程序。 |
| :---: | :--- |


| Tip | 熟悉 Hadoop 的 TextInputFormat 非常有用。 |
| :---: | :--- |


### `coalesce`transformation {#__a_id_coalesce_a_code_coalesce_code_transformation}

```
coalesce(numPartitions: Int, shuffle: Boolean = false)(implicit ord: Ordering[T] = null): RDD[T]
```

`coalesce`transformation 用于更改分区数。它可以根据 shuffle 标志（默认禁用，即 false）触发 RDD shuffling。

在下面的示例中，将并行化 local 10-number sequence 并首先合并\(coalesce\)它，然后使用 shuffling（注意 shuffle 参数分别为 false 和 true）。

| Tip | 使用 toDebugString 检出 RDD 谱系血统图。 |
| :---: | :--- |


```
scala> val rdd = sc.parallelize(0 to 10, 8)
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[0] at parallelize at <console>:24

scala> rdd.partitions.size
res0: Int = 8

scala> rdd.coalesce(numPartitions=8, shuffle=false)   (1)
res1: org.apache.spark.rdd.RDD[Int] = CoalescedRDD[1] at coalesce at <console>:27

scala> res1.toDebugString
res2: String =
(8) CoalescedRDD[1] at coalesce at <console>:27 []
 |  ParallelCollectionRDD[0] at parallelize at <console>:24 []

scala> rdd.coalesce(numPartitions=8, shuffle=true)
res3: org.apache.spark.rdd.RDD[Int] = MapPartitionsRDD[5] at coalesce at <console>:27

scala> res3.toDebugString
res4: String =
(8) MapPartitionsRDD[5] at coalesce at <console>:27 []
 |  CoalescedRDD[4] at coalesce at <console>:27 []
 |  ShuffledRDD[3] at coalesce at <console>:27 []
 +-(8) MapPartitionsRDD[2] at coalesce at <console>:27 []
    |  ParallelCollectionRDD[0] at parallelize at <console>:24 []
```

默认情况下 shuffle 是 false，它在这里显式地用于演示目的。请注意，与源 RDD rdd 中的分区数保持相同的分区数。

### Settings {#__a_id_settings_a_settings}

Table 1. Spark Properties

| Spark Property | Default Value | Description |
| :--- | :--- | :--- |
| `spark.default.parallelism` | 根据部署环境而变化\(varies per deployment environment\) | 设置要用于 HashPartitioner 的分区数。它对应于调度程序后端的默认并行性。 |

更具体地，spark.default.parallelism 对应于：

1. LocalBackend 的线程数。 
2. Mesos 上的 Spark 中的 CPU 核心数，默认为8. 
3. CoarseGrainedSchedulerBackend 中的 totalCoreCount 和2的最大值。



