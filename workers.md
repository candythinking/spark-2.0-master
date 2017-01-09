## Workers {#_workers}

worker（又名 slaves）正在运行 Spark 实例，执行者在其中执行任务。它们是 Spark 中的计算节点。

| Caution | worker 是否只是 Spark Standalone 的一部分？ |
| :---: | :--- |


| Caution | 每个 worker 产生多少个执行者？ |
| :---: | :--- |


worker 接收它在线程池中运行的序列化任务。

它托管一个本地块管理器，为 Spark 集群中的其他 worker 提供块。worker 之间使用他们的块管理器实例进行通信。

解释 Spark 中的任务执行，并了解 Spark 的底层执行模型。

Spark UI 中经常面临的新词汇

当创建 SparkContext 时，每个 worker 启动一个执行器。这是一个单独的进程（JVM），它也加载你的 jar。执行器连接回您的驱动程序。现在驱动程序可以发送他们的命令，像 flatMap，map 和 reduceByKey。当驱动程序退出时，执行程序关闭。

不为每个步骤启动新的进程。当构建 SparkContext 时，将对每个 worker 启动一个新进程。

执行器反序列化命令（这是可能的，因为它已经加载你的 jar），并在分区上执行它。

Shortly speaking, an application in Spark is executed in three steps:

1. 创建 RDD 图，即 RDD 的 DAG（有向无环图）以表示整个计算。

2. 创建阶段\(stage\)图，即作为基于 RDD 图的逻辑执行计划的阶段的 DAG。通过在随机边界处打破 RDD 图形来创建阶段。

3. 根据计划，调度并在worker上执行相应的任务。

In the WordCount example, the RDD graph is as follows:

file → lines → words → per-word count → global word count → output

基于该图，创建两个阶段。阶段创建规则基于流水线尽可能多的窄变换的想法。具有“窄”依赖关系的 RDD 操作（如 map（）和filter（））在每个阶段中流水线连接到一组任务中。

最后，每个阶段将只对其他阶段具有随机依赖性，并且可以计算其中的多个操作。

In the WordCount example, the narrow transformation finishes at per-word count. Therefore, you get two stages:

* file → lines → words → per-word count

* global word count → output

一旦阶段被定义，Spark 将从阶段生成任务。第一个阶段将创建 ShuffleMapTasks，最后一个阶段创建 ResultTasks，因为在最后一个阶段，包括一个 action 操作以产生结果。

要生成的任务数量取决于文件的分布方式。假设你有三个不同的文件在三个不同的节点，第一阶段将生成3个任务：每个分区一个任务

因此，您不应该直接将步骤映射到任务。任务属于阶段，并与分区相关。

在每个阶段中生成的任务数量将等于分区数。

### Cleanup {#__a_id_cleanup_a_cleanup}

| Caution | FIXME |
| :--- | :--- |


### Settings {#__a_id_settings_a_settings}

* `spark.worker.cleanup.enabled`\(default:`false`\) Cleanup enabled.





