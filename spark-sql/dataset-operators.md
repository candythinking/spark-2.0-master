## Dataset Operators {#_dataset_operators}

您可以对所有运算符的组进行分组，以便与每个目标的数据集一起使用，即它们应用于的数据集的一部分。

1. Column Operators

2. Standard Functions — `functions`object

3. User-Defined Functions \(UDFs\)

4. Aggregation — Typed and Untyped Grouping

5. `UserDefinedAggregateFunction` — User-Defined Aggregate Functions \(UDAFs\)

6. Window Aggregate Operators -- Windows

7. Joins

8. Caching

除了上面的操作符之外，还有下面的操作符作为一个整体使用一个数据集。

Table 1. Dataset Operators

| Operator | Description |
| :--- | :--- |
| as | Converting a`Dataset`to a`Dataset` |
| coalesce | Repartitioning a`Dataset`with shuffle disabled. |
| createGlobalTempView |  |
| createOrReplaceTempView |  |
| createTempView |  |
| explain | Explain logical and physical plans of a`Dataset` |
| filter |  |
| flatMap |  |
| foreachPartition |  |
| isLocal |  |
| isStreaming |  |
| mapPartition |  |
| randomSplit | Randomly split a`Dataset`into two`Dataset`s |
| rdd |  |
| repartition | Repartitioning a`Dataset`with shuffle enabled. |
| schema |  |
| select |  |
| selectExpr |  |
| show |  |
| take |  |
| toDF | Converts a`Dataset`to a`DataFrame` |
| toJSON |  |
| transform | Transforms a`Dataset` |
| where |  |
| write |  |
| writeStream |  |

### `createTempViewCommand`Internal Operator {#__a_id_createtempviewcommand_a_code_createtempviewcommand_code_internal_operator}

| Caution | FIXME |
| :--- | :--- |


### `createGlobalTempView`Operator {#__a_id_createglobaltempview_a_code_createglobaltempview_code_operator}

| Caution | FIXME |
| :--- | :--- |


### `createOrReplaceTempView`Operator {#__a_id_createorreplacetempview_a_code_createorreplacetempview_code_operator}

| Caution | FIXME |
| :--- | :--- |


### `createTempView`Operator {#__a_id_createtempview_a_code_createtempview_code_operator}

| Caution | FIXME |
| :--- | :--- |


### Transforming Datasets — `transform`Operator {#__a_id_transform_a_transforming_datasets_code_transform_code_operator}

```
transform[U](t: Dataset[T] => Dataset[U]): Dataset[U]
```

transform将t函数应用于源数据集\[T\]以产生结果Dataset \[U\]。它用于链接自定义转换。

```
val dataset = spark.range(5)

// Transformation t
import org.apache.spark.sql.Dataset
def withDoubled(longs: Dataset[java.lang.Long]) = longs.withColumn("doubled", 'id * 2)

scala> dataset.transform(withDoubled).show
+---+-------+
| id|doubled|
+---+-------+
|  0|      0|
|  1|      2|
|  2|      4|
|  3|      6|
|  4|      8|
+---+-------+
```

在内部，transform对当前数据集\[T\]执行t函数。

### Converting to`DataFrame` — `toDF`Methods {#__a_id_todf_a_converting_to_code_dataframe_code_code_todf_code_methods}

```
toDF(): DataFrame
toDF(colNames: String*): DataFrame
```

toDF将数据集转换为DataFrame。

在内部，空参数toDF使用数据集的SparkSession和QueryExecution（编码器为RowEncoder）创建一个Dataset \[Row\]。

| Caution | FIXME Describe`toDF(colNames: String*)` |
| :--- | :--- |


### Converting to`Dataset` — `as`Method {#__a_id_as_a_converting_to_code_dataset_code_code_as_code_method}

| Caution | FIXME |
| :--- | :--- |


### Accessing`DataFrameWriter` — `write`Method {#__a_id_write_a_accessing_code_dataframewriter_code_code_write_code_method}

```
write: DataFrameWriter[T]
```

写方法返回类型T的记录的DataFrameWriter。

```
import org.apache.spark.sql.{DataFrameWriter, Dataset}
val ints: Dataset[Int] = (0 to 5).toDS

val writer: DataFrameWriter[Int] = ints.write
```

### Accessing`DataStreamWriter` — `writeStream`Method {#__a_id_writestream_a_accessing_code_datastreamwriter_code_code_writestream_code_method}

```
writeStream: DataStreamWriter[T]
```

