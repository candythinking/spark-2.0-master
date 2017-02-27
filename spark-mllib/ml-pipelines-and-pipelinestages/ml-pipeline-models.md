## ML Pipeline Models {#_ml_pipeline_models}

模型抽象类是一个具有可选估计器的变换器，它已生成它（作为临时父字段）。

```
model: DataFrame =[predict]=> DataFrame (with predictions)
```

| Note | 估计器是可选的，只有在执行模型拟合后（估计器匹配）之后才可用。 |
| :---: | :--- |


作为一个变换器，它需要一个DataFrame并将其转换为一个添加了预测列的结果DataFrame。

有两个直接实现的Model类与一个具体的ML算法没有直接相关：

* PipelineModel

* PredictionModel

### PipelineModel {#__a_id_pipelinemodel_a_pipelinemodel}

| Caution | `PipelineModel`is a`private[ml]`class. |
| :--- | :--- |


PipelineModel是管道估计器的模型。

一旦拟合，您可以使用结果模型作为任何其他模型来转换数据集（如DataFrame）。

PipelineModel的一个非常有趣的用例是当流水线由Transformer实例组成时。

```
// Transformer #1
import org.apache.spark.ml.feature.Tokenizer
val tok = new Tokenizer().setInputCol("text")

// Transformer #2
import org.apache.spark.ml.feature.HashingTF
val hashingTF = new HashingTF().setInputCol(tok.getOutputCol).setOutputCol("features")

// Fuse the Transformers in a Pipeline
import org.apache.spark.ml.Pipeline
val pipeline = new Pipeline().setStages(Array(tok, hashingTF))

val dataset = Seq((0, "hello world")).toDF("id", "text")

// Since there's no fitting, any dataset works fine
val featurize = pipeline.fit(dataset)

// Use the pipelineModel as a series of Transformers
scala> featurize.transform(dataset).show(false)
+---+-----------+------------------------+--------------------------------+
|id |text       |tok_8aec9bfad04a__output|features                        |
+---+-----------+------------------------+--------------------------------+
|0  |hello world|[hello, world]          |(262144,[71890,72594],[1.0,1.0])|
+---+-----------+------------------------+--------------------------------+
```

### PredictionModel {#__a_id_predictionmodel_a_predictionmodel}

PredictionModel是一个抽象类，用于表示用于预测算法（如回归和分类）的模型（具有自己的专用模型 - 下面的细节）。

PredictionModel基本上是一个Transformer与predict方法来计算预测（最终在预测列）。

PredictionModel属于org.apache.spark.ml包。

```
import org.apache.spark.ml.PredictionModel
```

PredictionModel类的合同要求每个自定义实现都定义了预测方法（FeatureType类型是要素类型）。

```
predict(features: FeaturesType): Double
```

PredictionModel类的直接较少算法特定的扩展是：

* RegressionModel

* ClassificationModel

* RandomForestRegressionModel

作为一个自定义转换器，它配备了自己的自定义变换方法。

在内部，transform首先确保特征列的类型与模型的类型匹配，并将Double类型的预测列添加到结果DataFrame的模式。

然后，它创建结果DataFrame，并添加具有应用于要素列的值的predictUDF函数的预测列。

| Note | 显示从数据帧（左侧）和另一个（右侧）用箭头表示变换方法的变换的图。 |
| :---: | :--- |


