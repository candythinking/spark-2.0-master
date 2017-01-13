## RDD — Resilient Distributed Dataset {#__a_id_rdd_a_rdd_resilient_distributed_dataset}

### Introduction {#__a_id_introduction_a_introduction}

RDD 的起源

生成 RDD 概念的原始文章是弹性分布式数据集：Matei Zaharia 等人的内存中集群计算的容错抽象。

弹性分布式数据集（RDD）是 Apache Spark 和 Spark 的核心（通常称为 Spark Core）的主要数据抽象。

RDD 是记录的弹性和分布式集合。可以将 RDD 与位于单个 JVM 上的 Scala 集合（其位于许多 JVM 上，可能位于集群中的单独节点上）进行比较。

| Tip | RDD 是 Spark 的面包和黄油，掌握这个概念是成为一个 Spark pro 的最重要的。你想成为一个 Spark 亲，你不是吗？ |
| :---: | :--- |


使用 RDD，Spark 的创建者设法隐藏数据分区和分布，从而允许他们使用更高级的编程接口（API）为四种主流编程语言设计并行计算框架。

Learning about RDD by its name:

* **Resilient**, i.e. fault-tolerant with the help of RDD lineage graph and so able to recompute missing or damaged partitions due to node failures.

* **Distributed **with data residing on multiple nodes in a cluster.

* **Dataset **is a collection of partitioned data with primitive values or values of values, e.g. tuples or other objects \(that represent records of the data you work with\).

![](/img/mastering-apache-spark/spark core-rdd/figure9.png)

From the scaladoc of org.apache.spark.rdd.RDD:

一个弹性分布式数据集（RDD），Spark 中的基本抽象。表示可以并行操作的元素的不可变的，分区的集合。

From the original paper about RDD -

