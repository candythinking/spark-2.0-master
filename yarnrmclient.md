## YarnRMClient {#_yarnrmclient}

YarnRMClient 负责使用 YARN ResourceManager（以及名称中的 RM）注册和取消注册 Spark 应用程序（以 ApplicationMaster 的形式）。它只是 AMRMClient \[ContainerRequest\] 的包装器，它在注册 ApplicationMaster 时启动（并且从未明确停止！）。

除了负责注册和注销外，它还知道应用程序尝试标识符，并跟踪注册 ApplicationMaster 的最大尝试次数。

Enable`INFO`logging level for`org.apache.spark.deploy.yarn.YarnRMClient`logger to see what happens inside.

Add the following line to`conf/log4j.properties`:

```
log4j.logger.org.apache.spark.deploy.yarn.YarnRMClient=INFO
```

### Registering ApplicationMaster with YARN ResourceManager \(register method\) {#__a_id_register_a_registering_applicationmaster_with_yarn_resourcemanager_register_method}

要使用 YARN ResourceManager 注册 ApplicationMaster（对于 Spark 应用程序），Spark 使用 register。

```
register(
  driverUrl: String,
  driverRef: RpcEndpointRef,
  conf: YarnConfiguration,
  sparkConf: SparkConf,
  uiAddress: String,
  uiHistoryAddress: String,
  securityMgr: SecurityManager,
  localResources: Map[String, LocalResource]): YarnAllocator
```

register 实例化 YARN 的 AMRMClient，初始化它（使用 conf 输入参数）并立即开始。它在内部保存 ui HistoryAddress 输入参数以供以后使用。

![](/img/mastering-apache-spark/spark on yarn/figure3.png)

You should see the following INFO message in the logs \(in stderr in YARN\):

```
INFO YarnRMClient: Registering the ApplicationMaster
```

然后，它将本地主机上的应用程序主机注册为端口0和可以看到 master URL 的 uiAddress 输入参数。

The internal`registered`flag is enabled.

最终，它创建一个新的 YarnAllocator，其中包含传入的寄存器的输入参数和刚创建的 YARN AMRMClient。

### Unregistering ApplicationMaster from YARN ResourceManager \(unregister method\) {#__a_id_unregister_a_unregistering_applicationmaster_from_yarn_resourcemanager_unregister_method}

```
unregister(status: FinalApplicationStatus, diagnostics: String = ""): Unit
```

`unregister`unregisters the ApplicationMaster of a Spark application.

它基本上检查 ApplicationMaster 是否已注册，并且仅当它请求内部 AMRMClient 注销时。

`unregister`is called when`ApplicationMaster`wants to unregister。

### Maximum Number of Attempts to Register ApplicationMaster \(getMaxRegAttempts method\) {#__a_id_getmaxregattempts_a_maximum_number_of_attempts_to_register_applicationmaster_getmaxregattempts_method}

```
getMaxRegAttempts(sparkConf: SparkConf, yarnConf: YarnConfiguration): Int
```

getMaxRegAttempts 使用 SparkConf 和 YARN 的 YarnConfiguration 来读取配置设置，并返回 ApplicationMaster 注册 YARN 之前的最大应用程序尝试次数（因此，Spark 应用程序不成功）。

它读取 YARN 的 yarn.resourcemanager.am.max-attempts（可用作 YarnConfiguration.RM\_AM\_MAX\_ATTEMPTS）或回退到 YarnConfiguration.DEFAULT\_RM\_AM\_MAX\_ATTEMPTS（为 2）。

返回值是 YARN 和 Spark 的配置设置的最小值。

### Getting ApplicationAttemptId of Spark Application \(getAttemptId method\) {#__a_id_getattemptid_a_getting_applicationattemptid_of_spark_application_getattemptid_method}

```
getAttemptId(): ApplicationAttemptId
```

getAttemptId 返回 YARN 的 ApplicationAttemptId（容器被分配到的 Spark 应用程序的）。

在内部，它使用特定于 YARN 的方法，如 ConverterUtils.toContainerId 和 ContainerId.getApplicationAttemptId。

### getAmIpFilterParams {#__a_id_getamipfilterparams_a_getamipfilterparams}

| Caution | FIXME |
| :--- | :--- |














































