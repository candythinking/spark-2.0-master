## `SparkContext` — Entry Point to Spark \(Core\) {#__a_id_sparkcontext_a_code_sparkcontext_code_entry_point_to_spark_core}

SparkContext（也称为 Spark 上下文）是 Spark 应用程序的 Spark 的入口点。

| Note | 您还可以假定 SparkContext 实例是 Spark 应用程序。 |
| :---: | :--- |


它设置内部服务并建立到 Spark 执行环境的连接（部署模式）。

创建 SparkContext 实例后，您可以使用它创建 RDD，累加器和广播变量，访问 Spark 服务和运行作业（直到 SparkContext 停止）。

Spark 上下文本质上是 Spark 的执行环境的客户端，并且充当 Spark 应用程序的 master（不要与 Spark 中的 Master 的其他含义相混淆）。

![](/img/mastering-apache-spark/spark core-rdd/figure2.png)

`SparkContext`offers the following functions:

* Getting current configuration

  * SparkConf

  * deployment environment \(as master URL\)

  * application name

  * deploy mode

  * default level of parallelism+

  * Spark user

  * the time \(in milliseconds\) when`SparkContext`was created

  * Spark version

* Setting Configuration

  * master URL

  * Local Properties — Creating Logical Job Groups

  * Default Logging Level

* Creating Distributed Entities

  * RDDs

  * Accumulators

  * Broadcast variables

* Accessing services, e.g.TaskScheduler,LiveListenerBus,BlockManager,SchedulerBackends,ShuffleManagerand the optional ContextCleaner.

* Running jobs

* Cancelling job

* Setting up custom Scheduler Backend, TaskScheduler and DAGScheduler

* Closure Cleaning

* Submitting Jobs Asynchronously

* Unpersisting RDDs, i.e. marking RDDs as non-persistent

* Registering SparkListener

* Programmable Dynamic Allocation

| Tip | Read the scaladoc of [org.apache.spark.SparkContext](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.SparkContext). |
| :--- | :--- |


| Tip | Enable`INFO`logging level for`org.apache.spark.SparkContext`logger to see what happens inside.                                      Add the following line to`conf/log4j.properties`:                  log4j.logger.org.apache.spark.SparkContext=INFO              Refer to Logging. |
| :--- | :--- |


### Cancelling Job — `cancelJob`Method {#__a_id_canceljob_a_cancelling_job_code_canceljob_code_method}

```
cancelJob(jobId: Int)
```

`cancelJob`requests`DAGScheduler`to cancel a Spark job`jobId`.

### Persisted RDDs {#__a_id_persistentrdds_a_persisted_rdds}

| Caution | FIXME |
| :--- | :--- |


### `persistRDD`Method {#__a_id_persistrdd_a_code_persistrdd_code_method}

```
persistRDD(rdd: RDD[_])
```

persistRDD 是一个私有的 \[spark\] 方法在 persistentRdds 注册表中注册 rdd。

### Programmable Dynamic Allocation {#__a_id_dynamic_allocation_a_programmable_dynamic_allocation}

SparkContext 提供以下方法作为用于动态分配执行程序的开发人员 API：

* requestExecutors

* killExecutors

* requestTotalExecutors

* \(private!\)getExecutorIds

#### Requesting New Executors — `requestExecutors`Method {#__a_id_requestexecutors_a_requesting_new_executors_code_requestexecutors_code_method}

```
requestExecutors(numAdditionalExecutors: Int): Boolean
```

requestExecutors 从 CoarseGrainedSchedulerBackend 请求 numAdditionalExecutors 执行器。

#### Requesting to Kill Executors — `killExecutors`Method {#__a_id_killexecutors_a_requesting_to_kill_executors_code_killexecutors_code_method}

```
killExecutors(executorIds: Seq[String]): Boolean
```

| Caution | FIXME |
| :--- | :--- |


#### Requesting Total Executors — `requestTotalExecutors`Method {#__a_id_requesttotalexecutors_a_requesting_total_executors_code_requesttotalexecutors_code_method}

```
requestTotalExecutors(
  numExecutors: Int,
  localityAwareTasks: Int,
  hostToLocalTaskCount: Map[String, Int]): Boolean
```

