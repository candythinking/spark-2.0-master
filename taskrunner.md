## TaskRunner {#__a_id_taskrunner_a_taskrunner}

TaskRunner 是一个管理单个任务的执行线程。它可以运行或死亡归结为运行或杀死 TaskRunner 对象管理的任务。

| Tip | 启用 org.apache.spark.executor.Executor 记录器的 INFO 或 DEBUG 记录级别，以查看 TaskRunner 中发生的情况（因为 TaskRunner 是 Executor 的内部类）。 将以下行添加到   conf / log4j.properties：                                                                        log4j.logger.org.apache.spark.executor.Executor=DEBUG    请参阅 Logging    |
| :---: | :--- |


### Creating TaskRunner Instance {#__a_id_creating_instance_a_creating_taskrunner_instance}

`TaskRunner`takes…​FIXME

| Caution | FIXME |
| :--- | :--- |


### `setTaskFinishedAndClearInterruptStatus`Method {#__a_id_settaskfinishedandclearinterruptstatus_a_code_settaskfinishedandclearinterruptstatus_code_method}

| Caution | FIXME |
| :--- | :--- |


### Lifecycle {#_lifecycle}

| Caution | Image with state changes |
| :--- | :--- |


当请求执行器启动任务时，将创建一个 TaskRunner 对象。

它使用 ExecutorBackend（用于发送任务的状态更新），任务和尝试 ID，任务名称和任务的序列化版本（作为 ByteBuffer）创建。

### Running Task — `run`Method {#__a_id_run_a_running_task_code_run_code_method}

| Note | run 是 TaskRunner 遵循的 java.lang.Runnable 合约的一部分。 |
| :---: | :--- |


执行时，run 创建一个 TaskMemoryManager（用于 MemoryManager 和 taskId）。

| Note | 运行使用 SparkEnv 访问当前的 MemoryManager。 |
| :---: | :--- |


`run`开始测量对任务进行反序列化的时间。

`run`设置当前上下文类加载器。

| Cantion | 什么是类加载器的一部分？ |
| :---: | :--- |


run 创建一个闭包 Serializer。

| Note | 运行使用 SparkEnv 访问当前闭包 Serializer。 |
| :---: | :--- |


您应该在日志中看到以下INFO消息：

```
INFO Executor: Running [taskName] (TID [taskId])
```

此时，任务被视为正在运行，并且运行将状态更新发送到 ExecutorBackend（带有 taskId 和 TaskState.RUNNING 状态）。

运行反序列化任务的环境（从使用 Task.deserializeWithDependencies 的 serializedTask 字节）以具有任务的文件，jars 和属性，以及字节（即真实任务的主体）。

| Note | 要运行的目标任务尚未反序列化，但只有其环境 - 文件，jar 和属性。 |
| :---: | :--- |


| Caution | Describe`Task.deserializeWithDependencies`. |
| :--- | :--- |


`updateDependencies(taskFiles, taskJars)`is called.

| Caution | What does`updateDependencies`do? |
| :--- | :--- |


这是正确的 Task 对象使用早期创建的闭包序列化器反序列化（从 taskBytes）的时刻。本地属性（作为 localProperties）被初始化为任务的属性（来自先前对 Task.deserializeWithDependencies 的调用），并且 TaskMemoryManager（在方法中较早创建）被设置为任务。

| Note | 任务的属性是传递到当前 TaskRunner 对象的序列化对象的一部分。 |
| :---: | :--- |


| Note | 直到运行反序列化任务对象，它只能作为 serializedTask 字节缓冲区使用。 |
| :---: | :--- |


如果 kill 方法在此期间被调用，则运行抛出 TaskKilledException 异常。

您应该在日志中看到以下DEBUG消息：

```
DEBUG Executor: Task [taskId]'s epoch is [task.epoch]
```

运行通知 MapOutputTracker 有关任务的纪元\(epoch\)。

| Note | 运行使用 SparkEnv 访问当前的 MapOutputTracker。 |
| :---: | :--- |


