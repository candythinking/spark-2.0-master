## Datasets — Strongly-Typed DataFrames with Encoders {#_datasets_strongly_typed_dataframes_with_encoders}

Dataset是Spark SQL的强类型结构化查询，用于通过编码器处理半结构化数据和结构化数据，即具有已知模式的记录。

![](/img/mastering-apache-spark/spark sql/figure1.png)

| Note | 给定上面的图片，可以说数据集是编码器和QueryExecution（它又是SparkSession中的LogicalPlan）的元组 |
| :---: | :--- |


Dataset是延迟的，结构化查询表达式只有在调用操作时才会触发。在内部，Dataset表示描述产生数据（对于给定的Spark SQL会话）所需的计算查询的逻辑计划。

Dataset是针对数据存储（如文件，Hive表或JDBC数据库）执行查询表达式的结果。结构化查询表达式可以由SQL查询，基于列的SQL表达式或Scala / Java lambda函数描述。这就是为什么Dataset操作有三种变体的原因。

```
scala> val dataset = (0 to 4).toDS
dataset: org.apache.spark.sql.Dataset[Int] = [value: int]

// Variant 1: filter operator accepts a Scala function
dataset.filter(n => n % 2 == 0).count

// Variant 2: filter operator accepts a Column-based SQL expression
dataset.filter('value % 2 === 0).count

// Variant 3: filter operator accepts a SQL query
dataset.filter("value % 2 = 0").count
```

Dataset API提供了声明式和类型安全的运算符，可以提高数据处理的体验（与基于索引或列名称的Rows的DataFrames相比）。

Dataset首先在Apache Spark 1.6.0中被引入作为一个实验功能，并且自身已经成为一个完全支持的API。

从Spark 2.0.0开始，DataFrame - 以前版本的Spark SQL的旗舰数据抽象 - 目前是Dataset \[Row\]的一个类型别名：

```
type DataFrame = Dataset[Row]
```

数据集通过DataFrames的性能优化和Scala的强静态类型安全性提供了RDD的便利性。将强大的类型安全性带到DataFrame的最后一个特点使得Dataset如此吸引人。所有的功能一起给你一个更加功能的编程接口来处理结构化数据。

```
scala> spark.range(1).filter('id === 0).explain(true)
== Parsed Logical Plan ==
'Filter ('id = 0)
+- Range (0, 1, splits=8)

== Analyzed Logical Plan ==
id: bigint
Filter (id#51L = cast(0 as bigint))
+- Range (0, 1, splits=8)

== Optimized Logical Plan ==
Filter (id#51L = 0)
+- Range (0, 1, splits=8)

== Physical Plan ==
*Filter (id#51L = 0)
+- *Range (0, 1, splits=8)

scala> spark.range(1).filter(_ == 0).explain(true)
== Parsed Logical Plan ==
'TypedFilter <function1>, class java.lang.Long, [StructField(value,LongType,true)], unresolveddeserializer(newInstance(class java.lang.Long))
+- Range (0, 1, splits=8)

== Analyzed Logical Plan ==
id: bigint
TypedFilter <function1>, class java.lang.Long, [StructField(value,LongType,true)], newInstance(class java.lang.Long)
+- Range (0, 1, splits=8)

== Optimized Logical Plan ==
TypedFilter <function1>, class java.lang.Long, [StructField(value,LongType,true)], newInstance(class java.lang.Long)
+- Range (0, 1, splits=8)

== Physical Plan ==
*Filter <function1>.apply
+- *Range (0, 1, splits=8)
```

它只有在数据集在编译时进行语法和分析检查（这是不可能使用DataFrame，常规SQL查询或甚至RDDs）。

使用Dataset对象将Row实例的DataFrames转换为具有适当名称和类型的case类的DataFrames（跟在case类中的等价类）。而不是使用索引访问DataFrame中的相应字段并将其转换为类型，所有这些都由Datasets自动处理，并由Scala编译器检查。

Dataset使用Catalyst Query Optimizer和Tungsten来优化查询性能。

