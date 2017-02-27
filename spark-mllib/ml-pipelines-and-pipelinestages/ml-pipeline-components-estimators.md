## ML Pipeline Components — Estimators {#_ml_pipeline_components_estimators}

估计器（estimator）是适合数据集上的模型的学习算法的抽象。

| Note | 这就是机器学习以这种方式解释估计器，不是吗？这是我花更多的时间与Pipeline API，我经常使用这个空间的术语和短语。抱歉。 |
| :---: | :--- |


技术上，估计器为给定的DataFrame和参数（如ParamMap）产生一个模型（即变换器）。它适合一个模型到输入DataFrame和ParamMap，以产生一个Transformer（一个模型），可以计算任何基于DataFrame的输入数据集的预测。

它基本上是一个将DataFrame映射到Model通过拟合方法的函数，即它需要一个DataFrame并产生一个Transformer作为一个模型。

```
estimator: DataFrame =[fit]=> Model
```

估计器是org.apache.spark.ml.Estimator抽象类的实例，自带的fit方法（返回类型M是一个模型）：

```
fit(dataset: DataFrame): M
```

估计器是PipelineStage（因此它可以是流水线的一部分）。

| Note | 管道考虑Estimator特殊，并在变换之前执行fit方法（对于管道中的其他Transformer对象）。请咨询管道文档。 |
| :---: | :--- |


作为示例，您可以使用LinearRegression学习算法估计器来训练LinearRegressionModel。

Estimator抽象类的一些直接专用实现如下：

* StringIndexer

* KMeans

* TrainValidationSplit

* Predictors

### StringIndexer {#__a_id_stringindexer_a_stringindexer}

org.apache.spark.ml.feature.StringIndexer是一个产生StringIndexerModel的Estimator。

```
val df = ('a' to 'a' + 9).map(_.toString)
  .zip(0 to 9)
  .map(_.swap)
  .toDF("id", "label")

import org.apache.spark.ml.feature.StringIndexer
val strIdx = new StringIndexer()
  .setInputCol("label")
  .setOutputCol("index")

scala> println(strIdx.explainParams)
handleInvalid: how to handle invalid entries. Options are skip (which will filter out rows with bad values), or error (which will throw an error). More options may be added later (default: error)
inputCol: input column name (current: label)
outputCol: output column name (default: strIdx_ded89298e014__output, current: index)

val model = strIdx.fit(df)
val indexed = model.transform(df)

scala> indexed.show
+---+-----+-----+
| id|label|index|
+---+-----+-----+
|  0|    a|  3.0|
|  1|    b|  5.0|
|  2|    c|  7.0|
|  3|    d|  9.0|
|  4|    e|  0.0|
|  5|    f|  2.0|
|  6|    g|  6.0|
|  7|    h|  8.0|
|  8|    i|  4.0|
|  9|    j|  1.0|
+---+-----+-----+
```

### KMeans {#__a_id_kmeans_a_kmeans}

KMeans类是机器学习中的K-means聚类算法的实现，支持k-means \|\| （也称为k平均）在Spark MLlib。

大致地，k均值是无监督迭代算法，其将预定数量的k个簇中的输入数据分组。每个群集具有作为群集中心的质心。它是一种高度迭代的机器学习算法，测量距离（向量和质心之间）作为最近的平均值。重复算法步骤直到收敛指定数量的步骤。

| Note | K-Means算法在计算机科学中使用劳埃德算法。 |
| :---: | :--- |


它是一个估计器，产生一个KMeansModel。

| Tip | 请导入org.apache.spark.ml.clustering.KMeans以使用KMeans算法。 |
| :---: | :--- |


KMeans默认使用以下值：

* 聚类或形心数（k）：2

* 最大迭代次数（maxIter）：20

* 初始化算法（initMode）：k-means \|\|

* k均值\|\|的步长数（initSteps）：5

* 收敛公差（tol）：1e-4

