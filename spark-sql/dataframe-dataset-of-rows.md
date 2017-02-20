## DataFrame — Dataset of Rows {#_dataframe_dataset_of_rows}

Spark SQL引入了一个名为DataFrame的表式数据抽象。它旨在简化在Spark基础设施上处理大量结构化表格数据。

DataFrame是用于处理结构化和半结构化数据（即具有模式的Dataset）的数据抽象或领域专用语言（DSL）。因此，DataFrame是具有作为其描述的结构化查询的结果的模式的行的集合。

它使用RDD的不可变，内存，弹性，分布式和并行功能，并将一个称为模式的结构应用于数据。

在Spark 2.0.0中DataFrame是Dataset \[Row\]的纯类型别名。

```
type DataFrame = Dataset[Row]
```

DataFrame是组织成行和命名列的表式数据的分布式集合。它在概念上等同于关系数据库中具有用于对RDD进行投影（选择）project \(select\), filter, intersect, join, group, sort, join, aggregate或转换的操作的表（参见DataFrame API）

```
data.groupBy('Product_ID).sum('Score)
```

Spark SQL从pandas的DataFrame借用了DataFrame的概念，使它不可变，并行（一台机器，可能有许多处理器和核心）和分布式（许多机器，可能有许多处理器和核心）。

Spark SQL中的DataFrames强烈依赖RDD的功能 - 它基本上是一个RDD，通过适当的操作暴露为结构化DataFrame，以处理从第一天开始的非常大的数据。所以，PB数据不应该吓唬你（除非你是管理员创建这样的集群Spark环境）。

```
val df = Seq(("one", 1), ("one", 1), ("two", 1))
  .toDF("word", "count")

scala> df.show
+----+-----+
|word|count|
+----+-----+
| one|    1|
| one|    1|
| two|    1|
+----+-----+

val counted = df.groupBy('word).count

scala> counted.show
+----+-----+
|word|count|
+----+-----+
| two|    1|
| one|    2|
+----+-----+
```

您可以通过从结构化文件（JSON，Parquet，CSV），RDD，Hive中的表或外部数据库（JDBC）加载数据来创建DataFrames。您还可以从头开始创建DataFrames并构建它们（如上例所示）。请参阅DataFrame API。给定适当的DataFrameReader的Spark SQL扩展以适当地格式化数据集，您可以读取任何格式。

您可以使用两种方法通过DataFrames执行查询：

* 良好的SQL - 帮助从“SQL数据库”世界迁移到Spark SQL中的DataFrame世界。

* 查询DSL - 一种有助于在编译时确保正确语法的API。

DataFrame还允许您执行以下任务：

* Filtering

DataFrames使用Catalyst查询优化器来生成高效的查询（因此它们应该比相应的基于RDD的查询更快）。

| Note | 您的DataFrames也可以是类型安全的，此外通过专门的编码器可以显着减少序列化和反序列化时间进一步提高其性能。 |
| :---: | :--- |


您可以在通用行上强制执行类型，因此通过将行编码为类型安全的Dataset对象，从而提高类型安全性（在编译时）。从Spark 2.0开始，它是开发Spark应用程序的首选方法。

### Features of DataFrame {#__a_id_features_a_features_of_dataframe}

DataFrame是“通用”Row实例（如RDD \[Row\]）和模式的集合。

| Note | 不管你如何创建一个DataFrame，它总是一对RDD \[Row\]和StructType。 |
| :---: | :--- |


### Enforcing Types \(as method\) {#__a_id_as_a_enforcing_types_as_method}

DataFrame是Dataset \[Row\]的类型别名。您可以使用as方法强制实施字段的类型。 为您提供从Dataset \[Row\]到Dataset \[T\]的转换。

```
// Create DataFrame of pairs
val df = Seq("hello", "world!").zipWithIndex.map(_.swap).toDF("id", "token")

scala> df.printSchema
root
 |-- id: integer (nullable = false)
 |-- token: string (nullable = true)

scala> val ds = df.as[(Int, String)]
ds: org.apache.spark.sql.Dataset[(Int, String)] = [id: int, token: string]

// It's more helpful to have a case class for the conversion
final case class MyRecord(id: Int, token: String)

scala> val myRecords = df.as[MyRecord]
myRecords: org.apache.spark.sql.Dataset[MyRecord] = [id: int, token: string]
```