requestTotalExecutors 是一个私有的 \[spark\] 方法，它从粗粒度调度程序后端（coarse-grained scheduler backend）请求确切数量的执行程序。

| Note | 它只适用于粗粒度调度程序后端。 |
| :---: | :--- |


当调用其他调度程序后端时，您应该在日志中看到以下 WARN 消息：

```
WARN Requesting executors is only supported in coarse-grained mode
```

| Caution | 为什么 SparkContext 实现粗粒度调度程序后端的方法？为什么 SparkContext 不会在调用方法时抛出异常？没有人似乎在使用它（！） |
| :---: | :--- |


### Creating`SparkContext`Instance {#__a_id_creating_instance_a_creating_code_sparkcontext_code_instance}

您可以创建一个 SparkContext 实例，无论是否首先创建 SparkConf 对象。

| Note | 你可能想阅读里面创建 SparkContext 来了解当 SparkContext被创建时在幕后发生了什么。 |
| :---: | :--- |


#### Getting Existing or Creating New SparkContext — `getOrCreate`Methods {#__a_id_getorcreate_a_getting_existing_or_creating_new_sparkcontext_code_getorcreate_code_methods}

```
getOrCreate(): SparkContext
getOrCreate(conf: SparkConf): SparkContext
```

getOrCreate 方法允许您获取现有的 SparkContext 或创建一个新的。

```
import org.apache.spark.SparkContext
val sc = SparkContext.getOrCreate()

// Using an explicit SparkConf object
import org.apache.spark.SparkConf
val conf = new SparkConf()
  .setMaster("local[*]")
  .setAppName("SparkMe App")
val sc = SparkContext.getOrCreate(conf)
```

no-param getOrCreate 方法要求使用 spark-submit 指定两个强制的 Spark 设置 - master 和应用程序名称。

#### Constructors {#__a_id_constructors_a_constructors}

```
SparkContext()
SparkContext(conf: SparkConf)
SparkContext(master: String, appName: String, conf: SparkConf)
SparkContext(
  master: String,
  appName: String,
  sparkHome: String = null,
  jars: Seq[String] = Nil,
  environment: Map[String, String] = Map())
```

您可以使用四个构造函数创建一个 SparkContext 实例。

```
import org.apache.spark.SparkConf
val conf = new SparkConf()
  .setMaster("local[*]")
  .setAppName("SparkMe App")

import org.apache.spark.SparkContext
val sc = new SparkContext(conf)
```

当 Spark 上下文启动时，您应该在日志中看到以下 INFO（在来自 Spark 服务的其他消息中）：

```
INFO SparkContext: Running Spark version 2.0.0-SNAPSHOT
```

| Note | 只有一个 SparkContext 可能在单个 JVM 中运行（查看 SPARK-2243 在同一个 JVM 中支持多个 SparkContexts）。在 JVM中共享对 SparkContext 的访问是在 Spark 内共享数据的解决方案（不依赖于使用外部数据存储的其他数据共享方式）。 |
| :---: | :--- |


### Getting Current`SparkConf` — `getConf`Method {#__a_id_getconf_a_getting_current_code_sparkconf_code_code_getconf_code_method}

```
master: String
```

getConf 返回当前的 SparkConf。

| Note | 更改 SparkConf 对象不会更改当前配置（因为方法返回副本）。 |
| :---: | :--- |


### Getting Application Name — `appName`Method {#__a_id_appname_a_getting_application_name_code_appname_code_method}

```
appName: String
```

appName 必需返回 spark.app.name 设置的值。

| Note | 当 SparkDeploySchedulerBackend 启动时使用 appName，SparkUI 创建一个 web UI，执行 postApplicationStart 时，以及 Spark Streaming 中的 Mesos 和检查点。 |
| :---: | :--- |


### Getting Deploy Mode — `deployMode`Method {#__a_id_deploymode_a_getting_deploy_mode_code_deploymode_code_method}

```
deployMode: String
```

deployMode 返回 spark.submit.deployMode 设置或客户端的当前值（如果未设置）

### Getting Scheduling Mode — `getSchedulingMode`Method {#__a_id_getschedulingmode_a_getting_scheduling_mode_code_getschedulingmode_code_method}