```
import org.apache.spark.ml.clustering._
val kmeans = new KMeans()

scala> println(kmeans.explainParams)
featuresCol: features column name (default: features)
initMode: initialization algorithm (default: k-means||)
initSteps: number of steps for k-means|| (default: 5)
k: number of clusters to create (default: 2)
maxIter: maximum number of iterations (>= 0) (default: 20)
predictionCol: prediction column name (default: prediction)
seed: random seed (default: -1689246527)
tol: the convergence tolerance for iterative algorithms (default: 1.0E-4)
```

KMeans假定featuresCol类型为VectorUDT，并追加类型为IntegerType的predictionCol。

在内部，fit方法“展开”特征向量在输入DataFrame的featuresCol列中，并创建一个RDD \[Vector\]。然后它将调用传递到org.apache.spark.mllib.clustering.KMeans中的KMeans的MLlib变体。将结果复制到KMeansModel，并计算KMeansSummary。

数据集中的每个项（行）由称为特征的属性的数字向量描述。单个特征（向量的维度）表示具有值的词（标记），该值是定义该词或词在文档中的重要性的度量。

Enable`INFO`logging level for`org.apache.spark.mllib.clustering.KMeans`logger to see what happens inside a`KMeans`.+

Add the following line to`conf/log4j.properties`:

```
log4j.logger.org.apache.spark.mllib.clustering.KMeans=INFO
```

#### KMeans Example {#__a_id_kmeans_example_a_kmeans_example}

您可以使用向量空间模型表示文本语料库（文档集合）。在该表示中，向量具有作为语料库中不同词的数量的维度。具有大量零值的向量是非常自然的，因为并非所有单词都将在文档中。我们将使用优化的存储器表示以避免使用稀疏向量的零值。

此示例显示如何使用k-means将电子邮件分类为垃圾邮件。

```
// NOTE Don't copy and paste the final case class with the other lines
// It won't work with paste mode in spark-shell
final case class Email(id: Int, text: String)

val emails = Seq(
  "This is an email from your lovely wife. Your mom says...",
  "SPAM SPAM spam",
  "Hello, We'd like to offer you").zipWithIndex.map(_.swap).toDF("id", "text").as[Email]

// Prepare data for k-means
// Pass emails through a "pipeline" of transformers
import org.apache.spark.ml.feature._
val tok = new RegexTokenizer()
  .setInputCol("text")
  .setOutputCol("tokens")
  .setPattern("\\W+")

val hashTF = new HashingTF()
  .setInputCol("tokens")
  .setOutputCol("features")
  .setNumFeatures(20)

val preprocess = (tok.transform _).andThen(hashTF.transform)

val features = preprocess(emails.toDF)

scala> features.select('text, 'features).show(false)
+--------------------------------------------------------+------------------------------------------------------------+
|text                                                    |features                                                    |
+--------------------------------------------------------+------------------------------------------------------------+
|This is an email from your lovely wife. Your mom says...|(20,[0,3,6,8,10,11,17,19],[1.0,2.0,1.0,1.0,2.0,1.0,2.0,1.0])|
|SPAM SPAM spam                                          |(20,[13],[3.0])                                             |
|Hello, We'd like to offer you                           |(20,[0,2,7,10,11,19],[2.0,1.0,1.0,1.0,1.0,1.0])             |
+--------------------------------------------------------+------------------------------------------------------------+

import org.apache.spark.ml.clustering.KMeans
val kmeans = new KMeans

scala> val kmModel = kmeans.fit(features.toDF)
16/04/08 15:57:37 WARN KMeans: The input data is not directly cached, which may hurt performance if its parent RDDs are also uncached.
16/04/08 15:57:37 INFO KMeans: Initialization with k-means|| took 0.219 seconds.
16/04/08 15:57:37 INFO KMeans: Run 0 finished in 1 iterations
16/04/08 15:57:37 INFO KMeans: Iterations took 0.030 seconds.
16/04/08 15:57:37 INFO KMeans: KMeans converged in 1 iterations.
16/04/08 15:57:37 INFO KMeans: The cost for the best run is 5.000000000000002.
16/04/08 15:57:37 WARN KMeans: The input data was not directly cached, which may hurt performance if its parent RDDs are also uncached.
kmModel: org.apache.spark.ml.clustering.KMeansModel = kmeans_7a13a617ce0b

scala> kmModel.clusterCenters.map(_.toSparse)
res36: Array[org.apache.spark.mllib.linalg.SparseVector] = Array((20,[13],[3.0]), (20,[0,2,3,6,7,8,10,11,17,19],[1.5,0.5,1.0,0.5,0.5,0.5,1.5,1.0,1.0,1.0]))

val email = Seq("hello mom").toDF("text")
val result = kmModel.transform(preprocess(email))

scala> .show(false)
+---------+------------+---------------------+----------+
|text     |tokens      |features             |prediction|
+---------+------------+---------------------+----------+
|hello mom|[hello, mom]|(20,[2,19],[1.0,1.0])|1         |
+---------+------------+---------------------+----------+
```