writeStream方法为T类型的记录返回DataStreamWriter。

```
val papers = spark.readStream.text("papers").as[String]

import org.apache.spark.sql.streaming.DataStreamWriter
val writer: DataStreamWriter[String] = papers.writeStream
```

### Display Records — `show`Methods {#__a_id_show_a_display_records_code_show_code_methods}

```
show(): Unit
show(numRows: Int): Unit
show(truncate: Boolean): Unit
show(numRows: Int, truncate: Boolean): Unit
show(numRows: Int, truncate: Int): Unit
```

在内部，显示中继到私人showString做格式化。它将数据集转换为DataFrame（通过调用toDF（）），并获取前n个记录。

### Taking First n Records — `take`Action {#__a_id_take_a_taking_first_n_records_code_take_code_action}

```
take(n: Int): Array[T]
```

take是对返回n个记录的集合的数据集的操作。

| Warning | 将所有数据加载到Spark应用程序的驱动程序进程的内存中，对于大的n可能会导致OutOfMemoryError。 |
| :---: | :--- |


在内部，take为Literal表达式和当前LogicalPlan创建一个具有Limit逻辑计划的新数据集。然后运行SparkPlan产生一个Array \[InternalRow\]，然后使用有界编码器将其解码为Array \[T\]。

### `foreachPartition`Action {#__a_id_foreachpartition_a_code_foreachpartition_code_action}

```
foreachPartition(f: Iterator[T] => Unit): Unit
```

foreachPartition将f函数应用于数据集的每个分区。

```
case class Record(id: Int, city: String)
val ds = Seq(Record(0, "Warsaw"), Record(1, "London")).toDS

ds.foreachPartition { iter: Iterator[Record] => iter.foreach(println) }
```

| Note | foreachPartition用于将DataFrame保存到JDBC表（通过JdbcUtils.saveTable间接保存）和ForeachSink。 |
| :---: | :--- |


### `mapPartitions`Operator {#__a_id_mappartitions_a_code_mappartitions_code_operator}

```
mapPartitions[U: Encoder](func: Iterator[T] => Iterator[U]): Dataset[U]
```

mapPartitions返回一个新的数据集（类型U），函数func应用于每个分区。

| Caution | FIXME Example |
| :--- | :--- |


### Creating Zero or More Records — `flatMap`Operator {#__a_id_flatmap_a_creating_zero_or_more_records_code_flatmap_code_operator}

```
flatMap[U: Encoder](func: T => TraversableOnce[U]): Dataset[U]
```

flatMap返回一个新的数据集（类型U），所有记录（类型T）使用函数func映射，然后展平结果。

| Note | `flatMap`can create new records. It deprecated`explode`. |
| :--- | :--- |


```
final case class Sentence(id: Long, text: String)
val sentences = Seq(Sentence(0, "hello world"), Sentence(1, "witaj swiecie")).toDS

scala> sentences.flatMap(s => s.text.split("\\s+")).show
+-------+
|  value|
+-------+
|  hello|
|  world|
|  witaj|
|swiecie|
+-------+
```

在内部，flatMap使用分区flatMap（ped）调用mapPartition。

### Repartitioning Dataset with Shuffle Disabled — `coalesce`Operator {#__a_id_coalesce_a_repartitioning_dataset_with_shuffle_disabled_code_coalesce_code_operator}

```
coalesce(numPartitions: Int): Dataset[T]
```

`coalesce`合并操作符将数据集重新分配到正确的numPartitions分区。

在内部，coalesce创建一个具有shuffle禁用的Repartition逻辑运算符（在下面的explain输出中被标记为false）。

```
scala> spark.range(5).coalesce(1).explain(extended = true)
== Parsed Logical Plan ==
Repartition 1, false
+- Range (0, 5, step=1, splits=Some(8))

== Analyzed Logical Plan ==
id: bigint
Repartition 1, false
+- Range (0, 5, step=1, splits=Some(8))

== Optimized Logical Plan ==
Repartition 1, false
+- Range (0, 5, step=1, splits=Some(8))

== Physical Plan ==
Coalesce 1
+- *Range (0, 5, step=1, splits=Some(8))
```

### Repartitioning Dataset with Shuffle Enabled — `repartition`Operators {#__a_id_repartition_a_repartitioning_dataset_with_shuffle_enabled_code_repartition_code_operators}

