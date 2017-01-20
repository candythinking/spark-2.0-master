## `YarnClientSchedulerBackend` — SchedulerBackend for YARN in Client Deploy Mode {#__a_id_yarnclientschedulerbackend_a_code_yarnclientschedulerbackend_code_schedulerbackend_for_yarn_in_client_deploy_mode}

YarnClientSchedulerBackend 是在客户端部署模式下将 Spark 应用程序提交到 YARN 群集时使用的 YarnSchedulerBackend。

| Note | 客户端部署模式是提交到 YARN 集群的 Spark 应用程序的默认部署模式。 |
| :---: | :--- |


YarnClientSchedulerBackend 在启动时提交一个 Spark 应用程序，并等待 Spark 应用程序，直到它完成（成功或不成功）。

Table 1. YarnClientSchedulerBackend’s Internal Properties

| Name | Initial Value | Description |
| :--- | :--- | :--- |
| `client` | \(undefined\) | 客户端提交和监视 Spark 应用程序（当 YarnClientSchedulerBackend 启动时）。 当 YarnClientSchedulerBackend 启动时创建，并在 YarnClientSchedulerBackend 停止时停止。 |
| `monitorThread` | \(undefined\) | MonitorThread |

Enable`DEBUG`logging level for`org.apache.spark.scheduler.cluster.YarnClientSchedulerBackend`logger to see what happens inside`YarnClientSchedulerBackend`.

Add the following line to`conf/log4j.properties`:

```
log4j.logger.org.apache.spark.scheduler.cluster.YarnClientSchedulerBackend=DEBUG
```

Enable`DEBUG`logging level for`org.apache.hadoop`logger to see what happens inside Hadoop YARN.

Add the following line to`conf/log4j.properties`:

```
log4j.logger.org.apache.hadoop=DEBUG
```

使用时要小心，因为每秒会有大量的消息在日志中。

### Starting YarnClientSchedulerBackend — `start`Method {#__a_id_start_a_starting_yarnclientschedulerbackend_code_start_code_method}

```
start(): Unit
```

| Note | 客户端部署模式是提交到 YARN 集群的 Spark 应用程序的默认部署模式。 |
| :---: | :--- |


YarnClientSchedulerBackend 在启动时提交一个 Spark 应用程序，并等待 Spark 应用程序，直到它完成（成功或不成功）。

| Name | Initial Value | Description |
| :--- | :--- | :--- |
| `client` | \(undefined\) | 客户端提交和监视 Spark 应用程序（当 YarnClientSchedulerBackend 启动时）。 当 YarnClientSchedulerBackend 启动时创建，并在 YarnClientSchedulerBackend 停止时停止。 |
| `monitorThread` | \(undefined\) | MonitorThread |

Enable`DEBUG`logging level for`org.apache.spark.scheduler.cluster.YarnClientSchedulerBackend`logger to see what happens inside`YarnClientSchedulerBackend`.

Add the following line to`conf/log4j.properties`:

```
log4j.logger.org.apache.spark.scheduler.cluster.YarnClientSchedulerBackend=DEBUG
```

Enable`DEBUG`logging level for`org.apache.hadoop`logger to see what happens inside Hadoop YARN.

Add the following line to`conf/log4j.properties`:

```
log4j.logger.org.apache.hadoop=DEBUG
```

### Starting YarnClientSchedulerBackend — `start`Method {#__a_id_start_a_starting_yarnclientschedulerbackend_code_start_code_method}

```
start(): Unit
```

| Note | `start`is a part of SchedulerBackend contract executed when`TaskSchedulerImpl`starts. |
| :--- | :--- |


`start`创建 Client（与 YARN ResourceManager 通信），并向 YARN 集群提交 Spark 应用程序。

应用程序启动后，启动 MonitorThread 状态监视线程。在此期间，它也调用超类型的 start。

![](/img/mastering-apache-spark/spark on yarn/figure10.png)

在内部，start 分别为驱动程序的主机和端口使用 spark.driver.host 和 spark.driver.port 属性。

