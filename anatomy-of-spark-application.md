## Anatomy of Spark Application {#_anatomy_of_spark_application}

每个 Spark 应用程序都在实例化 Spark 上下文时启动。没有 Spark 上下文无法使用 Spark 服务启动计算。

| Note | Spark 应用程序是 SparkContext 的一个实例。或者，换句话说，Spark 上下文构成一个 Spark 应用程序。 |
| :---: | :--- |


为了使它工作，你必须使用 SparkConf 或使用自定义 SparkContext 构造函数创建 Spark 配置。

```
package pl.japila.spark

import org.apache.spark.{SparkContext, SparkConf}

object SparkMeApp {
  def main(args: Array[String]) {

    val masterURL = "local[*]"  (1)

    val conf = new SparkConf()  (2)
      .setAppName("SparkMe Application")
      .setMaster(masterURL)

    val sc = new SparkContext(conf) (3)

    val fileName = util.Try(args(0)).getOrElse("build.sbt")

    val lines = sc.textFile(fileName).cache() (4)

    val c = lines.count() (5)
    println(s"There are $c lines in $fileName")
  }
}
```

1. Master URL to connect the application to

2. Create Spark configuration

3. Create Spark context

4. Create`lines`RDD

5. Execute`count`action

| Tip | Spark shell creates a Spark context and SQL context for you at startup. |
| :--- | :--- |


当 Spark 应用程序启动（使用 spark-submit 脚本或作为独立应用程序）时，它会按 master URL 所述连接到 Spark 主机。它是 Spark 上下文初始化的一部分。

![](/img/mastering-apache-spark/spark core-rdd/figure1.png)

| Note | 您的 Spark 应用程序可以在本地或在基于集群管理器和部署模式（--deploy 模式）的集群上运行。请参阅部署模式。 |
| :---: | :--- |


然后，您可以创建 RDD，将它们转换为其他 RDD 并最终执行操作。您还可以缓存临时 RDDs 以加速数据处理。

在所有数据处理完成后，Spark 应用程序通过停止 Spark 上下文完成。











