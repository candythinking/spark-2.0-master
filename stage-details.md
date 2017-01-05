## Stage Details {#__a_id_stagepage_a_stage_details}

StagePage 显示给定其 ID 和尝试 ID 的阶段的任务详细信息。

![](/img/mastering-apache-spark/spark core-tools/figure18.png)

StagePage 在/ stage URL 下呈现一个页面，需要两个请求参数 - id 和尝试，例如。 http：// localhost：4040 / stages / stage /？id = 2＆attempt = 0。

StagePage 是StagesTab 的一部分。

StagePage 使用父级的 JobProgressListener 和 RDDOperationGraphListener 来计算度量。更具体地说，StagePage 使用JobProgressListener 的 stageIdToData 注册表来访问给定阶段 ID 和尝试的阶段。

StagePage 使用 ExecutorsListener 在任务部分中显示执行器的 stdout 和 stderr 日志。

### Tasks Section {#__a_id_tasks_a_a_id_taskpagedtable_a_tasks_section}

![](/img/mastering-apache-spark/spark core-tools/figure19.png)

任务分页表显示 JobProgressListener 为阶段和阶段尝试收集的 Stage UI Data。

注意

该部分使用 ExecutorsListener 访问 Executor ID / Host 的stdout和stderr日志。

### Summary Metrics for Completed Tasks in Stage {#__a_id_summary_task_metrics_a_summary_metrics_for_completed_tasks_in_stage}

摘要指标表显示给定阶段中已完成 SUCCESS 状态和指标的任务的指标。

该表由以下列组成：公制，最小，第25百分位数，中位数，第75百分位数，最大

![](/img/mastering-apache-spark/spark core-tools/figure20.png)

注意

所有分位数使用 Task UI Data.metrics（按升序排序）为双精度。

第一行是 Duration，其包括基于 executorRunTime 的分位数。

第二行是可选的调度程序延迟，包括将任务从调度程序发送到执行程序的时间，以及将任务结果从执行程序发送到调度程序的时间。它在默认情况下未启用，您应该选择“显示其他度量标准”下的“计划程序延迟”复选框，以将其包括在摘要表中。

提示

如果调度程序延迟较大，请考虑减少任务大小或减小任务结果大小。

第三行是可选的任务反序列化时间，其包括基于 executorDeserializeTime 任务度量的分位数。它在默认情况下未启用，您应该选择“显示其他度量标准”下的“任务反序列化时间”复选框，以将其包括在摘要表中。

第四行是 GC 时间，它是在任务运行时执行器为 Java 垃圾收集而暂停的时间（使用 jvmGCTime 任务度量）。

第5行是可选的结果序列化时间，它是在将执行器上的任务结果发送回驱动程序（使用 resultSerializationTime 任务指标）之前将其序列化的时间。它在默认情况下未启用，您应该选择“显示其他度量标准”下的“结果序列化时间”复选框，以将其包括在摘要表中。

第6行是可选的获取结果时间，它是驱动程序花费从工作程序获取任务结果的时间。它在默认情况下未启用，您应选择“显示其他度量标准”下的“获取结果时间”复选框将其包括在摘要表中。

提示

如果“获取结果时间”很大，请考虑减少从每个任务返回的数据量。

如果启用 Tungsten（默认情况下），则第7行是可选的 **Peak Execution Memory**，它是在 shuffles, aggregations and joins（使用peakExecutionMemory任务度量）期间创建的内部数据结构的峰值大小的总和。对于 SQL 作业，这仅跟踪所有不安全的运算符，广播连接和外部排序。它在默认情况下未启用，您应选择“显示其他度量标准”下的“**Peak Execution Memory**”复选框，以将其包括在摘要表中。

如果阶段有输入，第8行是输入大小/记录，它是从 Hadoop 或从 Spark 存储（使用 inputMetrics.bytesRead 和 inputMetrics.recordsRead 任务度量）读取的字节和记录。

如果阶段有输出，第9行是输出大小/记录，它是写入 Hadoop 或 Spark 存储器的字节和记录（使用 outputMetrics.bytesWritten 和 outputMetrics.recordsWritten 任务度量）。

