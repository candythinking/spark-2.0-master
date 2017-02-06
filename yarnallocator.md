## YarnAllocator — YARN Resource Container Allocator {#__a_id_yarnallocator_a_yarnallocator_yarn_resource_container_allocator}

YarnAllocator 从 YARN 集群（以来自 YARN ResourceManager 的容器的形式）请求资源，并通过将它们分配给 Spark 执行程序并在 Spark 应用程序不再需要时释放它们来管理容器分配。

YarnAllocator 使用 AMRMClient（创建 YarnAllocator 时 YarnRMClient 传入）管理资源。

![](/img/mastering-apache-spark/spark on yarn/figure12.png)

YarnAllocator 是 ApplicationMaster 的内部状态的一部分（通过内部分配器引用）。

![](/img/mastering-apache-spark/spark on yarn/figure13.png)

YarnAllocator 稍后在分配的 YARN 资源容器中启动 Spark 执行程序。

![](/img/mastering-apache-spark/spark on yarn/figure14.png)

Table 1. YarnAllocator’s Internal Registries and Counters

| Name | Description |
| :--- | :--- |
| `resource` | 设置单个执行程序的容量要求（即内存和虚拟核心）的 YARN资源。                                                                                                        注意：资源模型是集群中的一组计算机资源。目前有内存和虚拟CPU内核（vcore）。                                                                          在创建 YarnAllocator 时创建，并且是内存量的 executorMemory 和 memoryOverhead 以及虚拟核心数的 executorCore 的总和。 |
| executorIdCounter | `executorIdCounter`用于在分配的 YARN 资源容器中启动 Spark 执行程序时设置执行程序标识。                                                设置为最后分配的执行程序标识（在创建 YarnAllocator 时通过 RPC 系统接收）。 |
| `targetNumExecutors` | 当前所需的执行程序总数（作为 YARN 资源容器）。设置为创建 YarnAllocator 时的初始执行程序数。targetNumExecutors最终在 YarnAllocator 更新 YARN 容器分配请求后到达。稍后可以更改当 YarnAllocator 被请求的执行器的总数给定 locality 首选项。在请求缺少资源容器并在分配的资源容器中启动 Spark 执行程序时使用。 |
| `numExecutorsRunning` | 用于更新 YARN 容器分配请求，并获取当前运行的执行程序数。在分配的 YARN 资源容器中启动 Spark 执行程序时递增，并在为 Spark 执行程序释放资源容器时递减。 |
| `currentNodeBlacklist` | List of…​FIXME |
| `releasedContainers` | 不再需要的容器，它们的全局唯一标识符 ContainerId 将不被使用（对于集群中的容器）。NOTE: Hadoop YARN 的容器表示集群中分配的资源。 YARN ResourceManager 是将任何 Container 分配给应用程序的唯一权限。分配的容器始终位于单个节点上，并且具有唯一的 ContainerId。它具有特定数量的资源分配。 |
| `allocatedHostToContainersMap` | Lookup table |
| `allocatedContainerToHostMap` | Lookup Table |
| `pendingLossReasonRequests` |  |
| `releasedExecutorLossReasons` |  |
| `executorIdToContainer` |  |
| `numUnexpectedContainerRelease` |  |
| `containerIdToExecutorId` |  |
| `hostToLocalTaskCounts` | Lookup table |
| `failedExecutorsTimeStamps` |  |
| `executorMemory` |  |
| `memoryOverhead` |  |
| `executorCores` |  |
| `launchContainers` |  |
| `labelExpression` |  |
| `nodeLabelConstructor` |  |
| `containerPlacementStrategy` |  |
| `launcherPool` | ContainerLauncher Thread Pool |
| `numLocalityAwareTasks` | 当向指定位置首选项的执行程序请求 YarnAllocator 时，用作容器放置提示的区域感知任务数。创建 YarnAllocator 时设置为 0。当 YarnAllocator 更新 YARN 容器分配请求时，用作 containerPlacementStrategy.localityOfRequestedContainers 的输入。 |

| Tip | Enable`INFO`or`DEBUG`logging level for`org.apache.spark.deploy.yarn.YarnAllocator`logger to see what happens inside.Add the following line to`conf/log4j.properties`:       log4j.logger.org.apache.spark.deploy.yarn.YarnAllocator=DEBUG                                                                                                       Refer to Logging. |
| :--- | :--- |


### Creating YarnAllocator Instance {#__a_id_creating_instance_a_creating_yarnallocator_instance}

YarnAllocator 创建时采取以下操作：

1. `driverUrl`

