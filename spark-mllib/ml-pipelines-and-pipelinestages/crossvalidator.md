## CrossValidator {#_crossvalidator}

CrossValidator是一个估计器，用于产生CrossValidatorModel，即它可以适合给定输入数据集的CrossValidatorModel。

它属于org.apache.spark.ml.tuning包。

```
import org.apache.spark.ml.tuning.CrossValidator
```

CrossValidator接受numFolds参数（其他）。

```
import org.apache.spark.ml.tuning.CrossValidator
val cv = new CrossValidator

scala> println(cv.explainParams)
estimator: estimator for selection (undefined)
estimatorParamMaps: param maps for the estimator (undefined)
evaluator: evaluator used to select hyper-parameters that maximize the validated metric (undefined)
numFolds: number of folds for cross validation (>= 2) (default: 3)
seed: random seed (default: -1191137437)
```

| Note | CrossValidator是一个非常有用的模型选择工具，它能够使用任何Estimator实例，管道包括，可以在传递数据集之前预处理数据集。这让你有一种方法来处理任何数据集，并在新的（可能更好的）模型适合它之前对其进行预处理。 |
| :---: | :--- |


### Example — CrossValidator in Pipeline {#__a_id_example_a_example_crossvalidator_in_pipeline}

| Caution | k-means是否可以被交叉验证？它有什么意义吗？它只适用于监督学习吗？ |
| :---: | :--- |


```
// Let's create a pipeline with transformers and estimator
import org.apache.spark.ml.feature._

val tok = new Tokenizer().setInputCol("text")

val hashTF = new HashingTF()
  .setInputCol(tok.getOutputCol)
  .setOutputCol("features")
  .setNumFeatures(10)

import org.apache.spark.ml.classification.RandomForestClassifier
val rfc = new RandomForestClassifier

import org.apache.spark.ml.Pipeline
val pipeline = new Pipeline()
  .setStages(Array(tok, hashTF, rfc))

// CAUTION: label must be double
// 0 = scientific text
// 1 = non-scientific text
val trainDS = Seq(
  (0L, "[science] hello world", 0d),
  (1L, "long text", 1d),
  (2L, "[science] hello all people", 0d),
  (3L, "[science] hello hello", 0d)).toDF("id", "text", "label").cache

// Check out the train dataset
// Values in label and prediction columns should be alike
val sampleModel = pipeline.fit(trainDS)
sampleModel
  .transform(trainDS)
  .select('text, 'label, 'features, 'prediction)
  .show(truncate = false)

+--------------------------+-----+--------------------------+----------+
|text                      |label|features                  |prediction|
+--------------------------+-----+--------------------------+----------+
|[science] hello world     |0.0  |(10,[0,8],[2.0,1.0])      |0.0       |
|long text                 |1.0  |(10,[4,9],[1.0,1.0])      |1.0       |
|[science] hello all people|0.0  |(10,[0,6,8],[1.0,1.0,2.0])|0.0       |
|[science] hello hello     |0.0  |(10,[0,8],[1.0,2.0])      |0.0       |
+--------------------------+-----+--------------------------+----------+

val input = Seq("Hello ScienCE").toDF("text")
sampleModel
  .transform(input)
  .select('text, 'rawPrediction, 'prediction)
  .show(truncate = false)

+-------------+--------------------------------------+----------+
|text         |rawPrediction                         |prediction|
+-------------+--------------------------------------+----------+
|Hello ScienCE|[12.666666666666668,7.333333333333333]|0.0       |
+-------------+--------------------------------------+----------+

import org.apache.spark.ml.tuning.ParamGridBuilder
val paramGrid = new ParamGridBuilder().build

import org.apache.spark.ml.evaluation.BinaryClassificationEvaluator
val binEval = new BinaryClassificationEvaluator

import org.apache.spark.ml.tuning.CrossValidator
val cv = new CrossValidator()
  .setEstimator(pipeline) // <-- pipeline is the estimator
  .setEvaluator(binEval)  // has to match the estimator
  .setEstimatorParamMaps(paramGrid)

// WARNING: It does not work!!!
val cvModel = cv.fit(trainDS)
```

### Example \(no Pipeline\) {#__a_id_example_without_pipeline_a_example_no_pipeline}

```
import org.apache.spark.mllib.linalg.Vectors
val features = Vectors.sparse(3, Array(1), Array(1d))
val df = Seq(
  (0, "hello world", 0.0, features),
  (1, "just hello",  1.0, features)).toDF("id", "text", "label", "features")

import org.apache.spark.ml.classification.LogisticRegression
val lr = new LogisticRegression

import org.apache.spark.ml.evaluation.RegressionEvaluator
val regEval = new RegressionEvaluator

import org.apache.spark.ml.tuning.ParamGridBuilder
// Parameterize the only estimator used, i.e. LogisticRegression
// Use println(lr.explainParams) to learn about the supported parameters
val paramGrid = new ParamGridBuilder()
  .addGrid(lr.regParam, Array(0.1, 0.01))
  .build()

import org.apache.spark.ml.tuning.CrossValidator
val cv = new CrossValidator()
  .setEstimator(lr) // just LogisticRegression not Pipeline
  .setEvaluator(regEval)
  .setEstimatorParamMaps(paramGrid)

// FIXME

scala> val cvModel = cv.fit(df)
java.lang.IllegalArgumentException: requirement failed: Nothing has been added to this summarizer.
  at scala.Predef$.require(Predef.scala:219)
  at org.apache.spark.mllib.stat.MultivariateOnlineSummarizer.normL2(MultivariateOnlineSummarizer.scala:270)
  at org.apache.spark.mllib.evaluation.RegressionMetrics.SSerr$lzycompute(RegressionMetrics.scala:65)
  at org.apache.spark.mllib.evaluation.RegressionMetrics.SSerr(RegressionMetrics.scala:65)
  at org.apache.spark.mllib.evaluation.RegressionMetrics.meanSquaredError(RegressionMetrics.scala:99)
  at org.apache.spark.mllib.evaluation.RegressionMetrics.rootMeanSquaredError(RegressionMetrics.scala:108)
  at org.apache.spark.ml.evaluation.RegressionEvaluator.evaluate(RegressionEvaluator.scala:94)
  at org.apache.spark.ml.tuning.CrossValidator$$anonfun$fit$1.apply(CrossValidator.scala:115)
  at org.apache.spark.ml.tuning.CrossValidator$$anonfun$fit$1.apply(CrossValidator.scala:105)
  at scala.collection.IndexedSeqOptimized$class.foreach(IndexedSeqOptimized.scala:33)
  at scala.collection.mutable.ArrayOps$ofRef.foreach(ArrayOps.scala:186)
  at org.apache.spark.ml.tuning.CrossValidator.fit(CrossValidator.scala:105)
  ... 61 elided
```











