## Web UI — Spark Application’s Web Console {#_web_ui_spark_application_s_web_console}

Web UI（也称为应用程序UI或WebUI或Spark UI）是运行的Spark应用程序的Web界面，用于在Web浏览器中监视和检查Spark作业执行。

![](/img/mastering-apache-spark/spark core-tools/figure1.png)

每个 SparkContext 都会启动自己的 Web UI 实例，默认情况下可以在 http：// \[driver\]：4040 中获得（端口可以使用 spark.ui.port 设置更改），如果这个端口已经被占用，打开端口）。

Web UI 包含以下选项（可能不会一次可见，因为它们根据需要进行延迟创建，例如“流式”选项卡）：

1. Jobs
2. Stages
3. Storage with RDD size and memory use
4. Environment
5. Executors
6. SQL

提示

您可以在应用程序完成后通过使用 EventLoggingListener 持久化事件并使用 Spark History Server 来使用Web UI。

注意

由于 JobProgressListener 和其他 SparkListener，所有在 Web UI 中显示的信息都可用。可以说 Web UI 是 Spark 侦听器的 Web 层。

### Settings {#__a_id_settings_a_settings}

| Spark Property | Default Value | Description |
| :---: | :---: | :---: |
| spark.ui.enabled | true | 用于控制 Web UI 是否启动（true）或不是（false）的标志。 |
| spark.ui.port | 4040 | Web UI 端口的绑定。 如果多个 SparkContexts 尝试在同一个主机上运行（虽然在单个 JVM 上不可能有两个或更多的 Spark 上下文），它们将绑定到以 spark.ui.port 开头的连续端口。 |
| spark.ui.killEnabled | true | 该标志控制是否可以杀死 web UI 中的阶段（true）或不（false） |
| spark.ui.retainedDeadExecutors | 100 | executorToTaskSummary 注册表（在 ExecutorsListener 中）和 deadExecutorStorageStatus 注册表（在 StorageStatusListener 中）的最大条目数。 |



