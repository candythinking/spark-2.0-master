## Structured Streaming — Streaming Datasets

结构化流是Spark 2.0.0中引入的一种新的计算模型，用于构建称为连续应用程序的端到端流应用程序。结构化流提供了一个基于Datasets（在Spark SQL的引擎内）构建的高级声明式流API，用于连续增量执行结构化查询。

结构化流模型的语义如下（请参阅[Apache Spark中的结构化流](https://databricks.com/blog/2016/07/28/structured-streaming-in-apache-spark.html)）：

在任何时候，连续应用程序的输出等效于在数据的前缀上执行批处理作业。

结构化流是尝试统一流，交互式和批处理查询，为连续应用（如使用groupBy运算符的连续聚合或使用具有窗口函数的groupBy运算符的连续窗口聚合）​​铺平道路。

```scala
// business object
case class Person(id: Long, name: String, city: String)
// you could build your schema manually
import org.apache.spark.sql.types._
val schema = StructType(
StructField("id", LongType, false) ::
StructField("name", StringType, false) ::
StructField("city", StringType, false) :: Nil)
// ...but is error-prone and time-consuming, isn't it?
import org.apache.spark.sql.Encoders
val schema = Encoders.product[Person].schema
val people = spark.readStream
.schema(schema)
.csv("in/*.csv")
.as[Person]
// people has this additional capability of being streaming
scala> people.isStreaming
res0: Boolean = true
// ...but it is still a Dataset.
// (Almost) any Dataset operation is available
val population = people.groupBy('city).agg(count('city) as "population")
// Start the streaming query
// print the result out to the console
Structured Streaming — Streaming Datasets
418
// Only Complete output mode supported for groupBy
import org.apache.spark.sql.streaming.OutputMode.Complete
val populationStream = population.writeStream
.format("console")
.outputMode(Complete)
.queryName("Population")
.start
scala> populationStream.explain(extended = true)
== Parsed Logical Plan ==
Aggregate [city#112], [city#112, count(city#112) AS population#19L]
+- Relation[id#110L,name#111,city#112] csv
== Analyzed Logical Plan ==
city: string, population: bigint
Aggregate [city#112], [city#112, count(city#112) AS population#19L]
+- Relation[id#110L,name#111,city#112] csv
== Optimized Logical Plan ==
Aggregate [city#112], [city#112, count(city#112) AS population#19L]
+- Project [city#112]
+- Relation[id#110L,name#111,city#112] csv
== Physical Plan ==
*HashAggregate(keys=[city#112], functions=[count(city#112)], output=[city#112, populat
ion#19L])
+- Exchange hashpartitioning(city#112, 200)
+- *HashAggregate(keys=[city#112], functions=[partial_count(city#112)], output=[cit
y#112, count#118L])
+- *FileScan csv [city#112] Batched: false, Format: CSV, InputPaths: file:/Users
/jacek/dev/oss/spark/in/1.csv, file:/Users/jacek/dev/oss/spark/in/2.csv, file:/Users/j
..., PartitionFilters: [], PushedFilters: [], ReadSchema: struct<city:string>
// Let's query for all active streams
scala> spark.streams.active.foreach(println)
Streaming Query - Population [state = ACTIVE]
// You may eventually want to stop the streaming query
// spark.streams.active.head.stop
// ...or be more explicit about the right query to stop
scala> populationStream.isActive
res1: Boolean = true
scala> populationStream.stop
scala> populationStream.isActive
res2: Boolean = false
```

使用结构化流式处理，Spark 2.0旨在简化流分析，几乎不需要推断有效的数据流。这是Spark 2.0试图隐藏您的spark流分析架构中不必要的复杂性。

结构化流引入了无限数据集的流数据集，其具有像输入streaming sources和输出streaming sinks，事件时间\(event time\)，窗口\(windowing\)和会话\(sessions\)。您可以指定流数据集的输出模式，这是在有新数据可用时写入流接收器的内容。

| Tip | A Dataset is streaming when its logical plan is streaming. |
| :---: | :--- |


结构化流由org.apache.spark.sql.streaming包中的以下数据抽象定义：

* StreamingQuery

* Streaming Source

* Streaming Sink

* StreamingQueryManager

数据集是Spark SQL的结构化数据视图，结构化流检查每个触发器（时间）的新数据的输入源并执行（连续）查询。

| Tip | Watch [SPARK-8360 Streaming DataFrames](https://issues.apache.org/jira/browse/SPARK-8360) to track progress of the feature. |
| :---: | :--- |


| Tip | Read the official programming guide of Spark about [Structured Streaming](http://spark.apache.org/docs/latest/structured-streaming-programming-guide.html). |
| :---: | :--- |


| Note | 该功能也称为Streaming Spark SQL查询，流式DataFrames，连续数据框架或连续查询。在Spark项目结算结构化流之前，有很多名字。 |
| :---: | :--- |


## Example — Streaming Query for Running Counts \(over

Words from Socket with Output to Console\)

| Note | The example is "borrowed" from [the official documentation of Spark](http://spark.apache.org/docs/latest/structured-streaming-programming-guide.html). Changes and errors are only mine. |
| :---: | :--- |


| Tip | 在运行示例之前，您需要先运行nc -lk 9999。 |
| :---: | :--- |


```
val lines = spark.readStream
.format("socket")
.option("host", "localhost")
.option("port", 9999)
.load
.as[String]
val words = lines.flatMap(_.split("\\W+"))
scala> words.printSchema
root
|-- value: string (nullable = true)
val counter = words.groupBy("value").count
// nc -lk 9999 is supposed to be up at this point
import org.apache.spark.sql.streaming.OutputMode.Complete
val query = counter.writeStream
.outputMode(Complete)
.format("console")
.start
query.stop
```

## Example — Streaming Query over CSV Files with Output to Console Every 5 Seconds

下面你可以找到一个完整的例子，一个DataFrame的数据每5秒钟从一个给定的模式的csv格式的csv-logs文件的流式查询。

| Tip | 复制并粘贴到Spark Shell中：粘贴模式运行它。 |
| :---: | :--- |


```
// Explicit schema with nullables false
import org.apache.spark.sql.types._
val schemaExp = StructType(
StructField("name", StringType, false) ::
StructField("city", StringType, true) ::
StructField("country", StringType, true) ::
StructField("age", IntegerType, true) ::
StructField("alive", BooleanType, false) :: Nil
)
// Implicit inferred schema
val schemaImp = spark.read
.format("csv")
.option("header", true)
.option("inferSchema", true)
.load("csv-logs")
.schema
val in = spark.readStream
.schema(schemaImp)
.format("csv")
.option("header", true)
.option("maxFilesPerTrigger", 1)
.load("csv-logs")
scala> in.printSchema
root
|-- name: string (nullable = true)
|-- city: string (nullable = true)
|-- country: string (nullable = true)
|-- age: integer (nullable = true)
|-- alive: boolean (nullable = true)
println("Is the query streaming" + in.isStreaming)
println("Are there any streaming queries?" + spark.streams.active.isEmpty)
import scala.concurrent.duration._
import org.apache.spark.sql.streaming.ProcessingTime
import org.apache.spark.sql.streaming.OutputMode.Append
val out = in.writeStream
.format("console")
.trigger(ProcessingTime(5.seconds))
.queryName("consoleStream")
.outputMode(Append)
.start()
16/07/13 12:32:11 TRACE FileStreamSource: Listed 3 file(s) in 4.274022 ms
16/07/13 12:32:11 TRACE FileStreamSource: Files are:
file:///Users/jacek/dev/oss/spark/csv-logs/people-1.csv
file:///Users/jacek/dev/oss/spark/csv-logs/people-2.csv
file:///Users/jacek/dev/oss/spark/csv-logs/people-3.csv
16/07/13 12:32:11 DEBUG FileStreamSource: New file: file:///Users/jacek/dev/oss/spark/
csv-logs/people-1.csv
16/07/13 12:32:11 TRACE FileStreamSource: Number of new files = 3
16/07/13 12:32:11 TRACE FileStreamSource: Number of files selected for batch = 1
16/07/13 12:32:11 TRACE FileStreamSource: Number of seen files = 1
16/07/13 12:32:11 INFO FileStreamSource: Max batch id increased to 0 with 1 new files
16/07/13 12:32:11 INFO FileStreamSource: Processing 1 files from 0:0
16/07/13 12:32:11 TRACE FileStreamSource: Files are:
file:///Users/jacek/dev/oss/spark/csv-logs/people-1.csv
-------------------------------------------
Batch: 0
-------------------------------------------
+-----+--------+-------+---+-----+
| name| city|country|age|alive|
+-----+--------+-------+---+-----+
|Jacek|Warszawa| Polska| 42| true|
+-----+--------+-------+---+-----+
spark.streams
.active
.foreach(println)
// Streaming Query - consoleStream [state = ACTIVE]
scala> spark.streams.active(0).explain
== Physical Plan ==
*Scan csv [name#130,city#131,country#132,age#133,alive#134] Format: CSV, InputPaths: f
ile:/Users/jacek/dev/oss/spark/csv-logs/people-3.csv, PushedFilters: [], ReadSchema: s
truct<name:string,city:string,country:string,age:int,alive:boolean>
```

## Further reading or watching

[Structured Streaming In Apache Spark](https://databricks.com/blog/2016/07/28/structured-streaming-in-apache-spark.html)

\(video\) [The Future of Real Time in Spark](https://youtu.be/oXkxXDG0gNk) from Spark Summit East 2016 in which

Reynold Xin presents the concept of Streaming DataFrames to the public.

\(video\) [Structuring Spark: DataFrames, Datasets, and Streaming](https://youtu.be/i7l3JQRx7Qw?t=19m15s)

[What Spark’s Structured Streaming really means](http://www.infoworld.com/article/3052924/analytics/what-sparks-structured-streaming-really-means.html)

\(video\) [A Deep Dive Into Structured Streaming](https://youtu.be/rl8dIzTpxrI) by Tathagata "TD" Das from Spark

Summit 2016

