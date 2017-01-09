## Executors {#_executors}

执行器是执行任务的分布式代理。

它们通常在 Spark 应用程序的整个生命周期内运行，并称为执行程序的静态分配（但也可以选择进行动态分配）。

执行程序向驱动程序发送活动任务度量，并通知执行程序后端关于任务状态更新（包括任务结果）。

| Note | 执行器由执行器后端专门管理。 |
| :---: | :--- |


执行器为在 Spark 应用程序中缓存的 RDD 提供内存存储（通过块管理器Block Manager）。

当启动执行程序时，它们向驱动程序注册自己并直接进行通信以执行任务。

执行程序提供由 executor id 和执行程序运行的主机描述（请参阅本文档中的资源提供）。

执行器可以在其生命周期中并行和顺序地运行多个任务。它们跟踪运行的任务（通过其在 runningTasks 内部注册表中的任务标识）。请参阅启动任务部分。

执行程序使用线程池来启动任务和发送指标。

建议具有与数据节点一样多的执行程序以及从集群中获得的一样多内核。

执行器由它们的 id，主机名，环境（作为 SparkEnv）和类路径（以及不太重要，更多的内部优化，无论它们是以本地还是集群模式运行）来描述。

Table 1. Executors Internal Registries and Counters

| Name | Description |
| :--- | :--- |
| `runningTasks` |  |
| `heartbeatFailures` |  |

| Tip | 为org.apache.spark.executor.Executor记录器启用INFO或DEBUG日志记录级别，以查看内部发生了什么。                                log4j.logger.org.apache.spark.executor.Executor = INFO    请参阅 Logging |
| :--- | :--- |


### Stopping Executor — `stop`Method {#__a_id_stop_a_stopping_executor_code_stop_code_method}

| Caution | FIXME |
| :--- | :--- |


### Creating`Executor`Instance {#__a_id_creating_instance_a_creating_code_executor_code_instance}

执行器需要 executorId，executorHostname，SparkEnv，userClassPath 以及它是以本地还是集群模式运行（默认使用集群）。

| Note | isLocal 仅针对 LocalEndpoint（对于本地模式下的 Spark）启用。 |
| :---: | :--- |


创建时，您应该在日志中看到以下 INFO 消息：

```
INFO Executor: Starting executor ID [executorId] on host [executorHostname]
```

它创建了一个用于向驱动程序发送警报的 RPC 端点。

在非本地/集群模式下，将初始化 BlockManager。

| Tip | 一个执行器的 BlockManager 在 SparkEnv 中传递给构造函数。 |
| :---: | :--- |


A worker requires the additional services \(beside the common ones like …​\):

* executorActorSystemName

* RPC Environment\(for Akka only\)+

* MapOutputTrackerWorker

* MetricsSystem with the name`executor`

创建 ExecutorSource（使用 executorId）。并且，仅对于集群模式，请求 MetricsSystem 注册它。

（仅适用于集群模式）BlockManager 已初始化。

| Note | 当 CoarseGrainedExecutorBackend 接收到 RegisteredExecutor 消息，在 MesosExecutorBackend.registered 和当创建 LocalEndpoint 时，创建一个 Executor。 |
| :---: | :--- |


| Caution | 每个执行程序分配多少个核心？ |
| :--- | :--- |


### Launching Tasks — `launchTask`Method {#__a_id_launchtask_a_a_id_launching_tasks_a_launching_tasks_code_launchtask_code_method}

```
launchTask(
  context: ExecutorBackend,
  taskId: Long,
  attemptNumber: Int,
  taskName: String,
  serializedTask: ByteBuffer): Unit
```

launchTask 同时执行输入 serializedTask 任务。

在内部，launchTask 创建一个 TaskRunner，在 runningTasks 内部注册表中注册它（由 taskId），最后在 “Executor task launch worker” 线程池上执行它。

![](/img/mastering-apache-spark/spark core-architecture/figure4.png)

| Note | launchTask 由 CoarseGrainedExecutorBackend（当它处理 LaunchTask 消息时），MesosExecutorBackend 和 LocalEndpoint 调用。 |
| :---: | :--- |


### Sending Heartbeats and Active Tasks Metrics — `startDriverHeartbeater`Method {#__a_id_startdriverheartbeater_a_a_id_heartbeats_and_active_task_metrics_a_sending_heartbeats_and_active_tasks_metrics_code_startdriverheartbeater_code_method}

执行程序每隔 spark.executor.heartbeatInterval（默认为10秒，一些随机的初始延迟，所以来自不同执行器的心跳不会堆积在驱动程序上）继续向驱动程序发送活动任务的度量。

![](/img/mastering-apache-spark/spark core-architecture/figure5.png)

执行器使用内部心跳线 - 心跳发送器线程发送心跳。

![](/img/mastering-apache-spark/spark core-architecture/figure6.png)

对于 TaskRunner 中的每个任务（在 runningTasks 内部注册表中），计算成为心跳（包括累加器）一部分的任务度量（即 mergeShuffleReadMetrics 和 setJvmGCTime）。

| Caution | mergeShuffleReadMetrics 和 setJvmGCTime 如何影响累加器？ |
| :---: | :--- |


| Note | 执行程序跟踪运行任务的 TaskRunner。当执行程序发送心跳时，任务可能不会分配给 TaskRunner。 |
| :---: | :--- |


阻塞 Heartbeat 消息保存执行器 ID，所有累加器更新（每个任务 ID）和 BlockManagerId 发送到 HeartbeatReceiver RPC 端点（使用 spark.executor.heartbeatInterval 超时）。

