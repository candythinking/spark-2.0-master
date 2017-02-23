## ML Pipelines and PipelineStages \(spark.ml\) {#_ml_pipelines_and_pipelinestages_spark_ml}

ML Pipeline API（也称为Spark ML或spark.ml，由于API所在的包），Spark用户可以通过标准化不同机器学习概念的API，快速，轻松地组装和配置实用的分布式机器学习管道（又名工作流程）。

| Note | Scikit-learn和GraphLab都有在其系统中内置管道的概念。 |
| :---: | :--- |


ML Pipeline API是一个新的基于DataFrame的API，在org.apache.spark.ml包下开发，是MLlib和Spark 2.0的主要API。

| important | org.apache.spark.mllib包之前的基于RDD的API处于仅维护模式，这意味着它仍然保留有错误修复，但不需要新的功能。 |
| :---: | :--- |


Pipeline API（又名spark.ml组件）的关键概念：

* Pipelines and PipelineStages

* Transformers

  * Models

* Estimators

* Evaluators

* Params \(and ParamMaps\)

使用Spark ML的优点是ML数据集只是一个DataFrame（所有的计算都只是列上的UDF应用程序）。

机器学习算法的使用仅是预测分析工作流程的一个组成部分。还可以有用于机器学习算法工作的附加预处理步骤。

| Note | 虽然Spark Core中的RDD计算，Spark SQL中的数据集操作，Spark Streaming中的连续DStream计算是ML管道在Spark MLlib中的主要数据抽象。 |
| :---: | :--- |


典型的标准机器学习工作流程如下：

1. 加载数据（也称为数据提取） 
2. 提取特征（又称特征提取） 
3. 训练模型（又名模型训练） 
4. 评估（或预测）

在最终模型成为生产准备之前，您还可以考虑两个附加步骤，因此任何使用：

1. 测试模型（又叫做模型测试） 
2. 选择最佳模型（也称为模型选择或模型调整） 
3. 部署模型（也称为模型部署和集成）

| Note | Pipeline API位于org.apache.spark.ml包下。 |
| :---: | :--- |


给定管道组件，典型的机器学习管道如下：

* 使用Transformer实例集合来准备输入DataFrame - 具有适当输入数据（在列中）的数据集，用于所选择的ML算法。

* 然后你适合\(fit\)（又名构建\(build\)）一个模型。

* 使用模型，您可以通过DataFrame变换计算特征输入列上的预测（在预测列中）。

示例：在文本分类中，在训练类似SVM的分类模型之前，通常需要诸如n-gram提取和TF-IDF特征加权的预处理步骤。

部署模型时，系统不仅必须知道应用于输入要素的SVM权重，还必须将原始数据转换为模型训练的格式。

* Pipeline for text categorization\(分类\)

* Pipeline for image classification\(分类\)

管道（Pipeline）就像数据库系统中的查询计划。

ML管道组件：

* **Pipeline Construction Framework**– A DSL for the construction of pipelines that includes concepts of **Nodes **and **Pipelines**.\(管道构造框架 - 用于构建包含节点和管道概念的管道的DSL。\)

  * Nodes are data transformation steps \(Transformers\)

  * Pipelines are a DAG of Nodes.

  * Pipelines become objects that can be saved out and applied in real-time to new data.\(管道成为可以被保存并实时应用于新数据的对象。\)

它可以帮助创建特定于域的特征变换器，通用变换器，统计实用程序和节点。

您可以最终保存或加载机器学习组件，如永久机学习组件中所述。

| Note | 机器学习组件是属于Pipeline API的任何对象，例如管道，线性回归模型等 |
| :---: | :--- |


### Features of Pipeline API {#__a_id_features_a_features_of_pipeline_api}

The features of the Pipeline API in Spark MLlib:

* DataFrame as a dataset format

