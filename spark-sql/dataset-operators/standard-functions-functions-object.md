## Standard Functions — `functions`object {#_standard_functions_code_functions_code_object}

org.apache.spark.sql.functions对象提供了许多内置函数来处理数据集中的列中的值。

| Note | functions对象是Spark自1.3.0版本起的一个实验性功能。 |
| :---: | :--- |


您可以使用以下import语句访问函数：

```
import org.apache.spark.sql.functions._
```

函数对象中有超过300个函数。一些函数是将Column对象（或列名）转换为其他Column对象或将DataFrame转换为DataFrame。

functions按功能区分组：

Table 1. Functions in Spark SQL

|   | Functions | Description |
| :--- | :--- | :--- |
| **Window functions** | rank, dense\_rank, and percent\_rank | Ranking records per window partition |
|  | ntile | ntile |
|  | row\_number | Sequential numbering per window partition |
|  | cume\_dist | Cumulative distribution of records across window partitions |
|  | lag |  |
|  | lead | lead |

* Defining UDFs

* Creating Columns using`col`and`column`methods

* String functions

  * split

  * upper\(chained with`reverse`\)

* Aggregate functions

  * count

* Non-aggregate functions\(aka_normal functions_\)

  * struct

  * broadcast\(for`DataFrame`\)

  * expr

* Date and time functions

* explode

* window

* …​_and others_

| Tip | You should read the official documentation of the functions object. |
| :--- | :--- |


### explode {#__a_id_explode_a_explode}

| Caution | FIXME |
| :--- | :--- |


```
scala> Seq(Array(0,1,2)).toDF("array").withColumn("num", explode('array)).show
+---------+---+
|    array|num|
+---------+---+
|[0, 1, 2]|  0|
|[0, 1, 2]|  1|
|[0, 1, 2]|  2|
+---------+---+
```

### Ranking Records per Window Partition — `rank`functions {#__a_id_rank_a_a_id_dense_rank_a_a_id_percent_rank_a_ranking_records_per_window_partition_code_rank_code_functions}

```
rank(): Column
dense_rank(): Column
percent_rank(): Column
```

rank函数为每个窗口分区分配每个不同值的顺序排名。它们相当于好的SQL中的RANK，DENSE\_RANK和PERCENT\_RANK函数。

```
val dataset = spark.range(9).withColumn("bucket", 'id % 3)

import org.apache.spark.sql.expressions.Window
val byBucket = Window.partitionBy('bucket).orderBy('id)

scala> dataset.withColumn("rank", rank over byBucket).show
+---+------+----+
| id|bucket|rank|
+---+------+----+
|  0|     0|   1|
|  3|     0|   2|
|  6|     0|   3|
|  1|     1|   1|
|  4|     1|   2|
|  7|     1|   3|
|  2|     2|   1|
|  5|     2|   2|
|  8|     2|   3|
+---+------+----+

scala> dataset.withColumn("percent_rank", percent_rank over byBucket).show
+---+------+------------+
| id|bucket|percent_rank|
+---+------+------------+
|  0|     0|         0.0|
|  3|     0|         0.5|
|  6|     0|         1.0|
|  1|     1|         0.0|
|  4|     1|         0.5|
|  7|     1|         1.0|
|  2|     2|         0.0|
|  5|     2|         0.5|
|  8|     2|         1.0|
+---+------+------------+
```

rank函数为序列中具有间隙的重复行分配相同的排名（类似于奥运奖牌场所）。 dense\_rank类似于重复行的排名，但压缩等级并消除间隙。

