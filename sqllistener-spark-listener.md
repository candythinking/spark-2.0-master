## `SQLListener`Spark Listener {#__a_id_sqllistener_a_code_sqllistener_code_spark_listener}

SQLListener 是一个自定义 SparkListener，它收集有关用于 Web UI 的 SQL 查询执行的信息（以在 SQL 选项卡中显示）。它依靠 spark.sql.execution.id 键来区分查询。

在内部，它单独使用 SQLExecutionUIData 数据结构来记录单个 SQL 查询执行的所有必要数据。 SQLExecutionUIData 在内部注册表中被跟踪，即 activeExecutions，failedExecutions 和 completedExecutions  以及查找表，即 \_executionIdToData，\_jobIdToExecutionId 和 \_stageIdToStageMetrics。

SQLListener 通过拦截 SparkListenerSQLExecutionStart 事件（使用 onOtherEvent 回调）来开始记录查询执行。

当 SparkListenerSQLExecutionEnd 事件到达时，SQLListener 停止记录有关 SQL 查询执行的信息。

它定义其他回调（从SparkListener接口）：

* onJobStart

* onJobEnd

* onExecutorMetricsUpdate+

* onStageSubmitted

* onTaskEnd

### Registering Job and Stages under Active Execution \(onJobStart callback\) {#__a_id_onjobstart_a_registering_job_and_stages_under_active_execution_onjobstart_callback}

```
onJobStart(jobStart: SparkListenerJobStart): Unit
```

onJobStart 读取 spark.sql.execution.id 键，作业和阶段的标识符，然后在 activeExecutions 内部注册表中更新执行 id 的 SQLExecutionUIData。

注意

当执行 onJobStart 时，假定 SQLExecutionUIData 已在内部 activeExecutions 注册表中创建并可用。

SQLExecutionUIData 中的作业被标记为已添加（到阶段）的阶段。对于每个阶段，在内部 \_stageIdToStageMetrics 注册表中创建一个 SQLStageMetrics。最后，在内部 \_jobIdToExecutionId 中记录作业 ID 的执行 ID。

### onOtherEvent {#__a_id_onotherevent_a_onotherevent}

In`onOtherEvent`,`SQLListener`listens to the following SparkListenerEvent events:

* SparkListenerSQLExecutionStart

* SparkListenerSQLExecutionEnd

* SparkListenerDriverAccumUpdates

#### Registering Active Execution \(SparkListenerSQLExecutionStart Event\) {#__a_id_sparklistenersqlexecutionstart_a_registering_active_execution_sparklistenersqlexecutionstart_event}

```
case class SparkListenerSQLExecutionStart(

  executionId: Long,
  description: String,
  details: String,
  physicalPlanDescription: String,
  sparkPlanInfo: SparkPlanInfo,
  time: Long)
extends SparkListenerEvent
```

SparkListenerSQLExecutionStart 事件开始记录关于 executionId SQL 查询执行的信息。

当 SparkListenerSQLExecutionStart 事件到达时，将创建一个用于 executionId 查询执行的新 SQLExecutionUIData，并将其存储在 activeExecutions 内部注册表中。它也存储在 \_executionIdToData 查找表中。

#### SparkListenerSQLExecutionEnd {#__a_id_sparklistenersqlexecutionend_a_sparklistenersqlexecutionend}

```
case class SparkListenerSQLExecutionEnd(
  executionId: Long,
  time: Long)
extends SparkListenerEvent
```

SparkListenerSQLExecutionEnd 事件停止记录关于 executionId SQL 查询执行的信息（跟踪为 SQLExecutionUIData）。  SQLListener 将输入时间保存为 completionTime。

如果没有其他正在运行的作业（在 SQLExecutionUIData 中注册），查询执行将从 activeExecutions 内部注册表中删除，并移动到 completedExecutions 或 failedExecutions 注册表。

这是当 SQLListener 检查在任一注册表中的 SQLExecutionUIData 的数量 —— failedExecutions 或 completedExecutions —— 并删除超出 spark.sql.ui.retainedExecutions 的旧条目。

#### SparkListenerDriverAccumUpdates {#__a_id_sparklistenerdriveraccumupdates_a_sparklistenerdriveraccumupdates}

```
case class SparkListenerDriverAccumUpdates(
  executionId: Long,
  accumUpdates: Seq[(Long, Long)])
extends SparkListenerEvent
```

当 SparkListenerDriverAccumUpdates 到来时，查找输入 executionId 的 SQLExecutionUIData（在 \_executionIdToData 中），并使用输入 accumUpdates 更新 SQLExecutionUIData.driverAccumUpdates。

### onJobEnd {#__a_id_onjobend_a_onjobend}

```
onJobEnd(jobEnd: SparkListenerJobEnd): Unit
```

调用时，onJobEnd 将检索作业的 SQLExecutionUIData，并根据作业结果记录成功或失败。

如果它是查询执行的最后一个作业（跟踪为 SQLExecutionUIData），则从 activeExecutions 内部注册表中删除执行，并移动。 如果查询执行已标记为已完成（使用 completionTime）并且没有其他正在运行的作业（在 SQLExecutionUIData 中注册），查询执行将从 activeExecutions 内部注册表中删除，并移动到 completedExecutions 或 failedExecutions 注册表。

这是当 SQLListener 检查在任一注册表中的 SQLExecutionUIData 的数量 —— failedExecutions 或 completedExecutions —— 并删除超出 spark.sql.ui.retainedExecutions 的旧条目。

### Getting SQL Execution Data \(getExecution method\) {#__a_id_getexecution_a_getting_sql_execution_data_getexecution_method}

```
getExecution(executionId: Long): Option[SQLExecutionUIData]
```

### Getting Execution Metrics \(getExecutionMetrics method\) {#__a_id_getexecutionmetrics_a_getting_execution_metrics_getexecutionmetrics_method}

```
getExecutionMetrics(executionId: Long): Map[Long, String]
```

getExecutionMetrics 获取 executionId 的度量（也称为累加器更新）（通过它收集用于执行的所有任务）。

它专用于在 Web UI 中呈现 ExecutionPage 页面。

### mergeAccumulatorUpdates method {#__a_id_mergeaccumulatorupdates_a_mergeaccumulatorupdates_method}

mergeAccumulatorUpdates 是一个私人助手方法... TK

它只在 getExecutionMetrics 方法中使用。

### SQLExecutionUIData {#__a_id_sqlexecutionuidata_a_sqlexecutionuidata}

SQLExecutionUIData 是 SQLListener 的数据抽象，用于描述 SQL 查询执行。它是用于单个查询执行的作业，阶段和累加器更新的容器。

### Settings {#__a_id_settings_a_settings}

#### spark.sql.ui.retainedExecutions {#__a_id_spark_sql_ui_retainedexecutions_a_spark_sql_ui_retainedexecutions}

spark.sql.ui.retainedExecutions（默认值：1000）是保持在 failedExecutions 和 completedExecutions 内部注册表中的 SQLExecutionUIData 条目的数量。

当查询执行完成时，执行将从内部 activeExecutions 注册表中删除，并在给定结束执行状态的情况下存储在 failedExecutions 或 completedExecutions 中。它是当 SQLListener 确保 SQLExecutionUIData 所需的数量不超过 spark.sql.ui.retainedExecutions 并删除多余的旧条目。