### TrainValidationSplit {#__a_id_trainvalidationsplit_a_trainvalidationsplit}

| Caution | FIXME |
| :--- | :--- |


### Predictors {#__a_id_predictor_a_predictors}

预测器\(predictors\)是具有自己的抽象训练方法的PredictionModel的估计器的专门化。

```
train(dataset: DataFrame): M
```

train方法应该简化处理模式验证和复制参数到经过训练的PredictionModel模型。它还将模型的父级设置为自身。

Predictor基本上是一个将DataFrame映射到PredictionModel的函数。

```
predictor: DataFrame =[train]=> PredictionModel
```

它实现Estimator抽象类的抽象拟合（数据集：DataFrame），它验证并转换数据集的模式（使用PipelineStage的自定义transformSchema），然后调用抽象训练方法。

验证和转换模式（使用transformSchema）可确保：

1. 特征列存在，并且是正确类型（默认为Vector）。

2. 标签列存在并且是Double类型。

作为最后一步，它添加Double类型的预测列。

以下是不同学习算法的预测器示例列表：

* DecisionTreeClassifier

* LinearRegression

* RandomForestRegressor

#### DecisionTreeClassifier {#__a_id_decisiontreeclassifier_a_decisiontreeclassifier}

`DecisionTreeClassifier`is a`ProbabilisticClassifier`that…​

| Caution | FIXME |
| :--- | :--- |


#### LinearRegression {#__a_id_linearregression_a_linearregression}

LinearRegression是Predictor的一个例子（间接通过专用的Regressor私有抽象类），因此一个Estimator，它表示机器学习中的线性回归算法。

LinearRegression属于org.apache.spark.ml.regression包。

| Tip | Read the scaladoc of [LinearRegression](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.ml.regression.LinearRegression). |
| :--- | :--- |


它期望org.apache.spark.mllib.linalg.Vector作为数据集中列的输入类型，并生成LinearRegressionModel。

```
import org.apache.spark.ml.regression.LinearRegression
val lr = new LinearRegression
```

可接受的参数：

```
scala> println(lr.explainParams)
elasticNetParam: the ElasticNet mixing parameter, in range [0, 1]. For alpha = 0, the penalty is an L2 penalty. For alpha = 1, it is an L1 penalty (default: 0.0)
featuresCol: features column name (default: features)
fitIntercept: whether to fit an intercept term (default: true)
labelCol: label column name (default: label)
maxIter: maximum number of iterations (>= 0) (default: 100)
predictionCol: prediction column name (default: prediction)
regParam: regularization parameter (>= 0) (default: 0.0)
solver: the solver algorithm for optimization. If this is not set or empty, default value is 'auto' (default: auto)
standardization: whether to standardize the training features before fitting the model (default: true)
tol: the convergence tolerance for iterative algorithms (default: 1.0E-6)
weightCol: weight column name. If this is not set or empty, we treat all instance weights as 1.0 (default: )
```

##### LinearRegression.train {#__a_id_linearregression_train_a_linearregression_train}

```
train(dataset: DataFrame): LinearRegressionModel
```

LinearRegression的train（protected）方法希望数据集DataFrame具有两列：

1. `label`of type`DoubleType`.

2. `features`of type Vector.

It returns`LinearRegressionModel`.

它首先计算要素列中的元素数（通常为特征）。列必须是mllib.linalg.Vector类型（并且可以使用HashingTF变换器轻松地准备）。

