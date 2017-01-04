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