### Writing DataFrames to External Storage \(write method\) {#__a_id_write_a_writing_dataframes_to_external_storage_write_method}

| Caution | FIXME |
| :--- | :--- |


### SQLContext, spark, and Spark shell {#_sqlcontext_spark_and_spark_shell}

您可以使用org.apache.spark.sql.SQLContext构建DataFrames并执行SQL查询。

使用Spark SQL的最快最简单的方法是使用Spark shell和spark对象。

```
scala> spark
res1: org.apache.spark.sql.SQLContext = org.apache.spark.sql.hive.HiveContext@60ae950f
```

正如你可能已经注意到的，spark shell中的spark实际上是一个org.apache.spark.sql.hive.HiveContext，它将Spark SQL执行引擎与存储在Apache Hive中的数据集成。

Apache Hive™数据仓库软件便于查询和管理位于分布式存储中的大型数据集。

### Creating DataFrames from Scratch {#_creating_dataframes_from_scratch}

使用Spark shell，如Spark shell中所述。

#### Using toDF {#_using_todf}

在导入spark.implicits.\_（这是由Spark shell完成的）之后，您可以应用toDF方法将对象转换为DataFrames。

```
scala> val df = Seq("I am a DataFrame!").toDF("text")
df: org.apache.spark.sql.DataFrame = [text: string]
```

#### Creating DataFrame using Case Classes in Scala {#_creating_dataframe_using_case_classes_in_scala}

此方法假定数据来自将描述模式的Scala案例类。

```
scala> case class Person(name: String, age: Int)
defined class Person

scala> val people = Seq(Person("Jacek", 42), Person("Patryk", 19), Person("Maksym", 5))
people: Seq[Person] = List(Person(Jacek,42), Person(Patryk,19), Person(Maksym,5))

scala> val df = spark.createDataFrame(people)
df: org.apache.spark.sql.DataFrame = [name: string, age: int]

scala> df.show
+------+---+
|  name|age|
+------+---+
| Jacek| 42|
|Patryk| 19|
|Maksym|  5|
+------+---+
```

#### Custom DataFrame Creation using createDataFrame {#_custom_dataframe_creation_using_createdataframe}

SQLContext提供了一系列createDataFrame操作。

