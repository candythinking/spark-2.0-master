## Dataset Columns {#__a_id_column_a_dataset_columns}

`Column`类型表示数据集中的列，它们是给定字段的记录值。

| Note | A`Column`is a value generator for records of a`Dataset`. |
| :--- | :--- |


一个`Column`具有对使用expr创建的表达式的引用。

```
scala> window('time, "5 seconds").expr
res0: org.apache.spark.sql.catalyst.expressions.Expression = timewindow('time, 5000000, 5000000, 0) AS window#1
```

通过导入implicits转换，您可以使用Scala的符号创建“自由”列引用。

```
val spark: SparkSession = ...
import spark.implicits._

import org.apache.spark.sql.Column
scala> val nameCol: Column = 'name
nameCol: org.apache.spark.sql.Column = name
```

| Note | “自由”列引用是与数据集无关联的列。 |
| :---: | :--- |


您还可以从$前缀字符串创建自由列引用。

```
// Note that $ alone creates a ColumnName
scala> val idCol = $"id"
idCol: org.apache.spark.sql.ColumnName = id

import org.apache.spark.sql.Column

// The target type triggers the implicit conversion to Column
scala> val idCol: Column = $"id"
idCol: org.apache.spark.sql.Column = id
```

除了使用implicits转换来创建列之外，还可以使用functions对象的col和column方法。

```
import org.apache.spark.sql.functions._

scala> val nameCol = col("name")
nameCol: org.apache.spark.sql.Column = name

scala> val cityCol = column("city")
cityCol: org.apache.spark.sql.Column = city
```

最后，您可以使用Dataset.apply工厂方法或Dataset.col方法使用它所属的Dataset创建一个Column引用。您只能对从其创建的数据集使用此类列引用。

```
scala> val textCol = dataset.col("text")
textCol: org.apache.spark.sql.Column = text

scala> val idCol = dataset.apply("id")
idCol: org.apache.spark.sql.Column = id

scala> val idCol = dataset("id")
idCol: org.apache.spark.sql.Column = id
```

您可以使用引用嵌套列  . （点）。

### Adding Column to Dataset — `withColumn`Method {#__a_id_withcolumn_a_adding_column_to_dataset_code_withcolumn_code_method}

```
withColumn(colName: String, col: Column): DataFrame
```

withColumn方法返回一个新的DataFrame与添加colName名称的新列col。

| Note | withColumn可以替换现有的colName列。 |
| :---: | :--- |


```
scala> val df = Seq((1, "jeden"), (2, "dwa")).toDF("number", "polish")
df: org.apache.spark.sql.DataFrame = [number: int, polish: string]

scala> df.show
+------+------+
|number|polish|
+------+------+
|     1| jeden|
|     2|   dwa|
+------+------+

scala> df.withColumn("polish", lit(1)).show
+------+------+
|number|polish|
+------+------+
|     1|     1|
|     2|     1|
+------+------+
```

您可以使用withColumn方法添加新列做一个Dataset。

```
val spark: SparkSession = ...
val dataset = spark.range(5)

// Add a new column called "group"
scala> dataset.withColumn("group", 'id % 2).show
+---+-----+
| id|group|
+---+-----+
|  0|    0|
|  1|    1|
|  2|    0|
|  3|    1|
|  4|    0|
+---+-----+
```

### Referencing Column — `apply`Method {#__a_id_apply_a_referencing_column_code_apply_code_method}

```
val spark: SparkSession = ...
case class Word(id: Long, text: String)
val dataset = Seq(Word(0, "hello"), Word(1, "spark")).toDS

scala> val idCol = dataset.apply("id")
idCol: org.apache.spark.sql.Column = id

// or using Scala's magic a little bit
// the following is equivalent to the above explicit apply call
scala> val idCol = dataset("id")
idCol: org.apache.spark.sql.Column = id
```

### Creating Column — `col`method {#__a_id_col_a_creating_column_code_col_code_method}

```
val spark: SparkSession = ...
case class Word(id: Long, text: String)
val dataset = Seq(Word(0, "hello"), Word(1, "spark")).toDS

scala> val textCol = dataset.col("text")
textCol: org.apache.spark.sql.Column = text
```

### `like`Operator {#__a_id_like_a_code_like_code_operator}

| Caution | FIXME |
| :--- | :--- |


```
scala> df("id") like "0"
res0: org.apache.spark.sql.Column = id LIKE 0

scala> df.filter('id like "0").show
+---+-----+
| id| text|
+---+-----+
|  0|hello|
+---+-----+
```

### Symbols As Column Names {#__a_id_symbols_as_column_names_a_symbols_as_column_names}

```
scala> val df = Seq((0, "hello"), (1, "world")).toDF("id", "text")
df: org.apache.spark.sql.DataFrame = [id: int, text: string]

scala> df.select('id)
res0: org.apache.spark.sql.DataFrame = [id: int]

scala> df.select('id).show
+---+
| id|
+---+
|  0|
|  1|
+---+
```

### `over`Operator {#__a_id_over_a_code_over_code_operator}

```
over(window: expressions.WindowSpec): Column
```

over函数定义一个允许窗口计算应用于窗口的窗口列。窗口函数使用WindowSpec定义。

### `cast`Operator {#__a_id_cast_a_code_cast_code_operator}

cast方法将列转换为数据类型。它使得类型安全的maps具有正确类型的Row对象（不是任何）。

```
cast(to: String): Column
cast(to: DataType): Column
```

它使用CatalystSqlParser从其规范字符串表示中解析数据类型。

#### cast Example {#__a_id_cast_example_a_cast_example}

```
scala> val df = Seq((0f, "hello")).toDF("label", "text")
df: org.apache.spark.sql.DataFrame = [label: float, text: string]

scala> df.printSchema
root
 |-- label: float (nullable = false)
 |-- text: string (nullable = true)

// without cast
import org.apache.spark.sql.Row
scala> df.select("label").map { case Row(label) => label.getClass.getName }.show(false)
+---------------+
|value          |
+---------------+
|java.lang.Float|
+---------------+

// with cast
import org.apache.spark.sql.types.DoubleType
scala> df.select(col("label").cast(DoubleType)).map { case Row(label) => label.getClass.getName }.show(false)
+----------------+
|value           |
+----------------+
|java.lang.Double|
+----------------+
```























