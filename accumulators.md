## Accumulators {#__a_id_accumulatorv2_a_accumulators}

**Accumulators \(**累加器\)是通过关联和交换“加法”操作“添加”的变量。它们充当用于在运行于执行器上的多个任务中累积部分值的容器。它们被设计为安全有效地并行和分布式 Spark 计算，并用于分布式计数器和求和。

您可以为长整型，双精度型或集合创建内置累加器，或使用 SparkContext.register 方法注册自定义累加器。您可以创建具有或不具有名称的累加器，但是只有命名的累加器显示在 web UI 中（在给定阶段的“阶段”选项卡下）。

![](/img/mastering-apache-spark/spark core-optimizations/figure3.png)

累加器是执行器的只写变量。它们可以由执行程序\(executor\)添加到驱动程序\(driver\)中读取。

```
executor1: accumulator.add(incByExecutor1)
executor2: accumulator.add(incByExecutor2)

driver:  println(accumulator.value)
```

Accumulators 不是线程安全的。因为驱动程序在任务完成（成功或失败）后用于更新累加器的值的DAGScheduler.updateAccumulators 方法只在运行调度循环的单个线程上执行，因此它们并不需要。除此之外，它们是具有自己的本地累加器引用的工人的只写数据结构，而访问累加器的值只允许由驱动程序。

累加器是可序列化的，因此它们可以安全地在执行器中执行的代码中引用，然后安全地通过线发送以执行。

```
val counter = sc.longAccumulator("counter")
sc.parallelize(1 to 9).foreach(x => counter.add(x))
```

在内部，longAccumulator，doubleAccumulator 和 collectionAccumulator 方法创建内置的类型化累加器并调用 SparkContext.register。

| Tip | Read the official documentation about [Accumulators](http://spark.apache.org/docs/latest/programming-guide.html#accumulators). |
| :--- | :--- |


### `metadata`Property {#__a_id_metadata_a_code_metadata_code_property}

| Caution | FIXME |
| :--- | :--- |


### `merge`Method {#__a_id_merge_a_code_merge_code_method}

| Caution | FIXME |
| :--- | :--- |


### AccumulatorV2 {#__a_id_accumulatorv2_a_accumulatorv2}

```
abstract class AccumulatorV2[IN, OUT]
```

AccumulatorV2 参数化类表示累加 IN 值以产生 OUT 结果的累加器。

#### Registering Accumulator \(register method\) {#__a_id_register_a_registering_accumulator_register_method}

```
register(
  sc: SparkContext,
  name: Option[String] = None,
  countFailedValues: Boolean = false): Unit
```

register 是 AccumulatorV2 抽象类的私有 \[spark\] 方法。

它为累加器创建一个 AccumulatorMetadata 元数据对象（具有一个新的唯一标识符），并使用 AccumulatorContext 注册累加器。然后，累加器与 ContextCleaner 注册进行清理。

### AccumulatorContext {#__a_id_accumulatorcontext_a_accumulatorcontext}

AccumulatorContext 是一个私有的 \[spark\] 内部对象，用于使用内部原始查找表来跟踪 Spark 自身的累加器。 Spark 使用 AccumulatorContext 对象来注册和注销累加器。

原始查找表将累加器标识符映射到累加器本身。

每个累加器都有自己唯一的累加器 ID，它使用内部 nextId 计数器分配。

#### AccumulatorContext.SQL\_ACCUM\_IDENTIFIER {#__a_id_accumulatorcontext_sql_accum_identifier_a_accumulatorcontext_sql_accum_identifier}

AccumulatorContext.SQL\_ACCUM\_IDENTIFIER 是 Spark SQL 的内部累加器的内部标识符。值为 sql，Spark 使用它来区分 Spark SQL 指标与其他指标。

### Named Accumulators {#__a_id_named_a_named_accumulators}

累加器可以有一个可选名称，您可以在创建累加器时指定。

```
val counter = sc.longAccumulator("counter")
```

### AccumulableInfo {#__a_id_accumulableinfo_a_accumulableinfo}

AccumulableInfo 包含有关任务对 Accumulable 的本地更新的信息。

* `id`of the accumulator

* optional`name`of the accumulator

* optional partial`update`to the accumulator from a task

* `value`

* whether or not it is`internal`

* whether or not to`countFailedValues`to the final value of the accumulator for failed tasks

* optional`metadata`

AccumulableInfo 用于将每个执行器心跳或任务完成时累积器更新从执行器传输到驱动程序。

使用提供的值创建此表示。

### When are Accumulators Updated? {#_when_are_accumulators_updated}

### Examples {#__a_id_examples_a_examples}

#### Example: Distributed Counter {#__a_id_example_distributed_counter_a_example_distributed_counter}

想象一下，你被要求写一个分布式计数器。你对以下解决方案有什么看法？使用它的优点和缺点是什么？

```
val ints = sc.parallelize(0 to 9, 3)

var counter = 0
ints.foreach { n =>
  println(s"int: $n")
  counter = counter + 1
}
println(s"The number of elements is $counter")
```

你将如何使用累加器进行计算？

#### Example: Using Accumulators in Transformations and Guarantee Exactly-Once Update {#__a_id_example1_a_example_using_accumulators_in_transformations_and_guarantee_exactly_once_update}

| Caution | 具有用 TaskContext 信息更新累加器（Map）的失败转换（任务）的代码。 |
| :---: | :--- |


#### Example: Custom Accumulator {#__a_id_example2_a_example_custom_accumulator}

| Caution | FIXME Improve the earlier example |
| :---: | :--- |


#### Example: Distributed Stopwatch {#__a_id_example3_a_example_distributed_stopwatch}

| Note | This is\_almost\_a raw copy of org.apache.spark.ml.util.DistributedStopwatch. |
| :---: | :--- |


```
class DistributedStopwatch(sc: SparkContext, val name: String) {

  val elapsedTime: Accumulator[Long] = sc.accumulator(0L, s"DistributedStopwatch($name)")

  override def elapsed(): Long = elapsedTime.value

  override protected def add(duration: Long): Unit = {
    elapsedTime += duration
  }
}
```

### Further reading or watching {#__a_id_i_want_more_a_further_reading_or_watching}

* [Performance and Scalability of Broadcast in Spark](http://www.cs.berkeley.edu/~agearh/cs267.sp10/files/mosharaf-spark-bc-report-spring10.pdf)



