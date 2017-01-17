## AMEndpoint — ApplicationMaster RPC Endpoint {#__a_id_amendpoint_a_amendpoint_applicationmaster_rpc_endpoint}

### onStart Callback {#__a_id_onstart_a_onstart_callback}

当调用 onStart 时，AMEndpoint 通过发送一个带有对其引用的单向 RegisterClusterManager 消息与驱动程序（驱动程序远程 RPC 终端引用）通信。

RegisterClusterManager 发送（并由 YarnSchedulerEndpoint 接收）后，ApplicationMaster（YARN）和YarnSchedulerBackend（Spark 驱动程序）的 RPC 端点之间的通信被认为已建立。

### RPC Messages {#__a_id_messages_a_rpc_messages}

#### AddWebUIFilter {#__a_id_addwebuifilter_a_addwebuifilter}

```
AddWebUIFilter(
  filterName: String,
  filterParams: Map[String, String],
  proxyBase: String)
```

When`AddWebUIFilter`arrives, you should see the following INFO message in the logs:

```
INFO ApplicationMaster$AMEndpoint: Add WebUI Filter. [addWebUIFilter]
```

然后将 AddWebUIFilter 消息传递到驱动程序后端（通过 YarnScheduler RPC Endpoint）。

#### RequestExecutors {#__a_id_requestexecutors_a_requestexecutors}

```
RequestExecutors(
  requestedTotal: Int,
  localityAwareTasks: Int,
  hostToLocalTaskCount: Map[String, Int])
```

当 RequestExecutors 到达时，AMEndpoint 向给定局部性偏好的执行器请求 YarnAllocator。

如果 requestedTotal 执行器数量不同于当前数量，则执行 resetAllocatorInterval。

In case when`YarnAllocator`is not available yet, you should see the following WARN message in the logs:

```
WARN Container allocator is not ready to request executors yet.
```

The response is`false`then.

### resetAllocatorInterval {#__a_id_resetallocatorinterval_a_resetallocatorinterval}

当 RequestExecutors 消息到达时，它调用 resetAllocatorInterval 过程。

```
resetAllocatorInterval(): Unit
```

resetAllocatorInterval 请求 allocatorLock 监视器锁定，并将内部的 nextAllocationInterval 属性设置为 initialAllocationInterval 内部属性。然后它唤醒所有等待 allocatorLock 的线程。

| Note | 线程通过调用 Object.wait 方法之一在监视器上等待。 |
| :---: | :--- |


























