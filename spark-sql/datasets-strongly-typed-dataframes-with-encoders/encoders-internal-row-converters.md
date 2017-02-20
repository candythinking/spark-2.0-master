## Encoders — Internal Row Converters {#_encoders_internal_row_converters}

编码器是Spark SQL 2.0中序列化和反序列化（SerDe）框架中的基本概念。 Spark SQL使用SerDe框架为IO提供高效的时间和空间。

| Tip | Spark已经从Hive SerDe库借用了这个想法，所以也许值得熟悉Hive一点。 |
| :---: | :--- |


Encoders are modelled in Spark SQL 2.0 as`Encoder[T]`trait.

```
trait Encoder[T] extends Serializable {
  def schema: StructType
  def clsTag: ClassTag[T]
}
```

类型T代表编码器\[T\]可以处理的记录的类型。 T类型的编码器，即Encoder \[T\]，用于将任何JVM对象或T类型的原语（可能是您的域对象）转换（编码和解码）到Spark SQL的InternalRow，这是内部二进制行格​​式表示（使用Catalyst表达式和代码生成）。

| Note | 编码器也称为“Dataset中的serde表达式的容器”。 |
| :---: | :--- |


| Note | Spark SQL 2.0中Encoder trait的唯一实现是ExpressionEncoder。 |
| :---: | :--- |


编码器是任何Dataset\[T\]（具有类型T的记录）的整数（和内部）部分，具有编码器\[T\]，用于对该数据集的记录进行串行化和反序列化。

| Note | Dataset \[T\]类型是具有类型参数T的Scala类型构造函数。因此，Encoder \[T\]处理T到内部表示的序列化和反序列化。 |
| :---: | :--- |


编码器知道记录的模式。这是他们如何提供显着更快的序列化和反序列化（与默认的Java或Kryo序列化器相比）。

```
// The domain object for your records in a large dataset
case class Person(id: Long, name: String)

import org.apache.spark.sql.Encoders

scala> val personEncoder = Encoders.product[Person]
personEncoder: org.apache.spark.sql.Encoder[Person] = class[id[0]: bigint, name[0]: string]

scala> personEncoder.schema
res0: org.apache.spark.sql.types.StructType = StructType(StructField(id,LongType,false), StructField(name,StringType,true))

scala> personEncoder.clsTag
res1: scala.reflect.ClassTag[Person] = Person

import org.apache.spark.sql.catalyst.encoders.ExpressionEncoder

scala> val personExprEncoder = personEncoder.asInstanceOf[ExpressionEncoder[Person]]
personExprEncoder: org.apache.spark.sql.catalyst.encoders.ExpressionEncoder[Person] = class[id[0]: bigint, name[0]: string]

// ExpressionEncoders may or may not be flat
scala> personExprEncoder.flat
res2: Boolean = false

// The Serializer part of the encoder
scala> personExprEncoder.serializer
res3: Seq[org.apache.spark.sql.catalyst.expressions.Expression] = List(assertnotnull(input[0, Person, true], top level non-flat input object).id AS id#0L, staticinvoke(class org.apache.spark.unsafe.types.UTF8String, StringType, fromString, assertnotnull(input[0, Person, true], top level non-flat input object).name, true) AS name#1)

// The Deserializer part of the encoder
scala> personExprEncoder.deserializer
res4: org.apache.spark.sql.catalyst.expressions.Expression = newInstance(class Person)

scala> personExprEncoder.namedExpressions
res5: Seq[org.apache.spark.sql.catalyst.expressions.NamedExpression] = List(assertnotnull(input[0, Person, true], top level non-flat input object).id AS id#2L, staticinvoke(class org.apache.spark.unsafe.types.UTF8String, StringType, fromString, assertnotnull(input[0, Person, true], top level non-flat input object).name, true) AS name#3)

// A record in a Dataset[Person]
// A mere instance of Person case class
// There could be a thousand of Person in a large dataset
val jacek = Person(0, "Jacek")

// Serialize a record to the internal representation, i.e. InternalRow
scala> val row = personExprEncoder.toRow(jacek)
row: org.apache.spark.sql.catalyst.InternalRow = [0,0,1800000005,6b6563614a]

// Spark uses InternalRows internally for IO
// Let's deserialize it to a JVM object, i.e. a Scala object
import org.apache.spark.sql.catalyst.dsl.expressions._

// in spark-shell there are competing implicits
// That's why DslSymbol is used explicitly in the following line
scala> val attrs = Seq(DslSymbol('id).long, DslSymbol('name).string)
attrs: Seq[org.apache.spark.sql.catalyst.expressions.AttributeReference] = List(id#8L, name#9)

scala> val jacekReborn = personExprEncoder.resolveAndBind(attrs).fromRow(row)
jacekReborn: Person = Person(0,Jacek)

// Are the jacek instances same?
scala> jacek == jacekReborn
res6: Boolean = true
```

