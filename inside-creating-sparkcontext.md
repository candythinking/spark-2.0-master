## Inside Creating SparkContext {#_inside_creating_sparkcontext}

本文档描述了当您创建一个新的 SparkContext 时会发生什么。

```
import org.apache.spark.{SparkConf, SparkContext}

// 1. Create Spark configuration
val conf = new SparkConf()
  .setAppName("SparkMe Application")
  .setMaster("local[*]")  // local mode

// 2. Create Spark context
val sc = new SparkContext(conf)
```

| Note | 该示例在本地模式中使用 Spark，但是与其他群集模式的初始化将遵循类似的步骤。 |
| :---: | :--- |


创建 SparkContext 实例以设置带有 spark.driver.allowMultipleContexts 值的内部 allowMultipleContexts 字段开始，并将此 SparkContext 实例标记为部分构造。它确保没有其他线程在此 JVM 中创建 SparkContext 实例。它通过在 SPARK\_CONTEXT\_CONSTRUCTOR\_LOCK 上同步并使用内部原子引用 activeContext（最终具有完全创建的 SparkContext 实例）来实现。

创建一个完全工作的 SparkContext 实例的 SparkContext 的整个代码在两个语句之间：

```
SparkContext.markPartiallyConstructed(this, allowMultipleContexts)

// the SparkContext code goes here

SparkContext.setActiveContext(this, allowMultipleContexts)
```

startTime 设置为当前时间（以毫秒为单位）。

停止的内部标志设置为 false。

打印出的第一个信息是 Spark 作为 INFO 消息的版本：

```
INFO SparkContext: Running Spark version 2.0.0-SNAPSHOT
```

您可以使用 version 方法来了解当前的 Spark 版本或 org.apache.spark.SPARK\_VERSION 值。

创建一个 LiveListenerBus 实例（作为 listenerBus）。

计算当前用户名。

| Caution | sparkUser 在哪里使用？ |
| :---: | :--- |


它保存输入 SparkConf（如 \_conf）。

| Caution | 查看 \_conf.validateSettings（） |
| :---: | :--- |


它确保定义第一个强制性设置 - spark.master。如果不是则抛出 SparkException。

```
A master URL must be set in your configuration
```

它确保定义了其他强制设置 - spark.app.name。如果不是则抛出 SparkException。

```
An application name must be set in your configuration
```

对于在集群部署模式下的 YARN 上的 Spark，它检查 spark.yarn.app.id 的存在。如果它不存在，则抛出 SparkException。

```
Detected yarn cluster mode, but isn't running on a cluster. Deployment to YARN is not supported directly by SparkContext. Please use spark-submit.
```

| Caution | 如何“触发”异常？有什么步骤？ |
| :---: | :--- |


当 spark.logConf 被启用时 SparkConf.toDebugString 被调用。

| Note | SparkConf.toDebugString 在初始化过程的早期调用，并且不包括其后配置的其他设置。一旦 SparkContext 初始化就使用 sc.getConf.toDebugString 。 |
| :---: | :--- |


如果丢失，则设置驱动程序的主机和端口。 spark.driver.host 变为 Utils.localHostName 的值（或抛出异常），而 spark.driver.port 设置为0。

| Note | spark.driver.host 和 spark.driver.port 需要在驱动程序上设置。它后来被 SparkEnv 断言。 |
| :---: | :--- |


spark.executor.id 设置为驱动程序。

| Tip | 使用 sc.getConf.get（“spark.executor.id”）知道代码在哪里执行 - 驱动程序或执行程序。 |
| :---: | :--- |


它分别基于 spark.jars 和 spark.files 设置 jars 和文件。这些是执行程序正确执行任务所需的文件。

如果启用事件日志记录，即 spark.eventLog.enabled 标志为 true，则内部字段 \_eventLogDir 设置为 spark.eventLog.dir 设置的值或缺省值 / tmp / spark-events。

此外，如果启用 spark.eventLog.compress（它不是​​默认值），CompressionCodec 的短名称将分配给 \_eventLogCodec。配置键是 spark.io.compression.codec（默认值：lz4）。