```
scala> val lines = sc.textFile("Cartier+for+WinnersCurse.csv")
lines: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[3] at textFile at <console>:24

scala> val headers = lines.first
headers: String = auctionid,bid,bidtime,bidder,bidderrate,openbid,price

scala> import org.apache.spark.sql.types.{StructField, StringType}
import org.apache.spark.sql.types.{StructField, StringType}

scala> val fs = headers.split(",").map(f => StructField(f, StringType))
fs: Array[org.apache.spark.sql.types.StructField] = Array(StructField(auctionid,StringType,true), StructField(bid,StringType,true), StructField(bidtime,StringType,true), StructField(bidder,StringType,true), StructField(bidderrate,StringType,true), StructField(openbid,StringType,true), StructField(price,StringType,true))

scala> import org.apache.spark.sql.types.StructType
import org.apache.spark.sql.types.StructType

scala> val schema = StructType(fs)
schema: org.apache.spark.sql.types.StructType = StructType(StructField(auctionid,StringType,true), StructField(bid,StringType,true), StructField(bidtime,StringType,true), StructField(bidder,StringType,true), StructField(bidderrate,StringType,true), StructField(openbid,StringType,true), StructField(price,StringType,true))

scala> val noheaders = lines.filter(_ != header)
noheaders: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[10] at filter at <console>:33

scala> import org.apache.spark.sql.Row
import org.apache.spark.sql.Row

scala> val rows = noheaders.map(_.split(",")).map(a => Row.fromSeq(a))
rows: org.apache.spark.rdd.RDD[org.apache.spark.sql.Row] = MapPartitionsRDD[12] at map at <console>:35

scala> val auctions = spark.createDataFrame(rows, schema)
auctions: org.apache.spark.sql.DataFrame = [auctionid: string, bid: string, bidtime: string, bidder: string, bidderrate: string, openbid: string, price: string]

scala> auctions.printSchema
root
 |-- auctionid: string (nullable = true)
 |-- bid: string (nullable = true)
 |-- bidtime: string (nullable = true)
 |-- bidder: string (nullable = true)
 |-- bidderrate: string (nullable = true)
 |-- openbid: string (nullable = true)
 |-- price: string (nullable = true)

scala> auctions.dtypes
res28: Array[(String, String)] = Array((auctionid,StringType), (bid,StringType), (bidtime,StringType), (bidder,StringType), (bidderrate,StringType), (openbid,StringType), (price,StringType))

scala> auctions.show(5)
+----------+----+-----------+-----------+----------+-------+-----+
| auctionid| bid|    bidtime|     bidder|bidderrate|openbid|price|
+----------+----+-----------+-----------+----------+-------+-----+
|1638843936| 500|0.478368056|  kona-java|       181|    500| 1625|
|1638843936| 800|0.826388889|     doc213|        60|    500| 1625|
|1638843936| 600|3.761122685|       zmxu|         7|    500| 1625|
|1638843936|1500|5.226377315|carloss8055|         5|    500| 1625|
|1638843936|1600|   6.570625|    jdrinaz|         6|    500| 1625|
+----------+----+-----------+-----------+----------+-------+-----+
only showing top 5 rows
```

### Loading data from structured files {#_loading_data_from_structured_files}

#### Creating DataFrame from CSV file {#_creating_dataframe_from_csv_file}

让我们从一个示例开始，其中模式推理依赖于Scala中的自定义case类。

```
scala> val lines = sc.textFile("Cartier+for+WinnersCurse.csv")
lines: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[3] at textFile at <console>:24

scala> val header = lines.first
header: String = auctionid,bid,bidtime,bidder,bidderrate,openbid,price

scala> lines.count
res3: Long = 1349

scala> case class Auction(auctionid: String, bid: Float, bidtime: Float, bidder: String, bidderrate: Int, openbid: Float, price: Float)
defined class Auction

scala> val noheader = lines.filter(_ != header)
noheader: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[53] at filter at <console>:31

scala> val auctions = noheader.map(_.split(",")).map(r => Auction(r(0), r(1).toFloat, r(2).toFloat, r(3), r(4).toInt, r(5).toFloat, r(6).toFloat))
auctions: org.apache.spark.rdd.RDD[Auction] = MapPartitionsRDD[59] at map at <console>:35

scala> val df = auctions.toDF
df: org.apache.spark.sql.DataFrame = [auctionid: string, bid: float, bidtime: float, bidder: string, bidderrate: int, openbid: float, price: float]

scala> df.printSchema
root
 |-- auctionid: string (nullable = true)
 |-- bid: float (nullable = false)
 |-- bidtime: float (nullable = false)
 |-- bidder: string (nullable = true)
 |-- bidderrate: integer (nullable = false)
 |-- openbid: float (nullable = false)
 |-- price: float (nullable = false)

scala> df.show
+----------+------+----------+-----------------+----------+-------+------+
| auctionid|   bid|   bidtime|           bidder|bidderrate|openbid| price|
+----------+------+----------+-----------------+----------+-------+------+
|1638843936| 500.0|0.47836804|        kona-java|       181|  500.0|1625.0|
|1638843936| 800.0| 0.8263889|           doc213|        60|  500.0|1625.0|
|1638843936| 600.0| 3.7611227|             zmxu|         7|  500.0|1625.0|
|1638843936|1500.0| 5.2263775|      carloss8055|         5|  500.0|1625.0|
|1638843936|1600.0|  6.570625|          jdrinaz|         6|  500.0|1625.0|
|1638843936|1550.0| 6.8929167|      carloss8055|         5|  500.0|1625.0|
|1638843936|1625.0| 6.8931136|      carloss8055|         5|  500.0|1625.0|
|1638844284| 225.0|  1.237419|dre_313@yahoo.com|         0|  200.0| 500.0|
|1638844284| 500.0| 1.2524074|        njbirdmom|        33|  200.0| 500.0|
|1638844464| 300.0| 1.8111342|          aprefer|        58|  300.0| 740.0|
|1638844464| 305.0| 3.2126737|        19750926o|         3|  300.0| 740.0|
|1638844464| 450.0| 4.1657987|         coharley|        30|  300.0| 740.0|
|1638844464| 450.0| 6.7363195|        adammurry|         5|  300.0| 740.0|
|1638844464| 500.0| 6.7364697|        adammurry|         5|  300.0| 740.0|
|1638844464|505.78| 6.9881945|        19750926o|         3|  300.0| 740.0|
|1638844464| 551.0| 6.9896526|        19750926o|         3|  300.0| 740.0|
|1638844464| 570.0| 6.9931483|        19750926o|         3|  300.0| 740.0|
|1638844464| 601.0| 6.9939003|        19750926o|         3|  300.0| 740.0|
|1638844464| 610.0|  6.994965|        19750926o|         3|  300.0| 740.0|
|1638844464| 560.0| 6.9953704|            ps138|         5|  300.0| 740.0|
+----------+------+----------+-----------------+----------+-------+------+
only showing top 20 rows
```

