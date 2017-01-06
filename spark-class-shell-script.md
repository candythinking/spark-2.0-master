## spark-class shell script {#_spark_class_shell_script}

`spark-class`shell 脚本是 Spark 应用程序命令行启动器，它负责设置 JVM 环境和执行 Spark 应用程序。

| Note | 最终，Spark 中的任何 shell 脚本 spark-submit，调用 spark-class 脚本。 |
| :---: | :--- |


你可以在 Spark 发行版的 bin 目录中找到 spark-class 脚本。

启动时，spark 类首先加载 $ SPARK\_HOME / bin / load-spark-env.sh，收集 Spark 程序集 jar，然后执行 org.apache.spark.launcher.Main。

根据 Spark 的分布情况（或者根本不存在），即是否存在 RELEASE 文件，它会将 SPARK\_JARS\_DIR 环境变量分别设置为 \[SPARK\_HOME\] / jars 或 \[SPARK\_HOME\] / assembly / target / scala- \[SPARK\_SCALA\_VERSION\] / jars 后者是本地构建）。

如果 SPARK\_JARS\_DIR 不存在，则 spark-class 将打印以下错误消息并退出，代码为1。

```
Failed to find Spark jars directory ([SPARK_JARS_DIR]).
You need to build Spark with the target "package" before running this program.
```

spark-class 设置 LAUNCH\_CLASSPATH 环境变量以包括 SPARK\_JARS\_DIR 下的所有 jar。

如果启用了 SPARK\_PREPEND\_CLASSES，则会将 \[SPARK\_HOME\] / launcher / target / scala- \[SPARK\_SCALA\_VERSION\] / classes 目录添加到 LAUNCH\_CLASSPATH 作为第一个条目。

| Note | 使用 SPARK\_PREPEND\_CLASSES 使 Spark 启动器类（从 \[SPARK\_HOME\] / launcher / target / scala- \[SPARK\_SCALA\_VERSION\] / classes）出现在其他 Spark 程序集之前。它对开发有用，所以您的更改不需要重新重建 Spark。 |
| :---: | :--- |


SPARK\_TESTING 和 SPARK\_SQL\_TESTING 环境变量启用测试特殊模式。

| Caution | env vars有什么特别之处？ |
| :---: | :--- |


spark-class 使用 org.apache.spark.launcher.Main 命令行应用程序来计算要启动的 Spark 命令。 Main 类以编程方式计算 spark-class 之后执行的命令。

| Tip | 使用 JAVA\_HOME 指向要使用的 JVM。 |
| :---: | :--- |


### `org.apache.spark.launcher.Main`Standalone Application {#__a_id_main_a_code_org_apache_spark_launcher_main_code_standalone_application}

org.apache.spark.launcher.Main 是一个 Scala 独立应用程序，在 spark 类中用于准备 Spark 命令执行。

Main 期望第一个参数是类名称，即“操作模式”：

1. org.apache.spark.deploy.SparkSubmit - 主要使用 SparkSubmitCommandBuilder 来解析命令行参数。这是 spark 提交使用的模式。

2. anything- 主要使用 SparkClassCommandBuilder 来解析命令行参数。

```
$ ./bin/spark-class org.apache.spark.launcher.Main
Exception in thread "main" java.lang.IllegalArgumentException: Not enough arguments: missing class name.
	at org.apache.spark.launcher.CommandBuilderUtils.checkArgument(CommandBuilderUtils.java:241)
	at org.apache.spark.launcher.Main.main(Main.java:51)
```

`Main`使用 buildCommand 方法在构建器上构建 Spark 命令。

如果启用 SPARK\_PRINT\_LAUNCH\_COMMAND 环境变量，则 Main 将最终 Spark 命令打印为标准错误。

```
Spark Command: [cmd]
========================================
```

如果在 Windows 上，它调用 prepareWindowsCommand，而在非 Windows 操作系统的 prepareBashCommand 与 tokens \ 0 分隔。

| Caution | 什么是 prepareWindowsCommand？ prepareBashCommand？ |
| :---: | :--- |


`Main`uses the following environment variables:

* `SPARK_DAEMON_JAVA_OPTS`and`SPARK_MASTER_OPTS`to be added to the command line of the command.

* `SPARK_DAEMON_MEMORY`\(default:`1g`\) for`-Xms`and`-Xmx`.



