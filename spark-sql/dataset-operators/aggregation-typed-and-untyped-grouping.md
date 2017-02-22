## Aggregation — Typed and Untyped Grouping {#_aggregation_typed_and_untyped_grouping}

您可以使用条件来计算聚合（在分组记录的集合上），从而对数据集中的记录进行分组。

您可以使用agg方法计算整个数据集的每列聚合（无需先创建组，将整个数据集视为一个组）。

```
scala> spark.range(10).agg(sum('id) as "sum").show
+---+
|sum|
+---+
| 45|
+---+
```

可以使用以下聚合操作：

* groupBy用于具有基于列或字符串的列名称的无类型聚合。 

* groupByKey用于强类型聚合，其中数据按给定的键函数分组。

* rollup

* cube

无类型聚合groupBy，rollup和cube，返回RelationalGroupedDatasets，而groupByKey返回KeyValueGroupedDataset。

### Aggregates on Entire Dataset \(Without Groups\) — `agg`Operator {#__a_id_agg_a_aggregates_on_entire_dataset_without_groups_code_agg_code_operator}

```
agg(expr: Column, exprs: Column*): DataFrame
agg(exprs: Map[String, String]): DataFrame
agg(aggExpr: (String, String), aggExprs: (String, String)*): DataFrame
```

agg计算数据集中所有记录的聚合表达式。

| Note | agg只是groupBy\(\).agg\(...\)的一个快捷方式。 |
| :---: | :--- |


### Grouping by Columns — `groupBy`Untyped Operators {#__a_id_groupby_a_grouping_by_columns_code_groupby_code_untyped_operators}

```
groupBy(cols: Column*): RelationalGroupedDataset
groupBy(col1: String, cols: String*): RelationalGroupedDataset
```

groupBy方法将使用指定列作为列或其文本表示的数据集分组。它返回一个RelationalGroupedDataset以应用聚合。

```
// 10^3-record large data set
val ints = 1 to math.pow(10, 3).toInt

scala> val dataset = ints.toDF("n").withColumn("m", 'n % 2)
dataset: org.apache.spark.sql.DataFrame = [n: int, m: int]

scala> dataset.count
res0: Long = 1000

scala> dataset.groupBy('m).agg(sum('n)).show
+---+------+
|  m|sum(n)|
+---+------+
|  1|250000|
|  0|250500|
+---+------+
```

在内部，它首先解析列，然后构建一个RelationalGroupedDataset。

| Note | 以下会话使用数据设置，如测试设置一节中所述。 |
| :---: | :--- |


```
scala> dataset.show
+----+---------+-----+
|name|productId|score|
+----+---------+-----+
| aaa|      100| 0.12|
| aaa|      200| 0.29|
| bbb|      200| 0.53|
| bbb|      300| 0.42|
+----+---------+-----+

scala> dataset.groupBy('name).avg().show
+----+--------------+----------+
|name|avg(productId)|avg(score)|
+----+--------------+----------+
| aaa|         150.0|     0.205|
| bbb|         250.0|     0.475|
+----+--------------+----------+

scala> dataset.groupBy('name, 'productId).agg(Map("score" -> "avg")).show
+----+---------+----------+
|name|productId|avg(score)|
+----+---------+----------+
| aaa|      200|      0.29|
| bbb|      200|      0.53|
| bbb|      300|      0.42|
| aaa|      100|      0.12|
+----+---------+----------+

scala> dataset.groupBy('name).count.show
+----+-----+
|name|count|
+----+-----+
| aaa|    2|
| bbb|    2|
+----+-----+

scala> dataset.groupBy('name).max("score").show
+----+----------+
|name|max(score)|
+----+----------+
| aaa|      0.29|
| bbb|      0.53|
+----+----------+

scala> dataset.groupBy('name).sum("score").show
+----+----------+
|name|sum(score)|
+----+----------+
| aaa|      0.41|
| bbb|      0.95|
+----+----------+

scala> dataset.groupBy('productId).sum("score").show
+---------+------------------+
|productId|        sum(score)|
+---------+------------------+
|      300|              0.42|
|      100|              0.12|
|      200|0.8200000000000001|
+---------+------------------+
```

### `groupByKey`Typed Operator {#__a_id_groupbykey_a_code_groupbykey_code_typed_operator}

```
groupByKey[K: Encoder](func: T => K): KeyValueGroupedDataset[K, T]
```

groupByKey通过输入func组合记录（类型T）。它返回一个KeyValueGroupedDataset来应用聚合。

| Note | groupByKey是Dataset的实验API。 |
| :---: | :--- |


```
scala> dataset.groupByKey(_.productId).count.show
+-----+--------+
|value|count(1)|
+-----+--------+
|  300|       1|
|  100|       1|
|  200|       2|
+-----+--------+

import org.apache.spark.sql.expressions.scalalang._
scala> dataset.groupByKey(_.productId).agg(typed.sum[Token](_.score)).toDF("productId", "sum").orderBy('productId).show
+---------+------------------+
|productId|               sum|
+---------+------------------+
|      100|              0.12|
|      200|0.8200000000000001|
|      300|              0.42|
+---------+------------------+
```

### RelationalGroupedDataset {#__a_id_relationalgroupeddataset_a_relationalgroupeddataset}

RelationalGroupedDataset是执行非类型化运算符groupBy，rollup和cube的结果。

RelationalGroupedDataset也是对分组记录执行pivot操作符为RelationalGroupedDataset的结果。

它提供以下操作符来处理分组的记录集合：

* `agg`

* `count`

* `mean`

* `max`

* `avg`

* `min`

* `sum`

* `pivot`

### KeyValueGroupedDataset {#__a_id_keyvaluegroupeddataset_a_keyvaluegroupeddataset}

KeyValueGroupedDataset是执行强类型操作符groupByKey的结果的实验​​性接口。

```
scala> val tokensByName = dataset.groupByKey(_.name)
tokensByName: org.apache.spark.sql.KeyValueGroupedDataset[String,Token] = org.apache.spark.sql.KeyValueGroupedDataset@1e3aad46
```

它包含用于对象的keys。

```
scala> tokensByName.keys.show
+-----+
|value|
+-----+
|  aaa|
|  bbb|
+-----+
```

以下方法可用于任何KeyValueGroupedDataset在记录组上工作：

1. `agg`\(of 1 to 4 types\)

2. `mapGroups`

3. `flatMapGroups`

4. `reduceGroups`

5. `count`that is a special case of`agg`with[count](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/spark-sql-functions.html#count)function applied.

6. `cogroup`

### Test Setup {#__a_id_test_setup_a_test_setup}

这是一个用于学习GroupedData的设置。使用：粘贴到Spark Shell中。

```
import spark.implicits._

case class Token(name: String, productId: Int, score: Double)
val data = Token("aaa", 100, 0.12) ::
  Token("aaa", 200, 0.29) ::
  Token("bbb", 200, 0.53) ::
  Token("bbb", 300, 0.42) :: Nil
val dataset = data.toDS.cache  (1)
```

缓存数据集，以便以下查询不会重复加载/重新计算数据。



