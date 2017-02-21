## StructType Data Type {#_structtype_data_type}

StructType是Spark SQL中的内置数据类型，用于表示StructField的集合。

StructType是一个Seq \[StructField\]，因此所有的Seq在这里同样适用

```
scala> schemaTyped.foreach(println)
StructField(a,IntegerType,true)
StructField(b,StringType,true)
```

Read the official documentation of [scala.collection.Seq](http://www.scala-lang.org/api/current/scala/collection/Seq.html).

您可以比较两个StructType实例以查看它们是否相等。

```
import org.apache.spark.sql.types.StructType

val schemaUntyped = new StructType()
  .add("a", "int")
  .add("b", "string")

import org.apache.spark.sql.types.{IntegerType, StringType}
val schemaTyped = new StructType()
  .add("a", IntegerType)
  .add("b", StringType)

scala> schemaUntyped == schemaTyped
res0: Boolean = true
```

StructType在查询计划或SQL中呈现为&lt;struct&gt;或STRUCT。

### Adding Fields to Schema — `add`methods {#__a_id_add_a_adding_fields_to_schema_code_add_code_methods}

您可以向您的StructType添加一个新的StructField。有不同的add方法的变体，它们都为添加了字段的新的StructType做出。

```
add(field: StructField): StructType
add(name: String, dataType: DataType): StructType
add(name: String, dataType: DataType, nullable: Boolean): StructType
add(
  name: String,
  dataType: DataType,
  nullable: Boolean,
  metadata: Metadata): StructType
add(
  name: String,
  dataType: DataType,
  nullable: Boolean,
  comment: String): StructType
add(name: String, dataType: String): StructType
add(name: String, dataType: String, nullable: Boolean): StructType
add(
  name: String,
  dataType: String,
  nullable: Boolean,
  metadata: Metadata): StructType
add(
  name: String,
  dataType: String,
  nullable: Boolean,
  comment: String): StructType
```

### DataType Name Conversions {#__a_id_sql_a_a_id_catalogstring_a_a_id_simplestring_a_datatype_name_conversions}

```
simpleString: String
catalogString: String
sql: String
```

StructType作为自定义DataType用于查询计划或SQL。它可以使用simpleString，catalogString或sql（参见DataType Contract）呈现自身。

    scala> schemaTyped.simpleString
    res0: String = struct<a:int,b:string>

    scala> schemaTyped.catalogString
    res1: String = struct<a:int,b:string>

    scala> schemaTyped.sql
    res2: String = STRUCT<`a`: INT, `b`: STRING>

### Accessing StructField — `apply`method {#__a_id_apply_a_accessing_structfield_code_apply_code_method}

```
apply(name: String): StructField
```

StructType定义了自己的apply方法，使您可以通过名称轻松访问StructField。

```
scala> schemaTyped.printTreeString
root
 |-- a: integer (nullable = true)
 |-- b: string (nullable = true)

scala> schemaTyped("a")
res4: org.apache.spark.sql.types.StructField = StructField(a,IntegerType,true)
```

### Creating StructType from Existing StructType — `apply`method {#__a_id_apply_seq_a_creating_structtype_from_existing_structtype_code_apply_code_method}

```
apply(names: Set[String]): StructType
```

这种应用变体允许您从现有的仅具有名称的StructType中创建一个StructType。

```
scala> schemaTyped(names = Set("a"))
res0: org.apache.spark.sql.types.StructType = StructType(StructField(a,IntegerType,true))
```

当无法找到字段时，它将抛出IllegalArgumentException异常。

```
scala> schemaTyped(names = Set("a", "c"))
java.lang.IllegalArgumentException: Field c does not exist.
  at org.apache.spark.sql.types.StructType.apply(StructType.scala:275)
  ... 48 elided
```

### Displaying Schema As Tree — `printTreeString`method {#__a_id_printtreestring_a_displaying_schema_as_tree_code_printtreestring_code_method}

```
printTreeString(): Unit
```

printTreeString将模式打印到标准输出。

```
scala> schemaTyped.printTreeString
root
 |-- a: integer (nullable = true)
 |-- b: string (nullable = true)
```

在内部，它使用treeString方法来构建树然后println它。