| Caution | 什么时候创建 heartbeatReceiverRef？ |
| :---: | :--- |


如果响应请求重新注册 BlockManager，您应该在日志中看到以下 INFO 消息：

```
INFO Executor: Told to re-register on heartbeat
```

重新注册 BlockManager。

内部 heartbeatFailures 计数器复位（即变为0）。

如果与驱动程序通信有任何问题，您应该在日志中看到以下WARN消息：

```
WARN Executor: Issue communicating with driver in heartbeater
```

内部 heartbeatFailures 增加并检查为小于可接受的故障数。如果数字较大，则将以下 ERROR 打印到日志中：

```
ERROR Executor: Exit as unable to send heartbeats to driver more than [HEARTBEAT_MAX_FAILURES] times
```

执行器退出（使用 System.exit 和退出代码56）。

| Tip | 阅读 TaskMetrics 中的 TaskMetrics。 |
| :---: | :--- |


### heartbeater - Heartbeat Sender Thread {#__a_id_heartbeater_a_heartbeater_heartbeat_sender_thread}

heartbeater 是一个守护进程 ScheduledThreadPoolExecutor 与单线程。

线程池的名称是 driver-heartbeater。

### Coarse-Grained Executors {#__a_id_coarse_grained_executor_a_coarse_grained_executors}

粗粒度执行器 \(**Coarse-grained executors\) **是使用 CoarseGrainedExecutorBackend 执行任务调度的执行器。

### Resource Offers {#__a_id_resource_offers_a_resource_offers}

在 TaskSetManager 中的 TaskSchedulerImpl 和 resourceOffer 中读取 resourceOffers。

### "Executor task launch worker" Thread Pool {#__a_id_threadpool_a_a_id_thread_pool_a_executor_task_launch_worker_thread_pool}

执行程序使用名为 Executor 任务启动工作程序标识（ID 为任务标识）的守护程序缓存线程池来启动任务。

### Executor Memory — `spark.executor.memory`or`SPARK_EXECUTOR_MEMORY`settings {#__a_id_memory_a_executor_memory_code_spark_executor_memory_code_or_code_spark_executor_memory_code_settings}

您可以使用 spark.executor.memory 设置来控制每个执行程序的内存量。它为每个应用程序的所有执行程序平等地设置可用内存。

| Note | 创建 SparkContext 时，将查找每个执行程序的内存量。 |
| :---: | :--- |


您可以使用 SPARK\_EXECUTOR\_MEMORY 环境变量更改独立集群中每个节点的每个执行程序分配的内存。

您可以在独立主站的 Web UI 中找到作为每个节点的内存显示的值（如下图所示）。

![](/im/mastering-apache-spark/spark core-architecture/figure7.png)

上图显示了运行 Spark shell 的结果，其中显式地（在命令行上）定义了每个执行器的内存量，即

```
./bin/spark-shell --master spark://localhost:7077 -c spark.executor.memory=2g
```

### Metrics {#__a_id_metrics_a_metrics}

Every executor registers its own ExecutorSource to report metrics.

### Settings {#__a_id_settings_a_settings}

|  |  |  |
| :--- | :--- | :--- |
| Spark Property | Default Value | Description |
| `spark.executor.cores` |  | Number of cores for an executor. |
| `spark.executor.extraClassPath` |  | 表示用户的CLASSPATH的URL列表。每个条目由系统相关的路径分隔符分隔，即：在 Unix / MacOS 系统上;在 Microsoft Windows 上。 |
| `spark.executor.extraJavaOptions` |  | 执行器的额外 Java 选项。 用于准备在 YARN 容器中启动 CoarseGrainedExecutorBackend 的命令。 |
| `spark.executor.extraLibraryPath` |  | 由系统相关路径分隔符分隔的附加库路径列表，即：在 Unix / MacOS 系统上;在 Microsoft Windows 上。 用于准备在 YARN 容器中启动 CoarseGrainedExecutorBackend 的命令。 |
| `spark.executor.userClassPathFirst` | `false` | 控制是否在 Spark jars 中的用户 jar 之前加载类的标志。 |
| `spark.executor.heartbeatInterval` | `10s` | 执行器向驱动程序报告活动任务的心跳和度量的时间间隔。 请参阅本文档中的活动任务的发送心跳和部分指标。 |
| `spark.executor.heartbeat.maxFailures` | `60` | 执行器在放弃和退出之前尝试向驱动程序发送心跳的次数（带退出代码56）。 注意：它被引入在 SPARK-13​​522 当它无法心跳到驱动程序超过 N 次时，执行器应该杀死自己。 |
| `spark.executor.id` |  |  |
| `spark.executor.instances` | `0` | 要使用的执行程序数。 注意：大于0时，将禁用动态分配。 |
| `spark.executor.memory` | `1g` | 每个执行程序进程使用的内存量（相当于 SPARK\_EXECUTOR\_MEMORY 环境变量）。 请参阅本文档中的 Executor Memory - spark.executor.memory 或 SPARK\_EXECUTOR\_MEMORY 设置。 |
| `spark.executor.port` |  |  |
| `spark.executor.logs.rolling.maxSize` |  |  |
| `spark.executor.logs.rolling.maxRetainedFiles` |  |  |
| `spark.executor.logs.rolling.strategy` |  |  |
| `spark.executor.logs.rolling.time.interval` |  |  |
| `spark.executor.port` |  |  |
| `spark.executor.uri` |  | 相当于`SPARK_EXECUTOR_URI` |
| `spark.task.maxDirectResultSize` | `1048576B` |   |