```
val spam = Seq(
  (0, "Hi Jacek. Wanna more SPAM? Best!"),
  (1, "This is SPAM. This is SPAM")).toDF("id", "email")

import org.apache.spark.ml.feature.RegexTokenizer
val regexTok = new RegexTokenizer()
val spamTokens = regexTok.setInputCol("email").transform(spam)

scala> spamTokens.show(false)
+---+--------------------------------+---------------------------------------+
|id |email                           |regexTok_646b6bcc4548__output          |
+---+--------------------------------+---------------------------------------+
|0  |Hi Jacek. Wanna more SPAM? Best!|[hi, jacek., wanna, more, spam?, best!]|
|1  |This is SPAM. This is SPAM      |[this, is, spam., this, is, spam]      |
+---+--------------------------------+---------------------------------------+

import org.apache.spark.ml.feature.HashingTF
val hashTF = new HashingTF()
  .setInputCol(regexTok.getOutputCol)
  .setOutputCol("features")
  .setNumFeatures(5000)

val spamHashed = hashTF.transform(spamTokens)

scala> spamHashed.select("email", "features").show(false)
+--------------------------------+----------------------------------------------------------------+
|email                           |features                                                        |
+--------------------------------+----------------------------------------------------------------+
|Hi Jacek. Wanna more SPAM? Best!|(5000,[2525,2943,3093,3166,3329,3980],[1.0,1.0,1.0,1.0,1.0,1.0])|
|This is SPAM. This is SPAM      |(5000,[1713,3149,3370,4070],[1.0,1.0,2.0,2.0])                  |
+--------------------------------+----------------------------------------------------------------+

// Create labeled datasets for spam (1)
val spamLabeled = spamHashed.withColumn("label", lit(1d))

scala> spamLabeled.show
+---+--------------------+-----------------------------+--------------------+-----+
| id|               email|regexTok_646b6bcc4548__output|            features|label|
+---+--------------------+-----------------------------+--------------------+-----+
|  0|Hi Jacek. Wanna m...|         [hi, jacek., wann...|(5000,[2525,2943,...|  1.0|
|  1|This is SPAM. Thi...|         [this, is, spam.,...|(5000,[1713,3149,...|  1.0|
+---+--------------------+-----------------------------+--------------------+-----+

val regular = Seq(
  (2, "Hi Jacek. I hope this email finds you well. Spark up!"),
  (3, "Welcome to Apache Spark project")).toDF("id", "email")
val regularTokens = regexTok.setInputCol("email").transform(regular)
val regularHashed = hashTF.transform(regularTokens)
// Create labeled datasets for non-spam regular emails (0)
val regularLabeled = regularHashed.withColumn("label", lit(0d))

val training = regularLabeled.union(spamLabeled).cache

scala> training.show
+---+--------------------+-----------------------------+--------------------+-----+
| id|               email|regexTok_646b6bcc4548__output|            features|label|
+---+--------------------+-----------------------------+--------------------+-----+
|  2|Hi Jacek. I hope ...|         [hi, jacek., i, h...|(5000,[72,105,942...|  0.0|
|  3|Welcome to Apache...|         [welcome, to, apa...|(5000,[2894,3365,...|  0.0|
|  0|Hi Jacek. Wanna m...|         [hi, jacek., wann...|(5000,[2525,2943,...|  1.0|
|  1|This is SPAM. Thi...|         [this, is, spam.,...|(5000,[1713,3149,...|  1.0|
+---+--------------------+-----------------------------+--------------------+-----+

import org.apache.spark.ml.regression.LinearRegression
val lr = new LinearRegression

// the following calls train by the Predictor contract (see above)
val lrModel = lr.fit(training)

// Let's predict whether an email is a spam or not
val email = Seq("Hi Jacek. you doing well? Bye!").toDF("email")
val emailTokens = regexTok.setInputCol("email").transform(email)
val emailHashed = hashTF.transform(emailTokens)

scala> lrModel.transform(emailHashed).select("prediction").show
+-----------------+
|       prediction|
+-----------------+
|0.563603440350882|
+-----------------+
```

