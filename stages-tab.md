## Stages Tab — Stages for All Jobs {#__a_id_stagestab_a_stages_tab_stages_for_all_jobs}

Web UI 中的 Stages 选项卡显示 Spark 应用程序（即 SparkContext）中所有作业的所有阶段的当前状态，其中包含一个阶段的任务和统计信息的两个可选页面（选择阶段时）和池详细信息（当应用程序工作在 FAIR 调度模式）。

选项卡\(tab\)的标题是所有作业的阶段。

您可以访问/ stages URL下的阶段标签，即http：// localhost：4040 / stages。

没有提交任何作业（因此没有阶段显示），该页面只显示标题。

![](/img/mastering-apache-spark/spark core-tools/figure13.png)

“阶段”页面显示了 Spark 应用程序中每个状态的阶段，分别为“活动阶段”，“待处理阶段”，“完成阶段”和“失败阶段”。

![](/img/mastering-apache-spark/spark core-tools/figure14.png)

注意

仅当在给定状态中存在阶段时才显示状态部分。请参阅所有作业的阶段。

在 FAIR 调度模式下，您可以访问显示调度程序池的表。

![](/img/mastering-apache-spark/spark core-tools/figure15.png)

在内部，页面由 org.apache.spark.ui.jobs.StagesTab 类表示。

该页面使用父的 SparkUI 访问所需的服务，即 SparkContext，SparkConf，JobProgressListener，RDDOperationGraphListener，并知道是否启用 kill。

### `killEnabled`flag {#__a_id_killenabled_a_code_killenabled_code_flag}

| Caution | FIXME |
| :--- | :--- |




