## YarnShuffleService — ExternalShuffleService on YARN {#_yarnshuffleservice_externalshuffleservice_on_yarn}

YarnShuffleService 是 Spark on YARN 的外部 shuffle 服务。它是 YARN NodeManager 的辅助服务，实现 org.apache.hadoop.yarn.server.api.AuxiliaryService。

| Note | There is the ExternalShuffleService for Spark and despite their names they don’t share code. |
| :---: | :--- |


| Caution | What happens when the`spark.shuffle.service.enabled`flag is enabled? |
| :---: | :--- |


YarnShuffleService 在 yarn-site.xml 配置文件中配置，并在节点启动时在每个 YARN NodeManager 节点上初始化。

在 YARN 中配置外部 shuffle 服务后，您可以使用 spark.shuffle.service.enabled 标志在 Spark 应用程序中启用它。

| Note | `YarnShuffleService`was introduced in [SPARK-3797](https://issues.apache.org/jira/browse/SPARK-3797). |
| :--- | :--- |


在 YARN 日志系统中启用 org.apache.spark.network.yarn.YarnShuffleService 记录器的 INFO 日志记录级别，以查看内部发生了什么。

```
log4j.logger.org.apache.spark.network.yarn.YarnShuffleService=INFO
```

YARN 将日志保存在 Mac OS X 上的 /usr/local/Cellar/hadoop/2.7.2/libexec/logs 目录中，使用 brew，例如 /usr/local/Cellar/hadoop/2.7.2/libexec/logs/yarn-jacek-nodemanager-japila.local.log。

### Advantages {#__a_id_advantages_a_advantages}

使用 YARN Shuffle 服务的优点：

* 使用动态分配，启用的执行器可以被丢弃，Spark 应用程序仍然可以获得执行者写出的随机数据。

* 它允许单个执行器进入 GC 暂停（甚至崩溃），并仍然允许其他执行器读取洗牌数据并取得进展。

### Creating YarnShuffleService Instance {#__a_id_creating_instance_a_creating_yarnshuffleservice_instance}

当创建 YarnShuffleService 时，它将使用 spark\_shuffle 服务名称调用 YARN 的 AuxiliaryService。

您应该在日志中看到以下 INFO 消息：

```
INFO org.apache.spark.network.yarn.YarnShuffleService: Initializing YARN shuffle service for Spark
INFO org.apache.hadoop.yarn.server.nodemanager.containermanager.AuxServices: Adding auxiliary service spark_shuffle, "spark_shuffle"
```

### getRecoveryPath {#__a_id_getrecoverypath_a_getrecoverypath}

| Caution | FIXME |
| :--- | :--- |


### serviceStop {#__a_id_servicestop_a_servicestop}

```
void serviceStop()
```

`serviceStop`is a part of YARN’s`AuxiliaryService`contract and is called when…​FIXME

| Caution | FIXME The contract |
| :--- | :--- |


When called,`serviceStop`simply closes`shuffleServer`and`blockHandler`.

| Caution | FIXME What are`shuffleServer`and`blockHandler`? What’s their lifecycle? |
| :--- | :--- |


When an exception occurs, you should see the following ERROR message in the logs:

```
ERROR org.apache.spark.network.yarn.YarnShuffleService: Exception when stopping service
```

### stopContainer {#__a_id_stopcontainer_a_stopcontainer}

```
void stopContainer(ContainerTerminationContext context)
```

`stopContainer`is a part of YARN’s`AuxiliaryService`contract and is called when…​FIXME

| Caution | FIXME The contract |
| :--- | :--- |


When called,`stopContainer`simply prints out the following INFO message in the logs and exits.

```
INFO org.apache.spark.network.yarn.YarnShuffleService: Stopping container [containerId]
```

它使用 getContainerId 方法从上下文获取 containerId。

### initializeContainer {#__a_id_initializecontainer_a_initializecontainer}

```
void initializeContainer(ContainerInitializationContext context)
```

`initializeContainer`is a part of YARN’s`AuxiliaryService`contract and is called when…​FIXME

| Caution | FIXME The contract |
| :--- | :--- |


When called,`initializeContainer`simply prints out the following INFO message in the logs and exits.

```
INFO org.apache.spark.network.yarn.YarnShuffleService: Initializing container [containerId]
```

它使用 getContainerId 方法从上下文获取 containerId。

### stopApplication {#__a_id_stopapplication_a_stopapplication}

```
void stopApplication(ApplicationTerminationContext context)
```

`stopApplication`is a part of YARN’s`AuxiliaryService`contract and is called when…​FIXME

| Caution | FIXME The contract |
| :--- | :--- |


stopApplication 请求 ShuffleSecretManager 在启用身份验证时取消注册应用程序，并将 ExternalShuffleBlockHandler 应用于 applicationRemoved。

调用时，stopApplication 将获取应用程序的 YARN 的 ApplicationId（使用输入上下文）。

您应该在日志中看到以下 INFO 消息：

```
INFO org.apache.spark.network.yarn.YarnShuffleService: Stopping application [appId]
```

如果 isAuthenticationEnabled，则为应用程序标识执行 secretManager.unregisterApp。

它请求 ExternalShuffleBlockHandler 到 applicationRemoved（禁用 cleanupLocalDirs 标志）。

| Caution | FIXME What does`ExternalShuffleBlockHandler#applicationRemoved`do? |
| :--- | :--- |


When an exception occurs, you should see the following ERROR message in the logs:

```
ERROR org.apache.spark.network.yarn.YarnShuffleService: Exception when stopping application [appId]
```

### initializeApplication {#__a_id_initializeapplication_a_initializeapplication}

```
void initializeApplication(ApplicationInitializationContext context)
```

`initializeApplication`is a part of YARN’s`AuxiliaryService`contract and is called when…​FIXME

| Caution | FIXME The contract |
| :--- | :--- |


initializeApplication 在启用身份验证时请求 ShuffleSecretManager registerApp。

调用时，initializeApplication 将获取应用程序的 YARN 的 ApplicationId（使用输入上下文），并调用 shuffleSecret 的 context.getApplicationDataForService。

您应该在日志中看到以下 INFO 消息：

```
INFO org.apache.spark.network.yarn.YarnShuffleService: Initializing application [appId]
```

如果 isAuthenticationEnabled，则为应用程序标识和 shuffleSecret 执行 secretManager.registerApp。

发生异常时，应在日志中看到以下 ERROR 消息：

```
ERROR org.apache.spark.network.yarn.YarnShuffleService: Exception when initializing application [appId]
```

### serviceInit {#__a_id_serviceinit_a_serviceinit}

```
void serviceInit(Configuration conf)
```

`serviceInit`is a part of YARN’s`AuxiliaryService`contract and is called when…​FIXME

| Caution | FIXME |
| :--- | :--- |


当被调用时，serviceInit 为用于创建 ExternalShuffleBlockHandler（作为 blockHandler）的 shuffle 模块创建一个 TransportConf。

它在配置中检查 spark.authenticate 键（默认为 false），如果只启用了验证，它使用 ShuffleSecretManager 设置 SaslServerBootstrap 并将其添加到 TransportServerBootstraps 的集合中。

它创建一个 TransportServer 作为 shuffleServer 来监听 spark.shuffle.service.port（默认值：7337）。它在配置中读取 spark.shuffle.service.port 键。

您应该在日志中看到以下 INFO 消息：

```
INFO org.apache.spark.network.yarn.YarnShuffleService: Started YARN shuffle service for Spark on port [port]. Authentication is [authEnabled].  Registered executor file is [registeredExecutorFile]
```

### Installation {#__a_id_installation_a_installation}

#### YARN Shuffle Service Plugin {#__a_id_copy_plugin_a_yarn_shuffle_service_plugin}

将 YARN Shuffle Service 插件从 common / network-yarn 模块添加到 YARN NodeManager 的 CLASSPATH。

| Tip | 使用 yarn classpath 命令知道 YARN 的 CLASSPATH。 |
| :---: | :--- |


```
cp common/network-yarn/target/scala-2.11/spark-2.0.0-SNAPSHOT-yarn-shuffle.jar \
  /usr/local/Cellar/hadoop/2.7.2/libexec/share/hadoop/yarn/lib/
```

#### yarn-site.xml — NodeManager Configuration File {#__a_id_configuration_file_a_yarn_site_xml_nodemanager_configuration_file}

如果启用了外部 shuffle 服务，则需要在所有节点上的 yarn-site.xml 文件中的 yarn.nodemanager.aux-services 中添加 spark\_shuffle。

yarn-site.xml - NodeManager 配置属性

```
<?xml version="1.0"?>
<configuration>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>spark_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services.spark_shuffle.class</name>
    <value>org.apache.spark.network.yarn.YarnShuffleService</value>
  </property>
  <!-- optional -->
  <property>
      <name>spark.shuffle.service.port</name>
      <value>10000</value>
  </property>
  <property>
      <name>spark.authenticate</name>
      <value>true</value>
  </property>
</configuration>
```

yarn.nodemanager.aux-services 属性的辅助服务名称为 spark\_shuffle，yarn.nodemanager.aux-services.spark\_shuffle.class 属性为 org.apache.spark.network.yarn.YarnShuffleService。

### Exception — Attempting to Use External Shuffle Service in Spark Application in Spark on YARN {#_exception_attempting_to_use_external_shuffle_service_in_spark_application_in_spark_on_yarn}

当您在 Spark 应用程序中启用外部 shuffle 服务时，在 YARN 上使用 Spark 但不安装 YARN Shuffle Service 时，您会在日志中看到以下异常：

```
Exception in thread "ContainerLauncher-0" java.lang.Error: org.apache.spark.SparkException: Exception while starting container container_1465448245611_0002_01_000002 on host 192.168.99.1
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1148)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)
Caused by: org.apache.spark.SparkException: Exception while starting container container_1465448245611_0002_01_000002 on host 192.168.99.1
	at org.apache.spark.deploy.yarn.ExecutorRunnable.startContainer(ExecutorRunnable.scala:126)
	at org.apache.spark.deploy.yarn.ExecutorRunnable.run(ExecutorRunnable.scala:71)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	... 2 more
Caused by: org.apache.hadoop.yarn.exceptions.InvalidAuxServiceException: The auxService:spark_shuffle does not exist
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at org.apache.hadoop.yarn.api.records.impl.pb.SerializedExceptionPBImpl.instantiateException(SerializedExceptionPBImpl.java:168)
	at org.apache.hadoop.yarn.api.records.impl.pb.SerializedExceptionPBImpl.deSerialize(SerializedExceptionPBImpl.java:106)
	at org.apache.hadoop.yarn.client.api.impl.NMClientImpl.startContainer(NMClientImpl.java:207)
	at org.apache.spark.deploy.yarn.ExecutorRunnable.startContainer(ExecutorRunnable.scala:123)
	... 4 more
```















