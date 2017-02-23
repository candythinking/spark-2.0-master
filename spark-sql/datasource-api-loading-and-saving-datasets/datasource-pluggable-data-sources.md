## DataSource — Pluggable Data Sources {#__a_id_datasource_a_datasource_pluggable_data_sources}

DataSource属于数据源API（以及用于加载数据集的DataFrameReader，用于保存数据集的DataFrameWriter和用于创建流源的StreamSourceProvider）。

DataSource是一个内部类，它表示Spark SQL中的可插拔数据源，具有少量扩展点，以进一步丰富Spark SQL的功能。

Table 1.`DataSource`'s Extension Points

| Extension Point | Description |
| :--- | :--- |
| StreamSourceProvider | Used in:                                                                                                       1.sourceSchemaand createSource for streamed reading         2.createSink for streamed writing                                                 3.resolveRelation for resolved BaseRelation. |
| `FileFormat` | Used in:                                                                                                    1.sourceSchema for streamed reading                                          2.write for writing a`DataFrame`to a`DataSource`\(as part of creating a table as select\) |
| `CreatableRelationProvider` | Used in write for writing a`DataFrame`to a`DataSource`\(as part of creating a table as select\). |

作为用户，您通过DataFrameReader（当执行spark.read或spark.readStream时）或CREATE TABLE USING DDL与DataSource交互。

```
// Batch reading
val people: DataFrame = spark.read
  .format("csv")
  .load("people.csv")

// Streamed reading
val messages: DataFrame = spark.readStream
  .format("kafka")
  .option("subscribe", "topic")
  .option("kafka.bootstrap.servers", "localhost:9092")
  .load
```

DataSource使用SparkSession，类名，路径集合，可选的用户指定模式，分区列集合，存储桶规范和配置选项。

### `createSource`Method {#__a_id_createsource_a_code_createsource_code_method}

```
createSource(metadataPath: String): Source
```

| Caution | FIXME |
| :--- | :--- |


### `createSink`Method {#__a_id_createsink_a_code_createsink_code_method}

| Caution | FIXME |
| :--- | :--- |


### Creating`DataSource`Instance {#__a_id_creating_instance_a_creating_code_datasource_code_instance}

```
class DataSource(
  sparkSession: SparkSession,
  className: String,
  paths: Seq[String] = Nil,
  userSpecifiedSchema: Option[StructType] = None,
  partitionColumns: Seq[String] = Seq.empty,
  bucketSpec: Option[BucketSpec] = None,
  options: Map[String, String] = Map.empty,
  catalogTable: Option[CatalogTable] = None)
```

创建时，DataSource首先查找提供类className（考虑它是别名或完全限定类名），并计算数据源的名称和模式。

| Note | DataSource根据需要懒惰地初始化并且只有一次。 |
| :---: | :--- |


#### `sourceSchema`Internal Method {#__a_id_sourceschema_a_code_sourceschema_code_internal_method}

```
sourceSchema(): SourceInfo
```

sourceSchema返回流式读取的数据源的名称和模式。

| Caution | 为什么要调用该方法？为什么这会打扰流读取和数据源？ |
| :---: | :--- |


它支持两个类层次结构，即StreamSourceProvider和FileFormat数据源。

在内部，sourceSchema首先创建数据源的实例，然后...

对于StreamSourceProvider数据源，sourceSchema中继对StreamSourceProvider.sourceSchema的调用。

对于FileFormat数据源，sourceSchema确保指定了路径选项。

| Tip | path以不区分大小写的方式查找，因此paTh和PATH和pAtH都可以接受。使用path的小写版本。 |
| :---: | :--- |


| Note | path可以使用glob模式（不是regex语法），即包含{} \[\] \*？\字符中的任何一个。 |
| :---: | :--- |


如果不使用glob模式，它检查路径是否存在。如果它不存在，您将在日志中看到以下AnalysisException异常：

```
scala> spark.read.load("the.file.does.not.exist.parquet")
org.apache.spark.sql.AnalysisException: Path does not exist: file:/Users/jacek/dev/oss/spark/the.file.does.not.exist.parquet;
  at org.apache.spark.sql.execution.datasources.DataSource$$anonfun$12.apply(DataSource.scala:375)
  at org.apache.spark.sql.execution.datasources.DataSource$$anonfun$12.apply(DataSource.scala:364)
  at scala.collection.TraversableLike$$anonfun$flatMap$1.apply(TraversableLike.scala:241)
  at scala.collection.TraversableLike$$anonfun$flatMap$1.apply(TraversableLike.scala:241)
  at scala.collection.immutable.List.foreach(List.scala:381)
  at scala.collection.TraversableLike$class.flatMap(TraversableLike.scala:241)
  at scala.collection.immutable.List.flatMap(List.scala:344)
  at org.apache.spark.sql.execution.datasources.DataSource.resolveRelation(DataSource.scala:364)
  at org.apache.spark.sql.DataFrameReader.load(DataFrameReader.scala:149)
  at org.apache.spark.sql.DataFrameReader.load(DataFrameReader.scala:132)
  ... 48 elided
```

