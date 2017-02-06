## Settings {#_settings}

以下设置（也称为系统属性）特定于 YARN 上的 Spark。

| Spark Property | Default Value | Description |
| :--- | :--- | :--- |
| `spark.yarn.am.port` | `0` | ApplicationMaster 用于创建 sparkYarnAM RPC 环境的端口。 |
| `spark.yarn.am.waitTime` | `100s` | 以毫秒为单位，除非指定单位。 |
| `spark.yarn.app.id` |  |  |
| `spark.yarn.executor.memoryOverhead` | 10% of spark.executor.memory but not less than`384` | （以 MiB 为单位）是从 YARN 集群请求 YARN 资源容器时执行器内存开销（除 spark.executor.memory 之外）的可选设置。当客户端计算执行程序的内存开销时使用。 |

### spark.yarn.credentials.renewalTime {#__a_id_spark_yarn_credentials_renewaltime_a_spark_yarn_credentials_renewaltime}

spark.yarn.credentials.renewalTime（默认值：Long.MaxValue ms）是下次凭证更新时间的内部设置。

请参阅 prepareLocalResources。

### spark.yarn.credentials.updateTime {#__a_id_spark_yarn_credentials_updatetime_a_spark_yarn_credentials_updatetime}

spark.yarn.credentials.updateTime（默认值：Long.MaxValue ms）是下一个凭证更新时间的内部设置。

### spark.yarn.rolledLog.includePattern {#__a_id_spark_yarn_rolledlog_includepattern_a_spark_yarn_rolledlog_includepattern}

`spark.yarn.rolledLog.includePattern`

### spark.yarn.rolledLog.excludePattern {#__a_id_spark_yarn_rolledlog_excludepattern_a_spark_yarn_rolledlog_excludepattern}

`spark.yarn.rolledLog.excludePattern`

### spark.yarn.am.nodeLabelExpression {#__a_id_spark_yarn_am_nodelabelexpression_a_spark_yarn_am_nodelabelexpression}

`spark.yarn.am.nodeLabelExpression`

### spark.yarn.am.attemptFailuresValidityInterval {#__a_id_spark_yarn_am_attemptfailuresvalidityinterval_a_spark_yarn_am_attemptfailuresvalidityinterval}

`spark.yarn.am.attemptFailuresValidityInterval`

### spark.yarn.tags {#__a_id_spark_yarn_tags_a_spark_yarn_tags}

`spark.yarn.tags`

### spark.yarn.am.extraLibraryPath {#__a_id_spark_yarn_am_extralibrarypath_a_spark_yarn_am_extralibrarypath}

`spark.yarn.am.extraLibraryPath`

### spark.yarn.am.extraJavaOptions {#__a_id_spark_yarn_am_extrajavaoptions_a_spark_yarn_am_extrajavaoptions}

`spark.yarn.am.extraJavaOptions`

### spark.yarn.scheduler.initial-allocation.interval {#__a_id_spark_yarn_scheduler_initial_allocation_interval_a_spark_yarn_scheduler_initial_allocation_interval}

`spark.yarn.scheduler.initial-allocation.interval`\(default:`200ms`\) controls the initial allocation interval.

It is used when`ApplicationMaster`is instantiated.

### spark.yarn.scheduler.heartbeat.interval-ms {#__a_id_spark_yarn_scheduler_heartbeat_interval_ms_a_spark_yarn_scheduler_heartbeat_interval_ms}

`spark.yarn.scheduler.heartbeat.interval-ms`\(default:`3s`\) is the heartbeat interval to YARN ResourceManager.

It is used when`ApplicationMaster`is instantiated.

### spark.yarn.max.executor.failures {#__a_id_spark_yarn_max_executor_failures_a_spark_yarn_max_executor_failures}

`spark.yarn.max.executor.failures`is an optional setting that sets the maximum number of executor failures before…​TK

It is used when`ApplicationMaster`is instantiated.

| Caution | FIXME |
| :--- | :--- |


### spark.yarn.maxAppAttempts {#__a_id_spark_yarn_maxappattempts_a_spark_yarn_maxappattempts}

`spark.yarn.maxAppAttempts`is the maximum number of attempts to register ApplicationMaster before deploying a Spark application to YARN is deemed failed.

It is used when`YarnRMClient`computes`getMaxRegAttempts`.

### spark.yarn.user.classpath.first {#__a_id_spark_yarn_user_classpath_first_a_spark_yarn_user_classpath_first}

