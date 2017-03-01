## FileStreamSource

FileStreamSource是一个Source，用于从路径目录中读取文本文件。它使用LongOffset偏移。

| Note | 它由DataSource.createSource用于FileFormat。 |
| :--- | :--- |


您可以提供数据的模式和dataFrameBuilder - 在实例化时在getBatch中构建DataFrame的函数。

```
// NOTE The source directory must exist
// mkdir text-logs
val df = spark.readStream
.format("text")
.option("maxFilesPerTrigger", 1)
.load("text-logs")
scala> df.printSchema
root
|-- value: string (nullable = true)
```

批量索引。

它存在于org.apache.spark.sql.execution.streaming包中。

```
import org.apache.spark.sql.types._
val schema = StructType(
StructField("id", LongType, nullable = false) ::
StructField("name", StringType, nullable = false) ::
StructField("score", DoubleType, nullable = false) :: Nil)
// You should have input-json directory available
val in = spark.readStream
.format("json")
.schema(schema)
.load("input-json")
val input = in.transform { ds =>
println("transform executed") // <-- it's going to be executed once only
ds
}
scala> input.isStreaming
res9: Boolean = true
```

它跟踪已处理的文件在seenFiles哈希映射。

Enable  DEBUG  or  TRACE  logging level for  
 org.apache.spark.sql.execution.streaming.FileStreamSource  to see what happens

inside.

Add the following line to  conf/log4j.properties  :

```
log4j.logger.org.apache.spark.sql.execution.streaming.FileStreamSource=TRACE
```

## Options

## maxFilesPerTrigger

maxFilesPerTrigger选项指定每个触发器（批处理）的最大文件数。它限制文件流源读取maxFilesPerTrigger一次指定的文件数，因此启用速率限制。

它允许静态文件集\(file set\)，像流一样用于测试，因为文件集一次处理maxFilesPerTrigger个文件集。

## schema

如果在实例化时指定了模式（使用可选的dataSchema构造函数参数），则返回它。

否则，将调用fetchAllFiles内部方法以列出目录中的所有文件。

当有至少一个文件时，使用dataFrameBuilder构造函数参数函数计算模式。否则，将抛出IllegalArgumentException（“未指定模式”），除非它是用于文本提供程序（作为providerName构造函数参数），其中假定具有类型为StringType的单值列的默认模式。

| Note | 文本作为providerName构造函数参数的值表示文本文件流提供程序。 |
| :---: | :--- |


## getOffset

最大偏移量（getOffset）通过获取路径中的所有文件（不包括以\_（下划线）开头的文件）计算。

使用getOffset计算最大偏移量时，应在日志中看到以下DEBUG消息：

```
DEBUG Listed ${files.size} in ${(endTime.toDouble - startTime) / 1000000}ms
```

当使用getOffset计算最大偏移量时，它还过滤掉已经看到的文件（在seenFiles内部注册表中跟踪）。

您应该在日志中看到以下DEBUG消息（取决于文件的状态）：

```
new file: $file
// or
old file: $file
```

## getBatch

FileStreamSource.getBatch向批处理请求metadataLog。

您应该在日志中看到以下INFO和DEBUG消息：

```
INFO Processing ${files.length} files from ${startId + 1}:$endId
DEBUG Streaming ${files.mkString(", ")}
```

创建结果批处理的方法在实例化时提供（作为dataFrameBuilder构造函数参数）。

## metadataLog

metadataLog是使用metadataPath路径（这是一个构造函数参数）的元数据存储。

| Note | It extends  HDFSMetadataLog\[Seq\[String\]\]  . |
| :---: | :--- |