2. `driverRef` — RpcEndpointRef to the driver’s FIXME

3. YarnConfiguration

4. `sparkConf` — SparkConf

5. `amClient`AMRMClient for`ContainerRequest`

6. `ApplicationAttemptId`

7. `SecurityManager`

8. `localResources` — `Map[String, LocalResource]`

YarnAllocator（但 appAttemptId 和 amClient）的所有输入参数直接从 YarnRMClient 的输入参数传递。

YarnAllocator 将 org.apache.hadoop.yarn.util.RackResolver 记录器设置为 WARN（除非已设置为某个日志级别）。

YarnAllocator 初始化内部注册表和计数器（registries and counters）。

It sets the following internal counters:

* `numExecutorsRunning`to`0`

* `numUnexpectedContainerRelease`to`0L`

* `numLocalityAwareTasks`to`0`

* `targetNumExecutors`to the initial number of executors

它创建一个空的队列失败的执行器。

它将内部 executorFailuresValidityInterval 设置为 spark.yarn.executor.failuresValidityInterval。

它将内部 executorMemory 设置为 spark.executor.memory。

它将内部 memoryOverhead 设置为 spark.yarn.executor.memoryOverhead。如果不可用，则设置为 executorMemory 和 384 的最大值的 10％。

它将内部 executorCores 设置为 spark.executor.cores。

它使用 executorMemory + memoryOverhead 内存和 executorCores CPU 内核创建 Hadoop YARN 资源的内部资源。

它创建名为 ContainerLauncher 的内部 launcherPool，最大 spark.yarn.containerLauncherMaxThreads 线程。

它将内部 launchContainers 设置为 spark.yarn.launchContainers。

它将内部 labelExpression 设置为 spark.yarn.executor.nodeLabelExpression。

It sets the internal`nodeLabelConstructor`to…​FIXME

| Caution | FIXME nodeLabelConstructor? |
| :--- | :--- |


It sets the internal`containerPlacementStrategy`to…​FIXME

| Caution | FIXME LocalityPreferredContainerPlacementStrategy? |
| :--- | :--- |


#### `getNumExecutorsRunning`Method {#__a_id_getnumexecutorsrunning_a_code_getnumexecutorsrunning_code_method}

| Caution | FIXME |
| :--- | :--- |


#### `updateInternalState`Method {#__a_id_updateinternalstate_a_code_updateinternalstate_code_method}

| Caution | FIXME |
| :--- | :--- |


### `killExecutor`Method {#__a_id_killexecutor_a_code_killexecutor_code_method}

| Caution | FIXME |
| :--- | :--- |


### Specifying Current Total Number of Executors with Locality Preferences — `requestTotalExecutorsWithPreferredLocalities`Method {#__a_id_requesttotalexecutorswithpreferredlocalities_a_specifying_current_total_number_of_executors_with_locality_preferences_code_requesttotalexecutorswithpreferredlocalities_code_method}

```
requestTotalExecutorsWithPreferredLocalities(
  requestedTotal: Int,
  localityAwareTasks: Int,
  hostToLocalTaskCount: Map[String, Int],
  nodeBlacklist: Set[String]): Boolean
```

requestTotalExecutorsWithPreferredLocalities 返回当前所需的执行器总数是否与输入的 requestedTotal 不同。

| Note | requestTotalExecutorsWithPreferredLocalities 应该被调用 shouldRequestTotalExecutorsWithPreferredLocalities，因为它回答了是否请求新的总执行者的问题。 |
| :---: | :--- |


requestTotalExecutorsWithPreferredLocalities 分别将内部 numLocalityAwareTasks 和 hostToLocalTask​​Counts 属性设置为输入 localityAwareTasks 和 hostToLocalTask​​Count 参数。

If the input`requestedTotal`is different than the internal targetNumExecutors you should see the following INFO message in the logs:

```
INFO YarnAllocator: Driver requested a total number of [requestedTotal] executor(s).
```

requestTotalExecutorsWithPreferredLocalities 将输入 requestedTotal 保存为当前所需的执行程序总数。

requestTotalExecutorsWithPreferredLocalities 为此应用程序更新黑名单信息到 YARN ResouceManager，以避免在有问题的节点上分配新的容器。

| Caution | FIXME Describe the blacklisting |
| :--- | :--- |


| Note | `requestTotalExecutorsWithPreferredLocalities`is executed in response to`RequestExecutors`message to`ApplicationMaster`. |
| :--- | :--- |


