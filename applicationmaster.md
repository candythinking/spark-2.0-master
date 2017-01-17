## ApplicationMaster \(aka ExecutorLauncher\) {#__a_id_applicationmaster_a_a_id_executorlauncher_a_applicationmaster_aka_executorlauncher}

ApplicationMaster 类充当在 YARN 集群（通常在 YARN 上称为 Spark）上运行的 Spark 应用程序的 YARN ApplicationMaster。

它使用 YarnAllocator 来管理执行者的 YARN 容器。

ApplicationMaster 是一个独立的应用程序，YARN NodeManager 在 YARN 资源容器中运行，负责在 YARN 上执行 Spark 应用程序。

当创建的 ApplicationMaster 类给出一个 YarnRMClient（负责注册和注销一个 Spark 应用程序）。

ExecutorLauncher 是用于客户端部署模式的自定义 ApplicationMaster，目的是在使用 ps 或 jps 时轻松区分客户端和集群部署模式。

```
$ jps -lm

71253 org.apache.spark.deploy.yarn.ExecutorLauncher --arg 192.168.99.1:50188 --properties-file /tmp/hadoop-jacek/nm-local-dir/usercache/jacek/appcache/application_1468961163409_0001/container_1468961163409_0001_01_000001/__spark_conf__/__spark_conf__.properties

70631 org.apache.hadoop.yarn.server.resourcemanager.ResourceManager
70934 org.apache.spark.deploy.SparkSubmit --master yarn --class org.apache.spark.repl.Main --name Spark shell spark-shell
71320 sun.tools.jps.Jps -lm
70731 org.apache.hadoop.yarn.server.nodemanager.NodeManager
```

ApplicationMaster（和 ExecutorLauncher）作为客户端创建 ContainerLaunchContext 以在 YARN 上启动 Spark 的结果启动。

![](/img/mastering-apache-spark/spark on yarn/figure4.png)

| Note | ContainerLaunchContext 表示 YARN NodeManager 启动容器所需的所有信息。 |
| :---: | :--- |


### client Internal Reference to YarnRMClient {#__a_id_client_a_client_internal_reference_to_yarnrmclient}

client 是创建时 ApplicationMaster 给出的 YarnRMClient 的内部引用。

客户端主要用于注册 ApplicationMaster 和请求来自 YARN 的执行程序的容器，稍后从 YARN ResourceManager 注销 ApplicationMaster。

此外，它有助于获得应用程序尝试 ID 和注册 ApplicationMaster 的允许尝试次数。它还获取过滤器参数以保护 ApplicationMaster 的 UI。

### allocator Internal Reference to YarnAllocator {#__a_id_allocator_a_allocator_internal_reference_to_yarnallocator}

allocator 是 ApplicationMaster 用来向执行者请求新的或释放未完成容器的 YarnAllocator 的内部引用。

它在 ApplicationMaster 注册时创建（使用内部 YarnRMClient 引用）。

### main {#__a_id_main_a_main}

ApplicationMaster 作为节点上的 YARN 容器内的独立命令行应用程序启动。

| Note | 命令行应用程序作为发送 ContainerLaunchContext 请求以启动 ApplicationMaster 到 YARN ResourceManager（在为 ApplicationMaster 创建请求后）执行的结果 |
| :---: | :--- |


![](/img/mastering-apache-spark/spark on yarn/figure5.png)

当执行时，主要首先解析命令行参数，然后使用 SparkHadoopUtil.runAsSparkUser 运行带有 Hadoop UserGroupInformation 的主代码作为线程局部变量（分发给子线程），用于认证 HDFS 和 YARN 调用。

Enable`DEBUG`logging level for`org.apache.spark.deploy.SparkHadoopUtil`logger to see what happens inside.+

Add the following line to`conf/log4j.properties`:

```
log4j.logger.org.apache.spark.deploy.SparkHadoopUtil=DEBUG
```

You should see the following message in the logs:

```
DEBUG running as user: [user]
```