```
// rank function with duplicates
// Note the missing/sparse ranks, i.e. 2 and 4
scala> dataset.union(dataset).withColumn("rank", rank over byBucket).show
+---+------+----+
| id|bucket|rank|
+---+------+----+
|  0|     0|   1|
|  0|     0|   1|
|  3|     0|   3|
|  3|     0|   3|
|  6|     0|   5|
|  6|     0|   5|
|  1|     1|   1|
|  1|     1|   1|
|  4|     1|   3|
|  4|     1|   3|
|  7|     1|   5|
|  7|     1|   5|
|  2|     2|   1|
|  2|     2|   1|
|  5|     2|   3|
|  5|     2|   3|
|  8|     2|   5|
|  8|     2|   5|
+---+------+----+

// dense_rank function with duplicates
// Note that the missing ranks are now filled in
scala> dataset.union(dataset).withColumn("dense_rank", dense_rank over byBucket).show
+---+------+----------+
| id|bucket|dense_rank|
+---+------+----------+
|  0|     0|         1|
|  0|     0|         1|
|  3|     0|         2|
|  3|     0|         2|
|  6|     0|         3|
|  6|     0|         3|
|  1|     1|         1|
|  1|     1|         1|
|  4|     1|         2|
|  4|     1|         2|
|  7|     1|         3|
|  7|     1|         3|
|  2|     2|         1|
|  2|     2|         1|
|  5|     2|         2|
|  5|     2|         2|
|  8|     2|         3|
|  8|     2|         3|
+---+------+----------+

// percent_rank function with duplicates
scala> dataset.union(dataset).withColumn("percent_rank", percent_rank over byBucket).show
+---+------+------------+
| id|bucket|percent_rank|
+---+------+------------+
|  0|     0|         0.0|
|  0|     0|         0.0|
|  3|     0|         0.4|
|  3|     0|         0.4|
|  6|     0|         0.8|
|  6|     0|         0.8|
|  1|     1|         0.0|
|  1|     1|         0.0|
|  4|     1|         0.4|
|  4|     1|         0.4|
|  7|     1|         0.8|
|  7|     1|         0.8|
|  2|     2|         0.0|
|  2|     2|         0.0|
|  5|     2|         0.4|
|  5|     2|         0.4|
|  8|     2|         0.8|
|  8|     2|         0.8|
+---+------+------------+
```

### Cumulative Distribution of Records Across Window Partitions — `cume_dist`function {#__a_id_cume_dist_a_cumulative_distribution_of_records_across_window_partitions_code_cume_dist_code_function}

```
cume_dist(): Column
```

cume\_dist计算窗口分区中记录的累积分布。这相当于SQL的CUME\_DIST函数。

```
val buckets = spark.range(9).withColumn("bucket", 'id % 3)
// Make duplicates
val dataset = buckets.union(buckets)

import org.apache.spark.sql.expressions.Window
val windowSpec = Window.partitionBy('bucket).orderBy('id)
scala> dataset.withColumn("cume_dist", cume_dist over windowSpec).show
+---+------+------------------+
| id|bucket|         cume_dist|
+---+------+------------------+
|  0|     0|0.3333333333333333|
|  3|     0|0.6666666666666666|
|  6|     0|               1.0|
|  1|     1|0.3333333333333333|
|  4|     1|0.6666666666666666|
|  7|     1|               1.0|
|  2|     2|0.3333333333333333|
|  5|     2|0.6666666666666666|
|  8|     2|               1.0|
+---+------+------------------+
```

### `lag`functions {#__a_id_lag_a_code_lag_code_functions}

```
lag(e: Column, offset: Int): Column
lag(columnName: String, offset: Int): Column
lag(columnName: String, offset: Int, defaultValue: Any): Column
lag(e: Column, offset: Int, defaultValue: Any): Column
```

lag返回e / columnName列中的值，该值是当前记录之前的偏移记录。

如果窗口分区中的记录数小于offset或defaultValue，则lag返回null值。

