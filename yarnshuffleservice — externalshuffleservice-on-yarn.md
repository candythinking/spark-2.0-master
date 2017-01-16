## Spark on YARN {#_spark_on_yarn}

您可以使用 yarn master URL 将 Spark 应用程序提交到 Hadoop YARN 集群。

```
spark-submit --master yarn mySparkApp.jar
```

| Note | 从 Spark 2.0.0 开始，`yarn`master URL 是唯一正确的 master URL，您可以使用 --deploy-mode 在客户端（默认）或群集模式之间进行选择。 |
| :---: | :--- |


![](/img/mastering-apache-spark/spark on yarn/figure1.png)

如果不指定部署模式，则假定为客户端。

```
spark-submit --master yarn --deploy-mode client mySparkApp.jar
```

YARN有两种部署模式 - 客户端（默认）或群集。

| Tip | Deploy modes are all about where the Spark driver runs. |
| :---: | :--- |


在客户端模式下，Spark 驱动程序（和 SparkContext）在 YARN 群集外部的客户端节点上运行，而在群集模式下，它在 YARN 群集内部运行，即在 ApplicationMaster（用作 YARN 中的 Spark 应用程序）内部的 YARN 容器内运行。

```
spark-submit --master yarn --deploy-mode cluster mySparkApp.jar
```

在这个意义上，部署到 YARN 的 Spark 应用程序是一个可以部署到 YARN 集群（与其他 Hadoop 工作负载一起）的 YARN 兼容的执行框架。在 YARN 上，Spark 执行器映射到单个 YARN 容器。

| Note | 为了将应用程序部署到 YARN 集群，您需要使用 Spark 与 YARN 支持。 |
| :---: | :--- |


Spark on YARN 支持多个应用程序尝试，并支持 HDFS 中数据的数据本地化。您还可以利用 Hadoop 的安全性，并在使用 Kerberos 身份验证（也称为 Kerberos 群集）的安全 Hadoop 环境中运行 Spark。

有几个特定于 YARN 的设置（请参阅设置）。其中，您尤其可以支持 YARN 资源队列（根据高级策略划分群集资源并向不同的团队和用户分配共享）。

| Tip | 您可以使用 --verbose 命令行选项启动 spark-submit 以显示某些设置，包括特定于 YARN。请参阅 spark-submit 和 YARN 选项。 |
| :---: | :--- |


YARN 资源请求中的内存是 --executor-memory 对 spark.yarn.executor.memoryOverhead 设置的内容，默认为 --executor-memory 的10％。

如果 YARN 有足够的资源，它将部署分布在集群中的执行器，然后他们将尝试在本地处理数据（在 Spark Web UI 中为 NODE\_LOCAL），并且与您在 spark.executor.cores 中定义的并行数量并行。

### Multiple Application Attempts {#__a_id_multiple_application_attempts_a_multiple_application_attempts}

Spark on YARN 支持在集群模式下进行多个应用程序尝试。

请参阅 YarnRMClient.getMaxRegAttempts。

### spark-submit and YARN options {#__a_id_spark_submit_a_spark_submit_and_yarn_options}

使用 spark-submit 提交 Spark 应用程序时，可以使用以下 YARN 特定的命令行选项：

* `--archives`

* `--executor-cores`

* `--keytab`

* `--num-executors`

* `--principal`

* --queue

| Tip | Read about the corresponding settings in Settings in this document. |
| :---: | :--- |


### Memory Requirements {#__a_id_memory_a_memory_requirements}

当客户端向 YARN 集群提交 Spark 应用程序时，它确保应用程序不会请求超过 YARN 集群的最大内存能力。

ApplicationMaster 的内存由每个部署模式的自定义设置控制。

对于客户端部署模式，它是 spark.yarn.am.memory（默认值：512m）的总和，可选开销为 spark.yarn.am.memoryOverhead。

对于集群部署模式，它是 spark.driver.memory（默认值：1g）的总和，可选开销为 spark.yarn.driver.memoryOverhead。

如果未设置可选开销，则计算为主存储器的 10％（对于客户端模式为 spark.yarn.am.memory 的10%，对于集群模式为spark.driver.memory 的 10%）或 384m（无论大小）。

### Spark with YARN support {#__a_id_yarn_support_a_spark_with_yarn_support}

您需要使用已经使用 YARN 支持编译的 Spark，即类 org.apache.spark.deploy.yarn.Client 必须位于 CLASSPATH 上。

否则，您将在日志中看到以下错误，Spark 将退出。

```
Error: Could not load YARN classes. This copy of Spark may not have been compiled with YARN support.
```

### Master URL {#__a_id_masterurl_a_master_url}

从 Spark 2.0.0 开始，唯一正确的 master URL 是 yarn。

```
./bin/spark-submit --master yarn ...
```

在 Spark 2.0.0 之前，您可以使用 yarn-client 或 yarn-cluster，但现在已被弃用。当您使用已弃用的主网址时，您应该在日志中看到以下警告：

```
Warning: Master yarn-client is deprecated since 2.0. Please use master "yarn" with specified deploy mode instead.
```

### Keytab {#__a_id_keytab_a_keytab}

当指定 principal 时，也必须指定 keytab。

设置 spark.yarn.principal 和 spark.yarn.principal 将设置为相应的值，UserGroupInformation.loginUserFromKeytab 将以其值作为输入参数进行调用。

### Environment Variables {#__a_id_environment_variables_a_environment_variables}

#### SPARK\_DIST\_CLASSPATH {#__a_id_spark_dist_classpath_a_spark_dist_classpath}

SPARK\_DIST\_CLASSPATH 是要添加到进程的分布定义的 CLASSPATH。

它用于为 ApplicationMaster 和执行程序填充 CLASSPATH。

### Settings {#__a_id_settings_a_settings}

| Caution | Where and how are they used? |
| :--- | :--- |


### Further reading or watching {#__a_id_i_want_more_a_further_reading_or_watching}

* \(video\)[Spark on YARN: a Deep Dive — Sandy Ryza \(Cloudera\)](https://youtu.be/N6pJhxCPe-Y)

* \(video\)[Spark on YARN: The Road Ahead — Marcelo Vanzin \(Cloudera\)](https://youtu.be/sritCTJWQes)from Spark Summit 2015





