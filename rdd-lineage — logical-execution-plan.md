## RDD Lineage — Logical Execution Plan {#_rdd_lineage_logical_execution_plan}

RDD Lineage（也称为 RDD 运算符图或 RDD 依赖图）是 RDD 的所有父 RDD 的图。它是对 RDD 应用转换并创建逻辑执行计划的结果。

| Note | 执行DAG或物理执行计划是 stages 的DAG。 |
| :---: | :--- |


| Note | 下图使用笛卡尔或 zip 仅用于学习目的。您可以使用其他运算符来构建 RDD 图。 |
| :---: | :--- |


![](/img/mastering-apache-spark/spark core-rdd/figure14.png)

上述RDD图可以是以下一系列变换的结果：

```
val r00 = sc.parallelize(0 to 9)
val r01 = sc.parallelize(0 to 90 by 10)
val r10 = r00 cartesian r01
val r11 = r00.map(n => (n, n))
val r12 = r00 zip r01
val r13 = r01.keyBy(_ / 20)
val r20 = Seq(r11, r12, r13).foldLeft(r10)(_ union _)
```

因此，RDD 沿袭血统图是在调用动作之后需要执行什么变换的图。

您可以使用 RDD.toDebugString 方法了解 RDD 沿袭血统图。

### Logical Execution Plan {#__a_id_logical_execution_plan_a_logical_execution_plan}

**Logical Execution Plan **以最早的 RDD（与其他 RDD 或引用缓存数据没有依赖关系的 RDD）开始，并以产生已执行的操作结果的 RDD 结束。

| Note | 当 SparkContext 被请求运行 Spark 作业时，逻辑计划（即 DAG）被实现并执行。 |
| :---: | :--- |


### `toDebugString`Method {#__a_id_todebugstring_a_code_todebugstring_code_method}

```
toDebugString: String
```

您可以使用 toDebugString 方法了解 RDD 沿袭血统图。

```
scala> val wordCount = sc.textFile("README.md").flatMap(_.split("\\s+")).map((_, 1)).reduceByKey(_ + _)
wordCount: org.apache.spark.rdd.RDD[(String, Int)] = ShuffledRDD[21] at reduceByKey at <console>:24

scala> wordCount.toDebugString
res13: String =
(2) ShuffledRDD[21] at reduceByKey at <console>:24 []
 +-(2) MapPartitionsRDD[20] at map at <console>:24 []
    |  MapPartitionsRDD[19] at flatMap at <console>:24 []
    |  README.md MapPartitionsRDD[18] at textFile at <console>:24 []
    |  README.md HadoopRDD[17] at textFile at <console>:24 []
```

toDebugString 使用缩进来指示 shuffle 边界。

圆括号中的数字表示每个阶段的平行度，例如。 （2）在上述输出中。

```
scala> wordCount.getNumPartitions
res14: Int = 2
```

在启用 spark.logLineage 属性的情况下，执行操作时包括 toDebugString。

```
$ ./bin/spark-shell --conf spark.logLineage=true

scala> sc.textFile("README.md", 4).count
...
15/10/17 14:46:42 INFO SparkContext: Starting job: count at <console>:25
15/10/17 14:46:42 INFO SparkContext: RDD's recursive dependencies:
(4) MapPartitionsRDD[1] at textFile at <console>:25 []
 |  README.md HadoopRDD[0] at textFile at <console>:25 []
```

### Settings {#__a_id_settings_a_settings}

Table 1. Spark Properties

| Spark Property | Default Value | Description |
| :--- | :--- | :--- |
| `spark.logLineage` | `false` | 当启用（即 true）时，执行一个 action（从而运行一个作业）也将使用 RDD.toDebugString 打印出 RDD 沿袭血统图。 |























