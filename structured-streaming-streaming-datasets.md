## Structured Streaming — Streaming Datasets

结构化流是Spark 2.0.0中引入的一种新的计算模型，用于构建称为连续应用程序的端到端流应用程序。结构化流提供了一个基于Datasets（在Spark SQL的引擎内）构建的高级声明式流API，用于连续增量执行结构化查询。

结构化流模型的语义如下（请参阅[Apache Spark中的结构化流](https://databricks.com/blog/2016/07/28/structured-streaming-in-apache-spark.html)）：

在任何时候，连续应用程序的输出等效于在数据的前缀上执行批处理作业。

结构化流是尝试统一流，交互式和批处理查询，为连续应用（如使用groupBy运算符的连续聚合或使用具有窗口函数的groupBy运算符的连续窗口聚合）​​铺平道路。

```
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

