| Caution | FIXME |
| :--- | :--- |


### spark.yarn.archive {#__a_id_spark_yarn_archive_a_spark_yarn_archive}

spark.yarn.archive 是包含具有 Spark 类的 jar 文件的存档的位置。它不能是本地：URI。

它用于为 ApplicationMaster 和执行程序填充 CLASSPATH。

### spark.yarn.queue {#__a_id_spark_yarn_queue_a_spark_yarn_queue}

spark.yarn.queue（默认值：default）是客户端用来向其提交 Spark 应用程序的 YARN 资源队列的名称。

您可以使用 spark-submit 的 --queue 命令行参数指定值。

该值用于设置 YARN 的 ApplicationSubmissionContext.setQueue。

### spark.yarn.jars {#__a_id_spark_yarn_jars_a_spark_yarn_jars}

spark.yarn.jars 是 Spark jars 的位置。

```
--conf spark.yarn.jar=hdfs://master:8020/spark/spark-assembly-2.0.0-hadoop2.7.2.jar
```

它用于填充 ApplicationMaster 和 ExecutorRunnables 的 CLASSPATH（当未定义 spark.yarn.archive 时）。

| Note | spark.yarn.jar 设置从 Spark 2.0 开始弃用。 |
| :---: | :--- |


### spark.yarn.report.interval {#__a_id_spark_yarn_report_interval_a_spark_yarn_report_interval}

spark.yarn.report.interval（默认值：1s）是当前应用程序状态的报告之间的间隔（以毫秒为单位）。

它在 Client.monitorApplication 中使用。

### spark.yarn.dist.jars {#__a_id_spark_yarn_dist_jars_a_spark_yarn_dist_jars}

spark.yarn.dist.jars（默认值：空）是要分发的其他 jar 的集合。

当客户端使用 --jars 命令行选项为 spark-submit 分发额外的资源时使用。

### spark.yarn.dist.files {#__a_id_spark_yarn_dist_files_a_spark_yarn_dist_files}

spark.yarn.dist.files（默认值：空）是要分发的其他文件的集合。

当客户端使用 --files 命令行选项为 spark-submit 分发指定的其他资源时使用。

### spark.yarn.dist.archives {#__a_id_spark_yarn_dist_archives_a_spark_yarn_dist_archives}

spark.yarn.dist.archives（默认值：空）是要分发的其他归档的集合。

当客户端使用 --archive 命令行选项为 spark-submit 分发额外的资源时使用。

### spark.yarn.principal {#__a_id_spark_yarn_principal_a_spark_yarn_principal}

spark.yarn.principal - 请参阅 spark-submit 的相应的 --principal 命令行选项。

### spark.yarn.keytab {#__a_id_spark_yarn_keytab_a_spark_yarn_keytab}

spark.yarn.keytab - 请参阅 spark-submit 的相应 --keytab 命令行选项。

### spark.yarn.submit.file.replication {#__a_id_spark_yarn_submit_file_replication_a_spark_yarn_submit_file_replication}

spark.yarn.submit.file.replication 是 Spark 上传到 HDFS 的文件的复制因子（数字）。

### spark.yarn.config.gatewayPath {#__a_id_spark_yarn_config_gatewaypath_a_spark_yarn_config_gatewaypath}

spark.yarn.config.gatewayPath（default：null）是网关节点上存在的配置路径的根，并将替换为群集计算机中的相应路径。

当客户端将某个路径解析为可识别 YARN NodeManager 时使用。

### spark.yarn.config.replacementPath {#__a_id_spark_yarn_config_replacementpath_a_spark_yarn_config_replacementpath}

spark.yarn.config.replacementPath（default：null）是在 YARN 集群中启动进程时用作 spark.yarn.config.gatewayPath 的替代的路径。

当客户端将某个路径解析为可识别 YARN NodeManager 时使用。

### spark.yarn.historyServer.address {#__a_id_spark_yarn_historyserver_address_a_spark_yarn_historyserver_address}

spark.yarn.historyServer.address 是历史记录服务器的可选地址。

### spark.yarn.access.namenodes {#__a_id_spark_yarn_access_namenodes_a_spark_yarn_access_namenodes}

spark.yarn.access.namenodes（默认值：空）是要请求委托令牌的额外 NameNode URL 的列表。承载 fs.defaultFS 的 NameNode 不需要在此列出。

