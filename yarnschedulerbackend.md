## YarnSchedulerBackend — Coarse-Grained Scheduler Backend for YARN {#__a_id_yarnschedulerbackend_a_yarnschedulerbackend_coarse_grained_scheduler_backend_for_yarn}

YarnSchedulerBackend 是一个用于 YARN 的抽象 CoarseGrainedSchedulerBackend，它包含客户端和集群 YARN 调度程序后端的公共逻辑，即分别为 YarnClientSchedulerBackend 和 YarnClusterSchedulerBackend。

YarnSchedulerBackend 在 RPC 环境中可用作 YarnScheduler RPC 终端节点（或内部的 yarnSchedulerEndpointRef）。

YarnSchedulerBackend 期望 TaskSchedulerImpl 和 SparkContext 初始化自身。

它适用于单个 Spark 应用程序（作为 ApplicationId 类型的 appId）

| Caution | 它可能是一个调度程序后端的注释。 |
| :---: | :--- |


### attemptId Internal Attribute {#__a_id_attemptid_a_attemptid_internal_attribute}

```
attemptId: Option[ApplicationAttemptId] = None
```

attemptId 是此应用程序运行的应用程序尝试 ID。它仅适用于集群部署模式。

当 YarnClientSchedulerBackend 启动（并且 bindToYarn 被调用）时，它显式地设置为 None。

当 YarnClusterSchedulerBackend 启动时（和调用 bindToYarn），它被设置为当前的尝试 ID（使用 YARN API 的 ApplicationMaster.getAttemptId）。

| Note | attemptId 是使用 applicationAttemptId 公开的，这是 SchedulerBackend Contract 的一部分。 |
| :---: | :--- |


### applicationAttemptId {#__a_id_applicationattemptid_a_applicationattemptid}

| Note | applicationAttemptId 是 SchedulerBackend Contract 的一部分。 |
| :---: | :--- |


```
applicationAttemptId(): Option[String]
```

applicationAttemptId 返回 Spark 应用程序的应用程序尝试 ID。

### Resetting YarnSchedulerBackend {#__a_id_reset_a_resetting_yarnschedulerbackend}

| Note | `reset`is a part of CoarseGrainedSchedulerBackend Contract. |
| :--- | :--- |


`reset`重置父 CoarseGrainedSchedulerBackend 调度程序后端和 ExecutorAllocationManager（可由 SparkContext.executorAllocationManager 访问）。

### doRequestTotalExecutors {#__a_id_dorequesttotalexecutors_a_dorequesttotalexecutors}

```
def doRequestTotalExecutors(requestedTotal: Int): Boolean
```

| Note | `doRequestTotalExecutors`is a part of the CoarseGrainedSchedulerBackend Contract. |
| :---: | :--- |


![](/img/mastering-apache-spark/spark on yarn/figure8.png)

doRequestTotalExecutors 只是使用输入的 requestedTotal 和内部 localityAwareTasks 和 hostToLocalTask​​Count 属性发送阻塞的 RequestExecutors 消息到 YarnScheduler RPC Endpoint。

| Caution | The internal attributes are already set. When and how? |
| :---: | :--- |


### Reference to YarnScheduler RPC Endpoint \(yarnSchedulerEndpointRef attribute\) {#__a_id_yarnschedulerendpointref_a_reference_to_yarnscheduler_rpc_endpoint_yarnschedulerendpointref_attribute}

`yarnSchedulerEndpointRef`is the reference to YarnScheduler RPC Endpoint.

### totalExpectedExecutors {#__a_id_totalexpectedexecutors_a_totalexpectedexecutors}

totalExpectedExecutors 是一个值，在创建 YarnSchedulerBackend 实例时最初为0，但随后在 YARN 上的 Spark 启动时（在客户端模式或群集模式下）更改。

| Note | 启动 Spark on YARN 后，totalExpectedExecutors 将初始化为正确的值。 |
| :---: | :--- |


它用于 sufficientResourcesRegistered。