SparkHadoopUtil.runAsSparkUser 函数执行一个创建 ApplicationMaster（传递 ApplicationMasterArguments 实例和一个全新的 YarnRMClient）的块，然后运行它。

### Command-Line Parameters \(ApplicationMasterArguments class\) {#__a_id_command_line_parameters_a_a_id_applicationmasterarguments_a_command_line_parameters_applicationmasterarguments_class}

ApplicationMaster 使用 ApplicationMasterArguments 类来处理命令行参数。

ApplicationMasterArguments 是在对 args 命令行参数执行 main 方法之后创建的。

它接受以下命令行参数：

* `--jar JAR_PATH` — the path to the Spark application’s JAR file

* `--class CLASS_NAME` — the name of the Spark application’s main class

* `--arg ARG` — an argument to be passed to the Spark application’s main class. There can be multiple`--arg`arguments that are passed in order.

* `--properties-file FILE` — the path to a custom Spark properties file.+

* `--primary-py-file FILE` — the main Python file to run.

* `--primary-r-file FILE` — the main R file to run.

当找到不支持的参数时，会将以下消息输出到标准错误输出，ApplicationMaster 会退出，退出代码为1。

```
Unknown/unsupported param [unknownParam]

Usage: org.apache.spark.deploy.yarn.ApplicationMaster [options]
Options:
  --jar JAR_PATH       Path to your application's JAR file
  --class CLASS_NAME   Name of your application's main class
  --primary-py-file    A main Python file
  --primary-r-file     A main R file
  --arg ARG            Argument to be passed to your application's main class.
                       Multiple invocations are possible, each will be passed in order.
  --properties-file FILE Path to a custom Spark properties file.
```

### Registering ApplicationMaster with YARN ResourceManager and Requesting Resources \(registerAM method\) {#__a_id_registeram_a_registering_applicationmaster_with_yarn_resourcemanager_and_requesting_resources_registeram_method}

当执行 runDriver 或 runExecutorLauncher 时，它们使用私有帮助程序 registerAM 注册 ApplicationMaster（使用 YARN ResourceManager）和请求资源（给出关于在何处分配容器以尽可能接近数据的资源）。

```
registerAM(
  _rpcEnv: RpcEnv,
  driverRef: RpcEndpointRef,
  uiAddress: String,
  securityMgr: SecurityManager): Unit
```

在内部，它首先读取 spark.yarn.historyServer.address 设置，并替换 Hadoop 变量以创建历史记录服务器的完整地址，即 \[address\] / history / \[appId\] / \[attemptId\]。

| Caution | 替换 Hadoop 变量？ |
| :---: | :--- |


然后，registerAM 在 spark.driver.host 和 spark.driver.port Spark 属性上可用的驱动程序上为 CoarseGrainedScheduler RPC Endpoint 创建一个 RpcEndpointAddress。

它使用 YARN ResourceManager 和请求资源注册 ApplicationMaster（给出关于在何处分配容器以尽可能接近数据的提示）。

最终，registerAM启动记录线程。

![](/img/mastering-apahce-spark/spark on yarn/figure6.png)

### Running Driver in Cluster Mode \(runDriver method\) {#__a_id_rundriver_a_running_driver_in_cluster_mode_rundriver_method}

```
runDriver(securityMgr: SecurityManager): Unit
```

`runDriver`is a private procedure to…​???

它首先注册 Web UI 安全过滤器。

| Caution | 为什么需要这个？ addAmIpFilter |
| :---: | :--- |


It then starts the user class \(with the driver\) in a separate thread. You should see the following INFO message in the logs:

```
INFO Starting the user application in a separate Thread
```

| Caution | Review`startUserApplication`. |
| :--- | :--- |


You should see the following INFO message in the logs:

```
INFO Waiting for spark context initialization
```

| Caution | Review`waitForSparkContextInitialized` |
| :--- | :--- |


| Caution | Finish…​ |
| :--- | :--- |


### Running Executor Launcher \(runExecutorLauncher method\) {#__a_id_runexecutorlauncher_a_running_executor_launcher_runexecutorlauncher_method}

```
runExecutorLauncher(securityMgr: SecurityManager): Unit
```

