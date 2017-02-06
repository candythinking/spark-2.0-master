## YarnClusterSchedulerBackend - SchedulerBackend for YARN in Cluster Deploy Mode {#__a_id_yarnclusterschedulerbackend_a_yarnclusterschedulerbackend_schedulerbackend_for_yarn_in_cluster_deploy_mode}

YarnClusterSchedulerBackend 是集群部署模式下 YARN 上 Spark 的自定义 YarnSchedulerBackend。

这是一个调度程序后端，支持多个应用程序尝试和驱动程序日志的 URL，以在 Web UI 中的驱动程序的“执行程序”选项卡中显示为链接。

It uses`spark.yarn.app.attemptId`under the covers \(that the YARN resource manager sets?\).

| Note | `YarnClusterSchedulerBackend`is a`private[spark]`Scala class. You can find the sources in org.apache.spark.scheduler.cluster.YarnClusterSchedulerBackend. |
| :--- | :--- |


| Tip | Enable`DEBUG`logging level for`org.apache.spark.scheduler.cluster.YarnClusterSchedulerBackend`logger to see what happens inside.Add the following line to`conf/log4j.properties`:log4j.logger.org.apache.spark.scheduler.cluster.YarnClusterSchedulerBackend=DEBUG                                    Refer to Logging. |
| :--- | :--- |


### Creating YarnClusterSchedulerBackend {#__a_id_creating_instance_a_creating_yarnclusterschedulerbackend}

创建 YarnClusterSchedulerBackend 对象需要一个 TaskSchedulerImpl 和 SparkContext 对象。

### Starting YarnClusterSchedulerBackend \(start method\) {#__a_id_start_a_starting_yarnclusterschedulerbackend_start_method}

`YarnClusterSchedulerBackend`comes with a custom`start`method.

| Note | `start`is part of the SchedulerBackend Contract. |
| :--- | :--- |


在内部，它首先查询 ApplicationMaster 的 attemptId 并记录应用程序并尝试 ids。 然后它调用父的开始，并将父的 totalExpectedExecutors 设置为初始执行器数。

### Calculating Driver Log URLs \(getDriverLogUrls method\) {#__a_id_getdriverlogurls_a_calculating_driver_log_urls_getdriverlogurls_method}

YarnClusterSchedulerBackend 中的 getDriverLogUrls 计算驱动程序日志的 URL - 标准输出（stdout）和标准错误（stderr）。

| Note | `getDriverLogUrls`is part of the SchedulerBackend Contract. |
| :--- | :--- |


在内部，它检索容器 id，并通过环境变量计算基本 URL。 您应该在日志中看到以下 DEBUG：

```
DEBUG Base URL for logs: [baseUrl]
```