| Tip | Read about compression codecs in Compression. |
| :--- | :--- |


它将 spark.externalBlockStore.folderName 设置为 externalBlockStoreFolderName 的值。

| Caution | 什么是 externalBlockStoreFolderName？ |
| :---: | :--- |


对于客户端部署模式下的 YARN 上的 Spark，启用 SPARK\_YARN\_MODE 标志。

创建 JobProgressListener 并将其注册到 LiveListenerBus。

A`SparkEnv`is created.

`MetadataCleaner`is created.

| Caution | What’s MetadataCleaner? |
| :---: | :--- |


创建了带有 SparkStatusTracker 的可选 ConsoleProgressBar。

如果启用了 spark.ui.enabled 属性（即 true），SparkUI 将创建一个 Web UI（如 \_ui）。

| Caution | Where’s`_ui`used? |
| :--- | :--- |


创建 Hadoop 配置。请参阅 Hadoop 配置。

如果有通过 SparkContext 构造函数提供的 jar，它们将使用 addJar 添加。对于使用 addFile 的文件也是如此。

在这个时间点，计算要分配给每个执行器的内存量（作为 \_executorMemory）。它是 spark.executor.memory 设置的值，或 SPARK\_EXECUTOR\_MEMORY 环境变量（或当前已弃用的 SPARK\_MEM）的值，或默认为 1024。

\_executorMemory 稍后可用作 sc.executorMemory，并用于 LOCAL\_CLUSTER\_REGEX，Spark Standalone 的 SparkDeploySchedulerBackend，以设置 executorEnvs（“SPARK\_EXECUTOR\_MEMORY”），MesosSchedulerBackend，CoarseMesosSchedulerBackend。

SPARK\_PREPEND\_CLASSES 环境变量的值包含在 executorEnvs 中。

| Caution | 什么是 \_executorMemory？                                                                 \_executorMemory 的值的单位是什么？                                           什么是 “SPARK\_TESTING”，“spark.testing”？他们如何贡献执行者 Envs？                                                                                            什么是 executorEnvs？ |
| :---: | :--- |


Mesos 调度程序后端的配置包括在 executorEnvs 中，即 SPARK\_EXECUTOR\_MEMORY，\_conf.getExecutorEnv 和 SPARK\_USER。

HeartbeatReceiver 注册了 RPC 端点（作为 \_heartbeatReceiver）。

SparkContext.createTaskScheduler 被执行（使用 master URL），结果变成内部 \_schedulerBackend 和 \_taskScheduler。

| Note | 内部 \_schedulerBackend 和 \_taskScheduler 分别由 schedulerBackend 和 taskScheduler 方法使用。 |
| :---: | :--- |


DAGScheduler 已创建（作为 \_dagScheduler）。

SparkContext 向 HeartbeatReceiver RPC 端点发送阻塞 TaskSchedulerIsSet 消息（以通知 TaskScheduler 现在可用）。

TaskScheduler is started

设置内部字段 \_applicationId 和 \_applicationAttemptId（使用 TaskScheduler 合同中的 applicationId 和 applicationAttemptId）。

设置 spark.app.id 设置为当前应用程序标识，如果使用 Web UI，则会通知它（使用 setAppId（\_applicationId））。

BlockManager（用于驱动程序）已初始化（使用 \_applicationId）。

| Caution | 为什么 UI 应该知道应用程序 ID？ |
| :---: | :--- |


MetricsSystem 已启动（在使用 spark.app.id 设置应用程序标识后）。

| Caution | 为什么 Metric System 需要 application id？ |
| :---: | :--- |


在 metrics system 启动后，驱动程序度量（servlet 处理程序）附加到 Web ui。

如果 isEventLogEnabled，则创建并启动 \_eventLogger。它使用 EventLoggingListener 注册到 LiveListenerBus。

| Caution | 为什么 \_eventLogger 需要是 SparkContext 的内部字段？这在哪里使用？ |
| :---: | :--- |


