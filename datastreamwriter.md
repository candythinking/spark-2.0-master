## DataStreamWriter

DataFrameWriter是Spark 2.0中的Structured Streaming API的一部分，负责将流查询的输出写入sinks，从而开始执行。

```
val people: Dataset[Person] = ...
import org.apache.spark.sql.streaming.ProcessingTime
import scala.concurrent.duration._
import org.apache.spark.sql.streaming.OutputMode.Complete
df.writeStream
.queryName("textStream")
.outputMode(Complete)
.trigger(ProcessingTime(10.seconds))
.format("console")
.start
```

1. queryName to set the name of a query

2. outputMode to specify output mode.

3. trigger to set the Trigger for a stream query.

4. start to start continuous writing to a sink.

## Specifying Output Mode — outputMode method

```
outputMode(outputMode: OutputMode): DataStreamWriter[T]
```

outputMode指定流数据集的输出模式，这是在有新数据可用时写入流接收器的内容。

目前支持以下输出模式：

* OutputMode.Append - 只有流数据集中的新行将被写入sink。

* OutputMode.Complete - 每次有更新时，整个流数据集（包括所有行）将被写入接收器。仅对具有聚合的流查询支持它。

* OutputMode.Update - 还在完善当中

## Setting Query Name — queryName method

```
queryName(queryName: String): DataStreamWriter[T]
```

queryName设置流式查询的名称。

在内部，它只是一个带有queryName键的附加选项。

## Setting How Often to Execute Streaming Query — trigger method

```
trigger(trigger: Trigger): DataStreamWriter[T]
```

触发方法设置流查询的触发（批处理）的时间间隔。

| Note | 触发器指定StreamingQuery应该生成结果的频率。请参阅触发器。 |
| :---: | :--- |


默认触发器是处理时间（0L），尽可能经常运行流查询。

| Tip | 请参阅Trigger了解Trigger和ProcessingTime类型。 |
| :---: | :--- |


## Starting Continuous Writing to Sink — start methods

```
start(): StreamingQuery
start(path: String): StreamingQuery (1)
```

将路径选项设置为路径和调用start（）

start方法启动流式查询并返回StreamingQuery对象以连续写入数据。

| Note | 无论你是否有指定路径的选择取决于使用的数据源。 |
| :---: | :--- |


认可选项：

* queryName是活动流式查询的名称。

* checkpointLocation是用于检查点的目录。

| Note | 使用选项或选项方法定义选项。 |
| :---: | :--- |


## foreach method

