```
getSchedulingMode: SchedulingMode.SchedulingMode
```

getSchedulingMode 返回当前的调度模式。

### Getting Schedulable \(Pool\) by Name — `getPoolForName`Method {#__a_id_getpoolforname_a_getting_schedulable_pool_by_name_code_getpoolforname_code_method}

```
getPoolForName(pool: String): Option[Schedulable]
```

getPoolForName 通过池名称返回一个 Schedulable（如果存在）。

| Note | getPoolForName 是开发人员 API 的一部分，可能会在将来更改。 |
| :---: | :--- |


在内部，它请求 TaskScheduler 的根池，并通过池名称查找 Schedulable。

它专门用于在 Web UI 中显示池详细信息（对于阶段）。

### Getting All Pools — `getAllPools`Method {#__a_id_getallpools_a_getting_all_pools_code_getallpools_code_method}

```
getAllPools: Seq[Schedulable]
```

getAllPools 在 TaskScheduler.rootPool 中收集池。

| Note | `TaskScheduler.rootPool`is part of the TaskScheduler Contract. |
| :--- | :--- |


| Note | `getAllPools`is part of the Developer’s API. |
| :--- | :--- |


| Caution | FIXME Where is the method used? |
| :--- | :--- |


| Note | getAllPools用于在使用FAIR调度模式的Web UI中为“阶段”选项卡计算池名称。 |
| :--- | :--- |


### Computing Default Level of Parallelism {#__a_id_defaultparallelism_a_computing_default_level_of_parallelism}

默认的并行性级别是在 RDD 中创建时未由用户明确指定的 RDD 中的分区数。

它用于像 SparkContext.parallelize，SparkContext.range 和 SparkContext.makeRDD（以及 Spark Streaming 的 DStream.countByValue 和 DStream.countByValueAndWindow 和几个其他地方）等方法。它也用于实例化 HashPartitioner 或 HadoopRDD 中最小分区数。

在内部，defaultParallelism 将默认级别的请求传递到 TaskScheduler（它是它的合同的一部分）。

### Getting Spark Version — `version`Property {#__a_id_version_a_getting_spark_version_code_version_code_property}

```
version: String
```

version 返回 SparkContext 使用的 Spark 版本。

### `makeRDD`Method {#__a_id_makerdd_a_code_makerdd_code_method}

### Submitting Jobs Asynchronously — `submitJob`Method {#__a_id_submitjob_a_submitting_jobs_asynchronously_code_submitjob_code_method}

```
submitJob[T, U, R](
  rdd: RDD[T],
  processPartition: Iterator[T] => U,
  partitions: Seq[Int],
  resultHandler: (Int, U) => Unit,
  resultFunc: => R): SimpleFutureAction[R]
```

submitJob 以异步，非阻塞方式向 DAGScheduler 提交作业。

它清除 processPartition 输入函数参数，并返回一个保存 JobWaiter 实例的 SimpleFutureAction 的实例

| Caution | 什么是 resultFunc？ |
| :---: | :--- |


It is used in:

* AsyncRDDActions methods

* Spark Streaming for ReceiverTrackerEndpoint.startReceiver

### Spark Configuration {#__a_id_spark_configuration_a_spark_configuration}

### SparkContext and RDDs {#__a_id_sparkcontext_and_rdd_a_sparkcontext_and_rdds}

您使用 Spark 上下文创建 RDD（请参阅创建 RDD）。

当 RDD 被创建时，它属于并且完全由它源自的 Spark 上下文所拥有。 RDD 不能通过设计在 SparkContexts 之间共享。

![](/img/mastering-apache-spark/spark core-rdd/figure3.png)

### Creating RDD — `parallelize`Method {#__a_id_creating_rdds_a_a_id_parallelize_a_creating_rdd_code_parallelize_code_method}

`SparkContext`allows you to create many different RDDs from input sources like:

* Scala’s collections, i.e.`sc.parallelize(0 to 100)`+

* local or remote filesystems, i.e.`sc.textFile("README.md")`

* Any Hadoop`InputSource`using`sc.newAPIHadoopFile`