| Cantion | 为什么需要 MapOutputTracker.updateEpoch？ |
| :---: | :--- |


`run`将当前时间记住为任务的开始时间（作为 taskStart）。

`run`运行任务（使用 taskId，attemptNumber 和 MetricsSystem）。

| Note | run 使用 SparkEnv 访问当前 MetricsSystem。 |
| :---: | :--- |


任务在“监视”块（即 try-finally 块）内运行以在任务运行完成后清除，而不管最终结果 - 任务的值或抛出的异常。

在任务运行完成后（并且不管是否抛出异常），运行请求 BlockManager 释放任务的所有锁（使用当前任务的 taskId）。

`run`然后总是查询 TaskMemoryManager 内存泄漏。如果有任何（即在调用之后释放的内存大于0）和 spark.unsafe.exceptionOnMemoryLeak 被启用（默认情况下不是这样），任务运行时没有抛出异常，则抛出 SparkException：

```
Managed memory leak detected; size = [freedMemory] bytes, TID = [taskId]
```

否则，如果 spark.unsafe.exceptionOnMemoryLeak 被禁用或任务抛出异常，您应该在日志中看到以下 ERROR 消息：

```
ERROR Executor: Managed memory leak detected; size = [freedMemory] bytes, TID = [taskId]
```

| Note | 检测到的内存泄漏导致日志中出现 SparkException 或 ERROR 消息。 |
| :---: | :--- |


如果有任何 releasesLocks（在之前调用 BlockManager.releaseAllLocksForTask）和 spark.storage.exceptionOnPinLeak 被启用（它不是默认情况下），没有异常在任务运行时抛出，抛出 SparkException：

```
[releasedLocks] block locks were not released by TID = [taskId]:
[releasedLocks separated by comma]
```

否则，如果 spark.storage.exceptionOnPinLeak 被禁用或任务抛出异常，您应该在日志中看到以下 WARN 消息：

```
WARN Executor: [releasedLocks] block locks were not released by TID = [taskId]:
[releasedLocks separated by comma]
```

| Note | 如果有任何 releaseLocks，它们会在日志中导致 SparkException 或 WARN 消息。 |
| :---: | :--- |


`run`会将当前时间记住为任务的完成时间（作为 taskFinish）。

如果任务被终止，run 将抛出一个 TaskKilledException（并且 TaskRunner 退出）。

任务成功完成后，它返回一个值。该值被序列化（使用来自 SparkEnv 的 Serialize r的新实例，即 serializer）。

| Note | SparkEnv 中有两个 Serializer 对象。 |
| :---: | :--- |


序列化任务值的时间被跟踪（使用 beforeSerialization 和 afterSerialization）

设置任务的度量，即 executorDeserializeTime，executorRunTime，jvmGCTime 和 resultSerializationTime。

| Caution | 请更详细地描述指标。并包括一个数字来显示指标点。 |
| :---: | :--- |


run 收集累加器的最新值（as accumUpdates）。

将创建具有序列化结果和累加器的最新值的 DirectTaskResult 对象（作为 directResult）。然后对象被序列化（使用全局闭包序列化器）。

计算序列化 DirectTaskResult 对象的缓冲区的限制（作为 resultSize）。

计算 serializedResult（很快就会被发送到 ExecutorBackend）。它取决于 resultSize 的大小。

如果设置了 maxResultSize，并且序列化的 DirectTaskResult 的大小超过了它，日志中将显示以下 WARN 消息：

```
WARN Executor: Finished [taskName] (TID [taskId]). Result is larger than maxResultSize ([resultSize] > [maxResultSize]), dropping it.
```

| Tip | 阅读 spark.driver.maxResultSize。 |
| :---: | :--- |


