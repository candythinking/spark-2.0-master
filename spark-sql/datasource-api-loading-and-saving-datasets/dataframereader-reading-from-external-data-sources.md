## `DataFrameReader` — Reading from External Data Sources {#__code_dataframereader_code_reading_from_external_data_sources}

DataFrameReader是从外部数据源读取数据的接口，例如文件，Hive表或JDBC（包括Spark Thrift Server）转换为DataFrame。

| Note | 您可以定义自己的自定义文件格式。 |
| :---: | :--- |


您使用SparkSession.read访问DataFrameReader的实例。

```
val spark: SparkSession = SparkSession.builder.getOrCreate

import org.apache.spark.sql.DataFrameReader
val reader: DataFrameReader = spark.read
```

DataFrameReader supports many file formats and interface for new ones.

| Note | DataFrameReader默认为parquet文件格式，您可以使用spark.sql.sources.default设置更改。 |
| :---: | :--- |


从Spark 2.0开始，DataFrameReader可以使用返回Dataset \[String\]（而不是DataFrames，它是Dataset \[Row\]，因此无类型）的textFile方法读取文本文件。

在使用DataFrameReader的方法描述外部数据源之后，可以使用加载方法之一触发加载。

```
spark.read
  .format("csv")
  .option("header", true)
  .option("inferSchema", true)
  .load("*.csv")
```

### Specifying Data Format — `format`method {#__a_id_format_a_specifying_data_format_code_format_code_method}

```
format(source: String): DataFrameReader
```

Supported data formats:

* `json`

* `csv`\(since **2.0.0**\)

* parquet\(see Parquet\)

* `orc`

* `text`

* `jdbc`

* `libsvm` — only when used in`format("libsvm")`

| Note | 您可以通过使用JDBC和PostgreSQL从表创建DataFrames的练习来提高对格式（“jdbc”）的理解。 |
| :---: | :--- |


### Specifying Input Schema — `schema`method {#__a_id_schema_a_specifying_input_schema_code_schema_code_method}

```
schema(schema: StructType): DataFrameReader
```

您可以指定输入数据源的模式。

