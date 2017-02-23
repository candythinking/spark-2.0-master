## DataFrameWriter {#_dataframewriter}

DataFrameWriter是一个以批处理方式将数据集持久保存到外部存储系统的接口。

| Tip | 读取DataStreamWriter以进行流式写入。 |
| :---: | :--- |


您在Dataset上使用写入方法来访问DataFrameWriter。

```
import org.apache.spark.sql.DataFrameWriter

val nums: Dataset[Long] = ...
val writer: DataFrameWriter[Row] = nums.write
```

DataFrameWriter直接支持许多文件格式，JDBC数据库和插入新格式的接口。它假定parquet为默认数据源，您可以使用spark.sql.sources.default设置或格式方法更改。

```
// see above for writer definition

// Save dataset in Parquet format
writer.save(path = "nums")

// Save dataset in JSON format
writer.format("json").save(path = "nums-json")
```

最后，您触发使用保存方法实际保存数据集的内容。

```
writer.save
```

| Note | 有趣的是，DataFrameWriter实际上是Scala中的类型构造函数。它在其生命周期中保持对源DataFrame的引用（从创建它的那一刻开始）。 |
| :---: | :--- |


### Internal State {#__a_id_internal_state_a_internal_state}

DataFrameWriter使用以下可变属性为insertInto，saveAsTable和save创建正确定义的写入规范：

Table 1. Attributes and Corresponding Setters

| Attribute | Setters |
| :--- | :--- |
| `source` | format |
| `mode` | mode |
| `extraOptions` | option,options,save |
| `partitioningColumns` | partitionBy |
| `bucketColumnNames` | bucketBy |
| `numBuckets` | bucketBy |
| `sortColumnNames`+ | sortBy |

### `saveAsTable`Method {#__a_id_saveastable_a_code_saveastable_code_method}

```
saveAsTable(tableName: String): Unit
```

saveAsTable将DataFrame的内容保存为tableName表。

首先，tableName被解析为内部表标识符。 saveAsTable然后检查表是否存在，并使用保存模式决定要做什么。

saveAsTable为当前会话使用SessionCatalog。

Table 2.`saveAsTable`'s Behaviour per Save Mode

| Does table exist? | Save Mode | Behaviour |
| :--- | :--- | :--- |
| yes | `Ignore` | Do nothing |
| yes | `ErrorIfExists` | Throws a`AnalysisException`exception with`Table [tableIdent] already exists.`error message. |
| _anything_ | _anything_ | It creates a`CatalogTable`and executes the`CreateTable`plan. |

val

 ints = 

0

 to 

9

 toD

```
val ints = 0 to 9 toDF
val options = Map("path" -> "/tmp/ints")
ints.write.options(options).saveAsTable("ints")
sql("show tables").show
```

### Persisting DataFrame — `save`Method {#__a_id_save_a_persisting_dataframe_code_save_code_method}

```
save(): Unit
```

在内部，save首先检查DataFrame是否没有桶。

保存然后创建一个DataSource（对于源）并调用write。

| Note | save直接使用source，partitioningColumns，extraOptions和mode内部属性。它们通过API指定。 |
| :---: | :--- |


### `jdbc`Method {#__a_id_jdbc_a_code_jdbc_code_method}

```
jdbc(url: String, table: String, connectionProperties: Properties): Unit
```

jdbc方法通过JDBC将DataFrame的内容保存到外部数据库表。

您可以使用模式来控制保存模式，即在执行保存时存在外部表时会发生什么。

假设jdbc保存管道没有被partitioned 和 bucketed。

所有选项都由输入connectionProperties覆盖。

所需的选项是：

* 驱动程序，它是JDBC驱动程序的类名（即传递给Spark自己的DriverRegistry.register，后来用于连接（url，properties））。

当表存在并且正在使用覆盖保存模式时，将执行DROP TABLE表。

它创建输入表（使用CREATE TABLE表（模式），其中模式是DataFrame的模式）。

### `bucketBy`Method {#__a_id_bucketby_a_code_bucketby_code_method}

| Caution | FIXME |
| :--- | :--- |


### `partitionBy`Method {#__a_id_partitionby_a_code_partitionby_code_method}

```
partitionBy(colNames: String*): DataFrameWriter[T]
```

| Caution | FIXME |
| :--- | :--- |


### Specifying Save Mode — `mode`Method {#__a_id_mode_a_specifying_save_mode_code_mode_code_method}

```
mode(saveMode: String): DataFrameWriter[T]
mode(saveMode: SaveMode): DataFrameWriter[T]
```

您可以控制写使用模式方法的行为，即在执行保存时存在外部文件或表时会发生什么。

* `SaveMode.Ignore`or

* `SaveMode.ErrorIfExists`or

* `SaveMode.Overwrite`or

### Writer Configuration — `option`and`options`Methods {#__a_id_option_a_a_id_options_a_writer_configuration_code_option_code_and_code_options_code_methods}

| Caution | FIXME |
| :--- | :--- |


### Writing DataFrames to Files {#__a_id_writing_dataframes_to_files_a_writing_dataframes_to_files}

| Caution | FIXME |
| :--- | :--- |


### Specifying Alias or Fully-Qualified Class Name of DataSource — `format`Method {#__a_id_format_a_specifying_alias_or_fully_qualified_class_name_of_datasource_code_format_code_method}

| Caution | FIXME Compare to DataFrameReader. |
| :--- | :--- |


### Parquet {#__a_id_parquet_a_parquet}

| Caution | FIXME |
| :--- | :--- |


| Note | Parquet is the default data source format. |
| :--- | :--- |


### `insertInto`Method {#__a_id_insertinto_a_code_insertinto_code_method}

| Caution | FIXME |
| :--- | :--- |




