### Adding or Removing Container Requests to Launch Executors — `updateResourceRequests`Method {#__a_id_updateresourcerequests_a_adding_or_removing_container_requests_to_launch_executors_code_updateresourcerequests_code_method}

```
updateResourceRequests(): Unit
```

updateResourceRequests 从 YARN ResourceManager 请求新的或取消未完成的执行器容器。

| Note | 在 YARN 中，您必须首先请求资源容器（使用 AMRMClient.addContainerRequest），然后再调用 AMRMClient.allocate。 |
| :---: | :--- |


它获取未完成的 YARN 的 ContainerRequest 列表（使用构造函数的 AMRMClient \[ContainerRequest\]），并将它们的数量与当前工作负载对齐。

updateResourceRequests 包含两个主要分支：

1. 丢失的执行器，即当已经分配或挂起的执行器的数量不匹配需求，因此存在遗漏的执行器。

2. 执行者取消，即当待决执行器分配的数量为正，但所有执行器的数量超过 Spark 需要时。

| Note | 当 YarnAllocator 请求新的资源容器时，使用 updateResourceRequests。 |
| :---: | :--- |


#### Case 1. Missing Executors {#__a_id_updateresourcerequests_missing_executors_a_case_1_missing_executors}

You should see the following INFO message in the logs:

```
INFO YarnAllocator: Will request [count] executor containers, each with [vCores] cores and [memory] MB memory including [memoryOverhead] MB overhead
```

然后，它根据每个待处理任务的位置首选项（在内部 hostToLocalTask​​Counts 注册表中）拆分待处理的容器分配请求。

| Caution | 查看 splitPendingAllocationsByLocality |
| :---: | :--- |


它删除失效的容器分配请求（使用 YARN 的 AMRMClient.removeContainerRequest）。

| Caution | FIXME Stale? |
| :--- | :--- |


You should see the following INFO message in the logs:

```
INFO YarnAllocator: Canceled [cancelledContainers] container requests (locality no longer needed)
```

它计算所请求容器的位置（基于内部 numLocalityAwareTasks，hostToLocalTask​​Counts 和 allocatedHostToContainersMap 查找表）。

| Caution | 查看 containerPlacementStrategy.localityOfRequestedContainers  后面的代码。 |
| :---: | :--- |


对于任何新的容器，updateResourceRequests 添加一个容器请求（使用 YARN 的 AMRMClient.addContainerRequest）。

You should see the following INFO message in the logs:

```
INFO YarnAllocator: Submitted container request (host: [host], capability: [resource])
```

#### Case 2. Cancelling Pending Executor Allocations {#__a_id_updateresourcerequests_cancelling_executor_allocations_a_case_2_cancelling_pending_executor_allocations}

When there are executors to cancel \(case 2.\), you should see the following INFO message in the logs:

```
INFO Canceling requests for [numToCancel] executor container(s) to have a new desired total [targetNumExecutors] executors.
```

它检查是否有待处理的分配请求，并删除多余的（使用 YARN 的 AMRMClient.removeContainerRequest）。如果没有待处理的分配请求，您应该在日志中看到 WARN 消息：

```
WARN Expected to find pending requests, but found none.
```

### Handling Allocated Containers for Executors — `handleAllocatedContainers`Internal Method {#__a_id_handleallocatedcontainers_a_handling_allocated_containers_for_executors_code_handleallocatedcontainers_code_internal_method}

```
handleAllocatedContainers(allocatedContainers: Seq[Container]): Unit
```

handleAllocatedContainers 处理分配的 YARN 容器，即在匹配的容器上运行 Spark 执行器或释放不需要的容器。

| Note | YARN 容器表示集群中分配的资源。分配的容器始终位于单个节点上，并且具有唯一的 ContainerId。它具有特定数量的资源分配。 |
| :---: | :--- |


在内部，handleAllocatedContainers 将请求匹配到主机，机架和任何主机（容器分配）。

如果 handleAllocatedContainers 没有管理分配一些容器，您应该在日志中看到以下 DEBUG 消息：

```
DEBUG Releasing [size] unneeded containers that were allocated to us
```

handleAllocatedContainers 释放不需要的容器（如果有）。

handleAllocatedContainers 运行已分配和匹配的容器。

You should see the following INFO message in the logs:

```
INFO Received [allocatedContainersSize] containers from YARN, launching executors on [containersToUseSize] of them.
```

| Note | handleAllocatedContainers 仅在 YarnAllocator 为 Spark 执行程序分配 YARN 资源容器时使用。 |
| :---: | :--- |