如果启用了 Web UI，则将 spark.driver.appUIAddress 设置为 webUrl。

You should see the following DEBUG message in the logs:

```
DEBUG YarnClientSchedulerBackend: ClientArguments called with: --arg [hostport]
```

| Note | hostport 是 spark.driver.host 和 spark.driver.port 属性，它们之间用：，例如： 192.168.99.1:64905。 |
| :---: | :--- |


start 创建一个 ClientArguments（使用 --arg 和 hostport 传递两个元素的数组）。

start 计算 totalExpectedExecutors（使用初始执行器数）。

| Caution | Why is this part of subtypes since they both set it to the same value? |
| :---: | :--- |


start 创建一个 Client（使用 ClientArguments 和 SparkConf）。

start 将 Spark 应用程序提交到 YARN（通过客户端），并保存 ApplicationId（具有未定义的 ApplicationAttemptId）。

| Caution | FIXME Would be very nice to know why`start`does so in a NOTE. |
| :--- | :--- |


`start`waits until the Spark application is running.

（仅当定义 spark.yarn.credentials.file 时）start 启动 ConfigurableCredentialManager。

| Caution | FIXME Why? Include a NOTE to make things easier. |
| :--- | :--- |


`start`创建并启动 monitorThread（监视 Spark 应用程序并在停止时停止当前 SparkContext）。

### stop {#__a_id_stop_a_stop}

`stop`is part of the SchedulerBackend Contract.

它停止内部辅助对象，即 monitorThread 和客户端，以及通过 Client.reportLauncherState “停止”其他服务的停止。在此期间，它也调用超类型的停止。

stop 确保已经创建了内部客户端（即它不为 null），但不一定启动。

停止内部 monitorThread 使用 MonitorThread.stopMonitor 方法。

然后使用 Client.reportLauncherState（SparkAppHandle.State.FINISHED）“宣布”停止。

然后，它将调用传递到超类型的停止，并且一旦超类型的停止完成，它调用 YarnSparkHadoopUtil.stopExecutorDelegationTokenRenewer，随后停止内部客户端。

Eventually, when all went fine, you should see the following INFO message in the logs:

```
INFO YarnClientSchedulerBackend: Stopped
```

### Waiting Until Spark Application Runs — `waitForApplication`Internal Method {#__a_id_waitforapplication_a_waiting_until_spark_application_runs_code_waitforapplication_code_internal_method}

```
waitForApplication(): Unit
```

waitForApplication 等待，直到当前应用程序正在运行（使用 Client.monitorApplication）。

If the application has`FINISHED`,`FAILED`, or has been`KILLED`, a`SparkException`is thrown with the following message:

```
Yarn application has already ended! It might have been killed or unable to launch application master.
```

You should see the following INFO message in the logs for`RUNNING`state:

```
INFO YarnClientSchedulerBackend: Application [appId] has started running.
```

| Note | `waitForApplication`is used when`YarnClientSchedulerBackend`is started. |
| :--- | :--- |


### asyncMonitorApplication {#__a_id_asyncmonitorapplication_a_asyncmonitorapplication}

```
asyncMonitorApplication(): MonitorThread
```

asyncMonitorApplication 内部方法创建一个单独的守护进程 MonitorThread 线程调用 “Yarn application state monitor”。

| Note | `asyncMonitorApplication`does not start the daemon thread. |
| :--- | :--- |


### MonitorThread {#__a_id_monitorthread_a_monitorthread}

MonitorThread 内部类是在客户端部署模式下监视提交到 YARN 群集的 Spark 应用程序。

启动时，MonitorThread 请求客户端 &gt; 监视 Spark 应用程序（禁用 logApplicationReport）。

| Note | Client.monitorApplication 是一个阻塞操作，因此它被包装在 MonitorThread 中以在单独的线程上执行。 |
| :---: | :--- |


When the call to`Client.monitorApplication`has finished, it is assumed that the application has exited. You should see the following ERROR message in the logs:

```
ERROR Yarn application has already exited with state [state]!
```

这导致停止当前的 SparkContext（使用 SparkContext.stop）。