[Resilient Distributed Datasets: A Fault-Tolerant Abstraction for In-Memory Cluster Computing](https://www.cs.berkeley.edu/~matei/papers/2012/nsdi_spark.pdf):

弹性分布式数据集（RDD）是一种分布式存储器抽象，它允许程序员以容错方式在大型集群上执行内存中计算。

除了上述特性（直接嵌入在数据抽象的名称中 - RDD），它还有以下附加特性：

* **In-Memory**, i.e. data inside RDD is stored in memory as much \(size\) and long \(time\) as possible.

* **Immutable **or **Read-Only**, i.e. it does not change once created and can only be transformed using transformations to new RDDs.

* **Lazy evaluated**, i.e. the data inside RDD is not available or transformed until an action is executed that triggers the execution.

* **Cacheable**, i.e. you can hold all the data in a persistent "storage" like memory \(default and the most preferred\) or disk \(the least preferred due to access speed\).+

* **Parallel**, i.e. process data in parallel.

* **Typed**, i.e. values in a RDD have types, e.g.`RDD[Long]`or`RDD[(Int, String)]`.

* **Partitioned**, i.e. the data inside a RDD is partitioned \(split into partitions\) and then distributed across nodes in a cluster \(one partition per JVM that may or may not correspond to a single node\).

RDD 是通过设计分布的，并且实现均匀的数据分布以及利用数据局部性（在分布式系统中，如 HDFS 或 Cassandra，其中数据被默认地分割），它们被分割成固定数目的分区 - 逻辑块（部分）数据。逻辑划分仅用于处理，而在内部不被划分。每个分区包括记录。

![](/img/mastering-apache-spark/spark core-rdd/figure10.png)

分区是并行的单位。您可以使用重新分区或合并转换来控制 RDD 的分区数。 Spark 尝试尽可能接近数据，而不浪费时间通过 RDD shuffling 在网络上发送数据，并创建所需的多个分区以跟随存储布局，从而优化数据访问。它导致在分布式数据存储器中的（物理）数据之间的一对一映射，例如， HDFS 或 Cassandra 和 partitions。

RDDs support two kinds of operations:

* transformations- lazy operations that return another RDD.

* actions- operations that trigger\(触发\) computation and return values.

创建 RDD 的动机是（作者后）当前计算框架低效处理的两种类型的应用程序：

* **iterative algorithms **in machine learning and graph computations.

* **interactive data mining tools **as ad-hoc queries on the same dataset.

目标是跨多个数据密集型工作负载重用中间内存结果，而无需通过网络复制大量数据。

RDD 由五个主要固有属性定义：

* List of parent RDDs that is the list of the dependencies an RDD depends on for records.（父 RDD 的列表，RDD 依赖于记录的依赖列表。）

* An array of partitions that a dataset is divided to.

* A compute function to do a computation on partitions.（用于对分区执行计算的计算函数。）

* An optional Partitioner that defines how keys are hashed, and the pairs partitioned \(for key-value RDDs\)（一个可选的分区器，用于定义如何对键进行哈希，以及对键值对分区（对于键值 RDD））

* Optional preferred locations\(aka **locality info**\), i.e. hosts for a partition where the data will have been loaded.（可选的优选位置（也称为位置信息），即其中数据将被加载的分区的主机。）

此 RDD 抽象支持一组表达式操作，而不必为每个操作都修改调度程序。

RDD 是 SparkContext（可用作上下文属性）中的一个命名的（按名称）和唯一标识的（按 id）实体。

RDD 存在于一个且仅一个 SparkContext 中，其创建逻辑边界。

| Note | RDD 不能在 SparkContexts 之间共享（参见 SparkContext 和 RDDs）。 |
| :---: | :--- |


RDD 可以可选地具有可使用名称可访问的友好名称，可以使用可以改变的名称：=：

```
scala> val ns = sc.parallelize(0 to 10)
ns: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[2] at parallelize at <console>:24

scala> ns.id
res0: Int = 2

scala> ns.name
res1: String = null

scala> ns.name = "Friendly name"
ns.name: String = Friendly name

scala> ns.name
res2: String = Friendly name

scala> ns.toDebugString
res3: String = (8) Friendly name ParallelCollectionRDD[2] at parallelize at <console>:24 []
```

RDD 是一个关于如何实现大型（数组）分布式数据的指令容器，以及如何将它分成多个分区，这样 Spark（使用执行器）可以容纳其中的一些。

一般来说，数据分布可以帮助并行执行处理，以便任务处理最终可以在内存中保存的一块数据。

Spark  并行执行作业，并且 RDD 被分割为要被并行处理和写入的分区。在分区内，数据被顺序处理。

保存分区导致部分文件，而不是一个单个文件（除非有单个分区）。

### `partitions`Method {#__a_id_partitions_a_code_partitions_code_method}

| Caution | FIXME |
| :--- | :--- |


### RDD Contract {#__a_id_contract_a_rdd_contract}

| Caution | FIXME |
| :--- | :--- |


### Types of RDDs {#__a_id_rdd_types_a_types_of_rdds}

有一些最有趣的 RDD 类型：

* ParallelCollectionRDD

* CoGroupedRDD

* HadoopRDD 是一个RDD，它提供了使用较旧的 MapReduce API 读取存储在 HDFS 中的数据的核心功能。最值得注意的用例是 SparkContext.textFile 的返回 RDD。

* **MapPartitionsRDD **- 调用操作（如 map，flatMap，filter，mapPartitions 等）的结果。

* **CoalescedRDD **- a result of repartition or coalesce transformations.（重分区或聚结转换的结果。）

* ShuffledRDD - a result of shuffling, e.g. after repartition or coalesce transformations.

* **PipedRDD **- an RDD created by piping elements to a forked external process.（由管道元件创建到一个分叉的外部过程的RDD。）

* **PairRDD**\(implicit conversion by PairRDDFunctions\) that is an RDD of key-value pairs that is a result of`groupByKey`and`join`operations.（（PairRDDFunctions 的隐式转换），这是一个键值对的 RDD，它是 groupByKey 和连接操作的结果。）

* **DoubleRDD**\(implicit conversion as`org.apache.spark.rdd.DoubleRDDFunctions`\) that is an RDD of`Double`type.（隐式转换为 org.apache.spark.rdd.DoubleRDDFunctions），它是 Double 类型的 RDD。）

* **SequenceFileRDD**\(implicit conversion as`org.apache.spark.rdd.SequenceFileRDDFunctions`\) that is an RDD that can be saved as a`SequenceFile`.（隐式转换为 org.apache.spark.rdd.SequenceFileRDDFunctions），它是一个可以保存为 SequenceFile 的 RDD。）

给定 RDD 类型的适当操作在正确类型的 RDD 上自动可用，例如， RDD \[（Int，Int）\]，通过 Scala 中的隐式转换。

### Transformations {#__a_id_transformations_a_transformations}

**transformation **是一个对 RDD 的延迟操作，返回另一个 RDD，如 map，flatMap，filter，reduceByKey，join，cogroup 等。

### Actions {#__a_id_actions_a_actions}

**action **是触发 RDD 转换的执行并返回值（到 Spark 驱动程序 - 用户程序）的操作。

### Creating RDDs {#__a_id_creating_rdds_a_creating_rdds}

#### SparkContext.parallelize {#_sparkcontext_parallelize}

创建 RDD 的一种方法是使用 SparkContext.parallelize 方法。它接受一个元素的集合，如下所示（sc 是一个 SparkContext 实例）：

```
scala> val rdd = sc.parallelize(1 to 1000)
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[0] at parallelize at <console>:25
```

您还可能想要随机抽样数据：

```
scala> val data = Seq.fill(10)(util.Random.nextInt)
data: Seq[Int] = List(-964985204, 1662791, -1820544313, -383666422, -111039198, 310967683, 1114081267, 1244509086, 1797452433, 124035586)

scala> val rdd = sc.parallelize(data)
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[0] at parallelize at <console>:29
```

由于使用 Spark 来处理比你自己的笔记本电脑能处理的更多的数据的原因，SparkContext.parallelize 主要用来在 Spark shell 中学习 Spark。 SparkContext.parallelize 需要所有的数据在单个机器上可用 - Spark driver 驱动程序 - 最终达到你的笔记本电脑的极限。

#### SparkContext.makeRDD {#_sparkcontext_makerdd}

| Caution | makeRDD 的用例是什么？ |
| :---: | :--- |


```
scala> sc.makeRDD(0 to 1000)
res0: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[1] at makeRDD at <console>:25
```

#### SparkContext.textFile {#_sparkcontext_textfile}

创建 RDD 的最简单的方法之一是使用 SparkContext.textFile 读取文件。

你可以使用本地 README.md 文件（然后在 flatMap 的内部有一个 RDD 的单词）：

```
scala> val words = sc.textFile("README.md").flatMap(_.split("\\W+")).cache
words: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[27] at flatMap at <console>:24
```

| Note | 你缓存它，所以每次你处理单词时不执行计算。 |
| :---: | :--- |


#### Creating RDDs from Input {#__a_id_creating_rdds_from_input_a_creating_rdds_from_input}

请参阅使用输入和输出（I / O）了解用于创建 RDD 的 IO API。

#### Transformations {#_transformations}

RDD 转换定义将 RDD 转换为另一个 RDD，因此是创建新 RDD 的方法。

有关详细信息，请参阅 Transformations 部分。

### RDDs in Web UI {#_rdds_in_web_ui}

在 Web UI 中查看位于 http：// localhost：4040 的 Spark shell 的 RDD 是相当丰富的。

执行以下 Spark 应用程序（键入 spark-shell 中的所有行）：

```
val ints = sc.parallelize(1 to 100) (1)
ints.setName("Hundred ints")        (2)
ints.cache                          (3)
ints.count                          (4)
```

1. Creates an RDD with hundred of numbers \(with as many partitions as possible\)

2. Sets the name of the RDD

3. Caches the RDD for performance reasons that also makes it visible in Storage tab in the web UI

4. Executes action \(and materializes the RDD\)

![](/img/mastering-apache-spark/spark core-rdd/figure11.png)

单击 RDD 的名称（在 RDD 名称下），您将获得如何缓存 RDD 的详细信息。

![](/img/mastering-apache-spark/spark core-rdd/figure12.png)

执行以下 Spark 作业，您将看到分区数量如何减少。

```
ints.repartition(2).count
```

![](/img/mastering-apache-spark/spark core-rdd/figure13.png)

### Computing Partition \(in TaskContext\) — `compute`Method {#__a_id_compute_a_computing_partition_in_taskcontext_code_compute_code_method}

```
compute(split: Partition, context: TaskContext): Iterator[T]
```

抽象计算方法计算 TaskContext 中的输入拆分分区，以生成值（类型 T）的集合。

`compute`由 Spark 中的任何类型的 RDD 实现，并在每次请求记录时调用，除非 RDD 被缓存或检查点（并且记录可以从外部存储读取，但这次更靠近计算节点）。

当缓存 RDD 时，对于指定的存储级别（即除 NONE 之外的所有级别），请求 CacheManager 获取或计算分区。

| Note | 计算方法在 driver 上运行。 |
| :---: | :--- |


### Preferred Locations \(aka Locality Info\) — `getPreferredLocations`Method {#__a_id_preferred_locations_a_a_id_getpreferredlocations_a_preferred_locations_aka_locality_info_code_getpreferredlocations_code_method}

```
getPreferredLocations(split: Partition): Seq[String]
```

**preferred location **优选位置（也称为位置偏好或位置偏好或位置信息）是关于用于 HDFS 文件的分割块的位置的信息（放置计算分区）。

getPreferredLocations 返回（RDD 的）输入拆分分区的首选位置。

### Getting Partition Count — `getNumPartitions`Method {#__a_id_getnumpartitions_a_getting_partition_count_code_getnumpartitions_code_method}

```
getNumPartitions: Int
```

getNumPartitions 计算 RDD 的分区数。

```
scala> sc.textFile("README.md").getNumPartitions
res0: Int = 2

scala> sc.textFile("README.md", 5).getNumPartitions
res1: Int = 5
```

### Computing Partition \(Possibly by Reading From Checkpoint\) — `computeOrReadCheckpoint`Method {#__a_id_computeorreadcheckpoint_a_computing_partition_possibly_by_reading_from_checkpoint_code_computeorreadcheckpoint_code_method}

```
computeOrReadCheckpoint(split: Partition, context: TaskContext): Iterator[T]
```

computeOrReadCheckpoint 在启用时从检查点读取分割分区，或自行计算。

| Note | `computeOrReadCheckpoint`is a`private[spark]`method. |
| :--- | :--- |


| Note | `computeOrReadCheckpoint`is used to iterate over partition or when getOrCompute. |
| :--- | :--- |


### Computing Records For Partition — `iterator`Method {#__a_id_iterator_a_computing_records_for_partition_code_iterator_code_method}

```
iterator(split: Partition, context: TaskContext): Iterator[T]
```

迭代器在缓存或计算时（可能通过从检查点读取）获取（或计算）分割分区。

| Note | 迭代器是一个最终的方法，尽管是公共的，被认为是私有的，只能用于实现自定义 RDD。 |
| :---: | :--- |


### `getOrCompute`Method {#__a_id_getorcompute_a_code_getorcompute_code_method}

```
getOrCompute(partition: Partition, context: TaskContext): Iterator[T]
```

getOrCompute 请求块的 BlockManager 并返回一个 InterruptibleIterator。

| Note | getOrCompute 是私有的 \[spark\] 方法，当 RDD 被缓存时，在迭代分区时专门使用它。 |
| :---: | :--- |


### RDD Dependencies — `dependencies`Method {#__a_id_dependencies_a_rdd_dependencies_code_dependencies_code_method}

```
dependencies: Seq[Dependency[_]]
```

dependencies returns the dependencies of a RDD.

| Note | 依赖是一个 final 方法，Spark 中的任何类都不能覆盖。 |
| :---: | :--- |


在内部，依赖关系检查 RDD 是否设置检查点，并相应地执行。

对于被检查点的 RDD，依赖关系返回一个具有 OneToOneDependency 的单元素集合。

对于非检查点的 RDD，依赖项集合使用 getDependencies 方法计算。

| Note | getDependencies 方法是一种抽象方法，需要自定义 RDD 来提供。 |
| :---: | :--- |
















