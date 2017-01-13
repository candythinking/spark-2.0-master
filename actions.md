## Actions {#_actions}

**Actions** 是产生非 RDD 值的 RDD 操作。他们在 Spark 程序中实现一个值。换句话说，返回任何类型的值而不是 RDD \[T\] 的 RDD 操作是 action。

```
action: RDD => a value
```

| Note | Actions 是同步的。您可以使用 AsyncRDDActions 在调用操作时释放调用线程。 |
| :---: | :--- |


它们触发执行 RDD transformations 以返回值。简单地说，一个 action 评估 RDD 谱系血统图。

您可以将 action 视为阀门，直到 action 被触发，即使在管道中也不会处理数据，即 transformations 。只有 action 才能用实际数据实现整个处理流水线。

Actions 是从执行器向驱动程序发送数据的两种方法之一（另一种是累加器）。

Actions in org.apache.spark.rdd.RDD:

* `aggregate`

* `collect`

* `count`

* `countApprox*`

* `countByValue*`

* `first`

* `fold`

* `foreach`

* `foreachPartition`

* `max`

* `min`

* `reduce`

* saveAs\* actions, e.g.`saveAsTextFile`,`saveAsHadoopFile`

* `take`

* `takeOrdered`

* `takeSample`

* `toLocalIterator`

* `top`

* `treeAggregate`

* `treeReduce`

操作使用 SparkContext.runJob 或直接 DAGScheduler.runJob 运行作业。

```
scala> words.count  (1)
res0: Long = 502
```

1. words is an RDD of String.

当您希望对其执行两个或多个操作以获得更好的性能时，应该缓存您正在使用的 RDD。请参阅 RDD 缓存和持久性。

在调用动作之前，Spark 会执行闭包/函数清理（使用 SparkContext.clean），使其准备好进行序列化，并通过线程发送给执行器。如果计算无法清除，清除可能会抛出 SparkException。

| Note | Spark 使用 ClosureCleaner 来清理闭包。 |
| :---: | :--- |


### AsyncRDDActions {#__a_id_asyncrddactions_a_asyncrddactions}

AsyncRDDActions 类提供了可以在 RDD 上使用的异步操作（感谢 RDD 类中的隐式转换 rddToAsyncRDDActions）。这些方法返回一个 FutureAction。

The following asynchronous methods are available:

* `countAsync`

* `collectAsync`

* `takeAsync`

* `foreachAsync`

* `foreachPartitionAsync`

### FutureActions {#__a_id_futureaction_a_futureactions}

| Caution | FIXME |
| :--- | :--- |