一个Dataset对象需要一个SparkSession，一个QueryExecution计划和一个Encoder（用于快速序列化和反序列化从InternalRow）。

然而，如果使用LogicalPlan来创建数据集，则首先执行逻辑计划（使用SparkSession中的当前SessionState），产生QueryExecution计划。

Dataset是可查询和可序列化的，即可以保存到永久存储器。

| Note | SparkSession和QueryExecution是数据集的临时属性，因此不参与Dataset序列化。数据集的唯一牢固的特性是编码器。 |
| :---: | :--- |


您可以在类型安全数据集转换为“无类型”数据帧或访问被执行查询后产生的RDD。它应该给你一个更愉快的体验，同时从您可以在早期版本的星火SQL已经使用或鼓励spark core的RDD API迁移到spark SQL的数据集API基于RDD或遗留的基于数据帧的API过渡。

Datasets的默认存储级别是MEMORY\_AND\_DISK，因为重新计算基础表的内存中柱状表示是昂贵的。但是，您可以持久化一个数据集。

Spark 2.0引入了一个称为Structured Streaming的新的查询模型，用于连续增量执行结构化查询。这使得有可能考虑数据集静态和有界以及流和非绑定数据集与单个统一的API为不同的执行模型。

如果使用SparkSession.emptyDataset或SparkSession.createDataset方法从本地集合创建数据集，则数据集是本地的，并且它们的派生类似toDF。如果是，则可以优化数据集上的查询并在本地运行，即不使用Spark执行程序。

| Note | Dataset具有分析和检查的QueryExecution。 |
| :---: | :--- |


### `queryExecution`Attribute {#__a_id_queryexecution_a_code_queryexecution_code_attribute}

queryExecution是Dataset的必需参数。

```
val dataset: Dataset[Int] = ...
dataset.queryExecution
```

它是Dataset类的Developer API的一部分。

### Creating Datasets {#__a_id_creating_instance_a_creating_datasets}

如果LogicalPlan用于创建一个Dataset，它将被执行（使用当前的SessionState）来创建一个相应的QueryExecution。

### Implicit Type Conversions to Datasets — `toDS`and`toDF`methods {#__a_id_implicits_a_a_id_tods_a_a_id_todf_a_implicit_type_conversions_to_datasets_code_tods_code_and_code_todf_code_methods}

DatasetHolder case类提供了三种从Seq \[T\]或RDD \[T\]类型到数据集\[T\]的转换的方法：

* `toDS(): Dataset[T]`

* `toDF(): DataFrame`

* `toDF(colNames: String*): DataFrame`

| Note | `DataFrame`is a_mere_type alias for`Dataset[Row]`since Spark**2.0.0**. |
| :---: | :--- |


DatasetHolder由SQLImplicits使用，可在导入SparkSession的implicits对象后使用。

```
val spark: SparkSession = ...
import spark.implicits._

scala> val ds = Seq("I am a shiny Dataset!").toDS
ds: org.apache.spark.sql.Dataset[String] = [value: string]

scala> val df = Seq("I am an old grumpy DataFrame!").toDF
df: org.apache.spark.sql.DataFrame = [value: string]

scala> val df = Seq("I am an old grumpy DataFrame!").toDF("text")
df: org.apache.spark.sql.DataFrame = [text: string]

scala> val ds = sc.parallelize(Seq("hello")).toDS
ds: org.apache.spark.sql.Dataset[String] = [value: string]
```

这个implicits对象的值的导入是在Spark Shell中自动执行的，所以你不需要做任何事情，只使用转换。

```
scala> spark.version
res11: String = 2.0.0

scala> :imports
 1) import spark.implicits._  (59 terms, 38 are implicit)
 2) import spark.sql          (1 terms)
```

