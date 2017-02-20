像Apache Spark一般来说，Spark SQL特别是关于分布式内存计算。它们之间的主要区别 - Spark SQL和“裸”Spark Core的RDD计算模型 - 提供了一个框架，用于使用结构化查询加载，查询和持久存储结构化和半结构化数据，这些结构化查询可以使用SQL（使用子查询） ，Hive QL和定制的高级SQL级，声明式，类型安全的数据集API（对于结构化查询DSL）。无论您选择哪种结构化查询语言，它们都将成为Catalyst表达式树，并在进行大型分布式数据集的过程中进行进一步优化。

随着Spark 2.0最近的变化，Spark SQL现在实际上是Spark的基本内存分布式平台的主要和功能丰富的接口（将Spark Core的RDD隐藏在更高级别的抽象背后）。

```
// Found at http://stackoverflow.com/a/32514683/1305344
val dataset = Seq(
   "08/11/2015",
   "09/11/2015",
   "09/12/2015").toDF("date_string")

dataset.registerTempTable("dates")

// Inside spark-shell
scala > sql(
  """SELECT date_string,
        from_unixtime(unix_timestamp(date_string,'MM/dd/yyyy'), 'EEEEE') AS dow
      FROM dates""").show
+-----------+--------+
|date_string|     dow|
+-----------+--------+
| 08/11/2015| Tuesday|
| 09/11/2015|  Friday|
| 09/12/2015|Saturday|
+-----------+--------+
```

与SQL和NoSQL数据库一样，Spark SQL使用Catalyst的逻辑查询计划优化器，代码生成（通常可能比您自己的自定义代码更好）和Tungsten执行引擎使用自己的内部二进制行格​​式提供性能查询优化。

Spark SQL引入了名为Dataset（以前是DataFrame）的表式数据抽象。数据集数据抽象旨在使Spark基础设施上大量的结构化表格数据处理更简单，更快速。

| Note | 完美的引用适用于Spark SQL的Apache Drill：                               用于关系数据库和NoSQL数据库的SQL查询引擎，其具有对文件中的自描述和半结构化数据的直接查询，例如JSON或Parquet和HBase表，而无需在集中式存储中指定元数据定义。 |
| :---: | :--- |


以下代码段显示了一个批处理ETL管道，用于处理JSON文件并将其子集保存为CSV。

```
spark.read
  .format("json")
  .load("input-json")
  .select("name", "score")
  .where($"score" > 15)
  .write
  .format("csv")
  .save("output-csv")
```

然而，使用结构化流功能，上述静态批处理查询变为动态和连续铺设连续应用的方式。

```
import org.apache.spark.sql.types._
val schema = StructType(
  StructField("id", LongType, nullable = false) ::
  StructField("name", StringType, nullable = false) ::
  StructField("score", DoubleType, nullable = false) :: Nil)

spark.readStream
  .format("json")
  .schema(schema)
  .load("input-json")
  .select("name", "score")
  .where('score > 15)
  .writeStream
  .format("console")
  .start

// -------------------------------------------
// Batch: 1
// -------------------------------------------
// +-----+-----+
// | name|score|
// +-----+-----+
// |Jacek| 20.5|
// +-----+-----+
```

从Spark 2.0开始，Spark SQL的主要数据抽象是Dataset。它表示具有已知模式的记录的结构化数据。此结构化数据表示数据集使用存储在JVM堆外的受管对象中的压缩列格式启用紧凑二进制表示。它应该通过减少内存使用和GC来加快计算速度。

Spark SQL支持谓词下推，以优化数据集查询的性能，并且还可以在运行时生成优化的代码。

Spark SQL提供了不同的API来处理：

* Dataset API（以前的DataFrame API）与强类型的LINQ类查询DSL，Scala程序员可能会发现非常有吸引力的使用。

* 用于连续增量执行结构化查询的结构化流API（也称为流数据集）。

* 非程序员可能通过与Hive直接集成来使用SQL作为查询语言

* JDBC / ODBC fans 可以使用JDBC接口（通过Thrift JDBC / ODBC Server），并将它们的工具连接到Spark的分布式查询引擎。

Spark SQL提供了一个统一的接口，用于在使用专门的DataFrameReader和DataFrameWriter对象对Cassandra或HDFS（Hive，Parquet，JSON）等分布式存储系统中的数据进行访问。

Spark SQL允许您对可以驻留在Hadoop HDFS或Hadoop兼容文件系统（如S3）中的大量数据执行类似SQL的查询。它可以访问来自不同数据源（文件或表）的数据。

Spark SQL定义了三种类型的函数：

* 内置函数或用户定义函数（UDF），它从单个行获取值作为输入，为每个输入行生成单个返回值。

* 聚合函数，对一组行进行操作，并计算每个组的单个返回值。

* Windowed Aggregates（Windows），对一组行进行操作，并为组中的每一行计算一个返回值。

有两个支持的目录实现 - 内存（默认）和hive - 您可以使用spark.sql.catalogImplementation设置设置。

From user@spark:

如果已经将csv数据加载到数据框架中，为什么不将其注册为表，并使用Spark SQL查找max / min或任何其他聚合？ SELECT MAX（column\_name）FROM dftable\_name ...似乎很自然。

你更喜欢SQL，它可能值得注册这个DataFrame作为一个表，并生成SQL查询（生成一个字符串与一系列最小 - 最大调用）

您可以从外部数据源解析数据，并让模式参照器扣除模式。

```
// Example 1
val df = Seq(1 -> 2).toDF("i", "j")
val query = df.groupBy('i)
  .agg(max('j).as("aggOrdering"))
  .orderBy(sum('j))
  .as[(Int, Int)]
query.collect contains (1, 2) // true

// Example 2
val df = Seq((1, 1), (-1, 1)).toDF("key", "value")
df.createOrReplaceTempView("src")
scala> sql("SELECT IF(a > 0, a, 0) FROM (SELECT key a FROM src) temp").show
+-------------------+
|(IF((a > 0), a, 0))|
+-------------------+
|                  1|
|                  0|
+-------------------+
```

### Further reading or watching {#__a_id_i_want_more_a_further_reading_or_watching}

1. [Spark SQL](http://spark.apache.org/sql/)home page

2. \(video\)[Spark’s Role in the Big Data Ecosystem - Matei Zaharia](https://youtu.be/e-Ys-2uVxM0?t=6m44s)+

3. [Introducing Apache Spark 2.0](https://databricks.com/blog/2016/07/26/introducing-apache-spark-2-0.html)



















