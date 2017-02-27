## ML Pipeline Components — Transformers {#_ml_pipeline_components_transformers}

变换器是将DataFrame映射（也称为变换）到另一个DataFrame（两者都称为数据集）的函数对象。

```
transformer: DataFrame =[transform]=> DataFrame
```

变换器准备用于机器学习算法的数据集。它们也非常有助于一般地转换DataFrames（甚至在机器学习空间之外）。

变换器是org.apache.spark.ml.Transformer抽象类的实例，提供变换族方法：

```
transform(dataset: DataFrame): DataFrame
transform(dataset: DataFrame, paramMap: ParamMap): DataFrame
transform(dataset: DataFrame, firstParamPair: ParamPair[_], otherParamPairs: ParamPair[_]*): DataFrame
```

`Transformer`是PipelineStage，因此可以是Pipeline的一部分。

Transformer的一些可用实现：

* StopWordsRemover

* Binarizer

* SQLTransformer

* VectorAssembler — a feature transformer that assembles \(merges\) multiple columns into a \(feature\) vector column.

* UnaryTransformer

  * Tokenizer

  * RegexTokenizer

  * NGram

  * HashingTF

  * OneHotEncoder

* Model

See Custom UnaryTransformer section for a custom`Transformer`implementation.

### StopWordsRemover {#__a_id_stopwordsremover_a_stopwordsremover}

