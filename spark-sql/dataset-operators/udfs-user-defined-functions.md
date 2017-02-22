## UDFs — User-Defined Functions {#_udfs_user_defined_functions}

用户定义函数（又叫 UDF）是Spark SQL的一个特性，用于定义新的基于列的函数，扩展Spark SQL的DSL词汇表来转换数据集。

在恢复为使用自己的自定义UDF函数之前，尽可能使用具有Dataset运算符的更高级标准基于列的函数，因为UDF是Spark的黑盒，因此它甚至不尝试优化它们。

雷诺曾经在Spark的dev邮件列表上说：

有一些简单的例子，我们可以分析UDF字节码并推断它在做什么，但是一般来说很难做。

您通过将Scala函数定义为udf函数的输入参数来定义新的UDF。它接受最多10个输入参数的Scala函数。

```
val dataset = Seq((0, "hello"), (1, "world")).toDF("id", "text")

// Define a regular Scala function
val upper: String => String = _.toUpperCase

// Define a UDF that wraps the upper Scala function defined above
// You could also define the function in place, i.e. inside udf
// but separating Scala functions from Spark SQL's UDFs allows for easier testing
import org.apache.spark.sql.functions.udf
val upperUDF = udf(upper)

// Apply the UDF to change the source dataset
scala> dataset.withColumn("upper", upperUDF('text)).show
+---+-----+-----+
| id| text|upper|
+---+-----+-----+
|  0|hello|HELLO|
|  1|world|WORLD|
+---+-----+-----+
```

您可以通过UDFRegistration注册UDF以在基于SQL的查询表达式中使用（通过SparkSession.udf属性可用）。

```
val spark: SparkSession = ...
scala> spark.udf.register("myUpper", (input: String) => input.toUpperCase)
```

您可以使用Catalog接口（通过SparkSession.catalog属性可用）查询可用的标准和用户定义的函数。

```
val spark: SparkSession = ...
scala> spark.catalog.listFunctions.filter('name like "%upper%").show(false)
+-------+--------+-----------+-----------------------------------------------+-----------+
|name   |database|description|className                                      |isTemporary|
+-------+--------+-----------+-----------------------------------------------+-----------+
|myupper|null    |null       |null                                           |true       |
|upper  |null    |null       |org.apache.spark.sql.catalyst.expressions.Upper|true       |
+-------+--------+-----------+-----------------------------------------------+-----------+
```

| Note | UDF在Spark MLlib中起着至关重要的作用，可以定义新的变换器，它们是通过引入新列将DataFrames转换为DataFrames的函数对象。 |
| :---: | :--- |


### udf Functions \(in functions object\) {#__a_id_udf_function_a_udf_functions_in_functions_object}

```
udf[RT: TypeTag](f: Function0[RT]): UserDefinedFunction
...
udf[RT: TypeTag, A1: TypeTag, A2: TypeTag, A3: TypeTag, A4: TypeTag, A5: TypeTag, A6: TypeTag, A7: TypeTag, A8: TypeTag, A9: TypeTag, A10: TypeTag](f: Function10[A1, A2, A3, A4, A5, A6, A7, A8, A9, A10, RT]): UserDefinedFunction
```

org.apache.spark.sql.functions对象附带udf函数，让您为Scala函数f定义UDF。

```
val df = Seq(
  (0, "hello"),
  (1, "world")).toDF("id", "text")

// Define a "regular" Scala function
// It's a clone of upper UDF
val toUpper: String => String = _.toUpperCase

import org.apache.spark.sql.functions.udf
val upper = udf(toUpper)

scala> df.withColumn("upper", upper('text)).show
+---+-----+-----+
| id| text|upper|
+---+-----+-----+
|  0|hello|HELLO|
|  1|world|WORLD|
+---+-----+-----+

// You could have also defined the UDF this way
val upperUDF = udf { s: String => s.toUpperCase }

// or even this way
val upperUDF = udf[String, String](_.toUpperCase)

scala> df.withColumn("upper", upperUDF('text)).show
+---+-----+-----+
| id| text|upper|
+---+-----+-----+
|  0|hello|HELLO|
|  1|world|WORLD|
+---+-----+-----+
```

| Note | 基于“独立”Scala函数（例如toUpperUDF）定义自定义UDF，以便可以使用Scala方法（不使用Spark SQL的“噪声”）测试Scala函数，并且一旦定义它们，就可以在UnaryTransformers中重用UDF。 |
| :---: | :--- |


### UDFs are Blackbox {#__a_id_udfs_are_blackbox_a_udfs_are_blackbox}

让我们回顾一个UDF的例子。此示例仅转换大小为7个字符的字符串，并首先使用Dataset标准运算符，然后使用自定义UDF执行相同的转换。

```
scala> spark.conf.get("spark.sql.parquet.filterPushdown")
res0: String = true
```

您将要使用基于Parquet文件的下列城市数据集（用于Parquet数据源的谓词下推/过滤下推）部分。木地板的原因是它是一个外部数据源，它支持优化Spark使用来优化自己像谓词下推。

```
// no optimization as it is a more involved Scala function in filter
// 08/30 Asked on dev@spark mailing list for explanation
val cities6chars = cities.filter(_.name.length == 6).map(_.name.toUpperCase)

cities6chars.explain(true)

// or simpler when only concerned with PushedFilters attribute in Parquet
scala> cities6chars.queryExecution.optimizedPlan
res33: org.apache.spark.sql.catalyst.plans.logical.LogicalPlan =
SerializeFromObject [staticinvoke(class org.apache.spark.unsafe.types.UTF8String, StringType, fromString, input[0, java.lang.String, true], true) AS value#248]
+- MapElements <function1>, class City, [StructField(id,LongType,false), StructField(name,StringType,true)], obj#247: java.lang.String
   +- Filter <function1>.apply
      +- DeserializeToObject newInstance(class City), obj#246: City
         +- Relation[id#236L,name#237] parquet

// no optimization for Dataset[City]?!
// 08/30 Asked on dev@spark mailing list for explanation
val cities6chars = cities.filter(_.name == "Warsaw").map(_.name.toUpperCase)

cities6chars.explain(true)

// The filter predicate is pushed down fine for Dataset's Column-based query in where operator
scala> cities.where('name === "Warsaw").queryExecution.executedPlan
res29: org.apache.spark.sql.execution.SparkPlan =
*Project [id#128L, name#129]
+- *Filter (isnotnull(name#129) && (name#129 = Warsaw))
   +- *FileScan parquet [id#128L,name#129] Batched: true, Format: ParquetFormat, InputPaths: file:/Users/jacek/dev/oss/spark/cities.parquet, PartitionFilters: [], PushedFilters: [IsNotNull(name), EqualTo(name,Warsaw)], ReadSchema: struct<id:bigint,name:string>

// Let's define a UDF to do the filtering
val isWarsaw = udf { (s: String) => s == "Warsaw" }

// Use the UDF in where (replacing the Column-based query)
scala> cities.where(isWarsaw('name)).queryExecution.executedPlan
res33: org.apache.spark.sql.execution.SparkPlan =
*Filter UDF(name#129)
+- *FileScan parquet [id#128L,name#129] Batched: true, Format: ParquetFormat, InputPaths: file:/Users/jacek/dev/oss/spark/cities.parquet, PartitionFilters: [], PushedFilters: [], ReadSchema: struct<id:bigint,name:string>
```

