Enable`DEBUG`logging level for a`PredictionModel`implementation, e.g.[LinearRegressionModel](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/spark-mllib/spark-mllib-models.html#LinearRegressionModel), to see what happens inside.

Add the following line to`conf/log4j.properties`:

```
log4j.logger.org.apache.spark.ml.regression.LinearRegressionModel=DEBUG
```

### ClassificationModel {#__a_id_classificationmodel_a_classificationmodel}

ClassificationModel是一个PredictionModel，它将带有强制要素，标签和rawPrediction（类型为Vector）列的DataFrame转换为添加了预测列的DataFrame。

| Note | 带有ClassifierParams参数的模型，例如ClassificationModel，要求DataFrame具有必需的功能，标签（Double类型）和rawPrediction（类型为Vector）列。 |
| :---: | :--- |


ClassificationModel具有自己的变换（作为Transformer）和predict（作为PredictionModel）。

以下是已知的ClassificationModel自定义实现的列表（截至3月24日）：

* `ProbabilisticClassificationModel`\(the`abstract`parent of the following classification models\)

  * `DecisionTreeClassificationModel`\(`final`\)

  * `LogisticRegressionModel`

  * `NaiveBayesModel`+

  * `RandomForestClassificationModel`\(`final`\)

### RegressionModel {#__a_id_regressionmodel_a_regressionmodel}

RegressionModel是一个PredictionModel，它转换带有强制性标签，特征和预测列的DataFrame。

它没有自己的方法或值，因此更多的标记抽象类（组合不同特征的回归模型在一种类型下）。

#### LinearRegressionModel {#__a_id_linearregressionmodel_a_linearregressionmodel}

LinearRegressionModel表示由LinearRegression估计器生成的模型。它转换org.apache.spark.mllib.linalg.Vector类型的必需的特征列。

| Note | 它是一个私有的\[ml\]类，所以你最终可以使用的是更加一般的RegressionModel，因为RegressionModel只是一个无标记的抽象类，它更像是一个PredictionModel。 |
| :---: | :--- |


作为扩展LinearRegressionParams的线性回归模型，它期望输入DataFrame的以下模式：

* `label`\(required\)

* `features`\(required\)

* `prediction`

* `regParam`

* `elasticNetParam`

* `maxIter`\(Int\)

* `tol`\(Double\)

* `fitIntercept`\(Boolean\)

* `standardization`\(Boolean\)

* `weightCol`\(String\)

* `solver`\(String\)

（1.6.0中的新增内容）LinearRegressionModel也是一个MLWritable（因此您可以将其保存到持久存储中以供以后重用）。

启用DEBUG日志记录（参见上文），当调用transform并转换模式时，您可以在日志中看到以下消息。

```
16/03/21 06:55:32 DEBUG LinearRegressionModel: Input schema: {"type":"struct","fields":[{"name":"label","type":"double","nullable":false,"metadata":{}},{"name":"features","type":{"type":"udt","class":"org.apache.spark.mllib.linalg.VectorUDT","pyClass":"pyspark.mllib.linalg.VectorUDT","sqlType":{"type":"struct","fields":[{"name":"type","type":"byte","nullable":false,"metadata":{}},{"name":"size","type":"integer","nullable":true,"metadata":{}},{"name":"indices","type":{"type":"array","elementType":"integer","containsNull":false},"nullable":true,"metadata":{}},{"name":"values","type":{"type":"array","elementType":"double","containsNull":false},"nullable":true,"metadata":{}}]}},"nullable":true,"metadata":{}}]}
16/03/21 06:55:32 DEBUG LinearRegressionModel: Expected output schema: {"type":"struct","fields":[{"name":"label","type":"double","nullable":false,"metadata":{}},{"name":"features","type":{"type":"udt","class":"org.apache.spark.mllib.linalg.VectorUDT","pyClass":"pyspark.mllib.linalg.VectorUDT","sqlType":{"type":"struct","fields":[{"name":"type","type":"byte","nullable":false,"metadata":{}},{"name":"size","type":"integer","nullable":true,"metadata":{}},{"name":"indices","type":{"type":"array","elementType":"integer","containsNull":false},"nullable":true,"metadata":{}},{"name":"values","type":{"type":"array","elementType":"double","containsNull":false},"nullable":true,"metadata":{}}]}},"nullable":true,"metadata":{}},{"name":"prediction","type":"double","nullable":false,"metadata":{}}]}
```

LinearRegressionModel的predict的实现计算两个向量 - 相同大小的特征和系数（DenseVector或SparseVector类型）的点（v1，v2），并添加截距。

| Note | 系数Vector和intercept Double是LinearRegressionModel的组成部分，作为构造函数的必需输入参数。 |
| :---: | :--- |


#### LinearRegressionModel Example {#__a_id_linearregressionmodel_example_a_linearregressionmodel_example}

```
// Create a (sparse) Vector
import org.apache.spark.mllib.linalg.Vectors
val indices = 0 to 4
val elements = indices.zip(Stream.continually(1.0))
val sv = Vectors.sparse(elements.size, elements)

// Create a proper DataFrame
val ds = sc.parallelize(Seq((0.5, sv))).toDF("label", "features")

import org.apache.spark.ml.regression.LinearRegression
val lr = new LinearRegression

// Importing LinearRegressionModel and being explicit about the type of model value
// is for learning purposes only
import org.apache.spark.ml.regression.LinearRegressionModel
val model: LinearRegressionModel = lr.fit(ds)

// Use the same ds - just for learning purposes
scala> model.transform(ds).show
+-----+--------------------+----------+
|label|            features|prediction|
+-----+--------------------+----------+
|  0.5|(5,[0,1,2,3,4],[1...|       0.5|
+-----+--------------------+----------+
```

### RandomForestRegressionModel {#__a_id_randomforestregressionmodel_a_randomforestregressionmodel}

RandomForestRegressionModel是具有Vector类型的features列的PredictionModel。

有趣的是，DataFrame转换（作为Transformer合约的一部分）使用SparkContext.broadcast将自身发送到Spark集群中的节点，并调用对特征计算预测（作为预测列）。

### KMeansModel {#__a_id_kmeansmodel_a_kmeansmodel}

KMeansModel是KMeans算法的模型。

它属于org.apache.spark.ml.clustering包。

```
// See spark-mllib-estimators.adoc#KMeans
val kmeans: KMeans = ???
val trainingDF: DataFrame = ???
val kmModel = kmeans.fit(trainingDF)

// Know the cluster centers
scala> kmModel.clusterCenters
res0: Array[org.apache.spark.mllib.linalg.Vector] = Array([0.1,0.3], [0.1,0.1])

val inputDF = Seq((0.0, Vectors.dense(0.2, 0.4))).toDF("label", "features")

scala> kmModel.transform(inputDF).show(false)
+-----+---------+----------+
|label|features |prediction|
+-----+---------+----------+
|0.0  |[0.2,0.4]|0         |
+-----+---------+----------+
```

