如果spark.sql.streaming.schemaInference被禁用，并且数据源与TextFileFormat不同，并且未指定输入userSpecifiedSchema，则会抛出以下IllegalArgumentException异常：

```
Schema must be specified when creating a streaming source DataFrame. If some files already exist in the directory, then depending on the file format you may be able to create a static DataFrame on that directory with 'spark.read.load(directory)' and infer schema from it.
```

| Caution | 我不认为非流动源会发生异常，因为模式将被更早定义。什么时候？ |
| :---: | :--- |


最终，它返回一个SourceInfo与FileSource \[path\]和schema（使用inferFileFormatSchema内部方法计算）。

对于任何其他数据源，它会抛出UnsupportedOperationException异常：

```
Data source [className] does not support streamed reading
```

#### `inferFileFormatSchema`Internal Method {#__a_id_inferfileformatschema_a_code_inferfileformatschema_code_internal_method}

```
inferFileFormatSchema(format: FileFormat): StructType
```

inferFileFormatSchema私有方法计算（也称为）模式（作为StructType）。如果指定或返回userSpecifiedSchema或使用FileFormat.inferSchema。当无法推断模式时抛出AnalysisException。

它使用路径选项用于目录路径列表。

| Note | 当处理FileFormat时，它由DataSource.sourceSchema和DataSource.createSource使用。 |
| :---: | :--- |


### `write`Method {#__a_id_write_a_code_write_code_method}

```
write(
  mode: SaveMode,
  data: DataFrame): BaseRelation
```

在内部，写确保CalendarIntervalType不在数据DataFrame的模式中使用，并且当有一个AnalysisException时抛出一个AnalysisException。

write然后查找数据源实现（使用构造函数的className）。

| Note | DataSource实现可以是CreatableRelationProvider或FileFormat类型。 |
| :---: | :--- |


对于FileFormat数据源，写入使用所有路径和路径选项，并确保只有一个。

| Note | 写使用Hadoop的路径访问FileSystem并计算合格的输出路径。 |
| :---: | :--- |


`write`does`PartitioningUtils.validatePartitionColumn`.

| Caution | FIXME What is`PartitioningUtils.validatePartitionColumn`for? |
| :--- | :--- |


When appending to a table, …​FIXME

最后，write（对于FileFormat数据源）准备一个InsertIntoHadoopFsRelationCommand逻辑计划并执行它。

| Caution | FIXME Is`toRdd`a job execution? |
| :--- | :--- |


对于CreatableRelationProvider数据源，会执行CreatableRelationProvider.createRelation。

| Note | `write`is executed when…​ |
| :--- | :--- |


#### `lookupDataSource`Internal Method {#__a_id_lookupdatasource_a_code_lookupdatasource_code_internal_method}

```
lookupDataSource(provider0: String): Class[_]
```

在内部，lookupDataSource首先在类路径中搜索可用的DataSourceRegister提供者（使用Java的ServiceLoader.load方法），以通过短名称（别名）找到所请求的数据源。parquet或kafka。

如果无法通过短名称找到DataSource，lookupDataSource会尝试加载类，给定输入provider0或其变体provider0.DefaultSource（带.DefaultSource后缀）。

| Note | 您可以通过DataFrameWriter.format方法在代码中引用您自己的自定义DataSource，该方法是别名或完全限定类名。 |
| :---: | :--- |


只有一个数据源注册，或者您将看到以下RuntimeException：

```
Multiple sources found for [provider] ([comma-separated class names]), please specify the fully qualified class name.
```

### Creating`BaseRelation`for Reading or Writing — `resolveRelation`Method {#__a_id_resolverelation_a_creating_code_baserelation_code_for_reading_or_writing_code_resolverelation_code_method}

```
resolveRelation(checkFilesExist: Boolean = true): BaseRelation
```

resolveRelation解析（即创建）BaseRelation以读取或写入DataSource。

在内部，resolveRelation创建provideClass的实例（对于DataSource），并根据其类型（即SchemaRelationProvider，RelationProvider或FileFormat）操作。

Table 2.`resolveRelation`and Resolving`BaseRelation`per \(Schema\) Providers

| Provider | Behaviour |
| :--- | :--- |
| `SchemaRelationProvider` | Executes`SchemaRelationProvider.createRelation`with the provided schema. |
| `RelationProvider` | Executes`RelationProvider.createRelation`. |
| `FileFormat` | Creates a HadoopFsRelation. |