您可以使用Encoders对象的静态方法创建自定义编码器。但请注意，普通Scala类型的编码器及其产品类型已在implicits对象中提供。

```
val spark = SparkSession.builder.getOrCreate()
import spark.implicits._
```

| Note | 默认编码器已在spark-shell中导入。 |
| :---: | :--- |


编码器按名称将（数据集的）列映射到（JVM对象的）字段。通过Encoders，您可以将JVM对象桥接到数据源（CSV，JDBC，Parquet，Avro，JSON，Cassandra，Elasticsearch，memsql），反之亦然。

| Note | 在Spark SQL 2.0 DataFrame类型是Dataset\[row\]的纯类型别名，RowEncoder是编码器。 |
| :---: | :--- |


### ExpressionEncoder {#__a_id_expressionencoder_a_expressionencoder}

```
case class ExpressionEncoder[T](
  schema: StructType,
  flat: Boolean,
  serializer: Seq[Expression],
  deserializer: Expression,
  clsTag: ClassTag[T])
extends Encoder[T]
```

ExpressionEncoder是Spark 2.0中Encoder trait的唯一实现，具有附加属性，即平面，一个或多个序列化器和解串器表达式。

ExpressionEncoder可以是平的，这种情况下，序列化器只有一个Catalyst表达式。

序列化表达式（Serializer expressions）用于将类型T的对象编码为InternalRow。假设所有的序列化表达式包含至少一个和相同的BoundReference。

反序列化表达式（Deserializer expression）用于将InternalRow解码为类型T的对象。

在内部，ExpressionEncoder创建一个UnsafeProjection（对于输入serializer），一个InternalRow（大小为1）和一个安全的投影（对于输入deserializer）。它们都是编码器的内部惰性属性。

### Creating Custom Encoders \(Encoders object\) {#__a_id_creating_encoders_a_a_id_encoders_a_creating_custom_encoders_encoders_object}

`Encoders`编码器工厂对象定义创建编码器实例的方法。

```
import org.apache.spark.sql.Encoders

scala> Encoders.LONG
res1: org.apache.spark.sql.Encoder[Long] = class[value[0]: bigint]
```

您可以找到为Java对象类型创建编码器的方法，例如Boolean，Integer，Long，Double，String，java.sql.Timestamp或Byte数组，可以组合为Java bean类创建更高级的编码器（使用bean方法）。

```
import org.apache.spark.sql.Encoders

scala> Encoders.STRING
res2: org.apache.spark.sql.Encoder[String] = class[value[0]: string]
```

您还可以创建基于Kryo或Java序列化器的编码器。

```
import org.apache.spark.sql.Encoders

case class Person(id: Int, name: String, speaksPolish: Boolean)

scala> Encoders.kryo[Person]
res3: org.apache.spark.sql.Encoder[Person] = class[value[0]: binary]

scala> Encoders.javaSerialization[Person]
res5: org.apache.spark.sql.Encoder[Person] = class[value[0]: binary]
```

您可以为Scala的元组和case类创建编码器，Int，Long，Double等。

```
import org.apache.spark.sql.Encoders

scala> Encoders.tuple(Encoders.scalaLong, Encoders.STRING, Encoders.scalaBoolean)
res9: org.apache.spark.sql.Encoder[(Long, String, Boolean)] = class[_1[0]: bigint, _2[0]: string, _3[0]: boolean]
```

### Further reading or watching {#__a_id_i_want_more_a_further_reading_or_watching}

* \(video\)[Modern Spark DataFrame and Dataset \(Intermediate Tutorial\)](https://youtu.be/_1byVWTEK1s) by [Adam Breindel](https://twitter.com/adbreind) from Databricks.