Read[Creating RDDs](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/spark-rdd.html#creating-rdds)in[RDD - Resilient Distributed Dataset](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/spark-rdd.html).

### Unpersisting RDDs \(Marking RDDs as non-persistent\) — `unpersist`Method {#__a_id_unpersist_a_unpersisting_rdds_marking_rdds_as_non_persistent_code_unpersist_code_method}

它从 master’s Block Manager 中删除一个 RDD（调用 removeRdd（rddId：Int，blocking：Boolean））和内部 persistentRdds 映射。

它最终发布 SparkListenerUnpersistRDD 消息到 listenerBus。

### Setting Checkpoint Directory — `setCheckpointDir`Method {#__a_id_setcheckpointdir_a_setting_checkpoint_directory_code_setcheckpointdir_code_method}

```
setCheckpointDir(directory: String)
```

setCheckpointDir 方法用于设置检查点目录...

### Registering Custom Accumulators — `register`Methods {#__a_id_register_a_registering_custom_accumulators_code_register_code_methods}

```
register(acc: AccumulatorV2[_, _]): Unit
register(acc: AccumulatorV2[_, _], name: String): Unit
```

您可以使用专门的方法为长整型，双精度型和集合类型创建内置累加器。

在内部，寄存器将 SparkContext 注册到累加器。

### Creating Built-In Accumulators {#__a_id_creating_accumulators_a_a_id_longaccumulator_a_a_id_doubleaccumulator_a_a_id_collectionaccumulator_a_creating_built_in_accumulators}

```
longAccumulator: LongAccumulator
longAccumulator(name: String): LongAccumulator
doubleAccumulator: DoubleAccumulator
doubleAccumulator(name: String): DoubleAccumulator
collectionAccumulator[T]: CollectionAccumulator[T]
collectionAccumulator[T](name: String): CollectionAccumulator[T]
```

您可以使用 longAccumulator，doubleAccumulator 或 collectionAccumulator 创建和注册简单和集合值的累加器。

`longAccumulator`returns LongAccumulator with the zero value`0`.

`doubleAccumulator`returns DoubleAccumulator with the zero value`0.0`.

`collectionAccumulator`returns CollectionAccumulator with the zero value`java.util.List[T]`.

```
scala> val acc = sc.longAccumulator
acc: org.apache.spark.util.LongAccumulator = LongAccumulator(id: 0, name: None, value: 0)

scala> val counter = sc.longAccumulator("counter")
counter: org.apache.spark.util.LongAccumulator = LongAccumulator(id: 1, name: Some(counter), value: 0)

scala> counter.value
res0: Long = 0

scala> sc.parallelize(0 to 9).foreach(n => counter.add(n))

scala> counter.value
res3: Long = 45
```

name 输入参数允许您为累加器命名，并将其显示在 Spark UI 中（在给定阶段的“阶段”选项卡下）。

![](/img/mastering-apache-spark/spark core-rdd/figure4.png)

| Tip | You can register custom accumulators using register methods. |
| :--- | :--- |


### Creating Broadcast Variable — `broadcast`Method {#__a_id_broadcast_a_creating_broadcast_variable_code_broadcast_code_method}

```
broadcast[T](value: T): Broadcast[T]
```

`broadcast`方法创建广播变量。它是一个共享内存，在驱动程序上和以后在所有 Spark 执行器上具有值（作为广播块）。

```
val sc: SparkContext = ???
scala> val hello = sc.broadcast("hello")
hello: org.apache.spark.broadcast.Broadcast[String] = Broadcast(0)
```

Spark 将该值传输给 Spark 执行器一次，任务可以共享它，而不会在广播变量使用多次时导致重复的网络传输。

![](/img/mastering-apache-spark/spark core-rdd/figure5.png)

在内部，广播请求当前 BroadcastManager 创建一个新的广播变量。

| Note | 当前 BroadcastManager 是可用的 SparkEnv.broadcastManager 属性，并始终是 BroadcastManager（几乎没有内部配置更改，以反映它运行的地方，即在驱动程序或执行程序内）。 |
| :---: | :--- |


You should see the following INFO message in the logs:

```
INFO SparkContext: Created broadcast [id] from [callSite]
```

如果定义了 ContextCleaner，则会注册新的广播变量以进行清理。

Spark 不支持广播 RDD。如下会报错的：

```
scala> sc.broadcast(sc.range(0, 10))
java.lang.IllegalArgumentException: requirement failed: Can not directly broadcast RDDs; instead, call collect() and broadcast the result.
  at scala.Predef$.require(Predef.scala:224)
  at org.apache.spark.SparkContext.broadcast(SparkContext.scala:1392)
  ... 48 elided
```

一旦创建，广播变量（和其他块）显示每个执行者和驱动程序在 Web UI（在执行程序选项卡下）。

![](/img/mastering-apache-spark/spark core-rdd/figure6.png)

### Distribute JARs to workers {#__a_id_jars_a_distribute_jars_to_workers}

您使用 SparkContext.addJar 指定的 jar 将被复制到所有工作节点。

配置设置 spark.jars 是一个逗号分隔的 jar 路径列表，要包含在从此 SparkContext 执行的所有任务中。路径可以是本地文件，HDFS（或其他 Hadoop 支持的文件系统）中的文件，HTTP，HTTPS 或 FTP URI 或每个工作节点上文件的 local：/ path。

```
scala> sc.addJar("build.sbt")
15/11/11 21:54:54 INFO SparkContext: Added JAR build.sbt at http://192.168.1.4:49427/jars/build.sbt with timestamp 1447275294457
```

| Caution | 为什么是 HttpFileServer 用于 addJar？ |
| :---: | :--- |


### `SparkContext`as Application-Wide Counter {#__code_sparkcontext_code_as_application_wide_counter}

SparkContext keeps track\(跟踪\) of:

* shuffle id 使用 nextShuffleId 内部计数器注册 shuffle 服务的 shuffle 依赖。

### Running Job Synchronously — `runJob`Methods {#__a_id_runjob_a_running_job_synchronously_code_runjob_code_methods}

RDD 操作使用 runJob 方法之一运行作业。

```
runJob[T, U](
  rdd: RDD[T],
  func: (TaskContext, Iterator[T]) => U,
  partitions: Seq[Int],
  resultHandler: (Int, U) => Unit): Unit
runJob[T, U](
  rdd: RDD[T],
  func: (TaskContext, Iterator[T]) => U,
  partitions: Seq[Int]): Array[U]
runJob[T, U](
  rdd: RDD[T],
  func: Iterator[T] => U,
  partitions: Seq[Int]): Array[U]
runJob[T, U](rdd: RDD[T], func: (TaskContext, Iterator[T]) => U): Array[U]
runJob[T, U](rdd: RDD[T], func: Iterator[T] => U): Array[U]
runJob[T, U](
  rdd: RDD[T],
  processPartition: (TaskContext, Iterator[T]) => U,
  resultHandler: (Int, U) => Unit)
runJob[T, U: ClassTag](
  rdd: RDD[T],
  processPartition: Iterator[T] => U,
  resultHandler: (Int, U) => Unit)
```

runJob 在 RDD 的一个或多个分区（在 SparkContext 空间中）上执行一个函数，以产生每个分区的值集合。

| Tip | runJob 只能在 SparkContext 没有停止的时候工作。 |
| :---: | :--- |


在内部，runJob 首先确保 SparkContext 不停止。如果是，您应该在日志中看到以下 IllegalStateException 异常：

```
java.lang.IllegalStateException: SparkContext has been shutdown
  at org.apache.spark.SparkContext.runJob(SparkContext.scala:1893)
  at org.apache.spark.SparkContext.runJob(SparkContext.scala:1914)
  at org.apache.spark.SparkContext.runJob(SparkContext.scala:1934)
  ... 48 elided
```

runJob 然后计算调用位置并清除 func 闭包。

You should see the following INFO message in the logs:

```
INFO SparkContext: Starting job: [callSite]
```

启用 spark.logLineage（默认情况下不是这样），您应该看到以下 INFO 消息与 toDebugString（在 rdd 上执行）：

```
INFO SparkContext: RDD's recursive dependencies:
[toDebugString]
```

runJob 请求 DAGScheduler 运行作业。

| Tip | runJob 只是为 DAGScheduler 准备输入参数以运行作业。 |
| :---: | :--- |


DAGScheduler 完成并且作业完成后，runJob 停止 ConsoleProgressBar 并执行 rdd 的 RDD 检查点。

| Tip | 对于一些动作，例如 first（）和 lookup（），不需要计算作业中 RDD 的所有分区。和 Spark 知道。 |
| :---: | :--- |


```
// RDD to work with
val lines = sc.parallelize(Seq("hello world", "nice to see you"))

import org.apache.spark.TaskContext
scala> sc.runJob(lines, (t: TaskContext, i: Iterator[String]) => 1) (1)
res0: Array[Int] = Array(1, 1)  (2)
```

1. 使用 runJob 在行 RDD 上运行作业，该函数对每个分区（行 RDD）返回1。

2. 你能说说 RDD 的分区数量吗？你的结果 res0 不同于我的吗？为什么？

| Tip | Read TaskContext. |
| :--- | :--- |


运行作业本质上是对 rdd RDD 中的所有或一部分分区执行 func 函数，并将结果作为数组（其中元素是每个分区的结果）返回。

![](/img/mastering-apache-spark/spark core-rdd/figure7.png)

### `postApplicationEnd`Method {#__a_id_postapplicationend_a_code_postapplicationend_code_method}

| Caution | FIXME |
| :--- | :--- |


### `clearActiveContext`Method {#__a_id_clearactivecontext_a_code_clearactivecontext_code_method}

| Caution | FIXME |
| :--- | :--- |


### Stopping`SparkContext` — `stop`Method {#__a_id_stop_a_a_id_stopping_a_stopping_code_sparkcontext_code_code_stop_code_method}

```
stop(): Unit
```

stop stops the SparkContext.

在内部，停止使能停止的内部标志。如果已停止，您应该在日志中看到以下 INFO 消息：

```
INFO SparkContext: SparkContext already stopped.
```

`stop`then does the following:

1. Removes`_shutdownHookRef`from`ShutdownHookManager`.

2. Posts a`SparkListenerApplicationEnd`\(to`LiveListenerBus`Event Bus\).

3. Stops web UI

4. Requests`MetricSystem`to report metrics\(from all registered sinks\).

5. Stops`ContextCleaner`.

6. Requests`ExecutorAllocationManager`to stop.

7. If`LiveListenerBus`was started,requests`LiveListenerBus`to stop.

8. Requests`EventLoggingListener`to stop.

9. Requests`DAGScheduler`to stop.

10. Requests RpcEnv to stop`HeartbeatReceiver`endpoint.

11. Requests`ConsoleProgressBar`to stop.

12. Clears the reference to`TaskScheduler`, i.e.`_taskScheduler`is`null`.

13. Requests`SparkEnv`to stop and clears`SparkEnv`.

14. Clears`SPARK_YARN_MODE`flag.

15. Clears an active`SparkContext`.

Ultimately, you should see the following INFO message in the logs:

```
INFO SparkContext: Successfully stopped SparkContext
```

### Registering SparkListener — `addSparkListener`Method {#__a_id_addsparklistener_a_registering_sparklistener_code_addsparklistener_code_method}

```
addSparkListener(listener: SparkListenerInterface): Unit
```

您可以使用 addSparkListener 方法注册自定义 SparkListenerInterface

您还可以使用 spark.extraListeners 设置注册自定义侦听器。

### Custom SchedulerBackend, TaskScheduler and DAGScheduler {#__a_id_custom_schedulers_a_custom_schedulerbackend_taskscheduler_and_dagscheduler}

默认情况下，SparkContext 使用（private \[spark\] 类）org.apache.spark.scheduler.DAGScheduler，但你可以开发自己的定制DAGScheduler 实现，并使用（private \[spark\]）SparkContext.dagScheduler \_ =（ds：DAGScheduler）方法去分配你自己的。

它也适用于 SchedulerBackend 和 TaskScheduler，分别使用 schedulerBackend \_ =（sb：SchedulerBackend）和 taskScheduler \_ =（ts：TaskScheduler）方法。

| Caution | Make it an advanced exercise. |
| :--- | :--- |


### Events {#__a_id_events_a_events}

当 Spark 上下文启动时，它触发 SparkListenerEnvironmentUpdate 和 SparkListenerApplicationStart 消息。

请参阅 SparkContext 的初始化部分。

### Setting Default Logging Level — `setLogLevel`Method {#__a_id_setloglevel_a_a_id_setting_default_log_level_a_setting_default_logging_level_code_setloglevel_code_method}

```
setLogLevel(logLevel: String)
```

setLogLevel 允许您在 Spark 应用程序中设置根日志记录级别，例如。spark shell。

在内部，setLogLevel 调用 org.apache.log4j.Level.toLevel（logLevel），然后使用 org.apache.log4j.LogManager.getRootLogger（）。setLevel（level）来设置。

您可以使用 org.apache.log4j.LogManager.getLogger（）直接设置日志记录级别。

```
LogManager.getLogger("org").setLevel(Level.OFF)
```

### Closure Cleaning — `clean`Method {#__a_id_clean_a_a_id_closure_cleaning_a_closure_cleaning_code_clean_code_method}

```
clean(f: F, checkSerializable: Boolean = true): F
```

每次调用一个 action 时，Spark 在序列化之前先清理闭包，即 action 的主体，然后通过线路发送给执行者。

SparkContext 自带了 clean（f：F，checkSerializable：Boolean = true）方法。它又调用 ClosureCleaner.clean 方法。

ClosureCleaner.clean 方法不仅清洁闭包，而且它也是传递式的，即被引用的闭包被过滤清除。

一个闭包被认为是可序列化的，只要它不显式引用不可序列化的对象。它通过遍历封闭闭包的层次结构来实现这一点，并且清除起始闭包实际上未使用的任何引用。

为 org.apache.spark.util.ClosureCleaner 记录器启用 DEBUG 日志记录级别，以查看类中发生了什么。

将以下行添加到 conf / log4j.properties：

```
log4j.logger.org.apache.spark.util.ClosureCleaner=DEBUG
```

使用 DEBUG 日志记录级别，您应该在日志中看到以下消息：

```
+++ Cleaning closure [func] ([func.getClass.getName]) +++
 + declared fields: [declaredFields.size]
     [field]
 ...
+++ closure [func] ([func.getClass.getName]) is now cleaned +++
```

使用 Serializer 的新实例（作为闭包序列化程序）验证序列化。请参阅序列化。

### Hadoop Configuration {#__a_id_hadoopconfiguration_a_hadoop_configuration}

在创建 SparkContext 时，Hadoop 配置也是如此（作为 org.apache.hadoop.conf.Configuration 的实例，可用作 \_hadoopConfiguration）。

| Note | 使用 SparkHadoopUtil.get.newConfiguration。 |
| :---: | :--- |


如果提供了 SparkConf，它将用于构建如上所述的配置。否则，将返回默认的配置对象。

如果 AWS\_ACCESS\_KEY\_ID 和 AWS\_SECRET\_ACCESS\_KEY 都可用，则会为 Hadoop 配置设置以下设置：

* fs.s3.awsAccessKeyId，fs.s3n.awsAccessKeyId，fs.s3a.access.key 设置为 AWS\_ACCESS\_KEY\_ID 的值

* fs.s3.awsSecretAccessKey，fs.s3n.awsSecretAccessKey 和 fs.s3a.secret.key 设置为 AWS\_SECRET\_ACCESS\_KEY 的值

每一个 spark.hadoop。设置将成为带有前缀 spark.hadoop 的配置的设置。删除键。

spark.buffer.size 的值（默认值：65536）用作 io.file.buffer.size 的值。

### `listenerBus` — `LiveListenerBus`Event Bus {#__a_id_listenerbus_a_code_listenerbus_code_code_livelistenerbus_code_event_bus}

listenerBus 是一个 LiveListenerBus 对象，用作向驱动程序上的其他服务发布事件的机制。

| Note | 它是在 SparkContext 启动时创建和启动的，因为它是一个单 JVM 事件总线，专用于驱动程序。 |
| :---: | :--- |


listenerBus 是 SparkContext 中的私有 \[spark\] 值。

### Time when`SparkContext`was Created — `startTime`Property {#__a_id_starttime_a_time_when_code_sparkcontext_code_was_created_code_starttime_code_property}

```
startTime: Long
```

startTime 是 SparkContext 创建时的时间（以毫秒为单位）。

```
scala> sc.startTime
res0: Long = 1464425605653
```

### Spark User — `sparkUser`Property {#__a_id_sparkuser_a_spark_user_code_sparkuser_code_property}

```
sparkUser: String
```

sparkUser 是启动 SparkContext 实例的用户。

它是在使用 Utils.getCurrentUserName 创建 SparkContext 时计算的。

### Submitting`ShuffleDependency`for Execution — `submitMapStage`Internal Method {#__a_id_submitmapstage_a_submitting_code_shuffledependency_code_for_execution_code_submitmapstage_code_internal_method}

```
submitMapStage[K, V, C](
  dependency: ShuffleDependency[K, V, C]): SimpleFutureAction[MapOutputStatistics]
```

submitMapStage 将输入 ShuffleDependency 提交给 DAGScheduler 执行，并返回 SimpleFutureAction。

在内部，submitMapStage 首先计算调用站点，并使用 localProperties 提交它。

| Note | 有趣的是，submitMapStage 仅在 Spark SQL 的 ShuffleExchange 物理运算符被执行时使用。 |
| :---: | :--- |


| Note | submitMapStage 似乎与 Adaptive Query Planning / Adaptive Scheduling 相关。 |
| :---: | :--- |


### Calculating Call Site — `getCallSite`Method {#__a_id_getcallsite_a_calculating_call_site_code_getcallsite_code_method}

### `cancelJobGroup`Method {#__a_id_canceljobgroup_a_code_canceljobgroup_code_method}

```
cancelJobGroup(groupId: String)
```

cancelJobGroup 请求 DAGSchedule r取消一组活动的 Spark 作业。

### `cancelAllJobs`Method {#__a_id_cancelalljobs_a_code_cancelalljobs_code_method}

### `setJobGroup`Method {#__a_id_setjobgroup_a_code_setjobgroup_code_method}

```
setJobGroup(
  groupId: String,
  description: String,
  interruptOnCancel: Boolean = false): Unit
```

### `cleaner`Method {#__a_id_cleaner_a_code_cleaner_code_method}

```
cleaner: Option[ContextCleaner]
```

clean 是一个私有的 \[spark\] 方法来获取可选的应用程序范围的 ContextCleaner。

| Note | 当 SparkContext 使用 spark.cleaner.referenceTracking 创建 SparkCleaner 时，Spark 属性启用（默认情况下为）。 |
| :---: | :--- |


### Settings {#__a_id_settings_a_settings}

#### spark.driver.allowMultipleContexts {#__a_id_spark_driver_allowmultiplecontexts_a_spark_driver_allowmultiplecontexts}

引用 org.apache.spark.SparkContext 的 scaladoc：

每个JVM只能有一个 SparkContext。您必须在创建新的 SparkContext 之前停止（）。

但是，您可以使用 spark.driver.allowMultipleContexts 标志控制行为。

它被禁用，即默认情况下为 false。

如果启用（即 true），Spark 会向日志中输出以下 WARN 消息：

```
WARN Multiple running SparkContexts detected in the same JVM!
```

如果禁用（默认），它将抛出 SparkException 异常：

```
Only one SparkContext may be running in this JVM (see SPARK-2243). To ignore this error, set spark.driver.allowMultipleContexts = true. The currently running SparkContext was created at:
[ctx.creationSite.longForm]
```

当创建 SparkContext 的实例时，Spark 将当前线程标记为正在创建它（在实例化过程的早期）。

| Caution | 不能保证 Spark 能够与两个或多个 SparkContext 一起正常工作。考虑该功能正在进行中的工作。 |
| :---: | :--- |


### Environment Variables {#__a_id_environment_variables_a_environment_variables}

Table 1. Environment Variables

| Environment Variable | Default Value | Description |
| :--- | :--- | :--- |
| `SPARK_EXECUTOR_MEMORY` | `1024` | Amount of memory to allocate for a Spark executor in MB.See Executor Memory. |
| `SPARK_USER` |  | The user who is running`SparkContext`. Available later as sparkUser. |









