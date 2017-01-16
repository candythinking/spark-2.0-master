## Broadcast Variables {#_broadcast_variables}

From [the official documentation about Broadcast Variables](http://spark.apache.org/docs/latest/programming-guide.html#broadcast-variables):

广播变量允许程序员保存每个机器上缓存的只读变量，而不是发送带有任务的副本。

And later in the document:

显式创建广播变量仅在跨多个阶段的任务需要相同的数据或以反序列化形式缓存数据很重要时才有用。

![](/img/mastering-apache-spark/spark core-optimizations/figure1.png)

要在 Spark 转换中使用广播值，您必须先使用 SparkContext.broadcast 创建它，然后使用 value 方法访问共享值。在“入门示例”部分中了解。

Spark 中的 Broadcast 功能使用 SparkContext 创建广播值，使用 BroadcastManager 和 ContextCleaner 来管理它们的生命周期。

![](/img/mastering-apache-spark/spark core-optimizations/figure2.png)

| Tip | Spark 开发人员不仅可以使用广播变量进行有效的数据分发，而且 Spark 本身也经常使用它们。一个非常显着的用例是当 Spark 将任务分配给执行程序以执行它们。这改变了我对 Spark 中广播变量的作用的观点。 |
| :---: | :--- |


### `Broadcast`Spark Developer-Facing Contract {#__a_id_developer_contract_a_code_broadcast_code_spark_developer_facing_contract}

面向开发人员的广播合同允许 Spark 开发人员在其应用程序中使用它。

Table 1. Broadcast API

| Method Name | Description |
| :--- | :--- |
| `toString` | The string representation\(表示\) |
| `id` | The unique identifier |
| value | The value |
| unpersist | 异步删除执行器上的此广播的缓存副本。 |
| destroy | 销毁与此广播变量相关的所有数据和元数据。 |

### Lifecycle of Broadcast Variable {#__a_id_lifecycle_a_lifecycle_of_broadcast_variable}

您可以使用 SparkContext.broadcast 方法创建类型 T 的广播变量。

| Tip | 为 org.apache.spark.storage.BlockManager 记录器启用 DEBUG 日志记录级别以调试广播方法。                                                 阅读 BlockManager 以了解如何启用日志记录级别。 |
| :---: | :--- |


启用 DEBUG 日志记录级别后，您应该在日志中看到以下消息：

```
DEBUG BlockManager: Put block broadcast_0 locally took  430 ms
DEBUG BlockManager: Putting block broadcast_0 without replication took  431 ms
DEBUG BlockManager: Told master about block broadcast_0_piece0
DEBUG BlockManager: Put block broadcast_0_piece0 locally took  4 ms
DEBUG BlockManager: Putting block broadcast_0_piece0 without replication took  4 ms
```

创建广播变量的实例后，可以使用 value 方法引用值。

```
scala> b.value
res0: Int = 1
```

| Note | value method 是访问广播变量的值的唯一方法。 |
| :---: | :--- |


启用 DEBUG 日志记录级别后，您应该在日志中看到以下消息：

```
DEBUG BlockManager: Getting local block broadcast_0
DEBUG BlockManager: Level for block broadcast_0 is StorageLevel(disk, memory, deserialized, 1 replicas)
```

当你完成一个广播变量，你应该销毁它释放内存

```
scala> b.destroy
```

启用 DEBUG 日志记录级别后，您应该在日志中看到以下消息：

```
DEBUG BlockManager: Removing broadcast 0
DEBUG BlockManager: Removing block broadcast_0_piece0
DEBUG BlockManager: Told master about block broadcast_0_piece0
DEBUG BlockManager: Removing block broadcast_0
```

在销毁广播变量之前，您可能想要 unpersist 它。

```
scala> b.unpersist
```

### Getting the Value of Broadcast Variable — `value`Method {#__a_id_value_a_getting_the_value_of_broadcast_variable_code_value_code_method}

```
value: T
```

value 返回广播变量的值。您只能访问该值，直到它被销毁，然后您将在日志中看到以下 SparkException 异常：

```
org.apache.spark.SparkException: Attempted to use Broadcast(0) after it was destroyed (destroy at <console>:27)
  at org.apache.spark.broadcast.Broadcast.assertValid(Broadcast.scala:144)
  at org.apache.spark.broadcast.Broadcast.value(Broadcast.scala:69)
  ... 48 elided
```

在内部，value 确保广播变量有效，即 destroy 未被调用，如果是，则调用抽象 getValue 方法。

| Note | getValue 是抽象的，广播变量实现应该提供一个具体的行为。 参考 TorrentBroadcast。 |
| :---: | :--- |


### Unpersisting Broadcast Variable — `unpersist`Methods {#__a_id_unpersist_a_unpersisting_broadcast_variable_code_unpersist_code_methods}

```
unpersist(): Unit
unpersist(blocking: Boolean): Unit
```

### Destroying Broadcast Variable — `destroy`Method {#__a_id_destroy_a_destroying_broadcast_variable_code_destroy_code_method}

```
destroy(): Unit
```

destroy 删除广播变量。

| Note | 一旦广播变量被销毁，它不能再次使用。 |
| :---: | :--- |


如果您尝试多次销毁广播变量，则会在日志中看到以下 SparkException 异常：

```
scala> b.destroy
org.apache.spark.SparkException: Attempted to use Broadcast(0) after it was destroyed (destroy at <console>:27)
  at org.apache.spark.broadcast.Broadcast.assertValid(Broadcast.scala:144)
  at org.apache.spark.broadcast.Broadcast.destroy(Broadcast.scala:107)
  at org.apache.spark.broadcast.Broadcast.destroy(Broadcast.scala:98)
  ... 48 elided
```

在内部，destroy 执行内部销毁（启用阻塞）。

### Removing Persisted Data of Broadcast Variable — `destroy`Internal Method {#__a_id_destroy_internal_a_removing_persisted_data_of_broadcast_variable_code_destroy_code_internal_method}

```
destroy(blocking: Boolean): Unit
```

destroy 销毁广播变量的所有数据和元数据。

| Note | `destroy`is a`private[spark]`method. |
| :--- | :--- |


在内部，destroy 将一个广播变量标记为已销毁，即内部 \_isValid 标志被禁用。

您应该在日志中看到以下 INFO 消息：

```
INFO TorrentBroadcast: Destroying Broadcast([id]) (from [destroySite])
```

最后，doDestroy 方法被执行（广播实现应该提供）。

| Note | doDestroy 是广播实现的广播合同的一部分，所以他们可以提供自己的自定义行为。 |
| :---: | :--- |


### Introductory Example {#__a_id_introductory_example_a_introductory_example}

你将使用静态映射的有趣的项目与他们的网站，即 Map \[String，String\] 任务，即闭包（匿名函数）在转换中使用。

```
scala> val pws = Map("Apache Spark" -> "http://spark.apache.org/", "Scala" -> "http://www.scala-lang.org/")
pws: scala.collection.immutable.Map[String,String] = Map(Apache Spark -> http://spark.apache.org/, Scala -> http://www.scala-lang.org/)

scala> val websites = sc.parallelize(Seq("Apache Spark", "Scala")).map(pws).collect
...
websites: Array[String] = Array(http://spark.apache.org/, http://www.scala-lang.org/)
```

It works, but is very ineffective as the`pws`map is sent over the wire to executors while it could have been there already. If there were more tasks that need the`pws`map, you could improve their performance by minimizing the number of bytes that are going to be sent over the network for task execution.

输入广播变量。

```
val pwsB = sc.broadcast(pws)
val websites = sc.parallelize(Seq("Apache Spark", "Scala")).map(pwsB.value).collect
// websites: Array[String] = Array(http://spark.apache.org/, http://www.scala-lang.org/)
```

语义上，两个计算 - 有和没有广播值 - 是完全相同的，但是当有更多的执行器被执行许多任务使用 pws map 时，基于广播的一个赢得性能。

### Introduction {#__a_id_introduction_a_introduction}

广播是 Spark 的一部分，负责在群集中的节点之间广播信息。

您使用广播变量来实现 map-side join，即 a join using a`map`。为此，查找表使用广播分布在集群中的节点上，然后在 Map 中查找（以隐式方式执行 join）。

当您广播一个值时，它只被复制到执行器一次（否则为任务复制多次）。这意味着广播可以帮助更快地获得您的 Spark 应用程序，如果你有一个大的值在任务中使用或任务比执行者更多。

It appears that a Spark idiom emerges that uses`broadcast`with`collectAsMap`to create a`Map`for broadcast. When an RDD is`map`over to a smaller dataset \(column-wise not record-wise\),`collectAsMap`, and`broadcast`, using the very big RDD to map its elements to the broadcast RDDs is computationally faster.

```
val acMap = sc.broadcast(myRDD.map { case (a,b,c,b) => (a, c) }.collectAsMap)
val otherMap = sc.broadcast(myOtherRDD.collectAsMap)

myBigRDD.map { case (a, b, c, d) =>
  (acMap.value.get(a).get, otherMap.value.get(c).get)
}.collect
```

如果可能，使用大的广播 HashMaps 在 RDD 上，并留下带有 key 的 RDD 来查找必要的数据，如上所示。

Spark 自带的 BitTorrent 实现。

默认情况下不启用。

### `Broadcast`Contract {#__a_id_contract_a_code_broadcast_code_contract}

`Broadcast`contract \(广播合同\)由自定义广播实现应提供的以下方法组成：

1. `getValue`

2. `doUnpersist`

3. `doDestroy`

| Note | TorrentBroadcast 是广播合同的唯一实现。 |
| :---: | :--- |


| Note | Broadcast Spark Developer-Facing Contract 是面向开发人员的广播合同，允许 Spark 开发人员在其应用程序中使用它。 |
| :---: | :--- |


### Further Reading or Watching {#__a_id_i_want_more_a_further_reading_or_watching}

* [Map-Side Join in Spark](http://dmtolpeko.com/2015/02/20/map-side-join-in-spark/)





















