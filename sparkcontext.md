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







