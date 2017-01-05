## `JobProgressListener`Spark Listener {#__a_id_jobprogresslistener_a_code_jobprogresslistener_code_spark_listener}

`JobProgressListener`is a SparkListener for web UI.

`JobProgressListener`intercepts the following Spark events.

表1. JobProgressListener 事件

| Handler | Purpose |
| :--- | :--- |
| onJobStart | 创建 JobUIData。它更新 jobGroupToJobIds，pendingStages，jobIdToData，activeJobs，stageIdToActiveJobIds，stageIdToInfo 和 stageIdToData。 |
| onJobEnd | 删除 activeJobs 中的条目。它还删除 pendingStages 和 stageIdToActiveJobIds 中的条目。它更新 completedJobs，numCompletedJobs，failedJobs，numFailedJobs 和 skippedStages。 |
| onStageCompleted | 更新 StageUIData 和 JobUIData。 |
| onTaskStart | 更新任务的 StageUIData 和 JobUIData，并注册一个新的 TaskUIData。 |
| onTaskEnd | 更新任务的StageUIData（和TaskUIData），ExecutorSummary和JobUIData。 |
| onExecutorMetricsUpdate |  |
| `onEnvironmentUpdate` | 使用当前 spark.scheduler.mode（来自 Spark Properties 环境详细信息）设置 schedulingMode 属性。 用于作业选项卡（用于计划模式），以及在 JobsTab 和 StagesTab 中显示池。 |
| `onBlockManagerAdded` | 在内部 executorIdToBlockManagerId 注册表中记录执行程序及其块管理器。 |
| `onBlockManagerRemoved` | 从内部 executorIdToBlockManagerId 注册表中删除执行程序。 |
| `onApplicationStart` | 记录 Spark 应用程序的开始时间（在内部 startTime）。 用于作业选项卡（用于总正常运行时间和事件时间轴）和作业页面（用于事件时间线）。 |
| `onApplicationEnd` | 记录 Spark 应用程序的结束时间（在内部 endTime）。 用于作业选项卡（总正常运行时间）。 |
| `onTaskGettingResult` | 什么也没做。 |

### Registries and Counters {#__a_id_registries_a_registries_and_counters}

JobProgressListener 使用注册表收集有关作业执行的信息。

表2. JobProgressListener 注册表和计数器

| Name | Description |
| :--- | :--- |
| `numCompletedStages` |  |
| `numFailedStages` |  |
| `stageIdToData` | 每个阶段保持 StageUIData，即阶段和阶段尝试 ID |
| `stageIdToInfo` |  |
| `stageIdToActiveJobIds` |  |
| `poolToActiveStages` |  |
| `activeJobs` |  |
| `completedJobs` |  |
| `failedJobs` |  |
| `jobIdToData` |  |
| `jobGroupToJobIds` |  |
| `pendingStages` |  |
| `activeStages` |  |
| `completedStages` |  |
| `skippedStages` |  |
| `failedStages` |  |
| `executorIdToBlockManagerId` | 每个执行者 ID 的 BlockManagerId 的查找表。 用于跟踪块管理器，因此“阶段”页面可以显示执行程序的“聚合度量标准”中的地址。 |

### `onJobStart`Method {#__a_id_onjobstart_a_code_onjobstart_code_method}

```
onJobStart(jobStart: SparkListenerJobStart): Unit
```

onJobStart 创建一个 JobUIData。它更新 jobGroupToJobIds，pendingStages，jobIdToData，activeJobs，stageIdToActiveJobIds，stageIdToInfo 和 stageIdToData。

onJobStart 将可选的 Spark 作业组标识为 spark.jobGroup.id（来自输入 jobStart 中的属性）。

onJobStart 然后使用输入的 jobStart 创建一个 JobUIData，其状态属性设置为 JobExecutionStatus.RUNNING，并将其记录在jobIdToData 和 activeJobs 注册表中。

onJobStart 查找组 id 的作业 ID（在 jobGroupToJobIds 注册表中），并添加作业 ID。 内部 pendingStages 使用阶段 id 的 StageInfo进行更新（对于 SparkListenerJobStart.stageInfos 集合中的每个 StageInfo）。

onJobStart 在 stageIdToActiveJobIds 中记录作业的阶段。

onJobStart 在 stageIdToInfo 和 stageIdToData 中记录 StageInfos。

### `onJobEnd`Method {#__a_id_onjobend_a_code_onjobend_code_method}

```
onJobEnd(jobEnd: SparkListenerJobEnd): Unit
```

onJobEnd 删除 activeJobs 中的条目。它还删除 pendingStages 和 stageIdToActiveJobIds 中的条目。它更新 completedJobs，numCompletedJobs，failedJobs，numFailedJobs 和 skippedStages。

onJobEnd 从 activeJobs 注册表中删除作业。它从 pendingStages 注册表中删除阶段。

当成功完成后，作业将添加到 completedJobs 注册表中，状态属性设置为 JobExecutionStatus.SUCCEEDED。 numCompletedJobs 增加。

当失败时，作业将添加到 failedJobs 注册表中，状态属性设置为 JobExecutionStatus.FAILED。 numFailedJobs 增加。

对于作业中的每个阶段，该阶段将从活动作业（在 stageIdToActiveJobIds 中）中删除，如果没有活动作业存在，则该活动作业可以删除整个条目。

stageIdToInfo 中的每个挂起阶段都将添加到 skippedStages。