### Running ExecutorRunnables \(with CoarseGrainedExecutorBackends\) in Allocated YARN Resource Containers — `runAllocatedContainers`Internal Method {#__a_id_runallocatedcontainers_a_running_executorrunnables_with_coarsegrainedexecutorbackends_in_allocated_yarn_resource_containers_code_runallocatedcontainers_code_internal_method}

```
runAllocatedContainers(containersToUse: ArrayBuffer[Container]): Unit
```

runAllocatedContainers 遍历 YARN 容器集合（作为输入 containersToUse），并调度 ContainerLauncher 线程池上每个 YARN 容器的 ExecutorRunnables 的执行。

![](/img/mastering-apache-spark/spark on yarn/figure15.png)

| Note | YARN 中的容器表示集群中已分配的资源（内存和核心）。 |
| :---: | :--- |


在内部，runAllocatedContainers 递增 executorIdCounter 内部计数器。

| Note | `runAllocatedContainers`asserts that the amount of memory of a container not less than the requested memory for executors. And only memory! |
| :--- | :--- |


You should see the following INFO message in the logs:

```
INFO YarnAllocator: Launching container [containerId] for on host [executorHostname]
```

runAllocatedContainers 检查运行的执行程序的数量是否小于所需的执行程序的数量。

如果有执行者仍然缺失（并且 runAllocatedContainers 不处于测试模式），runAllocatedContainers 将调度在 ContainerLauncher 线程池上执行 ExecutorRunnable 并更新内部状态。当执行 ExecutorRunnable runAllocatedContainers 首先创建一个ExecutorRunnable 并启动它。

当 runAllocatedContainers 捕获非致命异常时，您应该在日志中看到以下 ERROR 消息，并立即释放容器（使用内部 AMRMClient）。

```
ERROR Failed to launch executor [executorId] on container [containerId]
```

如果 YarnAllocator 已达到目标执行程序数，则应在日志中看到以下 INFO 消息：

```
INFO Skip launching executorRunnable as running Executors count: [numExecutorsRunning] reached target Executors count: [targetNumExecutors].
```

| Note | runAllocatedContainers 仅在 YarnAllocator 处理分配的 YARN 容器时使用。 |
| :---: | :--- |


#### Releasing YARN Container — `internalReleaseContainer`Internal Procedure {#__a_id_internalreleasecontainer_a_releasing_yarn_container_code_internalreleasecontainer_code_internal_procedure}

所有不必要的 YARN 容器（已分配，但是没有使用或不再需要）使用内部 internalReleaseContainer 过程释放。

```
internalReleaseContainer(container: Container): Unit
```

internalReleaseContainer 在内部的 releasedContainers 注册表中记录容器，并将其释放到 YARN ResourceManager（使用内部 amClient 调用 AMRMClient \[ContainerRequest\] .releaseAssignedContainer）。

#### Deciding on Use of YARN Container — `matchContainerToRequest`Internal Method {#__a_id_matchcontainertorequest_a_deciding_on_use_of_yarn_container_code_matchcontainertorequest_code_internal_method}

当 handleAllocatedContainers 处理执行器的分配容器时，它使用 matchContainerToRequest 将容器与 ContainerRequest 匹配（从而匹配工作负载和位置首选项）。

```
matchContainerToRequest(
  allocatedContainer: Container,
  location: String,
  containersToUse: ArrayBuffer[Container],
  remaining: ArrayBuffer[Container]): Unit
```

matchContainerToRequest 将 allocatedContainer 置于 containersToUse 或剩余集合中，每个可用未完成的 ContainerRequests 匹配输入 allocateContainer 的优先级，输入位置以及 Spark 执行程序的内存和 vcore 功能。

| Note | The input`location`can be host, rack, or`*`\(star\), i.e. any host. |
| :--- | :--- |


它获取未完成的 ContainerRequests（来自 YARN ResourceManager）。

如果有任何未完成的 ContainerRequest 满足要求，它只需要第一个并将其放在输入 containersToUse 集合中。它也删除了 ContainerRequest，所以它不会再次提交（它使用内部 AMRMClient \[ContainerRequest\]）。

否则，它将输入 allocateContainer 放在输入剩余集合中。

### `processCompletedContainers`Method {#__a_id_processcompletedcontainers_a_code_processcompletedcontainers_code_method}

```
processCompletedContainers(completedContainers: Seq[ContainerStatus]): Unit
```

processCompletedContainers 接受 YARN 的 ContainerStatus 的集合。

| Note | `ContainerStatus`represents the current status of a YARN`Container`and provides details such as:                                              Id                                                                                                                State                                                                                                         Exit status of a completed container.                                             Diagnostic message for a failed container. |
| :--- | :--- |


