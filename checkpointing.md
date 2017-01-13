## Checkpointing {#_checkpointing}

### Introduction {#_introduction}

检查点是一个截断 RDD 沿袭血统图并将其保存到可靠分布式（HDFS）或本地文件系统的过程。、

There are two types of checkpointing:+

* **reliable**- in Spark \(core\), RDD checkpointing that saves the actual intermediate\(中间\) RDD data to a reliable distributed file system, e.g. HDFS.

* **local**- in Spark Streaming or GraphX - RDD checkpointing that truncates\(截断\) RDD lineage graph.

It’s up to a Spark application developer to decide when and how to checkpoint using`RDD.checkpoint()`method.

在使用检查点之前，Spark 开发人员必须使用 SparkContext.setCheckpointDir（directory：String）方法设置检查点目录。

### Reliable Checkpointing {#_reliable_checkpointing}

您调用 SparkContext.setCheckpointDir（directory：String）设置检查点目录 - RDD 的检查点目录。如果在集群上运行，该目录必须是 HDFS 路径。原因是驱动程序（driver）可能尝试从自己的本地文件系统重建检查点的 RDD，这是不正确的，因为检查点文件实际上在执行程序机器上。

您通过调用 RDD.checkpoint（）为 RDD 进行检查点标记。 RDD 将保存到检查点目录中的文件，并且将删除对其父 RDD 的所有引用。在对此 RDD 执行任何作业之前，必须调用此函数。

| Note | 强烈建议将一个检查点的 RDD 持久化在内存中，否则将其保存在文件中将需要重新计算。 |
| :---: | :--- |


当对已检查点的 RDD 调用操作时，将在日志中打印以下 INFO 消息：

```
15/10/10 21:08:57 INFO ReliableRDDCheckpointData: Done checkpointing RDD 5 to file:/Users/jacek/dev/oss/spark/checkpoints/91514c29-d44b-4d95-ba02-480027b7c174/rdd-5, new parent is RDD 6
```

#### ReliableRDDCheckpointData {#_reliablerddcheckpointdata}

当调用 RDD.checkpoint（）操作时，与 RDD 检查点相关的所有信息都在 ReliableRDDCheckpointData 中。

#### ReliableCheckpointRDD {#_reliablecheckpointrdd}

在 RDD.checkpoint 之后，RDD 具有 ReliableCheckpointRDD 作为具有作为 RDD 的分区的确切数目的新父 RDD。

### Local Checkpointing {#_local_checkpointing}

除了 RDD.checkpoint（）方法之外，还有类似的一个 - RDD.localCheckpoint（），它使用 Spark 的现有缓存图层标记用于本地检查点的 RDD。

此 RDD.localCheckpoint（）方法适用于希望截断 RDD 沿袭血统图的用户，同时跳过在可靠的分布式文件系统中复制实体化数据的昂贵步骤。这对于需要定期截断的具有长谱系血统的 RDD 是有用的，例如。 GraphX。

本地检查点以性能的容错作为交换代价。

不使用通过 SparkContext.setCheckpointDir 设置的检查点目录。

#### LocalRDDCheckpointData {#_localrddcheckpointdata}

FIXME

#### LocalCheckpointRDD {#_localcheckpointrdd}

FIXME

### `doCheckpoint`Method {#__a_id_docheckpoint_a_code_docheckpoint_code_method}

| Caution | FIXME |
| :--- | :--- |