StopWordsRemover是一个机器学习功能变换器，它接受一个字符串数组列，并输出一个字符串数组列，并删除所有定义的停用词。transformer附带一组标准的英语停用词作为默认值（与scikit-learn用法相同，即来自[the Glasgow Information Retrieval Group](http://ir.dcs.gla.ac.uk/resources/linguistic_utils/stop_words)\)。

| Note | 它的工作原理就像是一个UnaryTransformer，但它尚未迁移到扩展类。 |
| :---: | :--- |


StopWordsRemover类属于org.apache.spark.ml.feature包。

```
import org.apache.spark.ml.feature.StopWordsRemover
val stopWords = new StopWordsRemover
```

它接受以下参数：

```
scala> println(stopWords.explainParams)
caseSensitive: whether to do case-sensitive comparison during filtering (default: false)
inputCol: input column name (undefined)
outputCol: output column name (default: stopWords_9c2c0fdd8a68__output)
stopWords: stop words (default: [Ljava.lang.String;@5dabe7c8)
```

| Note | `null`values from the input array are preserved unless adding`null`to`stopWords`explicitly. |
| :--- | :--- |


```
import org.apache.spark.ml.feature.RegexTokenizer
val regexTok = new RegexTokenizer("regexTok")
  .setInputCol("text")
  .setPattern("\\W+")

import org.apache.spark.ml.feature.StopWordsRemover
val stopWords = new StopWordsRemover("stopWords")
  .setInputCol(regexTok.getOutputCol)

val df = Seq("please find it done (and empty)", "About to be rich!", "empty")
  .zipWithIndex
  .toDF("text", "id")

scala> stopWords.transform(regexTok.transform(df)).show(false)
+-------------------------------+---+------------------------------------+-----------------+
|text                           |id |regexTok__output                    |stopWords__output|
+-------------------------------+---+------------------------------------+-----------------+
|please find it done (and empty)|0  |[please, find, it, done, and, empty]|[]               |
|About to be rich!              |1  |[about, to, be, rich]               |[rich]           |
|empty                          |2  |[empty]                             |[]               |
+-------------------------------+---+------------------------------------+-----------------+
```

### Binarizer {#__a_id_binarizer_a_binarizer}

Binarizer是一个Transformer，将输入列中的值拆分为两组 - 大于阈值的值为“一”，其他值为“zeros”。

它与DataFrames与DoubleType或VectorUDT的输入列。结果输出列的类型与输入列的类型匹配，即DoubleType或VectorUDT。

```
import org.apache.spark.ml.feature.Binarizer
val bin = new Binarizer()
  .setInputCol("rating")
  .setOutputCol("label")
  .setThreshold(3.5)

scala> println(bin.explainParams)
inputCol: input column name (current: rating)
outputCol: output column name (default: binarizer_dd9710e2a831__output, current: label)
threshold: threshold used to binarize continuous features (default: 0.0, current: 3.5)

val doubles = Seq((0, 1d), (1, 1d), (2, 5d)).toDF("id", "rating")

scala> bin.transform(doubles).show
+---+------+-----+
| id|rating|label|
+---+------+-----+
|  0|   1.0|  0.0|
|  1|   1.0|  0.0|
|  2|   5.0|  1.0|
+---+------+-----+

import org.apache.spark.mllib.linalg.Vectors
val denseVec = Vectors.dense(Array(4.0, 0.4, 3.7, 1.5))
val vectors = Seq((0, denseVec)).toDF("id", "rating")

scala> bin.transform(vectors).show
+---+-----------------+-----------------+
| id|           rating|            label|
+---+-----------------+-----------------+
|  0|[4.0,0.4,3.7,1.5]|[1.0,0.0,1.0,0.0]|
+---+-----------------+-----------------+
```

### SQLTransformer {#__a_id_sqltransformer_a_sqltransformer}

SQLTransformer是一个变换器，通过执行SELECT ... FROM THIS执行转换，THIS是为输入数据集注册的基础临时表。

在内部，THIS被临时表的随机名替换（使用registerTempTable）。

| Note | 它已经从Spark 1.6.0开始可用。 |
| :---: | :--- |


它要求SELECT查询使用对应于临时表的THIS，并简单地使用sql方法执行必需语句。

您必须使用setStatement方法指定必需的语句参数。

```
import org.apache.spark.ml.feature.SQLTransformer
val sql = new SQLTransformer()

// dataset to work with
val df = Seq((0, s"""hello\tworld"""), (1, "two  spaces inside")).toDF("label", "sentence")

scala> sql.setStatement("SELECT sentence FROM __THIS__ WHERE label = 0").transform(df).show
+-----------+
|   sentence|
+-----------+
|hello	world|
+-----------+

scala> println(sql.explainParams)
statement: SQL statement (current: SELECT sentence FROM __THIS__ WHERE label = 0)
```

### VectorAssembler {#__a_id_vectorassembler_a_vectorassembler}

VectorAssembler是将多列组装（合并）为（特征）向量列的要素变换器。

它支持NumericType，BooleanType和VectorUDT类型的列。Doubles are passed on untouched。其他数字类型和布尔值被转换为双精度。

```
import org.apache.spark.ml.feature.VectorAssembler
val vecAssembler = new VectorAssembler()

scala> print(vecAssembler.explainParams)
inputCols: input column names (undefined)
outputCol: output column name (default: vecAssembler_5ac31099dbee__output)

final case class Record(id: Int, n1: Int, n2: Double, flag: Boolean)
val ds = Seq(Record(0, 4, 2.0, true)).toDS

scala> ds.printSchema
root
 |-- id: integer (nullable = false)
 |-- n1: integer (nullable = false)
 |-- n2: double (nullable = false)
 |-- flag: boolean (nullable = false)

val features = vecAssembler
  .setInputCols(Array("n1", "n2", "flag"))
  .setOutputCol("features")
  .transform(ds)

scala> features.printSchema
root
 |-- id: integer (nullable = false)
 |-- n1: integer (nullable = false)
 |-- n2: double (nullable = false)
 |-- flag: boolean (nullable = false)
 |-- features: vector (nullable = true)


scala> features.show
+---+---+---+----+-------------+
| id| n1| n2|flag|     features|
+---+---+---+----+-------------+
|  0|  4|2.0|true|[4.0,2.0,1.0]|
+---+---+---+----+-------------+
```

### UnaryTransformers {#__a_id_unarytransformer_a_unarytransformers}

UnaryTransformer抽象类是一种专用的变换器，它将变换应用于一个输入列，并将结果写入另一个（通过附加一个新列）。

每个UnaryTransformer使用以下“链”方法定义输入和输出列（它们返回它们被执行的变换器，因此是可链接的）：

* `setInputCol(value: String)`

* `setOutputCol(value: String)`

每个UnaryTransformer在执行transformSchema（schema：StructType）（即PipelineStage合同的一部分）时调用validateInputType。

| Note | UnaryTransformer是一个PipelineStage。 |
| :---: | :--- |


当调用transform时，它首先调用transformSchema（启用DEBUG日志记录），然后作为调用受保护的抽象createTransformFunc的结果添加列。

| Note | createTransformFunc函数是抽象的，由具体的UnaryTransformer对象定义。 |
| :---: | :--- |


在内部，transform方法使用Spark SQL的udf来定义一个函数（基于上面描述的createTransformFunc函数），它将创建新的输出列（使用适当的outputDataType）。 UDF稍后应用于输入DataFrame的输入列，结果将成为输出列（使用DataFrame.withColumn方法）。

| Note | 使用Spark SQL中的udf和withColumn方法展示了Spark模块之间的良好集成：MLlib和SQL。 |
| :---: | :--- |


以下是spark.ml中的UnaryTransformer实现：

* Tokenizer，将字符串列转换为小写，然后用空格分隔。

* RegexTokenizer提取tokens。

* NGram将输入的字符串数组转换为n元数组。

* HashingTF将术语序列映射到它们的术语频率（参考[SPARK-13998 HashingTF should extend UnaryTransformer](https://issues.apache.org/jira/browse/SPARK-13998)\)

* OneHotEncoder，用于将标签索引的数字输入列映射到二进制向量列。

#### RegexTokenizer {#__a_id_regextokenizer_a_regextokenizer}

RegexTokenizer是一个UnaryTransformer，它将String标记为String的集合。

```
import org.apache.spark.ml.feature.RegexTokenizer
val regexTok = new RegexTokenizer()

// dataset to transform with tabs and spaces
val df = Seq((0, s"""hello\tworld"""), (1, "two  spaces inside")).toDF("label", "sentence")

val tokenized = regexTok.setInputCol("sentence").transform(df)

scala> tokenized.show(false)
+-----+------------------+-----------------------------+
|label|sentence          |regexTok_810b87af9510__output|
+-----+------------------+-----------------------------+
|0    |hello	world       |[hello, world]               |
|1    |two  spaces inside|[two, spaces, inside]        |
+-----+------------------+-----------------------------+
```

| Note | Read the official scaladoc for [org.apache.spark.ml.feature.RegexTokenizer](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.ml.feature.RegexTokenizer). |
| :--- | :--- |


它支持minTokenLength参数，它是您可以使用setMinTokenLength方法更改的最小tokens长度。它只是过滤掉较小的标记，默认为1。

```
// see above to set up the vals

scala> rt.setInputCol("line").setMinTokenLength(6).transform(df).show
+-----+--------------------+-----------------------------+
|label|                line|regexTok_8c74c5e8b83a__output|
+-----+--------------------+-----------------------------+
|    1|         hello world|                           []|
|    2|yet another sentence|          [another, sentence]|
+-----+--------------------+-----------------------------+
```

它具有指示是以正则表达式在间隔（true）上split还是匹配tokens（false）的间隔参数。您可以使用setGaps设置它。它默认为true。

当设置为true（即在间隔上进行split）时，它使用Regex.split，而Regex.findAllIn为false。

```
scala> rt.setInputCol("line").setGaps(false).transform(df).show
+-----+--------------------+-----------------------------+
|label|                line|regexTok_8c74c5e8b83a__output|
+-----+--------------------+-----------------------------+
|    1|         hello world|                           []|
|    2|yet another sentence|          [another, sentence]|
+-----+--------------------+-----------------------------+

scala> rt.setInputCol("line").setGaps(false).setPattern("\\W").transform(df).show(false)
+-----+--------------------+-----------------------------+
|label|line                |regexTok_8c74c5e8b83a__output|
+-----+--------------------+-----------------------------+
|1    |hello world         |[]                           |
|2    |yet another sentence|[another, sentence]          |
+-----+--------------------+-----------------------------+
```

它有模式参数，它是用于标记化的正则表达式。它使用Scala的.r方法将字符串转换为regex。使用setPattern设置它。它默认为\\ s +。

它有toLowercase参数，指示是否在标记化之前将所有字符转换为小写。使用setToLowercase更改它。它默认为true。

#### NGram {#__a_id_ngram_a_ngram}

在本例中，您使用org.apache.spark.ml.feature.NGram将字符串的输入集合转换为n-gram（n个字）的集合。

```
import org.apache.spark.ml.feature.NGram

val bigram = new NGram("bigrams")
val df = Seq((0, Seq("hello", "world"))).toDF("id", "tokens")
bigram.setInputCol("tokens").transform(df).show

+---+--------------+---------------+
| id|        tokens|bigrams__output|
+---+--------------+---------------+
|  0|[hello, world]|  [hello world]|
+---+--------------+---------------+
```

#### HashingTF {#__a_id_hashingtf_a_hashingtf}

变换器的另一个例子是在ArrayType的Column上工作的org.apache.spark.ml.feature.HashingTF。

它将输入列的行转换为稀疏项频率向量。

```
import org.apache.spark.ml.feature.HashingTF
val hashingTF = new HashingTF()
  .setInputCol("words")
  .setOutputCol("features")
  .setNumFeatures(5000)

// see above for regexTok transformer
val regexedDF = regexTok.transform(df)

// Use HashingTF
val hashedDF = hashingTF.transform(regexedDF)

scala> hashedDF.show(false)
+---+------------------+---------------------+-----------------------------------+
|id |text              |words                |features                           |
+---+------------------+---------------------+-----------------------------------+
|0  |hello	world       |[hello, world]       |(5000,[2322,3802],[1.0,1.0])       |
|1  |two  spaces inside|[two, spaces, inside]|(5000,[276,940,2533],[1.0,1.0,1.0])|
+---+------------------+---------------------+-----------------------------------+
```

输出列的名称是可选的，如果未指定，它将成为带有\_\_output后缀的HashingTF对象的标识符。

```
scala> hashingTF.uid
res7: String = hashingTF_fe3554836819

scala> hashingTF.transform(regexDF).show(false)
+---+------------------+---------------------+-------------------------------------------+
|id |text              |words                |hashingTF_fe3554836819__output             |
+---+------------------+---------------------+-------------------------------------------+
|0  |hello	world       |[hello, world]       |(262144,[71890,72594],[1.0,1.0])           |
|1  |two  spaces inside|[two, spaces, inside]|(262144,[53244,77869,115276],[1.0,1.0,1.0])|
+---+------------------+---------------------+-------------------------------------------+
```

#### OneHotEncoder {#__a_id_onehotencoder_a_onehotencoder}

OneHotEncoder是一种Tokenizer，它将标签索引的数字输入列映射到二进制向量列。

```
// dataset to transform
val df = Seq(
  (0, "a"), (1, "b"),
  (2, "c"), (3, "a"),
  (4, "a"), (5, "c"))
  .toDF("label", "category")
import org.apache.spark.ml.feature.StringIndexer
val indexer = new StringIndexer().setInputCol("category").setOutputCol("cat_index").fit(df)
val indexed = indexer.transform(df)

import org.apache.spark.sql.types.NumericType

scala> indexed.schema("cat_index").dataType.isInstanceOf[NumericType]
res0: Boolean = true

import org.apache.spark.ml.feature.OneHotEncoder
val oneHot = new OneHotEncoder()
  .setInputCol("cat_index")
  .setOutputCol("cat_vec")

val oneHotted = oneHot.transform(indexed)

scala> oneHotted.show(false)
+-----+--------+---------+-------------+
|label|category|cat_index|cat_vec      |
+-----+--------+---------+-------------+
|0    |a       |0.0      |(2,[0],[1.0])|
|1    |b       |2.0      |(2,[],[])    |
|2    |c       |1.0      |(2,[1],[1.0])|
|3    |a       |0.0      |(2,[0],[1.0])|
|4    |a       |0.0      |(2,[0],[1.0])|
|5    |c       |1.0      |(2,[1],[1.0])|
+-----+--------+---------+-------------+

scala> oneHotted.printSchema
root
 |-- label: integer (nullable = false)
 |-- category: string (nullable = true)
 |-- cat_index: double (nullable = true)
 |-- cat_vec: vector (nullable = true)

scala> oneHotted.schema("cat_vec").dataType.isInstanceOf[VectorUDT]
res1: Boolean = true
```

#### Custom UnaryTransformer {#__a_id_custom_transformer_a_custom_unarytransformer}

下面的类是一个自定义的UnaryTransformer，它使用大写字母来转换单词。

```
package pl.japila.spark

import org.apache.spark.ml._
import org.apache.spark.ml.util.Identifiable
import org.apache.spark.sql.types._

class UpperTransformer(override val uid: String)
    extends UnaryTransformer[String, String, UpperTransformer] {

  def this() = this(Identifiable.randomUID("upper"))

  override protected def validateInputType(inputType: DataType): Unit = {
    require(inputType == StringType)
  }

  protected def createTransformFunc: String => String = {
    _.toUpperCase
  }

  protected def outputDataType: DataType = StringType
}
```

给定一个DataFrame，你可以使用它如下：

```
val upper = new UpperTransformer

scala> upper.setInputCol("text").transform(df).show
+---+-----+--------------------------+
| id| text|upper_0b559125fd61__output|
+---+-----+--------------------------+
|  0|hello|                     HELLO|
|  1|world|                     WORLD|
+---+-----+--------------------------+
```