```
repartition(numPartitions: Int): Dataset[T]
repartition(numPartitions: Int, partitionExprs: Column*): Dataset[T]
repartition(partitionExprs: Column*): Dataset[T]
```

`repartition`重分区操作符将`Dataset`重新分配到正确的numPartitions分区或使用partitionExprs表达式。

在内部，重新分区创建一个Repartition或RepartitionByExpression逻辑运算符，分别启用shuffle（在下面的explain输出中被标记为true）。

```
scala> spark.range(5).repartition(1).explain(extended = true)
== Parsed Logical Plan ==
Repartition 1, true
+- Range (0, 5, step=1, splits=Some(8))

== Analyzed Logical Plan ==
id: bigint
Repartition 1, true
+- Range (0, 5, step=1, splits=Some(8))

== Optimized Logical Plan ==
Repartition 1, true
+- Range (0, 5, step=1, splits=Some(8))

== Physical Plan ==
Exchange RoundRobinPartitioning(1)
+- *Range (0, 5, step=1, splits=Some(8))
```

| Note | `repartition`methods correspond to SQL’s`DISTRIBUTE BY`or`CLUSTER BY`. |
| :--- | :--- |


### Projecting Columns — `select`Operators {#__a_id_select_a_projecting_columns_code_select_code_operators}

```
select[U1: Encoder](c1: TypedColumn[T, U1]): Dataset[U1]
select[U1, U2](c1: TypedColumn[T, U1], c2: TypedColumn[T, U2]): Dataset[(U1, U2)]
select[U1, U2, U3](
  c1: TypedColumn[T, U1],
  c2: TypedColumn[T, U2],
  c3: TypedColumn[T, U3]): Dataset[(U1, U2, U3)]
select[U1, U2, U3, U4](
  c1: TypedColumn[T, U1],
  c2: TypedColumn[T, U2],
  c3: TypedColumn[T, U3],
  c4: TypedColumn[T, U4]): Dataset[(U1, U2, U3, U4)]
select[U1, U2, U3, U4, U5](
  c1: TypedColumn[T, U1],
  c2: TypedColumn[T, U2],
  c3: TypedColumn[T, U3],
  c4: TypedColumn[T, U4],
  c5: TypedColumn[T, U5]): Dataset[(U1, U2, U3, U4, U5)]
```

| Caution | FIXME |
| :--- | :--- |


### `filter`Operator {#__a_id_filter_a_code_filter_code_operator}

| Caution | FIXME |
| :--- | :--- |


### `where`Operators {#__a_id_where_a_code_where_code_operators}

```
where(condition: Column): Dataset[T]
where(conditionExpr: String): Dataset[T]
```

`where`是过滤器运算符的同义词，即它简单地将参数传递给过滤器。

### Projecting Columns using Expressions — `selectExpr`Operator {#__a_id_selectexpr_a_projecting_columns_using_expressions_code_selectexpr_code_operator}

```
selectExpr(exprs: String*): DataFrame
```

selectExpr就像select，但是接受SQL表达式exprs。

```
val ds = spark.range(5)

scala> ds.selectExpr("rand() as random").show
16/04/14 23:16:06 INFO HiveSqlParser: Parsing command: rand() as random
+-------------------+
|             random|
+-------------------+
|  0.887675894185651|
|0.36766085091074086|
| 0.2700020856675186|
| 0.1489033635529543|
| 0.5862990791950973|
+-------------------+
```

在内部，它使用映射到Column的exprs中的每个表达式执行select（使用SparkSqlParser.parseExpression）。

```
scala> ds.select(expr("rand() as random")).show
+------------------+
|            random|
+------------------+
|0.5514319279894851|
|0.2876221510433741|
|0.4599999092045741|
|0.5708558868374893|
|0.6223314406247136|
+------------------+
```

| Note | A new feature in Spark **2.0.0**. |
| :--- | :--- |


### Randomly Split Dataset — `randomSplit`Operators {#__a_id_randomsplit_a_randomly_split_dataset_code_randomsplit_code_operators}

```
randomSplit(weights: Array[Double]): Array[Dataset[T]]
randomSplit(weights: Array[Double], seed: Long): Array[Dataset[T]]
```

randomSplit每个权重随机拆分数据集。

`weights`权重双和应该总和为1，并且如果它们不是，则将被标准化。

您可以定义`seed`，如果不这样做，将使用随机`seed`。

| Note | 它用于TrainValidationSplit将数据集拆分成训练和验证数据集。 |
| :---: | :--- |