| Tip | Refer to [Schema](https://candythinking.gitbooks.io/spark-master-2-0/content/spark-sql/schema-structure-of-data.html). |
| :--- | :--- |


### Adding Extra Configuration Options — `option`and`options`methods {#__a_id_option_a_a_id_options_a_adding_extra_configuration_options_code_option_code_and_code_options_code_methods}

```
option(key: String, value: String): DataFrameReader
option(key: String, value: Boolean): DataFrameReader  (1)
option(key: String, value: Long): DataFrameReader     (1)
option(key: String, value: Double): DataFrameReader   (1)
```

从Spark 2.0.0起可用。

您还可以使用options方法在单个Map中描述不同的选项。

```
options(options: scala.collection.Map[String, String]): DataFrameReader
```

### Loading Datasets \(into`DataFrame`\) — `load`methods {#__a_id_load_a_loading_datasets_into_code_dataframe_code_code_load_code_methods}

```
load(): DataFrame
load(path: String): DataFrame
load(paths: String*): DataFrame
```

`load`将输入数据作为DataFrame加载。

在内部，load创建一个DataSource（用于当前SparkSession，用户指定的模式，源格式和选项）。然后它立即解析它并将BaseRelation转换为DataFrame。

### Creating DataFrames from Files {#__a_id_creating_dataframes_from_files_a_creating_dataframes_from_files}

`DataFrameReader`supports the following file formats:

* JSON

* CSV

* parquet

* ORC

* text

#### `json`method {#__a_id_json_a_code_json_code_method}

```
json(path: String): DataFrame
json(paths: String*): DataFrame
json(jsonRDD: RDD[String]): DataFrame
```

2.0.0中的新功能：prefersDecimal

#### `csv`method {#__a_id_csv_a_code_csv_code_method}

```
csv(path: String): DataFrame
csv(paths: String*): DataFrame
```

#### `parquet`method {#__a_id_parquet_a_code_parquet_code_method}

```
parquet(path: String): DataFrame
parquet(paths: String*): DataFrame
```

The supported options:

* compression\(default:`snappy`\)

New in **2.0.0**:`snappy`is the default Parquet codec. See [\[SPARK-14482\]\[SQL\] Change default Parquet codec from gzip to snappy](https://github.com/apache/spark/commit/2f0b882e5c8787b09bedcc8208e6dcc5662dbbab).

The compressions supported:

* `none`or`uncompressed`

* `snappy`- the default codec in Spark **2.0.0**.

* `gzip`- the default codec in Spark before **2.0.0**

* `lzo`

```
val tokens = Seq("hello", "henry", "and", "harry")
  .zipWithIndex
  .map(_.swap)
  .toDF("id", "token")

val parquetWriter = tokens.write
parquetWriter.option("compression", "none").save("hello-none")

// The exception is mostly for my learning purposes
// so I know where and how to find the trace to the compressions
// Sorry...
scala> parquetWriter.option("compression", "unsupported").save("hello-unsupported")
java.lang.IllegalArgumentException: Codec [unsupported] is not available. Available codecs are uncompressed, gzip, lzo, snappy, none.
  at org.apache.spark.sql.execution.datasources.parquet.ParquetOptions.<init>(ParquetOptions.scala:43)
  at org.apache.spark.sql.execution.datasources.parquet.DefaultSource.prepareWrite(ParquetRelation.scala:77)
  at org.apache.spark.sql.execution.datasources.InsertIntoHadoopFsRelation$$anonfun$run$1$$anonfun$4.apply(InsertIntoHadoopFsRelation.scala:122)
  at org.apache.spark.sql.execution.datasources.InsertIntoHadoopFsRelation$$anonfun$run$1$$anonfun$4.apply(InsertIntoHadoopFsRelation.scala:122)
  at org.apache.spark.sql.execution.datasources.BaseWriterContainer.driverSideSetup(WriterContainer.scala:103)
  at org.apache.spark.sql.execution.datasources.InsertIntoHadoopFsRelation$$anonfun$run$1.apply$mcV$sp(InsertIntoHadoopFsRelation.scala:141)
  at org.apache.spark.sql.execution.datasources.InsertIntoHadoopFsRelation$$anonfun$run$1.apply(InsertIntoHadoopFsRelation.scala:116)
  at org.apache.spark.sql.execution.datasources.InsertIntoHadoopFsRelation$$anonfun$run$1.apply(InsertIntoHadoopFsRelation.scala:116)
  at org.apache.spark.sql.execution.SQLExecution$.withNewExecutionId(SQLExecution.scala:53)
  at org.apache.spark.sql.execution.datasources.InsertIntoHadoopFsRelation.run(InsertIntoHadoopFsRelation.scala:116)
  at org.apache.spark.sql.execution.command.ExecutedCommand.sideEffectResult$lzycompute(commands.scala:61)
  at org.apache.spark.sql.execution.command.ExecutedCommand.sideEffectResult(commands.scala:59)
  at org.apache.spark.sql.execution.command.ExecutedCommand.doExecute(commands.scala:73)
  at org.apache.spark.sql.execution.SparkPlan$$anonfun$execute$1.apply(SparkPlan.scala:118)
  at org.apache.spark.sql.execution.SparkPlan$$anonfun$execute$1.apply(SparkPlan.scala:118)
  at org.apache.spark.sql.execution.SparkPlan$$anonfun$executeQuery$1.apply(SparkPlan.scala:137)
  at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:151)
  at org.apache.spark.sql.execution.SparkPlan.executeQuery(SparkPlan.scala:134)
  at org.apache.spark.sql.execution.SparkPlan.execute(SparkPlan.scala:117)
  at org.apache.spark.sql.execution.QueryExecution.toRdd$lzycompute(QueryExecution.scala:65)
  at org.apache.spark.sql.execution.QueryExecution.toRdd(QueryExecution.scala:65)
  at org.apache.spark.sql.execution.datasources.DataSource.write(DataSource.scala:390)
  at org.apache.spark.sql.DataFrameWriter.save(DataFrameWriter.scala:247)
  at org.apache.spark.sql.DataFrameWriter.save(DataFrameWriter.scala:230)
  ... 48 elided
```

#### `orc`method {#__a_id_orc_a_code_orc_code_method}

```
orc(path: String): DataFrame
orc(paths: String*): DataFrame
```

优化行列**Optimized Row Columnar**（ORC）文件格式是一种高效的列式格式，可存储具有超过1,000列的Hive数据，并提高性能。在Hive版本0.11中引入了ORC格式以使用和保留表定义中的类型信息。

| Tip | Read[ORC Files](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC)document to learn about the ORC file format. |
| :--- | :--- |


#### `text`method {#__a_id_text_a_code_text_code_method}

`text`method loads a text file.

```
text(path: String): DataFrame
text(paths: String*): DataFrame
```

##### Example {#__a_id_text_example_a_example}

```
val lines: Dataset[String] = spark.read.text("README.md").as[String]

scala> lines.show
+--------------------+
|               value|
+--------------------+
|      # Apache Spark|
|                    |
|Spark is a fast a...|
|high-level APIs i...|
|supports general ...|
|rich set of highe...|
|MLlib for machine...|
|and Spark Streami...|
|                    |
|<http://spark.apa...|
|                    |
|                    |
|## Online Documen...|
|                    |
|You can find the ...|
|guide, on the [pr...|
|and [project wiki...|
|This README file ...|
|                    |
|   ## Building Spark|
+--------------------+
only showing top 20 rows
```

### Creating DataFrames from Tables {#__a_id_creating_dataframes_from_tables_a_creating_dataframes_from_tables}

#### `table`method {#__a_id_table_a_code_table_code_method}

```
table(tableName: String): DataFrame
```

table方法将tableName表作为DataFrame返回。

```
scala> spark.sql("SHOW TABLES").show(false)
+---------+-----------+
|tableName|isTemporary|
+---------+-----------+
|dafa     |false      |
+---------+-----------+

scala> spark.read.table("dafa").show(false)
+---+-------+
|id |text   |
+---+-------+
|1  |swiecie|
|0  |hello  |
+---+-------+
```

| Note | 该方法使用spark.sessionState.sqlParser.parseTableIdentifier（tableName）和spark.sessionState.catalog.lookupRelation。很高兴学习一点他们的内部，嗯？ |
| :---: | :--- |


#### Accessing JDBC Data Sources — `jdbc`method {#__a_id_jdbc_a_accessing_jdbc_data_sources_code_jdbc_code_method}

| Note | jdbc方法使用java.util.Properties（并显示为以Java为中心）。请改用format（“jdbc”）。 |
| :---: | :--- |


```
jdbc(url: String, table: String, properties: Properties): DataFrame
jdbc(url: String, table: String,
  parts: Array[Partition],
  connectionProperties: Properties): DataFrame
jdbc(url: String, table: String,
  predicates: Array[String],
  connectionProperties: Properties): DataFrame
jdbc(url: String, table: String,
  columnName: String,
  lowerBound: Long,
  upperBound: Long,
  numPartitions: Int,
  connectionProperties: Properties): DataFrame
```

jdbc允许您创建表示数据库中可用作url的表的DataFrame。

### Reading Text Files — `textFile`methods {#__a_id_textfile_a_reading_text_files_code_textfile_code_methods}

```
textFile(path: String): Dataset[String]
textFile(paths: String*): Dataset[String]
```

textFile方法将文本文件作为Dataset \[String\]进行查询。

```
spark.read.textFile("README.md")
```

| Note | textFile类似于文本系列方法，因为它们都读取文本文件，但文本方法返回无类型的DataFrame，而textFile返回类型Dataset \[String\]。 |
| :---: | :--- |


在内部，textFile将调用传递给文本方法，并在应用Encoders.STRING编码器之前选择唯一的值列。