#### Creating DataFrame from CSV files using spark-csv module {#_creating_dataframe_from_csv_files_using_spark_csv_module}

您将使用spark-csv模块从处理正确解析和加载的CSV数据源加载数据。

| Note | 默认情况下，在Spark 2.0.0中支持CSV数据源。无需外部模块。 |
| :---: | :--- |


使用--packages选项启动Spark shell，如下所示：

```
➜  spark git:(master) ✗ ./bin/spark-shell --packages com.databricks:spark-csv_2.11:1.2.0
Ivy Default Cache set to: /Users/jacek/.ivy2/cache
The jars for the packages stored in: /Users/jacek/.ivy2/jars
:: loading settings :: url = jar:file:/Users/jacek/dev/oss/spark/assembly/target/scala-2.11/spark-assembly-1.5.0-SNAPSHOT-hadoop2.7.1.jar!/org/apache/ivy/core/settings/ivysettings.xml
com.databricks#spark-csv_2.11 added as a dependency

scala> val df = spark.read.format("com.databricks.spark.csv").option("header", "true").load("Cartier+for+WinnersCurse.csv")
df: org.apache.spark.sql.DataFrame = [auctionid: string, bid: string, bidtime: string, bidder: string, bidderrate: string, openbid: string, price: string]

scala> df.printSchema
root
 |-- auctionid: string (nullable = true)
 |-- bid: string (nullable = true)
 |-- bidtime: string (nullable = true)
 |-- bidder: string (nullable = true)
 |-- bidderrate: string (nullable = true)
 |-- openbid: string (nullable = true)
 |-- price: string (nullable = true)

 scala> df.show
 +----------+------+-----------+-----------------+----------+-------+-----+
 | auctionid|   bid|    bidtime|           bidder|bidderrate|openbid|price|
 +----------+------+-----------+-----------------+----------+-------+-----+
 |1638843936|   500|0.478368056|        kona-java|       181|    500| 1625|
 |1638843936|   800|0.826388889|           doc213|        60|    500| 1625|
 |1638843936|   600|3.761122685|             zmxu|         7|    500| 1625|
 |1638843936|  1500|5.226377315|      carloss8055|         5|    500| 1625|
 |1638843936|  1600|   6.570625|          jdrinaz|         6|    500| 1625|
 |1638843936|  1550|6.892916667|      carloss8055|         5|    500| 1625|
 |1638843936|  1625|6.893113426|      carloss8055|         5|    500| 1625|
 |1638844284|   225|1.237418982|dre_313@yahoo.com|         0|    200|  500|
 |1638844284|   500|1.252407407|        njbirdmom|        33|    200|  500|
 |1638844464|   300|1.811134259|          aprefer|        58|    300|  740|
 |1638844464|   305|3.212673611|        19750926o|         3|    300|  740|
 |1638844464|   450|4.165798611|         coharley|        30|    300|  740|
 |1638844464|   450|6.736319444|        adammurry|         5|    300|  740|
 |1638844464|   500|6.736469907|        adammurry|         5|    300|  740|
 |1638844464|505.78|6.988194444|        19750926o|         3|    300|  740|
 |1638844464|   551|6.989652778|        19750926o|         3|    300|  740|
 |1638844464|   570|6.993148148|        19750926o|         3|    300|  740|
 |1638844464|   601|6.993900463|        19750926o|         3|    300|  740|
 |1638844464|   610|6.994965278|        19750926o|         3|    300|  740|
 |1638844464|   560| 6.99537037|            ps138|         5|    300|  740|
 +----------+------+-----------+-----------------+----------+-------+-----+
 only showing top 20 rows
```