runExecutorLauncher 读取 spark.yarn.am.port（或假设为0）并启动 sparkYarnAM RPC 环境（在客户端模式下）。

| Caution | FIXME What’s client mode? |
| :--- | :--- |


It then waits for the driver to be available.

| Caution | FIXME Review`waitForSparkDriver` |
| :--- | :--- |


It registers Web UI security filters.

| Caution | FIXME Why is this needed?`addAmIpFilter` |
| :--- | :--- |


最终，runExecutorLauncher 注册 ApplicationMaster 并请求资源并等待，直到 reporterThread 死亡。

| Caution | FIXME Describe`registerAM` |
| :--- | :--- |


### reporterThread {#__a_id_reporterthread_a_reporterthread}

| Caution | FIXME |
| :--- | :--- |


### launchReporterThread {#__a_id_launchreporterthread_a_launchreporterthread}

| Caution | FIXME |
| :--- | :--- |


### Setting Internal SparkContext Reference \(sparkContextInitialized methods\) {#__a_id_sparkcontextinitialized_a_setting_internal_sparkcontext_reference_sparkcontextinitialized_methods}

```
sparkContextInitialized(sc: SparkContext): Unit
```

sparkContextInitialized 将调用传递到 ApplicationMaster.sparkContextInitialized，它设置内部 sparkContextRef 引用（为 sc）。

### Clearing Internal SparkContext Reference \(sparkContextStopped methods\) {#__a_id_sparkcontextstopped_a_clearing_internal_sparkcontext_reference_sparkcontextstopped_methods}

```
sparkContextStopped(sc: SparkContext): Boolean
```

sparkContextStopped 将调用传递到清除内部 sparkContextRef 引用（即将其设置为 null）的 ApplicationMaster.sparkContextStopped。

### Creating ApplicationMaster Instance {#__a_id_creating_instance_a_creating_applicationmaster_instance}

![](/img/mastering-apache-spark/spark on yarn/figure6.png)

当创建 ApplicationMaster 的实例时，它需要 ApplicationMasterArguments 和 YarnRMClient。

它实例化 SparkConf 和 Hadoop 的 YarnConfiguration（使用 SparkHadoopUtil.newConfiguration）。

当指定--class 时，它假定为集群部署模式。

它使用可选的 spark.yarn.max.executor.failures（如果设置）计算内部 maxNumExecutorFailures。否则，它是两次 spark.executor.instances 或 spark.dynamicAllocation.maxExecutors（启用动态分配），最小值为 3。

它从 YARN 读取 yarn.am.liveness-monitor.expiry-interval-ms（默认值：120000）以设置心跳间隔。它设置为 YARN 设置的最小值或 spark.yarn.scheduler.heartbeat.interval-ms 的最小值，最小值为0。

initialAllocationInterval 设置为心跳间隔的最小值或 spark.yarn.scheduler.initial-allocation.interval。

然后加载本地化文件（由客户端设置）。

| Caution | FIXME Who’s the client? |
| :--- | :--- |


### localResources attribute {#__a_id_localresources_a_localresources_attribute}

当 ApplicationMaster 被实例化时，它将根据内部 spark.yarn.cache.\* 配置设置，通过名称计算 YARN 的 LocalResource 的内部 localResources 集合。

```
localResources: Map[String, LocalResource]
```

You should see the following INFO message in the logs:

```
INFO ApplicationMaster: Preparing Local resources
```

它开始于读取内部 Spark 配置设置（当客户端准备本地资源分发时，之前设置）：

* spark.yarn.cache.filenames

* spark.yarn.cache.sizes

* spark.yarn.cache.timestamps

* spark.yarn.cache.visibilities+

* spark.yarn.cache.types

对于 spark.yarn.cache.filename 中的每个文件名，它将 spark.yarn.cache.types 映射到适当的 YARN 的 LocalResourceType 并创建一个新的 YARN LocalResource。 

| Note | LocalResource 表示运行容器所需的本地资源。 |
| :---: | :--- |


