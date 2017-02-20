## Row {#_row}

Row是一个有序字段集合的数据抽象，可以通过序数/索引（也称为序数的通用访问），名称（也称为本地原语访问）或使用Scala的模式匹配来访问。一个Row实例可能有也可能没有模式。

Row的特点：

* 长度或大小 - 行知道元素（列）的数量。

* schema - Row知道模式

Row属于org.apache.spark.sql.Row包。

```
import org.apache.spark.sql.Row
```

### Field Access by Index — `apply`and`get`methods {#__a_id_field_access_a_a_id_get_a_a_id_apply_a_field_access_by_index_code_apply_code_and_code_get_code_methods}

可以使用apply或get通过索引（从0开始）访问Row实例的字段。

```
scala> val row = Row(1, "hello")
row: org.apache.spark.sql.Row = [1,hello]

scala> row(1)
res0: Any = hello

scala> row.get(1)
res1: Any = hello
```

| Note | 通过序数的通用访问（使用apply或get）返回Any类型的值。 |
| :---: | :--- |


### Get Field As Type — `getAs`method {#__a_id_getas_a_get_field_as_type_code_getas_code_method}

您可以使用带有索引的getAs查询具有正确类型的字段

```
val row = Row(1, "hello")

scala> row.getAs[Int](0)
res1: Int = 1

scala> row.getAs[String](1)
res2: String = hello
```

### Schema {#__a_id_schema_a_schema}

一个Row实例可以定义一个模式。

| Note | 除非你自己实例化Row（使用Row Object），一个Row总是一个模式。 |
| :---: | :--- |


RowEncoder负责在数据集上的toDF或通过DataFrameReader实例化DataFrame时将模式分配给一行。

### Row Object {#__a_id_row_object_a_row_object}

Row companion对象提供了工厂方法来从一组元素（apply），元素序列（fromSeq）和元组（fromTuple）创建Row实例。

```
scala> Row(1, "hello")
res0: org.apache.spark.sql.Row = [1,hello]

scala> Row.fromSeq(Seq(1, "hello"))
res1: org.apache.spark.sql.Row = [1,hello]

scala> Row.fromTuple((0, "hello"))
res2: org.apache.spark.sql.Row = [0,hello]
```

Row对象可以合并Row实例。

```
scala> Row.merge(Row(1), Row("hello"))
res3: org.apache.spark.sql.Row = [1,hello]
```

它也可以返回一个空的Row实例。

```
scala> Row.empty == Row()
res4: Boolean = true
```

### Pattern Matching on Row {#__a_id_pattern_matching_on_row_a_pattern_matching_on_row}

Row可以用于模式匹配（因为Row Object带有unapplySeq）。

```
scala> Row.unapplySeq(Row(1, "hello"))
res5: Some[Seq[Any]] = Some(WrappedArray(1, hello))

Row(1, "hello") match { case Row(key: Int, value: String) =>
  key -> value
}
```





