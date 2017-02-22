## Window Aggregate Operators — Windows {#_window_aggregate_operators_windows}

Window Aggregate Operators（aka Windows）是对与当前记录有某种关系的一组记录（称为窗口）执行计算的运算符。他们计算窗口中每个记录的值。

| Note | 基于窗口的框架作为一个实验功能是从Spark 1.4.0开始的。 |
| :---: | :--- |


与常规聚合运算符不同，窗口聚合不将记录分组为单个记录，而是在与当前行位于同一分区中的行之间工作。

SQL查询，基于列的表达式和Scala API支持窗口聚合。

```
//
// Borrowed from 3.5. Window Functions in PostgreSQL documentation
// Example of window operators using Scala API
//
case class Salary(depName: String, empNo: Long, salary: Long)
val empsalary = Seq(
  Salary("sales", 1, 5000),
  Salary("personnel", 2, 3900),
  Salary("sales", 3, 4800),
  Salary("sales", 4, 4800),
  Salary("personnel", 5, 3500),
  Salary("develop", 7, 4200),
  Salary("develop", 8, 6000),
  Salary("develop", 9, 4500),
  Salary("develop", 10, 5200),
  Salary("develop", 11, 5200)).toDS

import org.apache.spark.sql.expressions.Window
// Windows are partitions of deptName
scala> val byDepName = Window.partitionBy('depName)
byDepName: org.apache.spark.sql.expressions.WindowSpec = org.apache.spark.sql.expressions.WindowSpec@1a711314

scala> empsalary.withColumn("avg", avg('salary) over byDepName).show
+---------+-----+------+-----------------+
|  depName|empNo|salary|              avg|
+---------+-----+------+-----------------+
|  develop|    7|  4200|           5020.0|
|  develop|    8|  6000|           5020.0|
|  develop|    9|  4500|           5020.0|
|  develop|   10|  5200|           5020.0|
|  develop|   11|  5200|           5020.0|
|    sales|    1|  5000|4866.666666666667|
|    sales|    3|  4800|4866.666666666667|
|    sales|    4|  4800|4866.666666666667|
|personnel|    2|  3900|           3700.0|
|personnel|    5|  3500|           3700.0|
+---------+-----+------+-----------------+
```

如上面的示例所示，您可以使用Window对象中的方便的工厂方法来创建一个窗口，创建一个窗口规范，您可以使用分区，排序和框架边界进一步细化。

描述窗口之后，您可以应用窗口聚合函数，例如排名函数（例如RANK），分析函数（例如LAG）和常规聚合函数，例如SUM，AVG，MAX。

### Window Aggregate Functions {#__a_id_functions_a_window_aggregate_functions}

窗口聚合函数计算一组与当前行相关的名为window的行的返回值。

| Note | 窗口函数也称为过度函数，因为它们如何使用Column的over函数。 |
| :---: | :--- |


尽管与聚合函数类似，窗口函数不会将行分组到单个输出行并保留其单独的标识。窗口函数可以访问链接到当前行的行。

Spark SQL支持三种窗口函数：

* **ranking **functions

* **analytic **functions

* **aggregate **functions

表1. Spark SQL中的窗口函数（请参阅Spark SQL中的窗口函数简介）

|   | SQL | Function |
| :--- | :--- | :--- |
| **Ranking functions** | RANK | rank |
|  | DENSE\_RANK | dense\_rank |
|  | PERCENT\_RANK | percent\_rank |
|  | NTILE | ntile |
|  | ROW\_NUMBER | row\_number |
| **Analytic functions** | CUME\_DIST | cume\_dist |
|  | LAG | lag |
|  | LEAD | lead |

对于聚合函数，可以使用现有的聚合函数作为窗口函数，例如。 sum，avg，min，max和count。

您可以在SQL中的函数之后通过OVER子句标记函数窗口，例如。 avg（revenue）OVER（...）或over方法。 rank（）.over（...）。

当执行时，窗口函数计算窗口中每行的值。

| Note | 窗口函数属于Spark的Scala API中的Window函数组。 |
| :---: | :--- |