#### Reading Data from External Data Sources \(read method\) {#__a_id_read_a_reading_data_from_external_data_sources_read_method}

您可以通过使用SQLContext.read方法从结构化文件（JSON，Parquet，CSV），RDD，Hive中的表或外部数据库（JDBC）加载数据来创建DataFrames。

```
read: DataFrameReader
```

read返回一个DataFrameReader实例。

支持的结构化数据（文件）格式包括（请参阅为DataFrameReader指定数据格式（格式方法））：

* JSON

* parquet

* JDBC

* ORC

* Tables in Hive and any JDBC-compliant database

* libsvm

```
val reader = spark.read
r: org.apache.spark.sql.DataFrameReader = org.apache.spark.sql.DataFrameReader@59e67a18

reader.parquet("file.parquet")
reader.json("file.json")
reader.format("libsvm").load("sample_libsvm_data.txt")
```

### Querying DataFrame {#_querying_dataframe}

| Note | Spark SQL提供了一个类似Pandas的查询DSL。 |
| :---: | :--- |


#### Using Query DSL {#__a_id_query_using_dsl_a_using_query_dsl}

您可以使用select方法选择特定列。

| Note | 此变体（其中使用字符串列名）只能选择现有列，即不能使用select表达式创建新列。 |
| :---: | :--- |


```
scala> predictions.printSchema
root
 |-- id: long (nullable = false)
 |-- topic: string (nullable = true)
 |-- text: string (nullable = true)
 |-- label: double (nullable = true)
 |-- words: array (nullable = true)
 |    |-- element: string (containsNull = true)
 |-- features: vector (nullable = true)
 |-- rawPrediction: vector (nullable = true)
 |-- probability: vector (nullable = true)
 |-- prediction: double (nullable = true)

scala> predictions.select("label", "words").show
+-----+-------------------+
|label|              words|
+-----+-------------------+
|  1.0|     [hello, math!]|
|  0.0| [hello, religion!]|
|  1.0|[hello, phy, ic, !]|
+-----+-------------------+
```

```
scala> auctions.groupBy("bidder").count().show(5)
+--------------------+-----+
|              bidder|count|
+--------------------+-----+
|    dennisthemenace1|    1|
|            amskymom|    5|
| nguyenat@san.rr.com|    4|
|           millyjohn|    1|
|ykelectro@hotmail...|    2|
+--------------------+-----+
only showing top 5 rows
```

在下面的示例中，您将查询前5个最活跃的出价工具。

注意微小的$和desc与列名称一起排序行。

```
scala> auctions.groupBy("bidder").count().sort($"count".desc).show(5)
+------------+-----+
|      bidder|count|
+------------+-----+
|    lass1004|   22|
|  pascal1666|   19|
|     freembd|   17|
|restdynamics|   17|
|   happyrova|   17|
+------------+-----+
only showing top 5 rows

scala> import org.apache.spark.sql.functions._
import org.apache.spark.sql.functions._

scala> auctions.groupBy("bidder").count().sort(desc("count")).show(5)
+------------+-----+
|      bidder|count|
+------------+-----+
|    lass1004|   22|
|  pascal1666|   19|
|     freembd|   17|
|restdynamics|   17|
|   happyrova|   17|
+------------+-----+
only showing top 5 rows
```

