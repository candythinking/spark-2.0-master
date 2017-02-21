## Data Types {#__a_id_datatype_a_data_types}

DataType抽象类是Spark SQL中所有内置数据类型的基本类型，例如。字符串，长。

Table 1. Built-In Data Types

|   | Functions | Description |
| :--- | :--- | :--- |
| **Atomic Types** | `TimestampType` | Represents`java.sql.Timestamp`values. |
|  | `StringType` |  |
|  | `BooleanType` |  |
|  | `DateType` |  |
|  | `BinaryType` |  |
| **Fractional Types** | `DoubleType` |  |
|  | `FloatType` |  |
| **Integral Types** | `ByteType` |  |
|  | `IntegerType` |  |
|  | `LongType` |  |
|  | `ShortType` |  |
|  | `CalendarIntervalType` |  |
|  | [StructType](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/spark-sql-StructType.html) |  |
|  | `MapType` |  |
|  | `ArrayType`+ |  |
|  | `NullType` |  |

您可以扩展类型系统并创建自己的用户定义类型（UDT）。

DataType Contract定义了构建SQL，JSON和字符串表示的方法。

| Note | DataType（和具体的Spark SQL类型）存在于org.apache.spark.sql.types包中。 |
| :---: | :--- |


```
import org.apache.spark.sql.types.StringType

scala> StringType.json
res0: String = "string"

scala> StringType.sql
res1: String = STRING

scala> StringType.catalogString
res2: String = string
```

您应该在代码中使用DataTypes对象来创建复杂的Spark SQL类型，即数组或映射。

```
import org.apache.spark.sql.types.DataTypes

scala> val arrayType = DataTypes.createArrayType(BooleanType)
arrayType: org.apache.spark.sql.types.ArrayType = ArrayType(BooleanType,true)

scala> val mapType = DataTypes.createMapType(StringType, LongType)
mapType: org.apache.spark.sql.types.MapType = MapType(StringType,LongType,true)
```

DataType支持使用unapply方法的Scala的模式匹配。

```
???
```

### DataType Contract {#__a_id_contract_a_datatype_contract}

Spark SQL中的任何类型遵循DataType合约，这意味着类型定义以下方法：

* json和prettyJson来构建数据类型的JSON表示

* defaultSize知道类型的默认值的大小

* simpleString和catalogString构建用户友好的字符串表示（后者用于外部目录）

* `sql`来构建SQL表示

    import org.apache.spark.sql.types.DataTypes._

    val maps = StructType(
      StructField("longs2strings", createMapType(LongType, StringType), false) :: Nil)

    scala> maps.prettyJson
    res0: String =
    {
      "type" : "struct",
      "fields" : [ {
        "name" : "longs2strings",
        "type" : {
          "type" : "map",
          "keyType" : "long",
          "valueType" : "string",
          "valueContainsNull" : true
        },
        "nullable" : false,
        "metadata" : { }
      } ]
    }

    scala> maps.defaultSize
    res1: Int = 2800

    scala> maps.simpleString
    res2: String = struct<longs2strings:map<bigint,string>>

    scala> maps.catalogString
    res3: String = struct<longs2strings:map<bigint,string>>

    scala> maps.sql
    res4: String = STRUCT<`longs2strings`: MAP<BIGINT, STRING>>

### DataTypes — Factory Methods for Data Types {#__a_id_datatypes_a_datatypes_factory_methods_for_data_types}

DataTypes是一个Java类，具有访问简单的方法或在Spark SQL中创建复杂的DataType类型，即数组和映射。

| Tip | 建议使用DataTypes类在模式中定义DataType类型。 |
| :---: | :--- |


DataTypes存在于org.apache.spark.sql.types包中。

```
import org.apache.spark.sql.types.DataTypes

scala> val arrayType = DataTypes.createArrayType(BooleanType)
arrayType: org.apache.spark.sql.types.ArrayType = ArrayType(BooleanType,true)

scala> val mapType = DataTypes.createMapType(StringType, LongType)
mapType: org.apache.spark.sql.types.MapType = MapType(StringType,LongType,true)
```

简单的DataType类型本身，即StringType或CalendarIntervalType，自带的Scala的case对象及其定义。

您还可以导入类型包并具有对类型的访问权限。

```
import org.apache.spark.sql.types._
```

### UDTs — User-Defined Types {#__a_id_user_defined_types_a_udts_user_defined_types}

| Caution | FIXME |
| :--- | :--- |
