### WindowSpec — Window Specification {#__a_id_windowspec_a_windowspec_window_specification}

窗口函数需要一个窗口规范，它是WindowSpec类的一个实例。

| Note | 从1.4.0开始，WindowSpec类被标记为实验。 |
| :---: | :--- |


| Tip | 请参阅org.apache.spark.sql.expressions.WindowSpec API。 |
| :---: | :--- |


窗口规范定义了哪些行被包括在与给定输入行相关联的窗口（也称为帧），即行集合中。它通过分割整个数据集并通过排序指定帧边界来实现。

| Note | 在Window对象中使用静态方法来创建WindowSpec。 |
| :---: | :--- |


```
import org.apache.spark.sql.expressions.Window

scala> val byHTokens = Window.partitionBy('token startsWith "h")
byHTokens: org.apache.spark.sql.expressions.WindowSpec = org.apache.spark.sql.expressions.WindowSpec@574985d8
```

窗口规范包括三个部分：

* 分区规范定义了哪些记录在同一个分区中。没有定义分区，所有记录属于单个分区。

* 排序规范定义分区中的记录如何排序，其又定义分区中的记录的位置。顺序可以是升序（在SQL中为ASC或在Scala中为asc）或降序（DESC或desc）。

* 框架规范（Hive中不支持;请参见为什么窗口函数失败，“窗口函数X不需要框架规范”？）根据它们与当前输入行的相对位置定义要包括在当前输入行的框架中的记录行。例如，“当前行之前的三行到当前行”描述了包括当前输入行和出现在当前行之前的三行的帧。

一旦使用Window对象创建了WindowSpec实例，您可以使用以下方法进一步扩展窗口规范以定义框架：

* `rowsBetween(start: Long, end: Long): WindowSpec`

* `rangeBetween(start: Long, end: Long): WindowSpec`

除了上面的两个，还可以使用以下方法（对应于Window对象中的方法）：

* `partitionBy`

* `orderBy`

### Window object {#__a_id_window_object_a_window_object}

Window对象提供了定义窗口（作为WindowSpec实例）的函数。

Window对象存在于org.apache.spark.sql.expressions包中。导入它以使用Window函数。

```
import org.apache.spark.sql.expressions.Window
```

在Window对象中有两个函数族可以为一个或多个Column实例创建WindowSpec实例：

* partitionBy

* orderBy

#### Partitioning Records — `partitionBy`Methods {#__a_id_partitionby_a_partitioning_records_code_partitionby_code_methods}

```
partitionBy(colName: String, colNames: String*): WindowSpec
partitionBy(cols: Column*): WindowSpec
```

partitionBy使用为一个或多个列定义的分区表达式创建WindowSpec的实例。

```
// partition records into two groups
// * tokens starting with "h"
// * others
val byHTokens = Window.partitionBy('token startsWith "h")

// count the sum of ids in each group
val result = tokens.select('*, sum('id) over byHTokens as "sum over h tokens").orderBy('id)

scala> .show
+---+-----+-----------------+
| id|token|sum over h tokens|
+---+-----+-----------------+
|  0|hello|                4|
|  1|henry|                4|
|  2|  and|                2|
|  3|harry|                4|
+---+-----+-----------------+
```

#### Ordering in Windows — `orderBy`Methods {#__a_id_orderby_a_ordering_in_windows_code_orderby_code_methods}

```
orderBy(colName: String, colNames: String*): WindowSpec
orderBy(cols: Column*): WindowSpec
```

orderBy允许您控制窗口中记录的顺序。