如果设置了 spark.yarn.cache.confArchive，则将其作为 ARCHIVE 资源类型和 PRIVATE 可见性添加到 localResources。

| Note | 当客户端准备本地资源时，将设置 spark.yarn.cache.confArchive。 |
| :---: | :--- |


| Note | ARCHIVE 是归档文件，由 NodeManager 自动取消归档。 |
| :---: | :--- |


| Note | PRIVATE 可见性意味着在节点上的同一用户的所有应用程序之间共享资源。 |
| :---: | :--- |


最终，它从 Spark 配置和系统属性中删除缓存相关的设置。 您应该在日志中看到以下 INFO 消息：

```
INFO ApplicationMaster: Prepared Local resources [resources]
```

### Running ApplicationMaster \(run method\) {#__a_id_run_a_running_applicationmaster_run_method}

当 ApplicationMaster 作为独立的命令行应用程序启动（在 YARN 集群中的节点上的 YARN 容器中）时，将最终运行。

```
run(): Int
```

调用 run 的结果是 ApplicationMaster 命令行应用程序的最终结果。

运行集群模式设置，注册清理关闭挂接，调度 AMDelegationTokenRenewer，最后为 Spark 应用程序注册 ApplicationMaster（为群集模式调用 runDriver 或为客户端模式调用 runExecutorLauncher）。

After the cluster mode settings are set, run prints the following INFO message out to the logs:

```
INFO ApplicationAttemptId: [appAttemptId]
```

appAttemptId 是当前应用程序尝试 ID（使用构造函数的 YarnRMClient 作为客户端）。 清除关闭挂钩注册关闭优先级低于 SparkContext（因此它在 SparkContext 之后执行）。

SecurityManager 使用内部 Spark 配置实例化。如果存在凭据文件配置（如 spark.yarn.credentials.file），则启动 AMDelegationTokenRenewer。

| Caution | 描述 AMDelegationTokenRenewer＃scheduleLoginFromKeytab |
| :---: | :--- |


它最终为 Spark 应用程序运行 ApplicationMaster（在集群模式下调用 runDriver，否则调用 runExecutorLauncher）。

It exits with`0`exit code。

In case of an exception,`run`prints the following ERROR message out to the logs:

```
ERROR Uncaught exception: [exception]
```

应用程序运行尝试以 FAILED 状态和 EXIT\_UNCAUGHT\_EXCEPTION（10）退出代码完成。

### Cluster Mode Settings {#__a_id_cluster_mode_settings_a_cluster_mode_settings}

When in cluster mode,`ApplicationMaster`sets the following system properties \(in run\):

* spark.ui.port as`0`

* spark.master as`yarn`

* spark.submit.deployMode as`cluster`

* spark.yarn.app.id as application id

| Caution | FIXME Why are the system properties required? Who’s expecting them? |
| :--- | :--- |


### isClusterMode Internal Flag {#__a_id_cluster_mode_a_a_id_isclustermode_a_isclustermode_internal_flag}

| Caution | 由于 org.apache.spark.deploy.yarn.ExecutorLauncher 用于客户端部署模式，isClusterMode 标志可以设置在那里（不依赖于 --class，这是正确的，但不是很明显）。 |
| :---: | :--- |


isClusterMode 是为集群模式启用（即 true）的内部标志。

具体来说，它说明是否指定了 Spark 应用程序的主类（通过 --class 命令行参数）。这就是当客户创建 YARN 的 ContainerLaunchContext（用于启动 ApplicationMaster）时，开发人员决定通知 ApplicationMaster 关于以集群模式运行的方式。

它用于在 run 和 runDriver（启用标志）或 runExecutorLauncher（禁用时）设置其他系统属性。

此外，它控制 Spark 应用程序的默认最终状态为 FinalApplicationStatus.FAILED（启用标志时）或 FinalApplicationStatus.UNDEFINED。

该标志还控制是否在 addAmIpFilter 中设置系统属性（当启用标志时）或发送 AddWebUIFilter。

### Unregistering ApplicationMaster from YARN ResourceManager \(unregister method\) {#__a_id_unregister_a_unregistering_applicationmaster_from_yarn_resourcemanager_unregister_method}