#### RandomForestRegressor {#__a_id_randomforestregressor_a_randomforestregressor}

RandomForestRegressor是随机森林学习算法的具体预测器。它使用DataFrame和Vector类型的features列来训练RandomForestRegressionModel（PredictionModel的子类型）。

```
import org.apache.spark.mllib.linalg.Vectors
val features = Vectors.sparse(10, Seq((2, 0.2), (4, 0.4)))

val data = (0.0 to 4.0 by 1).map(d => (d, features)).toDF("label", "features")
// data.as[LabeledPoint]

scala> data.show(false)
+-----+--------------------------+
|label|features                  |
+-----+--------------------------+
|0.0  |(10,[2,4,6],[0.2,0.4,0.6])|
|1.0  |(10,[2,4,6],[0.2,0.4,0.6])|
|2.0  |(10,[2,4,6],[0.2,0.4,0.6])|
|3.0  |(10,[2,4,6],[0.2,0.4,0.6])|
|4.0  |(10,[2,4,6],[0.2,0.4,0.6])|
+-----+--------------------------+

import org.apache.spark.ml.regression.{ RandomForestRegressor, RandomForestRegressionModel }
val rfr = new RandomForestRegressor
val model: RandomForestRegressionModel = rfr.fit(data)

scala> model.trees.foreach(println)
DecisionTreeRegressionModel (uid=dtr_247e77e2f8e0) of depth 1 with 3 nodes
DecisionTreeRegressionModel (uid=dtr_61f8eacb2b61) of depth 2 with 7 nodes
DecisionTreeRegressionModel (uid=dtr_63fc5bde051c) of depth 2 with 5 nodes
DecisionTreeRegressionModel (uid=dtr_64d4e42de85f) of depth 2 with 5 nodes
DecisionTreeRegressionModel (uid=dtr_693626422894) of depth 3 with 9 nodes
DecisionTreeRegressionModel (uid=dtr_927f8a0bc35e) of depth 2 with 5 nodes
DecisionTreeRegressionModel (uid=dtr_82da39f6e4e1) of depth 3 with 7 nodes
DecisionTreeRegressionModel (uid=dtr_cb94c2e75bd1) of depth 0 with 1 nodes
DecisionTreeRegressionModel (uid=dtr_29e3362adfb2) of depth 1 with 3 nodes
DecisionTreeRegressionModel (uid=dtr_d6d896abcc75) of depth 3 with 7 nodes
DecisionTreeRegressionModel (uid=dtr_aacb22a9143d) of depth 2 with 5 nodes
DecisionTreeRegressionModel (uid=dtr_18d07dadb5b9) of depth 2 with 7 nodes
DecisionTreeRegressionModel (uid=dtr_f0615c28637c) of depth 2 with 5 nodes
DecisionTreeRegressionModel (uid=dtr_4619362d02fc) of depth 2 with 5 nodes
DecisionTreeRegressionModel (uid=dtr_d39502f828f4) of depth 2 with 5 nodes
DecisionTreeRegressionModel (uid=dtr_896f3a4272ad) of depth 3 with 9 nodes
DecisionTreeRegressionModel (uid=dtr_891323c29838) of depth 3 with 7 nodes
DecisionTreeRegressionModel (uid=dtr_d658fe871e99) of depth 2 with 5 nodes
DecisionTreeRegressionModel (uid=dtr_d91227b13d41) of depth 2 with 5 nodes
DecisionTreeRegressionModel (uid=dtr_4a7976921f4b) of depth 2 with 5 nodes

scala> model.treeWeights
res12: Array[Double] = Array(1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0)

scala> model.featureImportances
res13: org.apache.spark.mllib.linalg.Vector = (1,[0],[1.0])
```

### Example {#__a_id_example_a_example}

以下示例使用LinearRegression估计器。

