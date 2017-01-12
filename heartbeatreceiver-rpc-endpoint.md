## HeartbeatReceiver RPC Endpoint {#__a_id_heartbeatreceiver_a_heartbeatreceiver_rpc_endpoint}

HeartbeatReceiver RPC 终结点是一个 ThreadSafeRpcEndpoint 和一个 SparkListener。

它跟踪执行者（通过消息），并通知 TaskScheduler 和 SparkContext 丢失的执行者。

创建时，它需要一个 SparkContext 和一个`Clock`。后来，它使用 SparkContext 将自己注册为 SparkListener 和 TaskScheduler（作为调度程序）。

HeartbeatReceiver 创建 SparkContext 时注册 RPC 终结点。

为 org.apache.spark.HeartbeatReceiver 启用 DEBUG 或 TRACE 日志记录级别，以查看内部发生了什么。

将以下行添加到 conf / log4j.properties：

```
log4j.logger.org.apache.spark.HeartbeatReceiver=TRACE
```

### Creating HeartbeatReceiver Instance {#__a_id_creating_instance_a_creating_heartbeatreceiver_instance}

```
HeartbeatReceiver(
  sc: SparkContext,
  clock: Clock)
extends SparkListener with ThreadSafeRpcEndpoint
```

HeartbeatReceiver 需要一个 SparkContext 和一个`Clock`。

创建时，HeartbeatReceiver 将自己注册为 SparkListener。

### Internal Registries and Counters {#__a_id_internal_registries_a_internal_registries_and_counters}

Table 1. Internal Registries and Counters

| Name | Description |
| :--- | :--- |
| `executorLastSeen` | 执行者 ID 的注册表和收到最后一次心跳时的时间戳。 |

### Starting — `onStart`Method {#__a_id_onstart_a_starting_code_onstart_code_method}

| Note | `onStart`is part of the RpcEndpoint contract |
| :--- | :--- |


当调用时，HeartbeatReceiver 在 eventLoopThread - Heartbeat Receiver 事件循环线程上发送每个 spark.network.timeoutInterval 的阻塞 ExpireDeadHosts。

### Stopping — `onStop`Method {#__a_id_onstop_a_stopping_code_onstop_code_method}

| Note | `onStop`is part of the RpcEndpoint Contract |
| :--- | :--- |


当调用时，HeartbeatReceiver 取消检查任务（在 eventLoopThread - Heartbeat Receiver 事件循环线程 - 参见启动（onStart 方法））上发送每个 spark.network.timeoutInterval 的阻塞 ExpireDeadHosts，并关闭 eventLoopThread 和 killExecutorThread 执行程序。

### `killExecutorThread` — Kill Executor Thread {#__a_id_killexecutorthread_a_a_id_kill_executor_thread_a_code_killexecutorthread_code_kill_executor_thread}

killExecutorThread 是一个守护进程 ScheduledThreadPoolExecutor 与单线程。

线程池的名称是 kill-executor-thread。

| Note | It is used to request SparkContext to kill the executor. |
| :--- | :--- |


### `eventLoopThread` — Heartbeat Receiver Event Loop Thread {#__a_id_eventloopthread_a_a_id_heartbeat_receiver_event_loop_thread_a_code_eventloopthread_code_heartbeat_receiver_event_loop_thread}

eventLoopThread 是一个守护进程 ScheduledThreadPoolExecutor 与单线程。

线程池的名称是 heartbeat-receiver-event-loop-thread。

### Messages {#__a_id_messages_a_messages}

#### ExecutorRegistered {#__a_id_executorregistered_a_executorregistered}

```
ExecutorRegistered(executorId: String)
```

当 ExecutorRegistered 到达时，executorId 只是添加到 executorLastSeen 内部注册表。

| Note | HeartbeatReceiver 向自己发送 ExecutorRegistered 消息（从 addExecutor 内部方法）。它是作为 SparkListener.onExecutorAdded 的后续，当驱动程序宣布新的执行程序注册。 |
| :---: | :--- |


| Note | It is an internal message. |
| :--- | :--- |


#### ExecutorRemoved {#__a_id_executorremoved_a_executorremoved}