```
val ds = spark.range(10)
scala> ds.randomSplit(Array[Double](2, 3)).foreach(_.show)
+---+
| id|
+---+
|  0|
|  1|
|  2|
+---+

+---+
| id|
+---+
|  3|
|  4|
|  5|
|  6|
|  7|
|  8|
|  9|
+---+
```

| Note | A new feature in Spark **2.0.0**. |
| :--- | :--- |


### Explaining Logical and Physical Plans — `explain`Operator {#__a_id_explain_a_explaining_logical_and_physical_plans_code_explain_code_operator}

```
explain(): Unit
explain(extended: Boolean): Unit
```

explain将逻辑和（带有扩展启用）物理计划打印到控制台。使用它来审查应用的结构化查询和优化。

| Tip | 如果您认真对待查询调试，还可以使用调试查询执行工具。 |
| :---: | :--- |


在内部，explain执行ExplainCommand logical命令。

```
scala> spark.range(10).explain(extended = true)
== Parsed Logical Plan ==
Range (0, 10, step=1, splits=Some(8))

== Analyzed Logical Plan ==
id: bigint
Range (0, 10, step=1, splits=Some(8))

== Optimized Logical Plan ==
Range (0, 10, step=1, splits=Some(8))

== Physical Plan ==
*Range (0, 10, step=1, splits=Some(8))
```

### `toJSON`method {#__a_id_tojson_a_code_tojson_code_method}

toJSON将Dataset的内容映射到JSON字符串的Dataset。

| Note | A new feature in Spark **2.0.0**. |
| :--- | :--- |


```
scala> val ds = Seq("hello", "world", "foo bar").toDS
ds: org.apache.spark.sql.Dataset[String] = [value: string]

scala> ds.toJSON.show
+-------------------+
|              value|
+-------------------+
|  {"value":"hello"}|
|  {"value":"world"}|
|{"value":"foo bar"}|
+-------------------+
```

在内部，toJSON获取RDD \[InternalRow\]（数据集的QueryExecution），并将记录（每个RDD分区）映射到JSON。

| Note | `toJSON`uses Jackson’s JSON parser — [jackson-module-scala](https://github.com/FasterXML/jackson-module-scala). |
| :--- | :--- |


### Accessing Schema — `schema`Method {#__a_id_schema_a_accessing_schema_code_schema_code_method}

A`Dataset`has a**schema**.

```
schema: StructType
```

You may also use the following methods to learn about the schema:

* `printSchema(): Unit`

* explain

### Converting Dataset into RDD — `rdd`Attribute {#__a_id_rdd_a_converting_dataset_into_rdd_code_rdd_code_attribute}

```
rdd: RDD[T]
```

每当需要将数据集转换为RDD时，执行rdd方法将为您提供位于数据集后面的正确输入对象类型（不是DataFrames中的Row）的RDD。

```
scala> val rdd = tokens.rdd
rdd: org.apache.spark.rdd.RDD[Token] = MapPartitionsRDD[11] at rdd at <console>:30
```

在内部，它查找ExpressionEncoder（用于数据集）并访问解串器表达式。这给出了表达式求值结果的DataType。

| Note | deserializer expression反序列化表达式用于将InternalRow解码为类型T的对象。请参阅ExpressionEncoder。 |
| :---: | :--- |


然后它执行一个DeserializeToObject逻辑运算符，将产生一个RDD \[InternalRow\]，使用DataType和T转换为正确的RDD \[T\]。

| Note | It is a lazy operation that "produces" a`RDD[T]`. |
| :--- | :--- |


### `isStreaming`Method {#__a_id_isstreaming_a_code_isstreaming_code_method}

当数据集包含StreamingRelation或StreamingExecutionRelation流源时，isStreaming返回true。

| Note | 流数据集使用DataFrameReader.stream方法（对于StreamingRelation）创建，并在DataStreamWriter.start之后包含StreamingExecutionRelation。 |
| :---: | :--- |


```
val reader = spark.read
val helloStream = reader.stream("hello")

scala> helloStream.isStreaming
res9: Boolean = true
```

| Note | A new feature in Spark **2.0.0**. |
| :--- | :--- |


### Is Dataset Local — `isLocal`method {#__a_id_islocal_a_is_dataset_local_code_islocal_code_method}

```
isLocal: Boolean
```

isLocal是一个标志，说明是否操作符collect或take可以在本地运行，即没有使用执行器。

在内部，isLocal检查数据集的逻辑查询计划是否为LocalRelation。







