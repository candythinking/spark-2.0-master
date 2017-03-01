## TextSocketSource

TextSocketSource是一个流源，从主机和端口（由参数定义）的套接字读取行。

它使用内部内存缓冲区来保持从套接字永远读取的所有行。

| Caution | 由于设计限制，该源不用于生产。无限内存收集的行读取和没有故障恢复。它仅用于教程和调试。 |
| :---: | :--- |


```
import org.apache.spark.sql.SparkSession
val spark: SparkSession = SparkSession.builder.getOrCreate()
// Connect to localhost:9999
// You can use "nc -lk 9999" for demos
val textSocket = spark.readStream
.format("socket")
.option("host", "localhost")
.option("port", 9999)
.load
import org.apache.spark.sql.Dataset
val lines: Dataset[String] = textSocket.as[String].map(_.toUpperCase)
val query = lines.writeStream.format("console").start
// Start typing the lines in nc session
// They will appear UPPERCASE in the terminal
-------------------------------------------
Batch: 0
-------------------------------------------
+---------+
| value|
+---------+
|UPPERCASE|
+---------+
scala> query.explain
== Physical Plan ==
*SerializeFromObject [staticinvoke(class org.apache.spark.unsafe.types.UTF8String, Str
ingType, fromString, input[0, java.lang.String, true], true) AS value#21]
+- *MapElements <function1>, obj#20: java.lang.String
+- *DeserializeToObject value#43.toString, obj#19: java.lang.String
+- LocalTableScan [value#43]
scala> query.stop
```

## lines Internal Buffer

```
lines: ArrayBuffer[(String, Timestamp)]
```

lines是从socket中读取的所有行TextSocketSource的内部缓冲区。

## Maximum Available Offset \(getOffset method\)

| Note | getOffset  is a part of the Streaming Source Contract. |
| :---: | :--- |


TextSocketSource的偏移量可以是内部行缓冲区中的行数的none或LongOffset。

## Schema \(schema method\)

TextSocketSource支持两种模式：

1. A single  value  field of String type.

2.  value  field of  StringType  type and  timestamp  field of TimestampType type of format

yyyy-MM-dd HH:mm:ss  

## Creating TextSocketSource Instance

```
TextSocketSource(
host: String,
port: Int,
includeTimestamp: Boolean,
sqlContext: SQLContext)
```

当TextSocketSource被创建（见TextSocketSourceProvider），它获得4个传递的参数：

1.  host

2.  port

3. includeTimestamp flag

4. SQLContext

它在给定的主机和端口参数下打开一个套接字，并使用默认字符集和默认大小的输入缓冲区（8192字节）逐行读取缓冲字符输入流。

它启动一个readThread守护线程（称为TextSocketSource（host，port））从套接字读取行。行被添加到内部行缓冲区。

## Stopping TextSocketSource \(stop method\)

停止时，TextSocketSource关闭套接字连接。

## MemoryStream

MemoryStream是产生存储在存储器中的值（类型T）的流源。它使用数据集的内部批次集合。

| Caution | 由于设计限制，该源不用于生产。无限内存收集的行读取和没有故障恢复。                                                                                             它主要用于单元测试，教程和调试。 |
| :---: | :--- |


```
import org.apache.spark.sql.execution.streaming.MemoryStream
import org.apache.spark.sql.SparkSession
val spark: SparkSession = SparkSession.builder.getOrCreate()
implicit val ctx = spark.sqlContext
// It uses two implicits: Encoder[Int] and SQLContext
val intsInput = MemoryStream[Int]
scala> val memoryQuery = ints.toDF.writeStream.format("memory").queryName("memStream")
.start
memoryQuery: org.apache.spark.sql.streaming.StreamingQuery = Streaming Query - memStre
am [state = ACTIVE]
scala> val zeroOffset = intsInput.addData(0, 1, 2)
zeroOffset: org.apache.spark.sql.execution.streaming.Offset = #0
memoryQuery.processAllAvailable()
val intsOut = spark.table("memStream").as[Int]
scala> intsOut.show
+-----+
|value|
+-----+
| 0|
| 1|
| 2|
+-----+
memoryQuery.stop()
```

Enable  DEBUG  logging level for org.apache.spark.sql.execution.streaming.MemoryStream  logger to see what

happens inside.

Add the following line to  conf/log4j.properties  :

```
log4j.logger.org.apache.spark.sql.execution.streaming.MemoryStream=DEBUG
```

## Creating MemoryStream Instance

```
apply[A : Encoder](implicit sqlContext: SQLContext): MemoryStream[A]
```

MemoryStream对象定义应用方法，您可以使用它来创建MemoryStream流源实例。

## Adding Data to Source \(addData methods\)

```
addData(data: A*): Offset
addData(data: TraversableOnce[A]): Offset
```

addData方法将输入数据添加到批量内部集合中。

执行时，addData添加一个DataFrame（使用toDS隐式方法创建），并增加内部currentOffset偏移量。

You should see the following DEBUG message in the logs:

```
DEBUG MemoryStream: Adding ds: [ds]
```

## Getting Next Batch \(getBatch method\)

| Note | getBatch  is a part of Streaming Source contract. |
| :---: | :--- |


执行时，getBatch使用内部批处理集合返回请求的偏移量。

You should see the following DEBUG message in the logs:

```
DEBUG MemoryStream: MemoryBatch [[startOrdinal], [endOrdinal]]: [newBlocks]
```

## StreamingExecutionRelation Logical Plan

MemoryStream使用StreamingExecutionRelation逻辑计划在请求时构建数据集或DataFrames。

StreamingExecutionRelation是为流源和属性的给定输出集合创建的叶逻辑节点。它是一个流逻辑计划，其名称是源的名称。

```
scala> val ints = MemoryStream[Int]
ints: org.apache.spark.sql.execution.streaming.MemoryStream[Int] = MemoryStream[value#
13]
scala> ints.toDS.queryExecution.logical.isStreaming
res14: Boolean = true
scala> ints.toDS.queryExecution.logical
res15: org.apache.spark.sql.catalyst.plans.logical.LogicalPlan = MemoryStream[value#13]
```

## Schema \(schema method\)

MemoryStream使用由（数据集的）编码器描述的模式的数据。