`unregister`unregisters the`ApplicationMaster`for the Spark application from the YARN ResourceManager

```
unregister(status: FinalApplicationStatus, diagnostics: String = null): Unit
```

| Note | 它从 cleanup shutdown hook（在 ApplicationMaster 中开始运行时注册）中调用，并且只在应用程序的最终结果成功或者是最后一次运行应用程序时尝试。 |
| :---: | :--- |


It first checks that the`ApplicationMaster`has not already been unregistered \(using the internal`unregistered`flag\). If so, you should see the following INFO message in the logs:

```
INFO ApplicationMaster: Unregistering ApplicationMaster with [status]
```

There can also be an optional diagnostic message in the logs:

```
(diag message: [msg])
```

内部未注册标志被设置为启用，即 true。

然后请求 YarnRMClient 注销。

### Cleanup Shutdown Hook {#__a_id_shutdown_hook_a_cleanup_shutdown_hook}

当 ApplicationMaster 开始运行时，它注册一个关闭挂钩\(shutdown hook\)，从 YARN ResourceManager 注销 Spark 应用程序，并清除暂存目录。

在内部，它检查内部完成标志，如果它被禁用，它标记 Spark 应用程序失败与 EXIT\_EARLY。

如果禁用了内部未注册标志，它将取消注册 Spark 应用程序，并仅在 ApplicationMaster 的注册的最终状态为 FinalApplicationStatus.SUCCEEDED 或应用程序尝试次数超过允许次数时清除暂存目录。

shutdown hook 在 SparkContext 关闭后运行，即关闭优先级比 SparkContext 少一个。

shutdown hook（关闭挂钩）使用 Spark 自己的 ShutdownHookManager.addShutdownHook 注册。

### finish {#__a_id_finish_a_finish}

| Caution | FIXME |
| :--- | :--- |


### ExecutorLauncher {#__a_id_executorlauncher_a_executorlauncher}

与 ApplicationMaster 相比，ExecutorLauncher 没有额外的功能。它作为一个帮助类，在客户端部署模式下在另一个类名下运行 ApplicationMaster。

使用两个不同的类名（指向同一个类 ApplicationMaster），您应该更成功地使用工具（如 ps 或 jps）来区分客户端部署模式下的 ExecutorLauncher（实际上是 ApplicationMaster）和集群部署模式下的 ApplicationMaster。

| Note | 考虑 ExecutorLauncher 用于客户端部署模式的 ApplicationMaster。 |
| :---: | :--- |


### Obtain Application Attempt Id \(getAttemptId method\) {#__a_id_getattemptid_a_obtain_application_attempt_id_getattemptid_method}

```
getAttemptId(): ApplicationAttemptId
```

getAttemptId 返回 YARN 的 ApplicationAttemptId（容器被分配到的 Spark 应用程序的）。

在内部，它通过 YarnRMClient 查询 YARN。

### addAmIpFilter helper method {#__a_id_addamipfilter_a_addamipfilter_helper_method}

```
addAmIpFilter(): Unit
```

`addAmIpFilter`is a helper method that …​???

它首先读取 Hadoop 的环境变量 ApplicationConstants.APPLICATION\_WEB\_PROXY\_BASE\_ENV，然后传递给 YarnRMClient 以计算 Web UI 的 AmIpFilter 的配置。

在集群部署模式下（当 ApplicationMaster 使用 Web UI 运行时），它将 spark.ui.filters 系统属性设置为 org.apache.hadoop.yarn.server.webproxy.amfilter.AmIpFilter。它还将来自 AmIpFilter（之前计算）的键值配置的系统属性设置为 spark.org.apache.hadoop.yarn.server.webproxy.amfilter.AmIpFilter.param.\[key\] being \[value\]。

在客户端部署模式下（当 ApplicationMaster 在另一个 JVM 甚至是主机而不是 Web UI 上运行时），它只是向 ApplicationMaster（即到 AMEndpoint RPC Endpoint）发送一个 AddWebUIFilter。





