```
import org.apache.spark.mllib.linalg.Vectors
import org.apache.spark.mllib.regression.LabeledPoint
val data = (0.0 to 9.0 by 1)                      // create a collection of Doubles
  .map(n => (n, n))                               // make it pairs
  .map { case (label, features) =>
    LabeledPoint(label, Vectors.dense(features)) } // create labeled points of dense vectors
  .toDF                                           // make it a DataFrame

scala> data.show
+-----+--------+
|label|features|
+-----+--------+
|  0.0|   [0.0]|
|  1.0|   [1.0]|
|  2.0|   [2.0]|
|  3.0|   [3.0]|
|  4.0|   [4.0]|
|  5.0|   [5.0]|
|  6.0|   [6.0]|
|  7.0|   [7.0]|
|  8.0|   [8.0]|
|  9.0|   [9.0]|
+-----+--------+

import org.apache.spark.ml.regression.LinearRegression
val lr = new LinearRegression

val model = lr.fit(data)

scala> model.intercept
res1: Double = 0.0

scala> model.coefficients
res2: org.apache.spark.mllib.linalg.Vector = [1.0]

// make predictions
scala> val predictions = model.transform(data)
predictions: org.apache.spark.sql.DataFrame = [label: double, features: vector ... 1 more field]

scala> predictions.show
+-----+--------+----------+
|label|features|prediction|
+-----+--------+----------+
|  0.0|   [0.0]|       0.0|
|  1.0|   [1.0]|       1.0|
|  2.0|   [2.0]|       2.0|
|  3.0|   [3.0]|       3.0|
|  4.0|   [4.0]|       4.0|
|  5.0|   [5.0]|       5.0|
|  6.0|   [6.0]|       6.0|
|  7.0|   [7.0]|       7.0|
|  8.0|   [8.0]|       8.0|
|  9.0|   [9.0]|       9.0|
+-----+--------+----------+

import org.apache.spark.ml.evaluation.RegressionEvaluator

// rmse is the default metric
// We're explicit here for learning purposes
val regEval = new RegressionEvaluator().setMetricName("rmse")
val rmse = regEval.evaluate(predictions)

scala> println(s"Root Mean Squared Error: $rmse")
Root Mean Squared Error: 0.0

import org.apache.spark.mllib.linalg.DenseVector
// NOTE Follow along to learn spark.ml-way (not RDD-way)
predictions.rdd.map { r =>
  (r(0).asInstanceOf[Double], r(1).asInstanceOf[DenseVector](0).toDouble, r(2).asInstanceOf[Double]))
  .toDF("label", "feature0", "prediction").show
+-----+--------+----------+
|label|feature0|prediction|
+-----+--------+----------+
|  0.0|     0.0|       0.0|
|  1.0|     1.0|       1.0|
|  2.0|     2.0|       2.0|
|  3.0|     3.0|       3.0|
|  4.0|     4.0|       4.0|
|  5.0|     5.0|       5.0|
|  6.0|     6.0|       6.0|
|  7.0|     7.0|       7.0|
|  8.0|     8.0|       8.0|
|  9.0|     9.0|       9.0|
+-----+--------+----------+

// Let's make it nicer to the eyes using a Scala case class
scala> :pa
// Entering paste mode (ctrl-D to finish)

import org.apache.spark.sql.Row
import org.apache.spark.mllib.linalg.DenseVector
case class Prediction(label: Double, feature0: Double, prediction: Double)
object Prediction {
  def apply(r: Row) = new Prediction(
    label = r(0).asInstanceOf[Double],
    feature0 = r(1).asInstanceOf[DenseVector](0).toDouble,
    prediction = r(2).asInstanceOf[Double])
}

// Exiting paste mode, now interpreting.

import org.apache.spark.sql.Row
import org.apache.spark.mllib.linalg.DenseVector
defined class Prediction
defined object Prediction

scala> predictions.rdd.map(Prediction.apply).toDF.show
+-----+--------+----------+
|label|feature0|prediction|
+-----+--------+----------+
|  0.0|     0.0|       0.0|
|  1.0|     1.0|       1.0|
|  2.0|     2.0|       2.0|
|  3.0|     3.0|       3.0|
|  4.0|     4.0|       4.0|
|  5.0|     5.0|       5.0|
|  6.0|     6.0|       6.0|
|  7.0|     7.0|       7.0|
|  8.0|     8.0|       8.0|
|  9.0|     9.0|       9.0|
+-----+--------+----------+
```













