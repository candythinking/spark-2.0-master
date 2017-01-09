## ExecutorSource {#__a_id_executorsource_a_executorsource}

ExecutorSource 是 Executor 的度量来源。它使用执行器的 threadPool 来计算度量。

| Note | 每个执行器都有自己的单独的 ExecutorSource，当 CoarseGrainedExecutorBackend 接收到一个 RegisteredExecutor 时注册它。 |
| :---: | :--- |


ExecutorSource的名称是executor。

![](/img/mastering-apache-spark/spark core-architecture/figure8.png)

Table 1. ExecutorSource 计量器\(Gauges\)

| Gauge | Description |
| :--- | :--- |
| threadpool.activeTasks | 正在活动执行任务的线程数。 使用 ThreadPoolExecutor.getActiveCount（）。 |
| threadpool.completeTasks | 已完成执行的任务的大致总数。 使用 ThreadPoolExecutor.getCompletedTaskCount（）。 |
| threadpool.currentPool\_size | 池中当前的线程数。 使用 ThreadPoolExecutor.getPoolSize（）。 |
| threadpool.maxPool\_size | 已同时在池中的最大允许线程数 使用 ThreadPoolExecutor.getMaximumPoolSize（）。 |
| filesystem.hdfs.read\_bytes | Uses Hadoop’s FileSystem.getAllStatistics\(\)and`getBytesRead()`. |
| filesystem.hdfs.write\_bytes | Uses Hadoop’s FileSystem.getAllStatistics\(\) and`getBytesWritten()`. |
| filesystem.hdfs.read\_ops | Uses Hadoop’s FileSystem.getAllStatistics\(\) and`getReadOps()` |
| filesystem.hdfs.largeRead\_ops | Uses Hadoop’s FileSystem.getAllStatistics\(\) and`getLargeReadOps()`. |
| filesystem.hdfs.write\_ops | Uses Hadoop’s FileSystem.getAllStatistics\(\) and`getWriteOps()`. |
| filesystem.file.read\_bytes | The same as`hdfs`but for`file`scheme. |
| filesystem.file.write\_bytes | The same as`hdfs`but for`file`scheme. |
| filesystem.file.read\_ops | The same as`hdfs`but for`file`scheme. |
| filesystem.file.largeRead\_ops | The same as`hdfs`but for`file`scheme.+ |
| filesystem.file.write\_ops | The same as`hdfs`but for`file`scheme. |