```
$ ./bin/spark-shell -c spark.driver.maxResultSize=1m

scala> sc.version
res0: String = 2.0.0-SNAPSHOT

scala> sc.getConf.get("spark.driver.maxResultSize")
res1: String = 1m

scala> sc.range(0, 1024 * 1024 + 10, 1).collect
WARN Executor: Finished task 4.0 in stage 0.0 (TID 4). Result is larger than maxResultSize (1031.4 KB > 1024.0 KB), dropping it.
...
ERROR TaskSetManager: Total size of serialized results of 1 tasks (1031.4 KB) is bigger than spark.driver.maxResultSize (1024.0 KB)
...
org.apache.spark.SparkException: Job aborted due to stage failure: Total size of serialized results of 1 tasks (1031.4 KB) is bigger than spark.driver.maxResultSize (1024.0 KB)
  at org.apache.spark.scheduler.DAGScheduler.org$apache$spark$scheduler$DAGScheduler$$failJobAndIndependentStages(DAGScheduler.scala:1448)
...
```

最后的 serializedResult 变成一个序列化的 IndirectTaskResult，带有任务 taskId 和 resultSize 的 TaskResultBlockId。

否则，当 maxResultSize 不为正或 resultSize 小于 maxResultSize 但大于 maxDirectResultSize 时，将创建任务 taskId 的 TaskResultBlockId 对象（作为 blockId），serializedDirectResult 作为 blockId 块存储到具有 MEMORY\_AND\_DISK\_SER 存储级别的 BlockManager。

| Caution | 描述 maxDirectResultSize。 |
| :---: | :--- |


以下INFO消息将打印到日志中：

```
INFO Executor: Finished [taskName] (TID [taskId]). [resultSize] bytes result sent via BlockManager)
```

最后的 serializedResult 变成一个序列化的 IndirectTaskResult，带有任务 taskId 和 resultSize 的 TaskResultBlockId。

| Note | 两种情况之间的区别是，结果被丢弃或通过 BlockManager 发送。 |
| :---: | :--- |


当上述两种情况不成立时，以下 INFO 消息将打印到日志中：

```
INFO Executor: Finished [taskName] (TID [taskId]). [resultSize] bytes result sent to driver
```

最后的 serializedResult 变成 serializedDirectResult（即序列化的 DirectTaskResult）。

当 TaskRunner 完成时，将从所拥有的 Executor 的内部 runningTasks 映射中删除 taskId（最终清除对 TaskRunner 的任何引用）。

| Note | TaskRunner 是 Java 的 Runnable，规定要求一旦 TaskRunner 完成执行，它可能不会重新启动。 |
| :---: | :--- |


#### FetchFailedException {#__a_id_run_fetchfailedexception_a_fetchfailedexception}

当运行任务时抛出 FetchFailedException 时，运行捕获它和 setTaskFinishedAndClearInterruptStatus。

运行请求 FetchFailedException 为 TaskFailedReason，序列化它，并通知 ExecutorBackend 任务已失败（与 taskId， TaskState.FAILED 和序列化的原因）。

| Note | 创建 TaskRunner 时指定了 ExecutorBackend。 |
| :---: | :--- |


| Note | run 使用闭包序列化程序来序列化失败原因。 Serializer 是在运行任务之前创建的。 |
| :---: | :--- |


### Killing Task — `kill`Method {#__a_id_kill_a_killing_task_code_kill_code_method}

```
kill(interruptThread: Boolean): Unit
```

kill 将 TaskRunner 的当前实例标记为已杀死，并将任务的调用传递给任务本身（如果可用）。

执行时，您应该在日志中看到以下 INFO 消息：

```
INFO TaskRunner: Executor is trying to kill [taskName] (TID [taskId])
```

在内部，kill 使内部标志被杀死，并且如果任务可用，则执行它的 Task.kill 方法。

| Note | 在运行中检查已停止的内部标志以停止执行任务。调用 Task.kill 方法允许稍后的任务中断。 |
| :---: | :--- |


### Settings {#__a_id_settings_a_settings}

Table 1. Spark Properties

| Spark Property | Default Value | Description |
| :--- | :--- | :--- |
| `spark.unsafe.exceptionOnMemoryLeak` | `false` |   |