```
import org.apache.spark.sql.expressions.Window
val byDepnameSalaryDesc = Window.partitionBy('depname).orderBy('salary desc)

// a numerical rank within the current row's partition for each distinct ORDER BY value
scala> val rankByDepname = rank().over(byDepnameSalaryDesc)
rankByDepname: org.apache.spark.sql.Column = RANK() OVER (PARTITION BY depname ORDER BY salary DESC UnspecifiedFrame)

scala> empsalary.select('*, rankByDepname as 'rank).show
+---------+-----+------+----+
|  depName|empNo|salary|rank|
+---------+-----+------+----+
|  develop|    8|  6000|   1|
|  develop|   10|  5200|   2|
|  develop|   11|  5200|   2|
|  develop|    9|  4500|   4|
|  develop|    7|  4200|   5|
|    sales|    1|  5000|   1|
|    sales|    3|  4800|   2|
|    sales|    4|  4800|   2|
|personnel|    2|  3900|   1|
|personnel|    5|  3500|   2|
+---------+-----+------+----+
```

#### Window Examples {#__a_id_windowspec_examples_a_window_examples}

来自scaladoc   org.apache.spark.sql.expressions.Window的两个示例：

```
// PARTITION BY country ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
Window.partitionBy('country).orderBy('date).rowsBetween(Long.MinValue, 0)
```

```
// PARTITION BY country ORDER BY date ROWS BETWEEN 3 PRECEDING AND 3 FOLLOWING
Window.partitionBy('country).orderBy('date).rowsBetween(-3, 3)
```

### Frame {#__a_id_frame_a_frame}

在其核心，窗口函数基于称为帧的一组行来计算表的每个输入行的返回值。每个输入行可以具有与其相关联的唯一帧。

定义帧时，必须指定帧规范的三个组件 - 开始和结束边界以及类型。

边界类型（两个位置和三个偏移）：

* `UNBOUNDED PRECEDING`- the first row of the partition

* `UNBOUNDED FOLLOWING`- the last row of the partition

* `CURRENT ROW`

* `<value> PRECEDING`

* `<value> FOLLOWING`

Offset偏移量指定与当前输入行的偏移量。

Types of frames:

* `ROW`- based on _physical offsets _from the position of the current input row

* `RANGE`- based on _logical offsets _from the position of the current input row

在WindowSpec的当前实现中，您可以使用两种方法来定义一个框架：

* `rowsBetween`

* `rangeBetween`

### Window Operators in SQL Queries {#__a_id_sql_a_window_operators_in_sql_queries}

SQL中的Windows运算符的语法接受以下内容：

1. `CLUSTER BY`or`PARTITION BY`or`DISTRIBUTE BY`for partitions,

2. `ORDER BY`or`SORT BY`for sorting order,

3. `RANGE`,`ROWS`,`RANGE BETWEEN`, and`ROWS BETWEEN`for window frame types,

4. `UNBOUNDED PRECEDING`,`UNBOUNDED FOLLOWING`,`CURRENT ROW`for frame bounds.

### Examples {#__a_id_examples_a_examples}

#### Top N per Group {#__a_id_example_top_n_a_top_n_per_group}

当您需要计算类别中的第一和第二畅销商时，每组的前N个是有用的。

