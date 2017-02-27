## GeneralizedLinearRegression \(GLM\) {#_generalizedlinearregression_glm}

`GeneralizedLinearRegression`广义线性回归是一种回归算法。它支持以下错误分布系列：

1. `gaussian`

2. `binomial`

3. `poisson`

4. `gamma`

`GeneralizedLinearRegression`支持线性预测变量和分布函数链接的平均值之间的以下关系：

1. `identity`

2. `logit`

3. `log`

4. `inverse`

5. `probit`

6. `cloglog`

7. `sqrt`

`GeneralizedLinearRegression`supports`4096`features.

The label column has to be of`DoubleType`type.

| Note | `GeneralizedLinearRegression`belongs to`org.apache.spark.ml.regression`package. |
| :--- | :--- |


```
import org.apache.spark.ml.regression._
val glm = new GeneralizedLinearRegression()

import org.apache.spark.ml.linalg._
val features = Vectors.sparse(5, Seq((3,1.0)))
val trainDF = Seq((0, features, 1)).toDF("id", "features", "label")
val glmModel = glm.fit(trainDF)
```

GeneralizedLinearRegression是一个具有Vector类型特征的回归函数，可以训练GeneralizedLinearRegressionModel。

### GeneralizedLinearRegressionModel {#__a_id_generalizedlinearregressionmodel_a_generalizedlinearregressionmodel}

### Regressor {#__a_id_regressor_a_regressor}

`Regressor`is a custom[Predictor](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/spark-mllib/spark-mllib-estimators.html#Predictor).



