| Caution | Where is this used? |
| :---: | :--- |


### Creating YarnSchedulerBackend Instance {#__a_id_initialization_a_a_id_creating_instance_a_creating_yarnschedulerbackend_instance}

创建时，YarnSchedulerBackend 设置内部的 minRegisteredRatio，当 spark.scheduler.minRegisteredResourcesRatio 未设置或父的minRegisteredRatio 时，它为 0.8。

totalExpectedExecutors 设置为0。

它创建一个 YarnSchedulerEndpoint（作为 yarnSchedulerEndpoint），并将它注册为 RPC 环境中的 YarnScheduler。

它使用 SparkContext 构造函数参数为 RPC ask 操作设置内部 askTimeout Spark 超时。

它设置可选的 appId（类型为 ApplicationId），attemptId（仅适用于集群模式和 ApplicationAttemptId 类型）。

它还创建 SchedulerExtensionServices 对象（作为服务）。

| Caution | What is SchedulerExtensionServices? |
| :---: | :--- |


内部的 shouldResetOnAmRegister flag 是关闭的。

### minRegisteredRatio {#__a_id_minregisteredratio_a_minregisteredratio}

minRegisteredRatio 在创建 YarnSchedulerBackend 时设置。

它用于 sufficientResourcesRegistered。

### Starting the Backend \(start method\) {#__a_id_start_a_starting_the_backend_start_method}

start 创建一个 SchedulerExtensionServiceBinding 对象（使用 SparkContext，appId 和 attemptId）并启动它（使用 SchedulerExtensionServices.start（binding））。

| Note | 当 YarnSchedulerBackend 初始化并可用作服务时，将创建 SchedulerExtensionServices 对象。 |
| :---: | :--- |


最终，它调用父的 CoarseGrainedSchedulerBackend.start。

`start`throws`IllegalArgumentException`when the internal`appId`has not been set yet.

```
java.lang.IllegalArgumentException: requirement failed: application ID unset

```

### Stopping the Backend \(stop method\) {#__a_id_stop_a_stopping_the_backend_stop_method}

`stop`调用父 CoarseGrainedSchedulerBackend.requestTotalExecutors（使用（0，0，Map.empty）参数）。

| Caution | Explain what 0, 0, Map.empty means after the method’s described for the parent. |
| :---: | :--- |


它调用父的 CoarseGrainedSchedulerBackend.stop。

最终，它停止内部 SchedulerExtensionServiceBinding 对象（使用 services.stop（））。

| Caution | Link the description of services.stop\(\) here. |
| :---: | :--- |


### Recording Application and Attempt Ids \(bindToYarn method\) {#__a_id_bindtoyarn_a_recording_application_and_attempt_ids_bindtoyarn_method}

```
bindToYarn(appId: ApplicationId, attemptId: Option[ApplicationAttemptId]): Unit
```

bindToYarn 将内部 appId 和 attemptId 分别设置为输入参数 appId 和 attemptId 的值。

| Note | start requires`appId`. |
| :--- | :--- |


### Internal Registries {#__a_id_internal_registries_a_internal_registries}

#### shouldResetOnAmRegister flag {#__a_id_shouldresetonamregister_a_shouldresetonamregister_flag}

当创建 YarnSchedulerBackend 时，shouldResetOnAmRegister 被禁用（即 false）。

shouldResetOnAmRegister 控制在另一个 RegisterClusterManager RPC 消息到达时是否重置 YarnSchedulerBackend。

它允许在初始 ApplicationManager 失败并注册了新的 ApplicationManager 后重置内部状态。

| Note | It can only happen in client deploy mode. |
| :--- | :--- |


### Settings {#__a_id_settings_a_settings}

#### spark.scheduler.minRegisteredResourcesRatio {#__a_id_spark_scheduler_minregisteredresourcesratio_a_spark_scheduler_minregisteredresourcesratio}

`spark.scheduler.minRegisteredResourcesRatio`\(default:`0.8`\)







