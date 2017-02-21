## Schema — Structure of Data {#_schema_structure_of_data}

schema是对数据结构的描述（它们一起在Spark SQL中创建一个数据集）。它可以是隐式的（在运行时推断）或显式的（在编译时已知）。

使用StructType描述schema，它是StructField对象的集合（依次是名称，类型和可空性分类器的元组）。

StructType和StructField属于org.apache.spark.sql.types包。

```
import org.apache.spark.sql.types.StructType
val schemaUntyped = new StructType()
  .add("a", "int")
  .add("b", "string")
```

您可以使用SQL类型的规范字符串表示来描述模式中的类型（在编译类型中固有的类型），或者使用org.apache.spark.sql.types包中的类型安全类型。

```
// it is equivalent to the above expression
import org.apache.spark.sql.types.{IntegerType, StringType}
val schemaTyped = new StructType()
  .add("a", IntegerType)
  .add("b", StringType)
```

| Tip | 阅读Spark SQL中的SQL Parser Framework，了解负责解析数据类型的CatalystSqlParser。 |
| :---: | :--- |


但是，建议使用带有静态方法的单例DataTypes类来创建模式类型。

```
import org.apache.spark.sql.types.DataTypes._
val schemaWithMap = StructType(
  StructField("map", createMapType(LongType, StringType), false) :: Nil)
```

StructType提供了printTreeString，使得模式更易于用户使用。

```
scala> schemaTyped.printTreeString
root
 |-- a: integer (nullable = true)
 |-- b: string (nullable = true)

scala> schemaWithMap.printTreeString
root
|-- map: map (nullable = false)
|    |-- key: long
|    |-- value: string (valueContainsNull = true)

// You can use prettyJson method on any DataType
scala> println(schema1.prettyJson)
{
 "type" : "struct",
 "fields" : [ {
   "name" : "a",
   "type" : "integer",
   "nullable" : true,
   "metadata" : { }
 }, {
   "name" : "b",
   "type" : "string",
   "nullable" : true,
   "metadata" : { }
 } ]
}
```

从Spark 2.0开始，您可以使用编码器描述强类型数据集的模式。

```
import org.apache.spark.sql.Encoders

scala> Encoders.INT.schema.printTreeString
root
 |-- value: integer (nullable = true)

scala> Encoders.product[(String, java.sql.Timestamp)].schema.printTreeString
root
|-- _1: string (nullable = true)
|-- _2: timestamp (nullable = true)

case class Person(id: Long, name: String)
scala> Encoders.product[Person].schema.printTreeString
root
 |-- id: long (nullable = false)
 |-- name: string (nullable = true)
```

### Implicit Schema {#__a_id_implicit_schema_a_implicit_schema}

```
val df = Seq((0, s"""hello\tworld"""), (1, "two  spaces inside")).toDF("label", "sentence")

scala> df.printSchema
root
 |-- label: integer (nullable = false)
 |-- sentence: string (nullable = true)

scala> df.schema
res0: org.apache.spark.sql.types.StructType = StructType(StructField(label,IntegerType,false), StructField(sentence,StringType,true))

scala> df.schema("label").dataType
res1: org.apache.spark.sql.types.DataType = IntegerType
```