如果启用动态分配，则会创建 ExecutorAllocationManager（作为 \_executorAllocationManager）并立即启动。

| Note | \_executorAllocationManager 被暴露（作为一种方法）到 YARN 调度程序后端，以将其状态重置为初始状态。 |
| :---: | :--- |


如果 spark.cleaner.referenceTracking Spark 属性启用（即 true），SparkContext 创建 ContextCleaner（如 \_cleaner）并立即启动。否则，\_cleaner 为空。

| Note | spark.cleaner.referenceTracking Spark 属性默认情况下启用。 |
| :---: | :--- |


| Caution | 在 sc.getConf.toDebugString 中使用所有属性及其默认值将是非常有用的，所以当没有包括配置但是改变 Spark 运行时配置时，应该将它添加到 \_conf。 |
| :---: | :--- |


它注册用户定义的侦听器并启动 SparkListenerEvent 事件传递到侦听器。

postEnvironmentUpdate 被调用，在 LiveListenerBus 上发布 SparkListenerEnvironmentUpdate 消息，其中包含有关任务计划程序的调度方式，添加的 jar 和文件路径以及其他环境详细信息。它们显示在 Web UI 的“环境”选项卡中。

SparkListenerApplicationStart 消息发布到 LiveListenerBus（使用内部 postApplicationStart 方法）。

通知 TaskScheduler SparkContext 已经启动（使用 postStartHook）。

| Note | TaskScheduler.postStartHook 在默认情况下不做任何操作，但是唯一的实现 TaskSchedulerImpl 自带的 postStartHook 和阻塞当前线程，直到 SchedulerBackend 准备好。 |
| :---: | :--- |


`MetricsSystem`is requested to registerthe following sources:

1. DAGSchedulerSource

2. BlockManagerSource

3. ExecutorAllocationManagerSource\(only if dynamic allocation is enabled\).

ShutdownHookManager.addShutdownHook（）被调用来做 SparkContext 的清理。

| Caution | 什么是 ShutdownHookManager.addShutdownHook（）？ |
| :---: | :--- |


任何非致命异常导致 Spark 上下文实例的终止。

| Caution | NonFatal 在 Scala 中表示什么？ |
| :---: | :--- |


nextShuffleId 和 nextRddId 以0开头。

NOTE:

| Caution | Where are`nextShuffleId`and`nextRddId`used? |
| :--- | :--- |


创建一个 Spark 上下文的新实例，并准备好进行操作。

### Creating SchedulerBackend and TaskScheduler \(createTaskScheduler method\) {#__a_id_createtaskscheduler_a_creating_schedulerbackend_and_taskscheduler_createtaskscheduler_method}

```
createTaskScheduler(
  sc: SparkContext,
  master: String,
  deployMode: String): (SchedulerBackend, TaskScheduler)
```

私有 createTaskScheduler 作为创建 SparkContext 实例的一部分来创建 TaskScheduler 和 SchedulerBackend 对象。

它使用 master URL 来选择正确的实现。

![](/img/mastering-apache-spark/spark core-rdd/figure8.png)

`createTaskScheduler`understands the following master URLs:

* `local`- local mode with 1 thread only

* `local[n]`or`local[*]`- local mode with`n`threads.

* `local[n, m]`or`local[*, m]` — local mode with`n`threads and`m`number of failures.

* `spark://hostname:port`for Spark Standalone.

* `local-cluster[n, m, z]` — local cluster with`n`workers,`m`cores per worker, and`z`memory per worker.

* `mesos://hostname:port`for Spark on Apache Mesos.

* any other URL is passed to`getClusterManager`to load an external cluster manager.

### Loading External Cluster Manager for URL \(getClusterManager method\) {#__a_id_getclustermanager_a_loading_external_cluster_manager_for_url_getclustermanager_method}

```
getClusterManager(url: String): Option[ExternalClusterManager]
```

getClusterManager 加载可以处理输入 url 的 ExternalClusterManager。

