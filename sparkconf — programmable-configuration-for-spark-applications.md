## SparkConf — Programmable Configuration for Spark Applications {#__a_id_sparkconf_a_sparkconf_programmable_configuration_for_spark_applications}

| Tip | Refer to [Spark Configuration](http://spark.apache.org/docs/latest/configuration.html) in the official documentation for an extensive coverage of how to configure Spark and user programs. |
| :--- | :--- |


| Caution | TODODescribe`SparkConf`object for the application configuration.the default configssystem properties |
| :--- | :--- |


There are three ways to configure Spark and user programs:+

* Spark Properties - use[Web UI](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/spark-webui.html)to learn the current properties.

* …​

### Mandatory Settings - spark.master and spark.app.name {#__a_id_mandatory_settings_a_mandatory_settings_spark_master_and_spark_app_name}

在 Spark 应用程序可以运行之前，必须定义任何 Spark 应用程序的两个强制设置 - spark.master 和 spark.app.name。

#### spark.master - Master URL {#__a_id_spark_master_a_spark_master_master_url}

| Caution | FIXME |
| :--- | :--- |


#### spark.app.name - Application Name {#__a_id_spark_app_name_a_spark_app_name_application_name}

### Spark Properties {#_spark_properties}

每个用户程序都从创建 SparkConf 的实例开始，该实例保存要连接到的 master URL（spark.master），Spark 应用程序的名称（稍后显示在 Web UI 中并变为 spark.app.name）和其他 Spark 属性正确运行所需。 SparkConf 的实例可以用来创建 SparkContext。

Tip

启动 Spark shell - - spark.logConf = true，以便在 SparkContext 启动时将有效的 Spark 配置记录为 INFO。

```
$ ./bin/spark-shell --conf spark.logConf=true
...
15/10/19 17:13:49 INFO SparkContext: Running Spark version 1.6.0-SNAPSHOT
15/10/19 17:13:49 INFO SparkContext: Spark configuration:
spark.app.name=Spark shell
spark.home=/Users/jacek/dev/oss/spark
spark.jars=
spark.logConf=true
spark.master=local[*]
spark.repl.class.uri=http://10.5.10.20:64055
spark.submit.deployMode=client
...
```

一旦 SparkContext 完成初始化，使用 sc.getConf.toDebugString 获得更丰富的输出。

您可以在 Spark shell 中查询 Spark 属性的值，如下所示：

```
scala> sc.getConf.getOption("spark.local.dir")
res0: Option[String] = None

scala> sc.getConf.getOption("spark.app.name")
res1: Option[String] = Some(Spark shell)

scala> sc.getConf.get("spark.master")
res2: String = local[*]
```

### Setting up Spark Properties {#_setting_up_spark_properties}

在以下场所，Spark 应用程序查找 Spark 属性（按照从最不重要到最重要的顺序）：

* `conf/spark-defaults.conf`- 该配置文件具有默认的 Spark 属性。请阅读 spark-defaults.conf。

* `--conf`or`-c`- spark 提交使用的命令行选项（以及在其下面使用 spark-submit 或 spark-class 的其他 shell 脚本，例如 spark-shell）

* `SparkConf`

### Default Configuration {#__a_id_default_configuration_a_default_configuration}

执行以下代码时将创建默认 Spark 配置：

```
import org.apache.spark.SparkConf
val conf = new SparkConf
```

它只是加载 spark.\* 的系统属性。

您可以使用 conf.toDebugString 或 conf.getAll 来打印 spark.\* 系统属性。

```
scala> conf.getAll
res0: Array[(String, String)] = Array((spark.app.name,Spark shell), (spark.jars,""), (spark.master,local[*]), (spark.submit.deployMode,client))

scala> conf.toDebugString
res1: String =
spark.app.name=Spark shell
spark.jars=
spark.master=local[*]
spark.submit.deployMode=client

scala> println(conf.toDebugString)
spark.app.name=Spark shell
spark.jars=
spark.master=local[*]
spark.submit.deployMode=client
```





