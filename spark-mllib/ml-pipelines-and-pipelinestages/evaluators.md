## Evaluators {#_evaluators}

评估器（**evaluator）**是将DataFrame映射到指示模型有多好的度量的变换。

```
evaluator: DataFrame =[evaluate]=> Double
```

Evaluator是一个带有evaluate方法的抽象类。

```
evaluate(dataset: DataFrame): Double
evaluate(dataset: DataFrame, paramMap: ParamMap): Double
```

它使用isLargerBetter方法来指示Double指标是应该最大化（true）还是最小化（false）。默认情况下，它认为较大的值更好（true）。

```
isLargerBetter: Boolean = true
```

以下是一些可用的Evaluator实现的列表：

* MulticlassClassificationEvaluator

* BinaryClassificationEvaluator

* RegressionEvaluator

### MulticlassClassificationEvaluator {#__a_id_multiclassclassificationevaluator_a_multiclassclassificationevaluator}

MulticlassClassificationEvaluator是一个具体的Evaluator，期望DataFrame数据集具有以下两列：

* `prediction`of`DoubleType`

* `label`of`float`or`double`values

### BinaryClassificationEvaluator {#__a_id_binaryclassificationevaluator_a_binaryclassificationevaluator}

BinaryClassificationEvaluator是一个二元分类的具体Evaluator，期望DataFrame类型的数据集具有两列：

* `rawPrediction`being`DoubleType`or`VectorUDT`.

* `label`being`NumericType`

| Note | It can cross-validate models LogisticRegression, RandomForestClassifier et al. |
| :--- | :--- |


### RegressionEvaluator {#__a_id_regressionevaluator_a_regressionevaluator}

RegressionEvaluator是一个回归的具体Evaluator，期望DataFrame类型的数据集具有以下两列：

* `prediction`of`float`or`double`values

* `label`of`float`or`double`values

当执行（通过求值），它准备一个带有（预测，标签）对的RDD \[Double，Double\]，并将其传递给org.apache.spark.mllib.evaluation.RegressionMetrics（来自“old”Spark MLlib）。

RegressionEvaluator可以评估以下指标：

* `rmse`\(default; larger is better? no\) is the **root mean squared error（均方根误差）**.

* `mse`\(larger is better? no\) is the **mean squared error（均方误差）**.

* `r2`\(larger is better?: yes\)

* `mae`\(larger is better? no\) is the **mean absolute error（平均绝对值误差）**.

```
// prepare a fake input dataset using transformers
import org.apache.spark.ml.feature.Tokenizer
val tok = new Tokenizer().setInputCol("text")

import org.apache.spark.ml.feature.HashingTF
val hashTF = new HashingTF()
  .setInputCol(tok.getOutputCol)  // it reads the output of tok
  .setOutputCol("features")

// Scala trick to chain transform methods
// It's of little to no use since we've got Pipelines
// Just to have it as an alternative
val transform = (tok.transform _).andThen(hashTF.transform _)

val dataset = Seq((0, "hello world", 0.0)).toDF("id", "text", "label")

// we're using Linear Regression algorithm
import org.apache.spark.ml.regression.LinearRegression
val lr = new LinearRegression

import org.apache.spark.ml.Pipeline
val pipeline = new Pipeline().setStages(Array(tok, hashTF, lr))

val model = pipeline.fit(dataset)

// Let's do prediction
// Note that we're using the same dataset as for fitting the model
// Something you'd definitely not be doing in prod
val predictions = model.transform(dataset)

// Now we're ready to evaluate the model
// Evaluator works on datasets with predictions

import org.apache.spark.ml.evaluation.RegressionEvaluator
val regEval = new RegressionEvaluator

// check the available parameters
scala> println(regEval.explainParams)
labelCol: label column name (default: label)
metricName: metric name in evaluation (mse|rmse|r2|mae) (default: rmse)
predictionCol: prediction column name (default: prediction)

scala> regEval.evaluate(predictions)
res0: Double = 0.0
```