```
ExecutorRemoved(executorId: String)
```

当 ExecutorRemoved 到达时，executorId 只是从 executorLastSeen 内部注册表中删除。

| Note | HeartbeatReceiver 本身发送 ExecutorRegistered 消息（从 removeExecutor 内部方法）。它是作为 SparkListener.onExecutorRemoved 的后续行为，当驱动程序删除执行程序。 |
| :---: | :--- |


| Note | It is an internal message. |
| :--- | :--- |


#### ExpireDeadHosts {#__a_id_expiredeadhosts_a_expiredeadhosts}

```
ExpireDeadHosts
```

当 ExpireDeadHosts 到达时，以下 TRACE 将打印到日志：

```
TRACE HeartbeatReceiver: Checking for hosts with no recent heartbeats in HeartbeatReceiver.
```

检查每个执行程序（在 executorLastSeen 注册表中）最后一次查看的时间是否不超过 spark.network.timeout。

对于任何此类执行程序，以下 WARN 消息将打印到日志中：

```
WARN HeartbeatReceiver: Removing executor [executorId] with no recent heartbeats: [time] ms exceeds timeout [timeout] ms
```

调用 TaskScheduler.executorLost（使用 SlaveLost（执行程序心跳超时\[timecout\]ms）。

SparkContext.killAndReplaceExecutor 被异步地为执行器调用（即在 killExecutorThread 上）。

执行程序从 executorLastSeen 中删除。

| Note | It is an internal message. |
| :--- | :--- |


#### Heartbeat {#__a_id_heartbeat_a_heartbeat}

```
Heartbeat(executorId: String,
  accumUpdates: Array[(Long, Seq[AccumulatorV2[_, _]])],
  blockManagerId: BlockManagerId)
```

当 Heartbeat 到达并且内部调度程序尚未设置（没有任何 TaskSchedulerIsSet 更早）时，以下 WARN 被打印到日志：

```
WARN HeartbeatReceiver: Dropping [heartbeat] because TaskScheduler is not ready yet
```

并且响应是 HeartbeatResponse（reregisterBlockManager = true）。

心跳消息是执行器通知它们活动并更新活动任务的状态的机制。

然而，如果内部调度程序已设置，HeartbeatReceiver 检查执行程序 executorId 是否已知（在 executorLastSeen 中）。

如果无法识别执行程序，则将以下 DEBUG 消息打印到日志中：

```
DEBUG HeartbeatReceiver: Received heartbeat from unknown executor [executorId]
```

并且响应是 HeartbeatResponse（reregisterBlockManager = true）。

然而，如果内部调度器被设置并且执行器被识别（在 executorLastSeen 中），当前时间被记录在 executorLastSeen 中，并且 TaskScheduler.executorHeartbeatReceived 在 eventLoopThread 上被异步调用（即，在单独的线程上）。

响应是 HeartbeatResponse（reregisterBlockManager = unknownExecutor），其中 unknownExecutor 对应于调用 TaskScheduler.executorHeartbeatReceived 的结果。

#### TaskSchedulerIsSet {#__a_id_taskschedulerisset_a_taskschedulerisset}

当 TaskSchedulerIsSet 到达时，HeartbeatReceiver 设置调度程序内部属性（使用 SparkContext.taskScheduler）。

| Note | TaskSchedulerIsSet 由 SparkContext 发送（在创建时）以通知 TaskScheduler 现在可用。 |
| :---: | :--- |


| Note | It is an internal message. |
| :--- | :--- |


### Settings {#__a_id_settings_a_settings}

Table 2. Spark Properties

| Spark Property | Default Value | Description |
| :--- | :--- | :--- |
| `spark.storage.blockManagerTimeoutIntervalMs` | `60s` |  |
| `spark.storage.blockManagerSlaveTimeoutMs` | `120s` |  |
| `spark.network.timeout` | spark.storage.blockManagerSlaveTimeoutMs | See spark.network.timeoutin RPC Environment \(RpcEnv\). |
| `spark.network.timeoutInterval` | `spark.storage.blockManagerTimeoutIntervalMs`+ |   |