如果阶段有shuffle读取，表中将有三个行。第一行是 Shuffle Read Blocked Time，这是任务被阻塞等待从远程机器读取 shuffle 数据的时间（使用 shuffleReadMetrics.fetchWaitTime 任务度量）。另一行是 Shuffle Read Size / Records，它是读取的总 shuffle 字节和记录（包括使用 shuffleReadMetrics.totalBytesRead 和 shuffleReadMetrics.recordsRead 任务指标从本地读取的数据和从远程执行器读取的数据）。最后一行是 Shuffle Remote Reads，它是从远程执行器读取的总 shuffle 字节（这是 shuffle 读取字节的子集;剩余的 shuffle 数据在本地读取）。它使用 shuffleReadMetrics.remoteBytesRead 任务度量。

如果阶段有 shuffle write，下面的行是 Shuffle Write Size / Records（使用 shuffleWriteMetrics.bytesWritten 和 shuffleWriteMetrics.recordsWritten 任务度量）。

如果阶段溢出字节，以下两行是 Shuffle 溢出（内存）（使用 memoryBytesSpilled 任务度量）和 Shuffle 溢出（磁盘）（使用 diskBytesSpilled 任务度量）。

### Request Parameters {#__a_id_parameters_a_request_parameters}

`id`is…​

`attempt`is…​

| Note | `id`and`attempt`uniquely identify the stage in JobProgressListener.stageIdToData to retrieve`StageUIData`. |
| :--- | :--- |


`task.page`\(default:`1`\) is…​

`task.sort`\(default:`Index`\)

`task.desc`\(default:`false`\)

`task.pageSize`\(default:`100`\)

`task.prevPageSize`\(default:`task.pageSize`\)

### Metrics {#__a_id_metrics_a_metrics}

Scheduler Delay is…​FIXME

Task Deserialization Time is…​FIXME

Result Serialization Time is…​FIXME

Getting Result Time is…​FIXME

Peak Execution Memory is…​FIXME

Shuffle Read Time is…​FIXME+

Executor Computing Time is…​FIXME

Shuffle Write Time is…​FIXME

![](/img/mastering-apache-spark/spark core-tools/figure21.png)

![](/img/mastering-apache-spark/spark core-tools/figure22.png)

![](/img/mastering-apache-spark/spark core-tools/figure23.png)

![](/img/mastering-apache-spark/spark core-tools/figure24.png)

### Aggregated Metrics by Executor {#__a_id_aggregated_metrics_by_executor_a_a_id_executortable_a_aggregated_metrics_by_executor}

`ExecutorTable`table shows the following columns:

* Executor ID

* Address

* Task Time

* Total Tasks

* Failed Tasks

* Killed Tasks

* Succeeded Tasks

* \(optional\) Input Size / Records \(only when the stage has an input\)

* \(optional\) Output Size / Records \(only when the stage has an output\)

* \(optional\) Shuffle Read Size / Records \(only when the stage read bytes for a shuffle\)

* \(optional\) Shuffle Write Size / Records \(only when the stage wrote bytes for a shuffle\)

* \(optional\) Shuffle Spill \(Memory\) \(only when the stage spilled memory bytes\)

* \(optional\) Shuffle Spill \(Disk\) \(only when the stage spilled bytes to disk\)

![](/img/mastering-apache-spark/spark core-tools/figure25.png)

它从 StageUIData（针对阶段和阶段尝试 ID）获取 executorSummary，并为每个执行者创建行。

它还请求 BlockManagers（从 JobProgressListener）将执行者 ID 映射到一对主机和端口以在地址列中显示。

### Accumulators（累加器） {#__a_id_accumulators_a_accumulators}

阶段页面显示具有命名累加器的表（仅在存在时）。它包含累加器的名称和值。

![](/img/mastering-apache-spark/spark core-tools/figure26.png)

注意

具有名称和值的信息存储在 Accumulable Info（在StageUIData中可用）中。

### Settings {#__a_id_settings_a_settings}

| Spark Property | Default Value | Description |
| :--- | :--- | :--- |
| `spark.ui.timeline.tasks.maximum` | `1000` |  |
| `spark.sql.unsafe.enabled` | `true` |  |



