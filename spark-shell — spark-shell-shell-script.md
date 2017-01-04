## Spark Shell — `spark-shell` shell script

Spark shell 是一个交互式 shell，学习如何充分利用 Apache Spark。这是一个在 Scala 中编写的 Spark 应用程序，用于提供具有自动完成功能的命令行环境（在 TAB 键下），您可以运行特殊查询并熟悉 Spark 的功能（帮助您开发自己的独立 Spark 应用程序）。它是一个非常方便的工具，立即反馈出以 Spark 探索的许多可用的东西。这是Spark为任务处理任何大小的数据集有帮助的许多原因之一。

有不同语言的 Spark shell 的变体：Scala 的 spark-shell 和 Python 的 pyspark。

| 注意 | 本文档仅使用spark-shell。 |
| :---: | :--- |


您可以使用 spark-shell 脚本启动 Spark shell。

```
$ ./bin/spark-shell
scala>
```

spark-shell 是 Scala REPL 的扩展，具有 SparkSession 作为 spark（和 SparkContext 为 sc）的自动实例化。

```
scala> :type spark
org.apache.spark.sql.SparkSession

// Learn the current version of Spark in use
scala> spark.version
res0: String = 2.1.0-SNAPSHOT
```

spark-shell 还导入 Scala SQL 的 implicits 和 sql 方法。

注意

当你执行 spark-shell 时，你实际执行 Spark 提交如下：

```
org.apache.spark.deploy.SparkSubmit --class org.apache.spark.repl.Main --name Spark shell spark-shell
```

设置 SPARK\_PRINT\_LAUNCH\_COMMAND 以查看要执行的整个命令。请参阅Spark Scripts的打印启动命令。

### Using Spark shell {#__a_id_using_spark_shell_a_using_spark_shell}

您使用 spark-shell 脚本启动 Spark shell（在 bin 目录中可用）。

    $ ./bin/spark-shell
    Setting default log level to "WARN".
    To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
    WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
    WARN ObjectStore: Failed to get database global_temp, returning NoSuchObjectException
    Spark context Web UI available at http://10.47.71.138:4040
    Spark context available as 'sc' (master = local[*], app id = local-1477858597347).
    Spark session available as 'spark'.
    Welcome to
          ____              __
         / __/__  ___ _____/ /__
        _\ \/ _ \/ _ `/ __/  '_/
       /___/ .__/\_,_/_/ /_/\_\   version 2.1.0-SNAPSHOT
          /_/

    Using Scala version 2.11.8 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_112)
    Type in expressions to have them evaluated.
    Type :help for more information.

    scala>

Spark shell 为你创建一个名为 spark 的 SparkSession 实例（所以你不必知道如何在第1天自己做）。

```
scala> :type spark
org.apache.spark.sql.SparkSession
```

此外，还有创建的 sc 值，它是 SparkContext 的一个实例。

| Spark Property | Default Value | Description |
| :---: | :---: | :---: |
| spark.repl.class.uri | null | 在 spark-shell 中用于创建 REPL ClassLoader 以在用户键入代码时加载在 Scala REPL 中定义的新类。 为 org.apache.spark.executor.Executor 记录器启用 INFO 日志记录级别以将值打印到日志中：  INFO 使用 REPL 类 URI：\[classUri\] |



