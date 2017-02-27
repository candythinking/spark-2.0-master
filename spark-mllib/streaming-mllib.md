## Streaming MLlib {#_streaming_mllib}

以下机器学习算法在MLlib中有其流式变体：

* k-means

* Linear Regression

* Logistic Regression

他们可以训练模型和预测流数据。

| Note | 流算法属于spark.mllib（较旧的基于RDD的API）。 |
| :---: | :--- |


### Streaming k-means {#__a_id_kmeans_a_streaming_k_means}

`org.apache.spark.mllib.clustering.StreamingKMeans`

### Streaming Linear Regression {#__a_id_linear_regression_a_streaming_linear_regression}

`org.apache.spark.mllib.regression.StreamingLinearRegressionWithSGD`

### Streaming Logistic Regression {#__a_id_logistic_regression_a_streaming_logistic_regression}

`org.apache.spark.mllib.classification.StreamingLogisticRegressionWithSGD`

### Sources {#_sources}

* [Streaming Machine Learning in Spark- Jeremy Freeman \(HHMI Janelia Research Center\)](https://youtu.be/uUQTSPvD1mc)





























