## Streaming Sinks

流式接收器表示将流式数据集写入的外部存储器。它被建模为Sink trait，可以处理作为DataFrames给出的批数据。

Spark目前提供以下接收器：

* ConsoleSink for  console  format.
* FileStreamSink for  parquet  format.
* ForeachSink used in foreach operator.
* MemorySink for  memory  format.

您可以创建自己的流格式实现StreamSinkProvider。

## Sink Contract

sink contract由Sink trait描述。它定义了一个唯一的addBatch方法来将数据添加为batchId。

```
package org.apache.spark.sql.execution.streaming
trait Sink {
def addBatch(batchId: Long, data: DataFrame): Unit
}
```

## FileStreamSink

FileStreamSink是parquet格式的流接收器。

```
import scala.concurrent.duration._
import org.apache.spark.sql.streaming.{OutputMode, ProcessingTime}
val out = in.writeStream
.format("parquet")
.option("path", "parquet-output-dir")
.option("checkpointLocation", "checkpoint-dir")
.trigger(ProcessingTime(5.seconds))
.outputMode(OutputMode.Append)
.start()
```

FileStreamSink仅支持附加输出模式。

它使用spark.sql.streaming.fileSink.log.deletion（as isDeletingExpiredLog）

## MemorySink

MemorySink是基于内存的Sink，特别适用于测试。它将结果存储在内存中。

它可用作需要查询名称（通过queryName方法或queryName选项）的内存格式。

| Note | It was introduced in the [pull request for \[SPARK-14288\]\[SQL\] Memory Sink for streaming](https://github.com/apache/spark/pull/12119). |
| :---: | :--- |


Use  toDebugString  to see the batches.

它的目的是允许用户在Spark shell或其他本地测试中测试流应用程序。

您可以使用选项方法设置checkpointLocation，也可以将其设置为spark.sql.streaming.checkpointLocation设置。

如果spark.sql.streaming.checkpointLocation设置，代码使用$ location / $ queryName目录。

最后，当没有设置spark.sql.streaming.checkpointLocation时，使用java.io.tmpdir下的临时目录memory.stream，其中包含offsets子目录。

| Note | 在关闭时使用ShutdownHookManager.registerShutdownDeleteDir清除目录 |
| :---: | :--- |


```
val nums = spark.range(10).withColumnRenamed("id", "num")
scala> val outStream = nums.writeStream
.format("memory")
.queryName("memStream")
.start()
16/04/11 19:37:05 INFO HiveSqlParser: Parsing command: memStream
outStream: org.apache.spark.sql.StreamingQuery = Continuous Query - memStream [state =
ACTIVE]
```

它基于它操作的DataFrame的模式创建MemorySink实例。

它使用先前创建的MemorySink实例创建一个新的DataFrame并将其注册为临时表（使用DataFrame.registerTempTable方法）。

| Note | 此时，您可以使用sql方法查询表，就像它是一个常规的非流表。 |
| :---: | :--- |


一个新的StreamingQuery开始（使用StreamingQueryManager.startQuery）并返回。