如果有两个或更多外部集群管理器可以处理 url，则抛出 SparkException：

```
Multiple Cluster Managers ([serviceLoaders]) registered for the url [url].
```

| Note | getClusterManager 使用 Java 的 ServiceLoader.load 方法。 |
| :---: | :--- |


| Note | getClusterManager 用于在为驱动程序创建 SchedulerBackend 和 TaskScheduler 时查找 master URL 的集群管理器。 |
| :---: | :--- |


### setupAndStartListenerBus {#__a_id_setupandstartlistenerbus_a_setupandstartlistenerbus}

```
setupAndStartListenerBus(): Unit
```

setupAndStartListenerBus 是一个内部方法，从当前 SparkConf 读取 spark.extraListeners 设置，以创建和注册 SparkListenerInterface 侦听器。

它期望类名代表具有以下构造函数之一的 SparkListenerInterface 侦听器（按此顺序）：

* 一个接受 SparkConf 的单参数构造函数

* 一个零参数构造函数

setupAndStartListenerBus 注册每个侦听器类。

您应该在日志中看到以下 INFO 消息：

```
INFO Registered listener [className]
```

它启动 LiveListenerBus 并将其记录在内部 \_listenerBusStarted 中。

当在 spark.extraListeners 设置中没有为类名找到单个 SparkConf 或零参数构造函数时，将抛出 SparkException，并显示以下消息：

```
[className] did not have a zero-argument constructor or a single-argument constructor that accepts SparkConf. Note: if the class is defined inside of another Scala class, then its constructors may accept an implicit parameter that references the enclosing class; in this case, you must define the listener as a top-level class in order to prevent this extra parameter from breaking Spark's ability to find a valid constructor.
```

注册 SparkListenerInterface 侦听器时发生的任何异常都会停止 SparkContext，并抛出 SparkException 和源异常的消息。

```
Exception when registering SparkListener
```

在 org.apache.spark.SparkContext 日志记录器上设置 INFO 以查看正在注册的其他侦听器。

```
INFO SparkContext: Registered listener pl.japila.spark.CustomSparkListener
```

### Creating SparkEnv for Driver \(createSparkEnv method\) {#__a_id_createsparkenv_a_creating_sparkenv_for_driver_createsparkenv_method}

```
createSparkEnv(
  conf: SparkConf,
  isLocal: Boolean,
  listenerBus: LiveListenerBus): SparkEnv
```

createSparkEnv 只是将对 SparkEnv 的调用委派给驱动程序创建一个 SparkEnv。

它为本地 master URL 计算核心数为1，JVM for \* 可用的处理器数量或 master URL 中的确切数字，或者为集群主机 URL 计算为0。

### Utils.getCurrentUserName {#__a_id_getcurrentusername_a_utils_getcurrentusername}

```
getCurrentUserName(): String
```

getCurrentUserName 计算已启动 SparkContext 实例的用户名。

| Note | 它以后可用作 SparkContext.sparkUser。 |
| :---: | :--- |


在内部，它读取 SPARK\_USER 环境变量，如果未设置，则还原为 Hadoop 安全 API 的 UserGroupInformation.getCurrentUser（）.getShortUserName（）。

| Note | 这是 Spark 依赖 Hadoop API 进行操作的另一个地方。 |
| :---: | :--- |


### Utils.localHostName {#__a_id_localhostname_a_utils_localhostname}

localHostName 计算本地主机名。

它首先检查 SPARK\_LOCAL\_HOSTNAME 环境变量的值。如果未定义，则使用 SPARK\_LOCAL\_IP 查找该名称（使用 InetAddress.getByName）。如果没有定义，它将调用 InetAddress.getLocalHost 作为名称。

| Note | Utils.localHostName 在创建 SparkContext 时执行，并且还计算 spark.driver.host Spark 属性的默认值。 |
| :---: | :--- |


| Caution | Review the rest. |
| :--- | :--- |


### stopped flag {#__a_id_stopped_a_stopped_flag}

| Caution | Where is this used? |
| :--- | :--- |