### `onExecutorMetricsUpdate`Method {#__a_id_onexecutormetricsupdate_a_code_onexecutormetricsupdate_code_method}

```
onExecutorMetricsUpdate(executorMetricsUpdate: SparkListenerExecutorMetricsUpdate): Unit
```

### `onTaskStart`Method {#__a_id_ontaskstart_a_code_ontaskstart_code_method}

```
onTaskStart(taskStart: SparkListenerTaskStart): Unit
```

onTaskStart 更新 StageUIData 和 JobUIData，并注册一个新的 TaskUIData。

onTaskStart 从输入的 taskStart 读取 TaskInfo。

onTaskStart 查找阶段和阶段尝试 ids 向上（在 stageIdToData 注册表中）的 StageUIData。

onTaskStart 增加 numActiveTasks 并将任务的 TaskUIData 放在 stageData.taskData 中。

最终，onTaskStart 在内部 stageIdToActiveJobIds 中查找阶段，并为每个活动作业读取其 JobUIData（从 jobIdToData）。然后它增加 numActiveTasks。

### `onTaskEnd`Method {#__a_id_ontaskend_a_code_ontaskend_code_method}

```
onTaskEnd(taskEnd: SparkListenerTaskEnd): Unit
```

onTaskEnd 更新 StageUIData（和 TaskUIData），ExecutorSummary 和 JobUIData。

onTaskEnd 从输入 taskEnd 中读取 TaskInfo。

注意

当 TaskInfo 可用且 stageAttemptId 不是 -1 时，onTaskEnd 执行其处理。

onTaskEnd 查看 StageUIData 的阶段和阶段尝试 ids up（在 stageIdToData 注册表中）。

onTaskEnd 在 StageUIData 中保存`accumulables`。

onTaskEnd 读取执行程序的 ExecutorSummary（任务已完成）。

根据任务结束的原因 onTaskEnd 增量 succeededTasks，killedTasks 或 failedTasks 计数器。

onTaskEnd 将任务的持续时间添加到 taskTime。

onTaskEnd 减少活动任务的数量（在 StageUIData 中）。

同样，根据任务结束的原因，onTaskEnd 计算 errorMessage 并更新 StageUIData。

| Caution | 为什么两个不同的注册表中有相同的信息 - stageData和execSummary？ |
| :--- | :--- |


如果 taskMetrics 可用，则执行 updateAggregateMetrics。

| Caution | 为什么是updateAggregateMetrics？ |
| :--- | :--- |


在 stageData.taskData 中查找任务的 TaskUIData，并执行 updateTaskInfo 和 updateTaskMetrics。 errorMessage 已更新。

onTaskEnd 确保 StageUIData（stageData.taskData）中的任务数量不高于 spark.ui.retainedTasks，并删除超额。

最终，onTaskEnd 在内部 stageIdToActiveJobIds 中查找阶段，并为每个活动作业读取其 JobUIData（从 jobIdToData）。然后，它减少 numActiveTasks 并根据任务的结束原因增加 numCompletedTasks，numKilledTasks 或 numFailedTasks。

### `onStageSubmitted`Method {#__a_id_onstagesubmitted_a_code_onstagesubmitted_code_method}

```
onStageSubmitted(stageSubmitted: SparkListenerStageSubmitted): Unit
```

### `onStageCompleted`Method {#__a_id_onstagecompleted_a_code_onstagecompleted_code_method}

```
onStageCompleted(stageCompleted: SparkListenerStageCompleted): Unit
```

onStageCompleted 更新 StageUIData 和 JobUIData。

onStageCompleted 从输入 stageCompleted 读取 stageInfo，并将其记录在 stageIdToInfo 注册表中。

onStageCompleted 在 StageIdToData 注册表中查找阶段的 StageUIData 和阶段尝试 ids。

onStageCompleted在StageUIData中记录 accumulables。

onStageCompleted 从 poolToActiveStages 和 activeStages 注册表中删除阶段。

如果阶段成功完成（即没有 failureReason），onStageCompleted 将阶段添加到 completedStages 注册表并增加 numCompletedStages 计数器。它修剪完成的阶段。

否则，当阶段失败时，onStageCompleted 将阶段添加到 failedStages 注册表并增加 numFailedStages 计数器。它修剪失败的阶段。

最终，onStageCompleted 看在内部 stageIdToActiveJobIds 的舞台，并为每个活动作业读取其 JobUIData（从 jobIdToData）。然后减少 numActiveStages。成功完成后，它将阶段添加到 completedStageIndices。失败，numFailedStages 会增加。

### JobUIData {#__a_id_jobuidata_a_jobuidata}

| Caution | FIXME |
| :--- | :--- |


### blockManagerIds method {#__a_id_blockmanagerids_a_blockmanagerids_method}

```
blockManagerIds: Seq[BlockManagerId]
```

| Caution | FIXME |
| :--- | :--- |


### StageUIData {#__a_id_stageuidata_a_stageuidata}

| Caution | FIXME |
| :--- | :--- |


### Settings {#__a_id_settings_a_settings}

| Setting | Default Value | Description |
| :--- | :--- | :--- |
| `spark.ui.retainedJobs` | `1000` | 要保存信息的作业数 |
| `spark.ui.retainedStages` | `1000` | 保存信息的阶段数 |
| `spark.ui.retainedTasks` | `100000` | 保存有关信息的任务数 |