### spark.yarn.cache.types {#__a_id_spark_yarn_cache_types_a_spark_yarn_cache_types}

`spark.yarn.cache.types`is an internal setting…​

### spark.yarn.cache.visibilities {#__a_id_spark_yarn_cache_visibilities_a_spark_yarn_cache_visibilities}

`spark.yarn.cache.visibilities`is an internal setting…​

### spark.yarn.cache.timestamps {#__a_id_spark_yarn_cache_timestamps_a_spark_yarn_cache_timestamps}

`spark.yarn.cache.timestamps`is an internal setting…​

### spark.yarn.cache.filenames {#__a_id_spark_yarn_cache_filenames_a_spark_yarn_cache_filenames}

`spark.yarn.cache.filenames`is an internal setting…​

### spark.yarn.cache.sizes {#__a_id_spark_yarn_cache_sizes_a_spark_yarn_cache_sizes}

`spark.yarn.cache.sizes`is an internal setting…​

### spark.yarn.cache.confArchive {#__a_id_spark_yarn_cache_confarchive_a_spark_yarn_cache_confarchive}

`spark.yarn.cache.confArchive`is an internal setting…​

### spark.yarn.secondary.jars {#__a_id_spark_yarn_secondary_jars_a_spark_yarn_secondary_jars}

`spark.yarn.secondary.jars`is…​

### spark.yarn.executor.nodeLabelExpression {#__a_id_spark_yarn_executor_nodelabelexpression_a_spark_yarn_executor_nodelabelexpression}

`spark.yarn.executor.nodeLabelExpression`is a node label expression for executors.

### spark.yarn.containerLauncherMaxThreads {#__a_id_spark_yarn_containerlaunchermaxthreads_a_spark_yarn_containerlaunchermaxthreads}

`spark.yarn.containerLauncherMaxThreads`\(default:`25`\)…​FIXME

### spark.yarn.executor.failuresValidityInterval {#__a_id_spark_yarn_executor_failuresvalidityinterval_a_spark_yarn_executor_failuresvalidityinterval}

spark.yarn.executor.failuresValidityInterval（默认值：-1L）是一个间隔（以毫秒为单位），之后 Executor 失败将被认为是独立的，并且不会向尝试计数累积。

### spark.yarn.submit.waitAppCompletion {#__a_id_spark_yarn_submit_waitappcompletion_a_spark_yarn_submit_waitappcompletion}

spark.yarn.submit.waitAppCompletion（default：true）是一个标志，用于控制在以集群模式退出启动程序进程之前是否等待应用程序完成。

### spark.yarn.am.cores {#__a_id_spark_yarn_am_cores_a_spark_yarn_am_cores}

`spark.yarn.am.cores`\(default:`1`\) sets the number of CPU cores for ApplicationMaster’s JVM.

### spark.yarn.driver.memoryOverhead {#__a_id_spark_yarn_driver_memoryoverhead_a_spark_yarn_driver_memoryoverhead}

`spark.yarn.driver.memoryOverhead`\(in MiBs\)

### spark.yarn.am.memoryOverhead {#__a_id_spark_yarn_am_memoryoverhead_a_spark_yarn_am_memoryoverhead}

`spark.yarn.am.memoryOverhead`\(in MiBs\)

### spark.yarn.am.memory {#__a_id_spark_yarn_am_memory_a_spark_yarn_am_memory}

`spark.yarn.am.memory`\(default:`512m`\) sets the memory size of ApplicationMaster’s JVM \(in MiBs\)

### spark.yarn.stagingDir {#__a_id_spark_yarn_stagingdir_a_spark_yarn_stagingdir}

spark.yarn.stagingDir 是在提交应用程序时使用的临时目录。

### spark.yarn.preserve.staging.files {#__a_id_spark_yarn_preserve_staging_files_a_spark_yarn_preserve_staging_files}

spark.yarn.preserve.staging.files（default：false）控制是否保留临时目录中的临时文件（由 spark.yarn.stagingDir 指向）。

### spark.yarn.credentials.file {#__a_id_spark_yarn_credentials_file_a_spark_yarn_credentials_file}

`spark.yarn.credentials.file`…​

### spark.yarn.launchContainers {#__a_id_spark_yarn_launchcontainers_a_spark_yarn_launchcontainers}

spark.yarn.launchContainers（默认值：true）是用于仅测试的标志，因此 YarnAllocator 不会在分配的 YARN 容器上运行启动 ExecutorRunnables。













