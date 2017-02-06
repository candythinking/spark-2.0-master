## YarnSchedulerEndpoint RPC Endpoint {#__a_id_yarnschedulerendpoint_a_yarnschedulerendpoint_rpc_endpoint}

YarnSchedulerEndpoint 是用于在驱动程序上的 YarnSchedulerBackend 和 YARN 上的 ApplicationMaster（在 YARN 容器内部）之间进行通信的线程安全 RPC 端点。

It uses the reference to the remote ApplicationMaster RPC Endpoint to send messages to.

| Tip | Enable`INFO`logging level for`org.apache.spark.scheduler.cluster.YarnSchedulerBackend$YarnSchedulerEndpoint`logger to see what happens inside.Add the following line to`conf/log4j.properties`:log4j.logger.org.apache.spark.scheduler.cluster.YarnSchedulerBackend$YarnSchedulerEndpoint=INFO                                                                                                   Refer to Logging. |
| :--- | :--- |


### RPC Messages {#__a_id_messages_a_rpc_messages}

#### RequestExecutors {#__a_id_requestexecutors_a_requestexecutors}

```
RequestExecutors(
  requestedTotal: Int,
  localityAwareTasks: Int,
  hostToLocalTaskCount: Map[String, Int])
extends CoarseGrainedClusterMessage
```

RequestExecutors 是通知 ApplicationMaster 关于执行器总数（如 requestedTotal）的当前需求，包括已经挂起和正在运行的执行器。

![](/img/mastering-apache-spark/spark on yarn/figure11.png)

当 RequestExecutors 到达时，YarnSchedulerEndpoint 简单地将其传递给 ApplicationMaster（通过内部 RPC 终结点引用）。正向呼叫的结果作为响应发回。

Any issues communicating with the remote`ApplicationMaster`RPC endpoint are reported as ERROR messages in the logs:

```
ERROR Sending RequestExecutors to AM was unsuccessful
```

#### RemoveExecutor {#__a_id_removeexecutor_a_removeexecutor}

#### KillExecutors {#__a_id_killexecutors_a_killexecutors}

#### AddWebUIFilter {#__a_id_addwebuifilter_a_addwebuifilter}

```
AddWebUIFilter(
  filterName: String,
  filterParams: Map[String, String],
  proxyBase: String)
```

AddWebUIFilter 触发器设置 spark.ui.proxyBase 系统属性，并将 filterName 过滤器添加到 Web UI。

AddWebUIFilter 由 ApplicationMaster 发送，当它添加 AmIpFilter 到 Web UI。

它首先将 spark.ui.proxyBase 系统属性设置为输入 proxyBase（如果不为空）。

如果它定义了一个过滤器，即输入 filterName 和 filterParams 都不为空，您应该在日志中看到以下 INFO 消息：

```
INFO Add WebUI Filter. [filterName], [filterParams], [proxyBase]
```

然后将 spark.ui.filters 设置为内部 conf SparkConf 属性中的输入 filterName。

所有 filterParams 也设置为 spark \[filterName\] .param.\[key\] 和 \[value\]。

使用 JettyUtils.addFilters（ui.getHandlers，conf）将过滤器添加到 Web UI。

| Caution | FIXME Review`JettyUtils.addFilters(ui.getHandlers, conf)`. |
| :--- | :--- |


#### RegisterClusterManager Message {#__a_id_registerclustermanager_a_registerclustermanager_message}

```
RegisterClusterManager(am: RpcEndpointRef)
```

When`RegisterClusterManager`message arrives, the following INFO message is printed out to the logs:

```
INFO YarnSchedulerBackend$YarnSchedulerEndpoint: ApplicationMaster registered as [am]
```

对远程 ApplicationMaster RPC 端点的内部引用设置为（到 am）。

如果内部的 shouldResetOnAmRegister 标志被启用，则 YarnSchedulerBackend 被复位。它最初被禁用，因此应启用 shouldResetOnAmRegister。

| Note | shouldResetOnAmRegister 控制是否在另一个 RegisterClusterManager RPC 消息到达时重置 YarnSchedulerBackend，这可能是因为 ApplicationManager 失败并且注册了一个新的。 |
| :---: | :--- |


#### RetrieveLastAllocatedExecutorId {#__a_id_retrievelastallocatedexecutorid_a_retrievelastallocatedexecutorid}

当接收到 RetrieveLastAllocatedExecutorId 时，YarnSchedulerEndpoint 将使用 currentExecutorIdCounter 的当前值进行响应。

| Note | 它由 YarnAllocator 用于初始化内部 executorIdCounter（因此，当 ApplicationMaster 重新启动时，它为新的执行器提供适当的标识符） |
| :---: | :--- |


### onDisconnected Callback {#__a_id_ondisconnected_a_ondisconnected_callback}

onDisconnected 清除远程 ApplicationMaster RPC 端点的内部引用（即它设置为 None），如果远程地址匹配引用。

| Note | It is a callback method to be called when…​FIXME |
| :--- | :--- |


You should see the following WARN message in the logs if that happens:

```
WARN ApplicationMaster has disassociated: [remoteAddress]
```

### onStop Callback {#__a_id_onstop_a_onstop_callback}

onStop 立即关闭 askAmThreadPool。

| Note | `onStop`is a callback method to be called when…​FIXME |
| :--- | :--- |


### Internal Reference to ApplicationMaster RPC Endpoint \(amEndpoint variable\) {#__a_id_amendpoint_a_internal_reference_to_applicationmaster_rpc_endpoint_amendpoint_variable}

amEndpoint 是对远程 ApplicationMaster RPC 端点的引用。

当 RegisterClusterManager 到达时，它被设置为当前的 ApplicationMaster RPC 终端节点，当与终端节点的连接断开时被清除。

### askAmThreadPool Thread Pool {#__a_id_askamthreadpool_a_askamthreadpool_thread_pool}

askAmThreadPool 是一个名为 yarn-scheduler-ask-am-thread-pool 的线程池，它根据需要创建新线程，并在以前构造的线程可用时重用它。