对于集合中的每个已完成容器，processCompletedContainers 将从内部 releasedContainers 注册表中删除它。

它看起来容器的主机（在内部 allocateContainerToHostMap 查找表）。主机可以存在或不存在于查找表中。

| Caution | FIXME The host may or may not exist in the lookup table? |
| :--- | :--- |


The`ExecutorExited`exit reason is computed.

当找到完成的容器的主机时，内部 numExecutorsRunning 计数器递减。

You should see the following INFO message in the logs:

```
INFO Completed container [containerId] [host] (state: [containerState], exit status: [containerExitStatus])
```

对于 ContainerExitStatus.SUCCESS 和 ContainerExitStatus.PREEMPTED 容器的退出状态（不认为应用程序失败），您应该在日志中看到两个可能的 INFO 消息之一：

```
INFO Executor for container [id] exited because of a YARN event (e.g., pre-emption) and not because of an error in the running job.
```

```
INFO Container [id] [host] was preempted.
```

容器的其他退出状态被视为应用程序故障，并在日志中作为 WARN 消息报告：

```
WARN Container killed by YARN for exceeding memory limits. [diagnostics] Consider boosting spark.yarn.executor.memoryOverhead.
```

或者

```
WARN Container marked as failed: [id] [host]. Exit status: [containerExitStatus]. Diagnostics: [containerDiagnostics]
```

主机在内部 allocateHostToContainersMap 查找表中查找。如果找到，则从为主机注册的容器中删除容器，或者当此容器是主机上的最后一个时，主机本身将从查找表中删除。

容器从内部 allocateContainerToHostMap 查找表中删除。

容器从内部 containerIdToExecutorId 转换表中删除。如果找到执行程序，它将从内部 executorIdToContainer 转换表中删除。

如果执行器被记录在内部的 pendingLossReasonRequests 查找表中，则为记录的每个未决 RPC 消息发送退出原因（早先计算为 ExecutorExited）。

如果没有找到执行器，则执行器和退出原因记录在内部的 releasedExecutorLossReasons 查找表中。

如果容器不在内部 releasedContainers 注册表中，内部 numUnexpectedContainerRelease 计数器增加，并向驱动程序发送 RemoveExecutor RPC 消息（如创建 YarnAllocator 时指定）以通知执行程序的失败。

### Requesting and Allocating YARN Resource Containers to Spark Executors \(and Cancelling Outstanding Containers\) — `allocateResources`Method {#__a_id_allocateresources_a_requesting_and_allocating_yarn_resource_containers_to_spark_executors_and_cancelling_outstanding_containers_code_allocateresources_code_method}

```
allocateResources(): Unit
```

allocateResources 从 YARN ResourceManager 声明新的资源容器，并取消任何未完成的资源容器请求。

| Note | 在 YARN 中，您首先必须向 YARN ResourceManager 提交对 YARN 资源容器的请求（使用 AMRMClient.addContainerRequest），然后通过调用 AMRMClient.allocate 声明它们。 |
| :---: | :--- |


在内部，allocateResources 提交对新容器的请求并取消先前的容器请求。

allocateResources 然后声明容器（使用 YARN 的 AMRMClient 的内部引用），进度指示符为 0.1f。

您可以在 YARN 控制台中查看 Spark 应用程序的精确时刻，进度条为 10％。

![](/img/mastering-apache-spark/spark on yarn/figure16.png)

If the number of allocated containers is greater than`0`, you should see the following DEBUG message in the logs \(in stderr on YARN\):

```
DEBUG YarnAllocator: Allocated containers: [allocatedContainersSize]. Current executor count: [numExecutorsRunning]. Cluster resources: [availableResources].
```

allocateResources 在分配的 YARN 资源容器上启动执行程序。

allocateResources 从 YARN ResourceManager 获取已完成容器的状态列表。

If the number of completed containers is greater than`0`, you should see the following DEBUG message in the logs \(in stderr on YARN\):

```
DEBUG YarnAllocator: Completed [completedContainersSize] containers
```

`allocateResources`processes completed containers.

You should see the following DEBUG message in the logs \(in stderr on YARN\):

```
DEBUG YarnAllocator: Finished processing [completedContainersSize] completed containers. Current running executor count: [numExecutorsRunning].
```

| Note | 当 ApplicationMaster 注册到 YARN ResourceManager 并启动 progress Reporter 线程时，使用 allocateResources。 |
| :---: | :--- |
















