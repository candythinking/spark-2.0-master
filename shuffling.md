## RDD shuffling {#_rdd_shuffling}

| Tip | Read the official documentation about the topic [Shuffle operations](http://people.apache.org/~pwendell/spark-nightly/spark-master-docs/latest/programming-guide.html#shuffle-operations). It is _still _better than this page. |
| :--- | :--- |


**Shuffling **是跨分区重新分配数据的过程（也称为重新分区），可能或可能不会导致跨 JVM 进程或甚至通过有线（在单独的机器上的执行器之间）移动数据。

shuffling 是 stages 之间的数据传输过程。

| Tip | 避免随意洗牌。想想如何利用现有的分区。利用部分聚合以减少数据传输。 |
| :---: | :--- |


By default, shuffling doesn’t change the number of partitions, but their content.

* 避免 groupByKey 并使用 reduceByKey 或 combineByKey。

* groupByKey 洗牌所有的数据，这是慢的。

* reduceByKey 只洗牌数据的每个分区中的子聚合的结果

### Example - join {#_example_join}

PairRDD 提供连接转换（引用官方文档）：

当调用类型（K，V）和（K，W）的数据集时，返回每个键的所有元素对的（K，（V，W））对的数据集。

让我们看看一个例子，看看它的工作原理：

```
scala> val kv = (0 to 5) zip Stream.continually(5)
kv: scala.collection.immutable.IndexedSeq[(Int, Int)] = Vector((0,5), (1,5), (2,5), (3,5), (4,5), (5,5))

scala> val kw  = (0 to 5) zip Stream.continually(10)
kw: scala.collection.immutable.IndexedSeq[(Int, Int)] = Vector((0,10), (1,10), (2,10), (3,10), (4,10), (5,10))

scala> val kvR = sc.parallelize(kv)
kvR: org.apache.spark.rdd.RDD[(Int, Int)] = ParallelCollectionRDD[3] at parallelize at <console>:26

scala> val kwR = sc.parallelize(kw)
kwR: org.apache.spark.rdd.RDD[(Int, Int)] = ParallelCollectionRDD[4] at parallelize at <console>:26

scala> val joined = kvR join kwR
joined: org.apache.spark.rdd.RDD[(Int, (Int, Int))] = MapPartitionsRDD[10] at join at <console>:32

scala> joined.toDebugString
res7: String =
(8) MapPartitionsRDD[10] at join at <console>:32 []
 |  MapPartitionsRDD[9] at join at <console>:32 []
 |  CoGroupedRDD[8] at join at <console>:32 []
 +-(8) ParallelCollectionRDD[3] at parallelize at <console>:26 []
 +-(8) ParallelCollectionRDD[4] at parallelize at <console>:26 []
```

当在操作图中的“节点”之间存在“角度”时，看起来不好。它出现在连接操作之前，因此需要随机播放。

下面是在 Web UI 中执行 joined.count 的工作。

![](/img/mastering-apache-spark/spark core-rdd/figure20.png)

Web UI 的屏幕截图显示3个阶段，两个并行化为随机写入和计数为随机读取。这意味着洗牌确实发生了。

| Note | 刚刚学习了 sc.range（0，5）作为 sc.parallelize \(0 to 5\)的缩写版本 |
| :---: | :--- |


`join`操作是使用 defaultPartitioner 的 cogroup 操作之一，即遍历 RDD 沿袭血统图（按分区数减少排序），并选择具有正数输出分区的分区器。否则，它检查 spark.default.parallelism 属性，如果定义选择 HashPartitioner 与 SchedulerBackend 的默认并行性。

`join`is almost`CoGroupedRDD.mapValues`

| Caution | 调度程序后端的默认并行性 |
| :---: | :--- |