| Note | This example is borrowed from an_excellent_article [Introducing Window Functions in Spark SQL](https://databricks.com/blog/2015/07/15/introducing-window-functions-in-spark-sql.html). |
| :--- | :--- |


Table 2. Table PRODUCT\_REVENUE

| product | category | revenue |
| :--- | :--- | :--- |
| Thin | cell phone | 6000 |
| Normal | tablet | 1500 |
| Mini | tablet | 5500 |
| Ultra thin | cell phone | 5000 |
| Very thin | cell phone | 6000 |
| Big | tablet | 2500 |
| Bendable | cell phone | 3000 |
| Foldable | cell phone | 3000 |
| Pro | tablet+ | 4500 |
| Pro2 | tablet | 6500 |

问题：在每个类别中最畅销和第二畅销的产品是什么？

```
val dataset = Seq(
  ("Thin",       "cell phone", 6000),
  ("Normal",     "tablet",     1500),
  ("Mini",       "tablet",     5500),
  ("Ultra thin", "cell phone", 5000),
  ("Very thin",  "cell phone", 6000),
  ("Big",        "tablet",     2500),
  ("Bendable",   "cell phone", 3000),
  ("Foldable",   "cell phone", 3000),
  ("Pro",        "tablet",     4500),
  ("Pro2",       "tablet",     6500))
  .toDF("product", "category", "revenue")

scala> dataset.show
+----------+----------+-------+
|   product|  category|revenue|
+----------+----------+-------+
|      Thin|cell phone|   6000|
|    Normal|    tablet|   1500|
|      Mini|    tablet|   5500|
|Ultra thin|cell phone|   5000|
| Very thin|cell phone|   6000|
|       Big|    tablet|   2500|
|  Bendable|cell phone|   3000|
|  Foldable|cell phone|   3000|
|       Pro|    tablet|   4500|
|      Pro2|    tablet|   6500|
+----------+----------+-------+

scala> data.where('category === "tablet").show
+-------+--------+-------+
|product|category|revenue|
+-------+--------+-------+
| Normal|  tablet|   1500|
|   Mini|  tablet|   5500|
|    Big|  tablet|   2500|
|    Pro|  tablet|   4500|
|   Pro2|  tablet|   6500|
+-------+--------+-------+
```

问题归结为根据收入对某个类别中的产品进行排名，并根据排名选择最畅销的产品和第二畅销产品。

```
import org.apache.spark.sql.expressions.Window
val overCategory = Window.partitionBy('category).orderBy('revenue.desc)

val ranked = data.withColumn("rank", dense_rank.over(overCategory))

scala> ranked.show
+----------+----------+-------+----+
|   product|  category|revenue|rank|
+----------+----------+-------+----+
|      Pro2|    tablet|   6500|   1|
|      Mini|    tablet|   5500|   2|
|       Pro|    tablet|   4500|   3|
|       Big|    tablet|   2500|   4|
|    Normal|    tablet|   1500|   5|
|      Thin|cell phone|   6000|   1|
| Very thin|cell phone|   6000|   1|
|Ultra thin|cell phone|   5000|   2|
|  Bendable|cell phone|   3000|   3|
|  Foldable|cell phone|   3000|   3|
+----------+----------+-------+----+

scala> ranked.where('rank <= 2).show
+----------+----------+-------+----+
|   product|  category|revenue|rank|
+----------+----------+-------+----+
|      Pro2|    tablet|   6500|   1|
|      Mini|    tablet|   5500|   2|
|      Thin|cell phone|   6000|   1|
| Very thin|cell phone|   6000|   1|
|Ultra thin|cell phone|   5000|   2|
+----------+----------+-------+----+
```

#### Revenue Difference per Category {#_revenue_difference_per_category}

| Note | This example is the 2nd example from an _excellent _article [Introducing Window Functions in Spark SQL](https://databricks.com/blog/2015/07/15/introducing-window-functions-in-spark-sql.html). |
| :--- | :--- |


```
import org.apache.spark.sql.expressions.Window
val reveDesc = Window.partitionBy('category).orderBy('revenue.desc)
val reveDiff = max('revenue).over(reveDesc) - 'revenue

scala> data.select('*, reveDiff as 'revenue_diff).show
+----------+----------+-------+------------+
|   product|  category|revenue|revenue_diff|
+----------+----------+-------+------------+
|      Pro2|    tablet|   6500|           0|
|      Mini|    tablet|   5500|        1000|
|       Pro|    tablet|   4500|        2000|
|       Big|    tablet|   2500|        4000|
|    Normal|    tablet|   1500|        5000|
|      Thin|cell phone|   6000|           0|
| Very thin|cell phone|   6000|           0|
|Ultra thin|cell phone|   5000|        1000|
|  Bendable|cell phone|   3000|        3000|
|  Foldable|cell phone|   3000|        3000|
+----------+----------+-------+------------+
```

#### Difference on Column {#_difference_on_column}

计算列中行的值之间的差异。

```
val pairs = for {
  x <- 1 to 5
  y <- 1 to 2
} yield (x, 10 * x * y)
val ds = pairs.toDF("ns", "tens")

scala> ds.show
+---+----+
| ns|tens|
+---+----+
|  1|  10|
|  1|  20|
|  2|  20|
|  2|  40|
|  3|  30|
|  3|  60|
|  4|  40|
|  4|  80|
|  5|  50|
|  5| 100|
+---+----+

import org.apache.spark.sql.expressions.Window
val overNs = Window.partitionBy('ns).orderBy('tens)
val diff = lead('tens, 1).over(overNs)

scala> ds.withColumn("diff", diff - 'tens).show
+---+----+----+
| ns|tens|diff|
+---+----+----+
|  1|  10|  10|
|  1|  20|null|
|  3|  30|  30|
|  3|  60|null|
|  5|  50|  50|
|  5| 100|null|
|  4|  40|  40|
|  4|  80|null|
|  2|  20|  20|
|  2|  40|null|
+---+----+----+
```

请注意，为什么窗口函数失败，“窗口函数X不采取框架规范”？

这里的关键是要记住DataFrames是RDD下的覆盖，因此聚合像DataFrames中的一个键分组是RDD的groupBy（或更糟糕的是，reduceByKey或aggregateByKey转换）。

#### Running Total {#__a_id_example_running_total_a_running_total}

运行总和是包括当前行在内的所有先前行的总和。

```
val sales = Seq(
  (0, 0, 0, 5),
  (1, 0, 1, 3),
  (2, 0, 2, 1),
  (3, 1, 0, 2),
  (4, 2, 0, 8),
  (5, 2, 2, 8))
  .toDF("id", "orderID", "prodID", "orderQty")

scala> sales.show
+---+-------+------+--------+
| id|orderID|prodID|orderQty|
+---+-------+------+--------+
|  0|      0|     0|       5|
|  1|      0|     1|       3|
|  2|      0|     2|       1|
|  3|      1|     0|       2|
|  4|      2|     0|       8|
|  5|      2|     2|       8|
+---+-------+------+--------+

val orderedByID = Window.orderBy('id)

val totalQty = sum('orderQty).over(orderedByID).as('running_total)
val salesTotalQty = sales.select('*, totalQty).orderBy('id)

scala> salesTotalQty.show
16/04/10 23:01:52 WARN Window: No Partition Defined for Window operation! Moving all data to a single partition, this can cause serious performance degradation.
+---+-------+------+--------+-------------+
| id|orderID|prodID|orderQty|running_total|
+---+-------+------+--------+-------------+
|  0|      0|     0|       5|            5|
|  1|      0|     1|       3|            8|
|  2|      0|     2|       1|            9|
|  3|      1|     0|       2|           11|
|  4|      2|     0|       8|           19|
|  5|      2|     2|       8|           27|
+---+-------+------+--------+-------------+

val byOrderId = orderedByID.partitionBy('orderID)
val totalQtyPerOrder = sum('orderQty).over(byOrderId).as('running_total_per_order)
val salesTotalQtyPerOrder = sales.select('*, totalQtyPerOrder).orderBy('id)

scala> salesTotalQtyPerOrder.show
+---+-------+------+--------+-----------------------+
| id|orderID|prodID|orderQty|running_total_per_order|
+---+-------+------+--------+-----------------------+
|  0|      0|     0|       5|                      5|
|  1|      0|     1|       3|                      8|
|  2|      0|     2|       1|                      9|
|  3|      1|     0|       2|                      2|
|  4|      2|     0|       8|                      8|
|  5|      2|     2|       8|                     16|
+---+-------+------+--------+-----------------------+
```

#### Calculate rank of row {#__a_id_example_rank_a_calculate_rank_of_row}

有关详细示例，请参见“解释”Windows的查询计划。

### Interval data type for Date and Timestamp types {#_interval_data_type_for_date_and_timestamp_types}

See [\[SPARK-8943\] CalendarIntervalType for time intervals](https://issues.apache.org/jira/browse/SPARK-8943).

使用间隔数据类型，可以使用间隔作为&lt;value&gt;PRECEDING和&lt;value&gt; FOLLOWING中为RANGE帧指定的值。它特别适合于具有窗口函数的时间序列分析。

#### Accessing values of earlier rows {#_accessing_values_of_earlier_rows}

FIXME What’s the value of rows before current one?

#### Moving Average {#__a_id_example_moving_average_a_moving_average}

#### Cumulative Aggregates {#__a_id_example_cumulative_aggregates_a_cumulative_aggregates}

Eg. cumulative sum

### User-defined aggregate functions {#_user_defined_aggregate_functions}

See[\[SPARK-3947\] Support Scala/Java UDAF](https://issues.apache.org/jira/browse/SPARK-3947).

With the window function support, you could use user-defined aggregate functions as window functions.

### "Explaining" Query Plans of Windows {#__a_id_explain_windows_a_explaining_query_plans_of_windows}

```
import org.apache.spark.sql.expressions.Window
val byDepnameSalaryDesc = Window.partitionBy('depname).orderBy('salary desc)

scala> val rankByDepname = rank().over(byDepnameSalaryDesc)
rankByDepname: org.apache.spark.sql.Column = RANK() OVER (PARTITION BY depname ORDER BY salary DESC UnspecifiedFrame)

// empsalary defined at the top of the page
scala> empsalary.select('*, rankByDepname as 'rank).explain(extended = true)
== Parsed Logical Plan ==
'Project [*, rank() windowspecdefinition('depname, 'salary DESC, UnspecifiedFrame) AS rank#9]
+- LocalRelation [depName#5, empNo#6L, salary#7L]

== Analyzed Logical Plan ==
depName: string, empNo: bigint, salary: bigint, rank: int
Project [depName#5, empNo#6L, salary#7L, rank#9]
+- Project [depName#5, empNo#6L, salary#7L, rank#9, rank#9]
   +- Window [rank(salary#7L) windowspecdefinition(depname#5, salary#7L DESC, ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS rank#9], [depname#5], [salary#7L DESC]
      +- Project [depName#5, empNo#6L, salary#7L]
         +- LocalRelation [depName#5, empNo#6L, salary#7L]

== Optimized Logical Plan ==
Window [rank(salary#7L) windowspecdefinition(depname#5, salary#7L DESC, ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS rank#9], [depname#5], [salary#7L DESC]
+- LocalRelation [depName#5, empNo#6L, salary#7L]

== Physical Plan ==
Window [rank(salary#7L) windowspecdefinition(depname#5, salary#7L DESC, ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS rank#9], [depname#5], [salary#7L DESC]
+- *Sort [depname#5 ASC, salary#7L DESC], false, 0
   +- Exchange hashpartitioning(depname#5, 200)
      +- LocalTableScan [depName#5, empNo#6L, salary#7L]
```

#### Window Unary Logical Plan {#__a_id_window_a_window_unary_logical_plan}

Window是为NamedExpressions（对于Windows），Expressions（对于分区），SortOrder（用于排序）和子逻辑计划的集合的集合创建的一元逻辑计划。

The`output`\(collection ofAttributes\) is the child’s attributes and the window’s.

窗口逻辑计划是在ColumnPruning规则中修剪不必要的窗口表达式并在PushDownPredicate规则中推送过滤器运算符的主题。

### Further reading or watching {#__a_id_i_want_more_a_further_reading_or_watching}

* [3.5. Window Functions](http://www.postgresql.org/docs/current/static/tutorial-window.html)in the official documentation of PostgreSQL

* [Window Functions in SQL](https://www.simple-talk.com/sql/t-sql-programming/window-functions-in-sql/)

* [Working with Window Functions in SQL Server](https://www.simple-talk.com/sql/learn-sql-server/working-with-window-functions-in-sql-server/)

* [OVER Clause \(Transact-SQL\)](https://msdn.microsoft.com/en-CA/library/ms189461.aspx)

* [An introduction to windowed functions](https://sqlsunday.com/2013/03/31/windowed-functions/)

* [Probably the Coolest SQL Feature: Window Functions](https://blog.jooq.org/2013/11/03/probably-the-coolest-sql-feature-window-functions/)

* [Window Functions](https://sqlschool.modeanalytics.com/advanced/window-functions/)