```
scala> df.select("auctionid").distinct.count
res88: Long = 97

scala> df.groupBy("bidder").count.show
+--------------------+-----+
|              bidder|count|
+--------------------+-----+
|    dennisthemenace1|    1|
|            amskymom|    5|
| nguyenat@san.rr.com|    4|
|           millyjohn|    1|
|ykelectro@hotmail...|    2|
|   shetellia@aol.com|    1|
|              rrolex|    1|
|            bupper99|    2|
|           cheddaboy|    2|
|             adcc007|    1|
|           varvara_b|    1|
|            yokarine|    4|
|          steven1328|    1|
|              anjara|    2|
|              roysco|    1|
|lennonjasonmia@ne...|    2|
|northwestportland...|    4|
|             bosspad|   10|
|        31strawberry|    6|
|          nana-tyler|   11|
+--------------------+-----+
only showing top 20 rows
```

#### Using SQL {#__a_id_query_using_sql_a_a_id_registertemptable_a_using_sql}

将DataFrame注册为命名临时表以运行SQL。

```
scala> df.registerTempTable("auctions") (1)

scala> val sql = spark.sql("SELECT count(*) AS count FROM auctions")
sql: org.apache.spark.sql.DataFrame = [count: bigint]
```

注册临时表，使SQL查询有意义

您可以使用sql操作在DataFrame上执行SQL查询，但在执行查询之前，它会由Catalyst查询优化程序进行优化。您可以使用explain操作打印DataFrame的物理计划。

```
scala> sql.explain
== Physical Plan ==
TungstenAggregate(key=[], functions=[(count(1),mode=Final,isDistinct=false)], output=[count#148L])
 TungstenExchange SinglePartition
  TungstenAggregate(key=[], functions=[(count(1),mode=Partial,isDistinct=false)], output=[currentCount#156L])
   TungstenProject
    Scan PhysicalRDD[auctionid#49,bid#50,bidtime#51,bidder#52,bidderrate#53,openbid#54,price#55]

scala> sql.show
+-----+
|count|
+-----+
| 1348|
+-----+

scala> val count = sql.collect()(0).getLong(0)
count: Long = 1348
```

### Filtering {#__a_id_filter_a_filtering}

```
scala> df.show
+----+---------+-----+
|name|productId|score|
+----+---------+-----+
| aaa|      100| 0.12|
| aaa|      200| 0.29|
| bbb|      200| 0.53|
| bbb|      300| 0.42|
+----+---------+-----+

scala> df.filter($"name".like("a%")).show
+----+---------+-----+
|name|productId|score|
+----+---------+-----+
| aaa|      100| 0.12|
| aaa|      200| 0.29|
+----+---------+-----+
```

### Handling data in Avro format {#_handling_data_in_avro_format}

使用自定义序列化器使用spark-avro。

运行Spark shell与--packages com.databricks：spark-avro\_2.11：2.0.0（见2.0.0工件不在任何公共maven仓库为什么--repositories是必需的）。

```
./bin/spark-shell --packages com.databricks:spark-avro_2.11:2.0.0 --repositories "http://dl.bintray.com/databricks/maven"
```

然后...

```
val fileRdd = sc.textFile("README.md")
val df = fileRdd.toDF

import org.apache.spark.sql.SaveMode

val outputF = "test.avro"
df.write.mode(SaveMode.Append).format("com.databricks.spark.avro").save(outputF)
```

参见org.apache.spark.sql.SaveMode（或者从Scala的角度来看org.apache.spark.sql.SaveMode）。

```
val df = spark.read.format("com.databricks.spark.avro").load("test.avro")
```

### Example Datasets {#_example_datasets}

* [eBay online auctions](http://www.modelingonlineauctions.com/datasets)

* [SFPD Crime Incident Reporting system](https://data.sfgov.org/Public-Safety/SFPD-Incidents-from-1-January-2003/tmnf-yvry)