```
val spark: SparkSession = ...
import spark.implicits._

case class Token(name: String, productId: Int, score: Double)
val data = Seq(
  Token("aaa", 100, 0.12),
  Token("aaa", 200, 0.29),
  Token("bbb", 200, 0.53),
  Token("bbb", 300, 0.42))

// Transform data to a Dataset[Token]
// It doesn't work with type annotation
// https://issues.apache.org/jira/browse/SPARK-13456
val ds = data.toDS

// ds: org.apache.spark.sql.Dataset[Token] = [name: string, productId: int ... 1 more field]

// Transform data into a DataFrame with no explicit schema
val df = data.toDF

// Transform DataFrame into a Dataset
val ds = df.as[Token]

scala> ds.show
+----+---------+-----+
|name|productId|score|
+----+---------+-----+
| aaa|      100| 0.12|
| aaa|      200| 0.29|
| bbb|      200| 0.53|
| bbb|      300| 0.42|
+----+---------+-----+

scala> ds.printSchema
root
 |-- name: string (nullable = true)
 |-- productId: integer (nullable = false)
 |-- score: double (nullable = false)

// In DataFrames we work with Row instances
scala> df.map(_.getClass.getName).show(false)
+--------------------------------------------------------------+
|value                                                         |
+--------------------------------------------------------------+
|org.apache.spark.sql.catalyst.expressions.GenericRowWithSchema|
|org.apache.spark.sql.catalyst.expressions.GenericRowWithSchema|
|org.apache.spark.sql.catalyst.expressions.GenericRowWithSchema|
|org.apache.spark.sql.catalyst.expressions.GenericRowWithSchema|
+--------------------------------------------------------------+

// In Datasets we work with case class instances
scala> ds.map(_.getClass.getName).show(false)
+---------------------------+
|value                      |
+---------------------------+
|$line40.$read$$iw$$iw$Token|
|$line40.$read$$iw$$iw$Token|
|$line40.$read$$iw$$iw$Token|
|$line40.$read$$iw$$iw$Token|
+---------------------------+
```

#### Internals of toDS {#__a_id_tods_internals_a_internals_of_tods}

在内部，Scala编译器使得toDS隐式可用于任何Seq \[T\]（使用SQLImplicits.localSeqToDatasetHolder隐式方法）。

| Note | 每当你导入spark.implicits.\_时，此方法和其他隐式方法都在作用域中。 |
| :---: | :--- |


输入Seq \[T\]通过SQLContext.createDataset转换为Dataset \[T\]，然后将所有调用传递给SparkSession.createDataset。一旦创建，数据集\[T\]被封装在DatasetHolder \[T\]中，并使用只返回输入ds的toDS。

### Queryable {#__a_id_queryable_a_queryable}

| Caution | [FIXME](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/GLOSSARY.html#fixme) |
| :--- | :--- |


### Tracking Multi-Job SQL Query Executions — `withNewExecutionId`Internal Method {#__a_id_withnewexecutionid_a_tracking_multi_job_sql_query_executions_code_withnewexecutionid_code_internal_method}

```
withNewExecutionId[U](body: => U): U
```

withNewExecutionId是一个私有\[sql\]运算符，它使用SQLExecution.withNewExecutionId执行输入正文操作，该操作设置执行id本地属性集。

| Note | It is used in`foreach`,foreachPartition, and \(private\)`collect`. |
| :--- | :--- |


### Creating DataFrame — `ofRows`Internal Method {#__a_id_ofrows_a_creating_dataframe_code_ofrows_code_internal_method}

```
ofRows(sparkSession: SparkSession, logicalPlan: LogicalPlan): DataFrame
```

| Note | ofRows是一个私有的\[sql\]运算符，只能从org.apache.spark.sql包中的代码访问。它不是Dataset的公共API的一部分。 |
| :---: | :--- |


ofRows返回DataFrame（它是Dataset \[Row\]的类型别名）。 ofRows使用RowEncoder转换模式（基于输入logicalPlan逻辑计划）。

在内部，ofRows准备输入logicalPlan以执行，并使用当前SparkSession，QueryExecution和RowEncoder创建一个Dataset \[Row\]。

### Further reading or watching {#__a_id_i_want_more_a_further_reading_or_watching}

* \(video\)[Structuring Spark: DataFrames, Datasets, and Streaming](https://youtu.be/i7l3JQRx7Qw)



























