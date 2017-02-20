## `RowEncoder` — DataFrame Encoder {#__a_id_rowencoder_a_code_rowencoder_code_dataframe_encoder}

RowEncoder是Encoder框架的一部分，用作DataFrames的编码器，即Dataset \[Row\] - Rows的数据集。

| Note | DataFrame类型是Dataset \[Row\]的纯类型别名，期望一个Encoder \[Row\]可用于作为RowEncoder本身的作用域。 |
| :---: | :--- |


RowEncoder是Scala中的一个对象，带有apply和其他工厂方法。

RowEncoder可以从模式（使用apply方法）创建ExpressionEncoder \[Row\]。

```
import org.apache.spark.sql.types._
val schema = StructType(
  StructField("id", LongType, nullable = false) ::
  StructField("name", StringType, nullable = false) :: Nil)

import org.apache.spark.sql.catalyst.encoders.RowEncoder
scala> val encoder = RowEncoder(schema)
encoder: org.apache.spark.sql.catalyst.encoders.ExpressionEncoder[org.apache.spark.sql.Row] = class[id[0]: bigint, name[0]: string]

// RowEncoder is never flat
scala> encoder.flat
res0: Boolean = false
```

RowEncoder对象属于org.apache.spark.sql.catalyst.encoders包。

### Creating ExpressionEncoder of Rows — `apply`method {#__a_id_apply_a_creating_expressionencoder_of_rows_code_apply_code_method}

```
apply(schema: StructType): ExpressionEncoder[Row]
```

从输入StructType（作为模式）应用构建Row的ExpressionEncoder，即ExpressionEncoder \[Row\]。

在内部，apply为Row类型创建一个BoundReference，并为输入模式返回一个ExpressionEncoder \[Row\]，一个CreateNamedStruct序列化器（使用serializerFor内部方法），一个模式的解串器和Row类型。

### `serializerFor`Internal Method {#__a_id_serializerfor_a_code_serializerfor_code_internal_method}

```
serializerFor(inputObject: Expression, inputType: DataType): Expression
```

serializerFor创建一个假定为CreateNamedStruct的表达式。

serializerFor接受inputType和：

1.返回与原生类型一样的输入inputObject，即NullType，BooleanType，ByteType，ShortType，IntegerType，LongType，FloatType，DoubleType，BinaryType，CalendarIntervalType。

2.对于UserDefinedTypes，它从SQLUserDefinedType注记或UDTRegistration对象获取UDT类，并返回一个带有Invoke的表达式，以在UDT类的NewInstance上调用serialize方法。

3.对于TimestampType，它返回一个带有StaticInvoke的表达式，以从DateTimeUtils类调用fromJavaTimestamp。

4.....fixme

### `StaticInvoke`NonSQLExpression {#__a_id_staticinvoke_a_code_staticinvoke_code_nonsqlexpression}

```
case class StaticInvoke(
  staticObject: Class[_],
  dataType: DataType,
  functionName: String,
  arguments: Seq[Expression] = Nil,
  propagateNull: Boolean = true) extends NonSQLExpression
```

StaticInvoke是一个没有SQL表示的表达式，表示Scala或Java中的静态方法调用。它支持生成Java代码来评估自身。

StaticInvoke在带有参数输入参数的staticObject对象上调用functionName静态方法，以产生dataType类型的值。如果启用了propagateNull并且任何参数为null，则null是结果（不调用functionName函数）。

StaticInvoke用于RowEncoder和Java的编码器。

```
import org.apache.spark.sql.types._
val schema = StructType(
  StructField("id", LongType, nullable = false) ::
  StructField("name", StringType, nullable = false) :: Nil)

import org.apache.spark.sql.catalyst.encoders.RowEncoder
val encoder = RowEncoder(schema)

scala> encoder.serializer
res0: Seq[org.apache.spark.sql.catalyst.expressions.Expression] = List(validateexternaltype(getexternalrowfield(assertnotnull(input[0, org.apache.spark.sql.Row, true], top level row object), 0, id), LongType) AS id#69L, staticinvoke(class org.apache.spark.unsafe.types.UTF8String, StringType, fromString, validateexternaltype(getexternalrowfield(assertnotnull(input[0, org.apache.spark.sql.Row, true], top level row object), 1, name), StringType), true) AS name#70)
```