* ML Pipelines API is similar to [scikit-learn](http://scikit-learn.org/stable/modules/generated/sklearn.pipeline.Pipeline.html)

* Easy debugging \(via inspecting columns added during execution\)

* Parameter tuning

* Compositions \(to build more complex pipelines out of existing ones\)

### Pipelines {#__a_id_pipelines_a_a_id_pipeline_a_pipelines}

ML流水线（或ML工作流）是用于将PipelineModel拟合到输入数据集的变换器和估计器序列。

```
pipeline: DataFrame =[fit]=> DataFrame (using transformers and estimators)
```

管道由Pipeline类表示。

```
import org.apache.spark.ml.Pipeline
```

管道也是一个估计器\(Estimator\)（因此可以接受与其他管道实例建立一个管道）。 

Pipeline对象可以读取或加载流水线（参考持久化机器学习组件页面）。

```
read: MLReader[Pipeline]
load(path: String): Pipeline
```

您可以使用可选的uid标识符创建流水线。未指定时，它的格式为pipeline\_ \[randomUid\]。

```
val pipeline = new Pipeline()

scala> println(pipeline.uid)
pipeline_94be47c3b709

val pipeline = new Pipeline("my_pipeline")

scala> println(pipeline.uid)
my_pipeline
```

标识符uid用于创建从fit（dataset：DataFrame）：PipelineModel方法返回的PipelineModel的实例。

```
scala> val pipeline = new Pipeline("my_pipeline")
pipeline: org.apache.spark.ml.Pipeline = my_pipeline

scala> val df = (0 to 9).toDF("num")
df: org.apache.spark.sql.DataFrame = [num: int]

scala> val model = pipeline.setStages(Array()).fit(df)
model: org.apache.spark.ml.PipelineModel = my_pipeline
```

阶段强制参数可以使用setStages（value：Array \[PipelineStage\]）：this.type方法设置。

#### Pipeline Fitting \(fit method\) {#__a_id_pipeline_fit_a_pipeline_fitting_fit_method}

```
fit(dataset: DataFrame): PipelineModel
```

fit方法返回一个PipelineModel，它保存了一个Transformer对象集合，这些对象是Pipeline中每个Estimator（可能已修改的数据集）或简单地输入Transformer对象的Estimator.fit方法的结果。将输入数据集DataFrame传递给流水线中的每个Transformer实例进行变换。

它首先转换输入数据集DataFrame的模式。

然后，它搜索最后一个估计器的索引，以计算估计器的变换器，并简单地将变换器返回到流水线中的索引。对于每个估计器，使用输入数据集调用fit方法。结果DataFrame被传递到链中的下一个Transformer。

| Note | 当阶段既不是估计器也不是变换器时抛出IllegalArgumentException异常。 |
| :---: | :--- |


变换方法被调用每个Transformer计算，但最后一个（这是执行适合在最后一个Estimator的结果）。

收集计算的Transformers。

在最后一个Estimator之后，只能有Transformer阶段。

该方法返回具有uid和变换器的PipelineModel。父估计器`Estimator`是流水线本身。

### PipelineStage {#__a_id_pipelinestage_a_pipelinestage}

PipelineStage抽象类表示管道中的单个阶段。

PipelineStage有以下直接实现（其中很少是抽象类）：

* Estimators

* Models

* Pipeline

* Predictor

* Transformer

每个PipelineStage使用transformSchema系列方法转换模式：

```
transformSchema(schema: StructType): StructType
transformSchema(schema: StructType, logging: Boolean): StructType
```

| Note | StructType描述了DataFrame的模式。 |
| :---: | :--- |


| Tip | 为相应的PipelineStage实现启用DEBUG日志记录级别，以查看下面发生的情况。 |
| :---: | :--- |


### Further reading or watching {#__a_id_i_want_more_a_further_reading_or_watching}

* [ML Pipelines](https://amplab.cs.berkeley.edu/ml-pipelines/)

* [ML Pipelines: A New High-Level API for MLlib](https://databricks.com/blog/2015/01/07/ml-pipelines-a-new-high-level-api-for-mllib.html)

* \(video\)[Building, Debugging, and Tuning Spark Machine Learning Pipelines - Joseph Bradley \(Databricks\)](https://youtu.be/OednhGRp938)+

* \(video\)[Spark MLlib: Making Practical Machine Learning Easy and Scalable](https://youtu.be/7gHlgk8F58w)

* \(video\)[Apache Spark MLlib 2 0 Preview: Data Science and Production](https://youtu.be/kvk4gnXL9H4)by Joseph K. Bradley \(Databricks\)













