## HadoopRDD {#_hadooprdd}

HadoopRDD 是一个 RDD，它提供用于读取存储在 HDFS，本地文件系统（在所有节点上可用）或任何使用旧版 MapReduce API（org.apache.hadoop.mapred）的 Hadoop 支持的文件系统 URI 的数据的核心功能。

HadoopRDD 是在 SparkContext 中调用以下方法的结果：

* `hadoopFile`

* `textFile`\(the most often used in examples!\)

* `sequenceFile`

分区类型为 HadoopPartition。

当计算 HadoopRDD，即调用操作时，您应该在日志中看到 INFO 消息输入 split：

```
scala> sc.textFile("README.md").count
...
15/10/10 18:03:21 INFO HadoopRDD: Input split: file:/Users/jacek/dev/oss/spark/README.md:0+1784
15/10/10 18:03:21 INFO HadoopRDD: Input split: file:/Users/jacek/dev/oss/spark/README.md:1784+1784
...
```

The following properties are set upon partition execution:

* **mapred.tip.id**- task id of this task’s attempt

* **mapred.task.id**- task attempt’s id

* **mapred.task.is.map**as`true`+

* **mapred.task.partition**- split id

* **mapred.job.id**

Spark settings for`HadoopRDD`:

* **spark.hadoop.cloneConf**\(default:`false`\) - shouldCloneJobConf - 应该在生成 Hadoop 作业之前克隆 Hadoop 作业配置 JobConf 对象。参考 \[SPARK-2546\] 配置对象线程安全问题。如果为 true，您应该看到 DEBUG 消息克隆 Hadoop 配置。

您可以在 TaskContext 上注册回调。

HadoopRDD 不设置检查点。当调用 checkpoint（）时，它们什么也不做。

| Caution | 什么是 InputMetrics？                                                                       什么是 JobConf？                                                                                  什么是 InputSplits：FileSplit 和 CombineFileSplit？ \*什么是 InputFormat 和可配置子类型？                                                     什么是 InputFormat 的 RecordReader？它创建一个键和一个值。他们是什么？                                                                                     什么是 Hadoop Split？输入拆分为 Hadoop 读取？请参阅 InputFormat.getSplits |
| :---: | :--- |


### getPartitions {#__a_id_getpartitions_a_getpartitions}

HadoopRDD 的分区数，即 getPartitions 的返回值，是使用 InputFormat.getSplits（jobConf，minPartitions）计算的，其中 minPartitions 只是最少需要多少个分区的提示。作为提示，这并不意味着分区的数量将完全是给定的数量。

对于 SparkContext.textFile，输入格式类是 org.apache.hadoop.mapred.TextInputFormat。

org.apache.hadoop.mapred.FileInputFormat 的 javadoc 表示：

FileInputFormat 是所有基于文件的 InputFormat 的基类。这提供了 getSplits（JobConf，int）的通用实现。 FileInputFormat 的子类还可以覆盖 isSplitable（FileSystem，Path）方法，以确保输入文件不被分割，并由 Mappers 作为一个整体进行处理。

| Tip | 您可以找到 org.apache.hadoop.mapred.FileInputFormat.getSplits 启发的源。 |
| :---: | :--- |








