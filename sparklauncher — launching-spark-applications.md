## `SparkLauncher` — Launching Spark Applications  {#__a_id_sparklauncher_a_code_sparklauncher_code_launching_spark_applications_programmatically}

## Programmatically {#__a_id_sparklauncher_a_code_sparklauncher_code_launching_spark_applications_programmatically}

SparkLauncher 是一个以编程方式（即从代码（而不是直接 spark 提交））启动Spark应用程序的接口。它使用构建器模式配置 Spark 应用程序，并使用 spark-submit 将其作为子进程启动。

SparkLauncher 属于 spark-launcher 构建模块中的 org.apache.spark.launcher Scala 包。 SparkLauncher 使用 SparkSubmitCommandBuilder 构建 Spark 应用程序的 Spark 命令来启动。

表1. SparkLauncher 的 Builder 方法设置 Spark 应用程序的调用

| Setter | Description |
| :--- | :--- |
| `addAppArgs(String…​ args)` | 为 Spark 应用程序添加命令行参数。 |
| `addFile(String file)` | 使用 Spark 应用程序添加要提交的文件。 |
| `addJar(String jar)` | 添加要与应用程序一起提交的 jar 文件。 |
| `addPyFile(String file)` | 添加一个 python 文件/ zip / egg 与 Spark 应用程序提交。 |
| `addSparkArg(String arg)` | 向 Spark 调用添加无值参数。 |
| `addSparkArg(String name, String value)` | 将一个带有值的参数添加到 Spark 调用中。它识别已知的命令行参数，即 --master，--properties-files，--conf，--class， - jar， --files 和 --py-files。 |
| `directory(File dir)` | S设置spark-submit的工作目录。 |
| `redirectError()` | 将 stderr 重定向到 stdout。 |
| `redirectError(File errFile)` | 将 error output 重定向到指定的`errFile 文件。` |
| `redirectError(ProcessBuilder.Redirect to)` | 将 error output 重定向到制定的重定向。 |
| `redirectOutput(File outFile)` | 将 output 重定向到指定的`outFile 文件。` |
| `redirectOutput(ProcessBuilder.Redirect to)` | 将 standard output 重定向到指定的重定向。 |
| `redirectToLog(String loggerName)` | 将所有输出设置为记录日志并重定向到具有指定名称的日志记录器。 |
| `setAppName(String appName)` | Sets the name of an Spark application |
| `setAppResource(String resource)` | Sets the main application resource, i.e. the location of a jar file for Scala/Java applications. |
| `setConf(String key, String value)` | Sets a Spark property. Expects`key`starting with`spark.`prefix. |
| `setDeployMode(String mode)` | Sets the deploy mode. |
| `setJavaHome(String javaHome)` | Sets a custom`JAVA_HOME`. |
| `setMainClass(String mainClass)` | Sets the main class. |
| `setMaster(String master)` | Sets the master URL. |
| `setPropertiesFile(String path)` | Sets the internal`propertiesFile`.See[`loadPropertiesFile`Internal Method](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/spark-AbstractCommandBuilder.html#loadPropertiesFile). |
| `setSparkHome(String sparkHome)` | Sets a custom`SPARK_HOME`. |
| `setVerbose(boolean verbose)` | Enables verbose reporting for SparkSubmit. |

在设置 Spark 应用程序的调用后，使用 launch（）方法启动将启动已配置的 Spark 应用程序的子过程。但是，建议使用 startApplication 方法。

```
import org.apache.spark.launcher.SparkLauncher

val command = new SparkLauncher()
  .setAppResource("SparkPi")
  .setVerbose(true)

val appHandle = command.startApplication()
```