```
val buckets = spark.range(9).withColumn("bucket", 'id % 3)
// Make duplicates
val dataset = buckets.union(buckets)

import org.apache.spark.sql.expressions.Window
val windowSpec = Window.partitionBy('bucket).orderBy('id)
scala> dataset.withColumn("lag", lag('id, 1) over windowSpec).show
+---+------+----+
| id|bucket| lag|
+---+------+----+
|  0|     0|null|
|  3|     0|   0|
|  6|     0|   3|
|  1|     1|null|
|  4|     1|   1|
|  7|     1|   4|
|  2|     2|null|
|  5|     2|   2|
|  8|     2|   5|
+---+------+----+

scala> dataset.withColumn("lag", lag('id, 2, "<default_value>") over windowSpec).show
+---+------+----+
| id|bucket| lag|
+---+------+----+
|  0|     0|null|
|  3|     0|null|
|  6|     0|   0|
|  1|     1|null|
|  4|     1|null|
|  7|     1|   1|
|  2|     2|null|
|  5|     2|null|
|  8|     2|   2|
+---+------+----+
```

| Caution | 它看起来像滞后与默认值有一个错误 - 默认值不使用。 |
| :---: | :--- |


### Sequential numbering per window partition — `row_number`functions {#__a_id_row_number_a_sequential_numbering_per_window_partition_code_row_number_code_functions}

```
row_number(): Column
```

row\_number返回在窗口分区中从1开始的序列号。

```
val buckets = spark.range(9).withColumn("bucket", 'id % 3)
// Make duplicates
val dataset = buckets.union(buckets)

import org.apache.spark.sql.expressions.Window
val windowSpec = Window.partitionBy('bucket).orderBy('id)
scala> dataset.withColumn("row_number", row_number() over windowSpec).show
+---+------+----------+
| id|bucket|row_number|
+---+------+----------+
|  0|     0|         1|
|  0|     0|         2|
|  3|     0|         3|
|  3|     0|         4|
|  6|     0|         5|
|  6|     0|         6|
|  1|     1|         1|
|  1|     1|         2|
|  4|     1|         3|
|  4|     1|         4|
|  7|     1|         5|
|  7|     1|         6|
|  2|     2|         1|
|  2|     2|         2|
|  5|     2|         3|
|  5|     2|         4|
|  8|     2|         5|
|  8|     2|         6|
+---+------+----------+
```

### `ntile`function {#__a_id_ntile_a_code_ntile_code_function}

```
ntile(n: Int): Column
```

ntile计算有序窗口分区中的ntile组ID（从1到n）。

```
val dataset = spark.range(7).select('*, 'id % 3 as "bucket")

import org.apache.spark.sql.expressions.Window
val byBuckets = Window.partitionBy('bucket).orderBy('id)
scala> dataset.select('*, ntile(3) over byBuckets as "ntile").show
+---+------+-----+
| id|bucket|ntile|
+---+------+-----+
|  0|     0|    1|
|  3|     0|    2|
|  6|     0|    3|
|  1|     1|    1|
|  4|     1|    2|
|  2|     2|    1|
|  5|     2|    2|
+---+------+-----+
```

| Caution | ntile与rank有什么不同？性能怎么样？ |
| :---: | :--- |


### Creating Columns — `col`and`column`methods {#__a_id_creating_columns_a_a_id_col_a_a_id_column_a_creating_columns_code_col_code_and_code_column_code_methods}

```
col(colName: String): Column
column(colName: String): Column
```

col和column方法创建一个Column，您稍后可以使用它来引用数据集中的列。

```
import org.apache.spark.sql.functions._

scala> val nameCol = col("name")
nameCol: org.apache.spark.sql.Column = name

scala> val cityCol = column("city")
cityCol: org.apache.spark.sql.Column = city
```

### Defining UDFs \(udf factories\) {#__a_id_udf_a_defining_udfs_udf_factories}

```
udf(f: FunctionN[...]): UserDefinedFunction
```

udf系列函数允许您在Scala中基于用户定义的函数创建用户定义的函数（UDF）。它接受0到10个参数的f函数，并且自动推断输入和输出类型（给定函数f的相应输入和输出类型的类型）。

```
import org.apache.spark.sql.functions._
val _length: String => Int = _.length
val _lengthUDF = udf(_length)

// define a dataframe
val df = sc.parallelize(0 to 3).toDF("num")

// apply the user-defined function to "num" column
scala> df.withColumn("len", _lengthUDF($"num")).show
+---+---+
|num|len|
+---+---+
|  0|  1|
|  1|  1|
|  2|  1|
|  3|  1|
+---+---+
```

