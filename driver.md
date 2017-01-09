## Driver {#_driver}

Spark 驱动程序（也称为应用程序的驱动程序进程）是一个 JVM 进程，它为 Spark 应用程序提供 SparkContext。它是 Spark 应用程序中的主节点。

它是作业和任务执行的驾驶舱（使用 DAGScheduler 和任务计划程序）。它托管环境的 Web UI。

![](/img/mastering-apache-spark/spark core-architecture/figure3.png)

它将 Spark 应用程序分成任务，并将它们调度到执行器上运行。

驱动程序是任务调度器生成并在 workers 中生成任务的地方。

驱动程序协调 worker 和任务的总体执行。

| Tip | Spark shell 是一个 Spark 应用程序和驱动程序。它创建一个可用作 sc 的 SparkContext。 |
| :---: | :--- |


驱动程序需要额外的服务（除了像 ShuffleManager，MemoryManager，BlockTransferService，BroadcastManager，CacheManager 等常见的服务）：

* Listener Bus

* RPC Environment

* MapOutputTrackerMaster with the name**MapOutputTracker**

* BlockManagerMaster with the name**BlockManagerMaster**

* HttpFileServer

* MetricsSystem with the name**driver**

* OutputCommitCoordinator with the endpoint’s name **OutputCommitCoordinator**

| Caution | 驱动程序（及以后的执行程序）的RpcEnv图。也许应该在关于 RpcEnv 的注释？ |
| :--- | :--- |


* High-level control flow of work

* Your Spark application runs as long as the Spark driver.

  * Once the driver terminates, so does your Spark application.

* Creates`SparkContext`, \`RDD’s, and executes transformations and actions

* Launches tasks

### Driver’s Memory {#__a_id_driver_memory_a_driver_s_memory}

它可以首先使用 spark-submit 的 --driver-memory 命令行选项或 spark.driver.memory 进行设置，如果未设置，则会回到 SPARK\_DRIVER\_MEMORY。

| Note | 它以 spark-submit 的详细模式打印到标准错误输出。 |
| :---: | :--- |


### Driver’s Cores {#__a_id_driver_memory_a_driver_s_cores}

它可以首先使用 spark-submit 的 --driver-cores 命令行选项为集群部署模式设置

| Note | 在客户端部署模式下，驱动程序的内存对应于运行 Spark 应用程序的 JVM 进程的内存。 |
| :---: | :--- |


| Note | 它以 spark-submit 的详细模式打印到标准错误输出。 |
| :---: | :--- |


### Settings {#__a_id_settings_a_settings}

Table 1. Spark Properties

| Spark Property | Default Value | Description |
| :--- | :--- | :--- |
| `spark.driver.blockManager.port` | spark.blockManager.port | 驱动程序上的 BlockManager 使用的端口。 更准确地说，当创建 NettyBlockTransferService 时（当为驱动程序创建 SparkEnv 时）使用 spark.driver.blockManager.port。 |
| `spark.driver.host` | localHostName | 驱动程序运行的节点的地址。 创建 SparkContext 时设置 |
| `spark.driver.memory` | `1g` | 驱动程序的内存大小（以 MiB 为单位）。 请参阅驱动程序内存。 |
| `spark.driver.cores` | `1` | 在集群部署模式下分配给驱动程序的 CPU 核心数。 注意：创建客户端时（仅适用于集群模式下的 YARN 上的 Spark），它将使用 spark.driver.cores 设置 ApplicationManager 的核心数。 参考驱动程序的内核。 |
| `spark.driver.extraLibraryPath` |  |  |
| `spark.driver.extraJavaOptions` |  | Additional JVM options for the driver. |
| spark.driver.appUIAddress | `spark.driver.libraryPath` | spark.driver.appUIAddress 仅用于 YARN 上的 Spark。它在 YarnClientSchedulerBackend 开始运行 ExecutorLauncher（并注册 Application 应用程序的 ApplicationMaster）时设置。 |

#### spark.driver.extraClassPath {#__a_id_spark_driver_extraclasspath_a_spark_driver_extraclasspath}

spark.driver.extraClassPath 系统属性设置应在集群部署模式下添加到驱动程序类路径的附加类路径条目（例如 jars 和目录）。

| Note | 对于客户端部署模式，您可以使用属性文件或命令行来设置 spark.driver.extraClassPath。 不要使用 SparkConf，因为对于客户端部署模式来说，由于已经设置了 JVM 以启动 Spark 应用程序，因此太迟了。 有关如何在内部处理的非常低级别的细节，请参考 buildSparkSubmitCommand 内部方法。 |
| :--- | :--- |


`spark.driver.extraClassPath`uses a OS-specific path separator.

| Note | 在命令行中使用 spark-submit 的 --driver-class-path 命令行选项来从 Spark 属性文件覆盖 spark.driver.extraClassPath。 |
| :--- | :--- |




