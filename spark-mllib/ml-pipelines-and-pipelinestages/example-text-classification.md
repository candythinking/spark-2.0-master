## Example — Text Classification {#_example_text_classification}

| Note | The example was inspired by the video [Building, Debugging, and Tuning Spark Machine Learning Pipelines - Joseph Bradley \(Databricks\)](https://youtu.be/OednhGRp938). |
| :--- | :--- |


问题：给定文本文档，将其分类为科学或非科学文档。

加载输入数据时，它是a。

| Note | 该示例使用一个案例类LabeledText来具有良好的描述模式。 |
| :---: | :--- |


```
import spark.implicits._

sealed trait Category
case object Scientific extends Category
case object NonScientific extends Category

// FIXME: Define schema for Category

case class LabeledText(id: Long, category: Category, text: String)

val data = Seq(LabeledText(0, Scientific, "hello world"), LabeledText(1, NonScientific, "witaj swiecie")).toDF

scala> data.show
+-----+-------------+
|label|         text|
+-----+-------------+
|    0|  hello world|
|    1|witaj swiecie|
+-----+-------------+
```

然后将其标记化并转换为另一个具有称为特征的列的数据帧，特征是数值的向量。

| Note | 使用：粘贴模式将以下代码粘贴到Spark Shell中。 |
| :---: | :--- |


```
import spark.implicits._

case class Article(id: Long, topic: String, text: String)
val articles = Seq(
  Article(0, "sci.math", "Hello, Math!"),
  Article(1, "alt.religion", "Hello, Religion!"),
  Article(2, "sci.physics", "Hello, Physics!"),
  Article(3, "sci.math", "Hello, Math Revised!"),
  Article(4, "sci.math", "Better Math"),
  Article(5, "alt.religion", "TGIF")).toDS
```

现在，令牌化部分将每个文本文档的输入文本映射到令牌\(tokens\)（一个Seq \[String\]），然后到数值的向量，然后才能被机器学习算法（对Vector实例操作）理解 。

```
scala> articles.show
+---+------------+--------------------+
| id|       topic|                text|
+---+------------+--------------------+
|  0|    sci.math|        Hello, Math!|
|  1|alt.religion|    Hello, Religion!|
|  2| sci.physics|     Hello, Physics!|
|  3|    sci.math|Hello, Math Revised!|
|  4|    sci.math|         Better Math|
|  5|alt.religion|                TGIF|
+---+------------+--------------------+

val topic2Label: Boolean => Double = isSci => if (isSci) 1 else 0
val toLabel = udf(topic2Label)

val labelled = articles.withColumn("label", toLabel($"topic".like("sci%"))).cache

val Array(trainDF, testDF) = labelled.randomSplit(Array(0.75, 0.25))

scala> trainDF.show
+---+------------+--------------------+-----+
| id|       topic|                text|label|
+---+------------+--------------------+-----+
|  1|alt.religion|    Hello, Religion!|  0.0|
|  3|    sci.math|Hello, Math Revised!|  1.0|
+---+------------+--------------------+-----+


scala> testDF.show
+---+------------+---------------+-----+
| id|       topic|           text|label|
+---+------------+---------------+-----+
|  0|    sci.math|   Hello, Math!|  1.0|
|  2| sci.physics|Hello, Physics!|  1.0|
|  4|    sci.math|    Better Math|  1.0|
|  5|alt.religion|           TGIF|  0.0|
+---+------------+---------------+-----+
```

列车模型阶段使用逻辑回归机器学习算法来构建模型并预测未来输入文本文档的标签（并因此将其分类为科学或非科学）。

```
import org.apache.spark.ml.feature.RegexTokenizer
val tokenizer = new RegexTokenizer()
  .setInputCol("text")
  .setOutputCol("words")

import org.apache.spark.ml.feature.HashingTF
val hashingTF = new HashingTF()
  .setInputCol(tokenizer.getOutputCol)  // it does not wire transformers -- it's just a column name
  .setOutputCol("features")
  .setNumFeatures(5000)

import org.apache.spark.ml.classification.LogisticRegression
val lr = new LogisticRegression().setMaxIter(20).setRegParam(0.01)

import org.apache.spark.ml.Pipeline
val pipeline = new Pipeline().setStages(Array(tokenizer, hashingTF, lr))
```

它使用两列，即标签和特征向量来构建逻辑回归模型以进行预测。

```
val model = pipeline.fit(trainDF)

val trainPredictions = model.transform(trainDF)
val testPredictions = model.transform(testDF)

scala> trainPredictions.select('id, 'topic, 'text, 'label, 'prediction).show
+---+------------+--------------------+-----+----------+
| id|       topic|                text|label|prediction|
+---+------------+--------------------+-----+----------+
|  1|alt.religion|    Hello, Religion!|  0.0|       0.0|
|  3|    sci.math|Hello, Math Revised!|  1.0|       1.0|
+---+------------+--------------------+-----+----------+

// Notice that the computations add new columns
scala> trainPredictions.printSchema
root
 |-- id: long (nullable = false)
 |-- topic: string (nullable = true)
 |-- text: string (nullable = true)
 |-- label: double (nullable = true)
 |-- words: array (nullable = true)
 |    |-- element: string (containsNull = true)
 |-- features: vector (nullable = true)
 |-- rawPrediction: vector (nullable = true)
 |-- probability: vector (nullable = true)
 |-- prediction: double (nullable = true)

import org.apache.spark.ml.evaluation.BinaryClassificationEvaluator
val evaluator = new BinaryClassificationEvaluator().setMetricName("areaUnderROC")

import org.apache.spark.ml.param.ParamMap
val evaluatorParams = ParamMap(evaluator.metricName -> "areaUnderROC")

scala> val areaTrain = evaluator.evaluate(trainPredictions, evaluatorParams)
areaTrain: Double = 1.0

scala> val areaTest = evaluator.evaluate(testPredictions, evaluatorParams)
areaTest: Double = 0.6666666666666666
```

让我们调整模型的超参数（使用org.apache.spark.ml.tuning包中的“工具”）。

| Caution | 查看org.apache.spark.ml.tuning包中的可用类。 |
| :---: | :--- |


```
import org.apache.spark.ml.tuning.ParamGridBuilder
val paramGrid = new ParamGridBuilder()
  .addGrid(hashingTF.numFeatures, Array(100, 1000))
  .addGrid(lr.regParam, Array(0.05, 0.2))
  .addGrid(lr.maxIter, Array(5, 10, 15))
  .build

// That gives all the combinations of the parameters

paramGrid: Array[org.apache.spark.ml.param.ParamMap] =
Array({
	logreg_cdb8970c1f11-maxIter: 5,
	hashingTF_8d7033d05904-numFeatures: 100,
	logreg_cdb8970c1f11-regParam: 0.05
}, {
	logreg_cdb8970c1f11-maxIter: 5,
	hashingTF_8d7033d05904-numFeatures: 1000,
	logreg_cdb8970c1f11-regParam: 0.05
}, {
	logreg_cdb8970c1f11-maxIter: 10,
	hashingTF_8d7033d05904-numFeatures: 100,
	logreg_cdb8970c1f11-regParam: 0.05
}, {
	logreg_cdb8970c1f11-maxIter: 10,
	hashingTF_8d7033d05904-numFeatures: 1000,
	logreg_cdb8970c1f11-regParam: 0.05
}, {
	logreg_cdb8970c1f11-maxIter: 15,
	hashingTF_8d7033d05904-numFeatures: 100,
	logreg_cdb8970c1f11-regParam: 0.05
}, {
	logreg_cdb8970c1f11-maxIter: 15,
	hashingTF_8d7033d05904-numFeatures: 1000,
	logreg_cdb8970c1f11-...

import org.apache.spark.ml.tuning.CrossValidator
import org.apache.spark.ml.param._
val cv = new CrossValidator()
  .setEstimator(pipeline)
  .setEstimatorParamMaps(paramGrid)
  .setEvaluator(evaluator)
  .setNumFolds(10)

val cvModel = cv.fit(trainDF)
```

让我们使用交叉验证模型来计算预测并评估它们的精度。

```
val cvPredictions = cvModel.transform(testDF)

scala> cvPredictions.select('topic, 'text, 'prediction).show
+------------+---------------+----------+
|       topic|           text|prediction|
+------------+---------------+----------+
|    sci.math|   Hello, Math!|       0.0|
| sci.physics|Hello, Physics!|       0.0|
|    sci.math|    Better Math|       1.0|
|alt.religion|           TGIF|       0.0|
+------------+---------------+----------+

scala> evaluator.evaluate(cvPredictions, evaluatorParams)
res26: Double = 0.6666666666666666

scala> val bestModel = cvModel.bestModel
bestModel: org.apache.spark.ml.Model[_] = pipeline_8873b744aac7
```

| Caution | Review [https://github.com/apache/spark/blob/master/mllib/src/test/scala/org/apache/spark/ml/tuning/CrossValidatorSuite.scala](https://github.com/apache/spark/blob/master/mllib/src/test/scala/org/apache/spark/ml/tuning/CrossValidatorSuite.scala) |
| :--- | :--- |


You can eventually save the model for later use.

```
cvModel.write.overwrite.save(
"model"
)
```

Congratulations! You’re done.