从Spark 2.0.0开始，还有udf函数的另一个变体：

```
udf(f: AnyRef, dataType: DataType): UserDefinedFunction
```

udf（f：AnyRef，dataType：DataType）允许您对函数参数（如f）使用Scala闭包，并显式声明输出数据类型（作为dataType）。

```
// given the dataframe above

import org.apache.spark.sql.types.IntegerType
val byTwo = udf((n: Int) => n * 2, IntegerType)

scala> df.withColumn("len", byTwo($"num")).show
+---+---+
|num|len|
+---+---+
|  0|  0|
|  1|  2|
|  2|  4|
|  3|  6|
+---+---+
```

### String functions {#__a_id_string_functions_a_string_functions}

#### split function {#__a_id_split_a_split_function}

```
split(str: Column, pattern: String): Column
```

split函数拆分str列使用模式。它返回一个新的列。

| Note | `split`UDF uses [java.lang.String.split\(String regex, int limit\)](https://docs.oracle.com/javase/8/docs/api/java/lang/String.html#split-java.lang.String-int-)method. |
| :--- | :--- |


```
val df = Seq((0, "hello|world"), (1, "witaj|swiecie")).toDF("num", "input")
val withSplit = df.withColumn("split", split($"input", "[|]"))

scala> withSplit.show
+---+-------------+----------------+
|num|        input|           split|
+---+-------------+----------------+
|  0|  hello|world|  [hello, world]|
|  1|witaj|swiecie|[witaj, swiecie]|
+---+-------------+----------------+
```

| Note | `.$|()[{^?*+\`are RegEx’s meta characters and are considered special. |
| :--- | :--- |


#### upper function {#__a_id_upper_a_upper_function}

```
upper(e: Column): Column
```

upper函数将字符串列转换为所有字母上的一列。它返回一个新的列。

| Note | 以下示例使用两个函数接受列并返回另一个以显示如何链接它们。 |
| :---: | :--- |


```
val df = Seq((0,1,"hello"), (2,3,"world"), (2,4, "ala")).toDF("id", "val", "name")
val withUpperReversed = df.withColumn("upper", reverse(upper($"name")))

scala> withUpperReversed.show
+---+---+-----+-----+
| id|val| name|upper|
+---+---+-----+-----+
|  0|  1|hello|OLLEH|
|  2|  3|world|DLROW|
|  2|  4|  ala|  ALA|
+---+---+-----+-----+
```

### Non-aggregate functions {#__a_id_non_aggregate_functions_a_non_aggregate_functions}

它们也称为正常函数。

#### struct functions {#__a_id_struct_a_struct_functions}

```
struct(cols: Column*): Column
struct(colName: String, colNames: String*): Column
```

struct系列函数允许您基于Column或其名称的集合创建新的结构列。

| Note | struct和另一个类似数组函数的区别在于，列的类型可以不同（在struct中）。 |
| :---: | :--- |


```
scala> df.withColumn("struct", struct($"name", $"val")).show
+---+---+-----+---------+
| id|val| name|   struct|
+---+---+-----+---------+
|  0|  1|hello|[hello,1]|
|  2|  3|world|[world,3]|
|  2|  4|  ala|  [ala,4]|
+---+---+-----+---------+
```

#### broadcast function {#__a_id_broadcast_a_broadcast_function}

```
broadcast[T](df: Dataset[T]): Dataset[T]
```

broadcast功能将输入数据集标记为足够小以在broadcast join中使用。

```
val left = Seq((0, "aa"), (0, "bb")).toDF("id", "token").as[(Int, String)]
val right = Seq(("aa", 0.99), ("bb", 0.57)).toDF("token", "prob").as[(String, Double)]

scala> left.join(broadcast(right), "token").explain(extended = true)
== Parsed Logical Plan ==
'Join UsingJoin(Inner,List('token))
:- Project [_1#42 AS id#45, _2#43 AS token#46]
:  +- LocalRelation [_1#42, _2#43]
+- BroadcastHint
   +- Project [_1#55 AS token#58, _2#56 AS prob#59]
      +- LocalRelation [_1#55, _2#56]

== Analyzed Logical Plan ==
token: string, id: int, prob: double
Project [token#46, id#45, prob#59]
+- Join Inner, (token#46 = token#58)
   :- Project [_1#42 AS id#45, _2#43 AS token#46]
   :  +- LocalRelation [_1#42, _2#43]
   +- BroadcastHint
      +- Project [_1#55 AS token#58, _2#56 AS prob#59]
         +- LocalRelation [_1#55, _2#56]

== Optimized Logical Plan ==
Project [token#46, id#45, prob#59]
+- Join Inner, (token#46 = token#58)
   :- Project [_1#42 AS id#45, _2#43 AS token#46]
   :  +- Filter isnotnull(_2#43)
   :     +- LocalRelation [_1#42, _2#43]
   +- BroadcastHint
      +- Project [_1#55 AS token#58, _2#56 AS prob#59]
         +- Filter isnotnull(_1#55)
            +- LocalRelation [_1#55, _2#56]

== Physical Plan ==
*Project [token#46, id#45, prob#59]
+- *BroadcastHashJoin [token#46], [token#58], Inner, BuildRight
   :- *Project [_1#42 AS id#45, _2#43 AS token#46]
   :  +- *Filter isnotnull(_2#43)
   :     +- LocalTableScan [_1#42, _2#43]
   +- BroadcastExchange HashedRelationBroadcastMode(List(input[0, string, true]))
      +- *Project [_1#55 AS token#58, _2#56 AS prob#59]
         +- *Filter isnotnull(_1#55)
            +- LocalTableScan [_1#55, _2#56]
```

#### expr function {#__a_id_expr_a_expr_function}

```
expr(expr: String): Column
```

expr函数将输入expr SQL字符串解析为它表示的列。

```
val ds = Seq((0, "hello"), (1, "world"))
  .toDF("id", "token")
  .as[(Long, String)]

scala> ds.show
+---+-----+
| id|token|
+---+-----+
|  0|hello|
|  1|world|
+---+-----+

val filterExpr = expr("token = 'hello'")

scala> ds.filter(filterExpr).show
+---+-----+
| id|token|
+---+-----+
|  0|hello|
+---+-----+
```

在内部，expr使用活动会话的sqlParser或创建一个新的SparkSqlParser来调用parseExpression方法。

### count {#__a_id_count_a_count}

| Caution | FIXME |
| :--- | :--- |


### Generating Tumbling Time Windows — `window`functions {#__a_id_window_a_generating_tumbling_time_windows_code_window_code_functions}

```
window(
  timeColumn: Column,
  windowDuration: String): Column  (1)
window(
  timeColumn: Column,
  windowDuration: String,
  slideDuration: String): Column   (2)
window(
  timeColumn: Column,
  windowDuration: String,
  slideDuration: String,
  startTime: String): Column
```

1.使用slideDuration作为windowDuration和0秒的startTime

2.对于startTime为0秒

window生成windowDuration持续时间的翻转时间窗口，给定timeColumn时间戳指定列。

    scala> window('time, "5 seconds")
    res0: org.apache.spark.sql.Column = timewindow(time, 5000000, 5000000, 0) AS `window`

timeColumn必须是TimestampType，即java.sql.Timestamp值。

| Tip | 使用java.sql.Timestamp.from或java.sql.Timestamp.valueOf工厂方法创建Timestamp实例。 |
| :---: | :--- |


windowDuration和slideDuration是分别指定持续时间和滑动标识符的窗口宽度的字符串。

| Tip | 对有效的窗口标识符使用CalendarInterval。 |
| :---: | :--- |


| Note | `window`is available as of Spark **2.0.0**. |
| :--- | :--- |


Internally,`window`creates a Column with`TimeWindow`expression.





