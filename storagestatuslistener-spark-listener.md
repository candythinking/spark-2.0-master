## `StorageStatusListener`Spark Listener {#__a_id_storagestatuslistener_a_code_storagestatuslistener_code_spark_listener}

StorageStatusListener 是跟踪 Spark 应用程序中“节点”上的 BlockManager 的状态的 SparkListener，即驱动程序和执行程序。

注意

创建 SparkUI 时创建并注册 StorageStatusListener。它以后用于创建 ExecutorsListener 和 StorageListener Spark 侦听器。

表1. StorageStatusListener 注册表

| Registry | Description |
| :--- | :--- |
| `executorIdToStorageStatus` | 每个执行程序或驱动程序的 StorageStatus 的查找表 |
| `deadExecutorStorageStatus` | 已删除/非活动 BlockManagers 的 StorageStatus 的集合。 |

表2. StorageStatusListener 事件处理程序

| Event Handler | Description |
| :--- | :--- |
| `onUnpersistRDD` | 从 executorIdToStorageStatus 内部注册表中的每个 StorageStatus 中删除未分配的 rddId 的 RDD 块。 |
| `onBlockManagerAdded` | 在 executorIdToStorageStatus 内部注册表中的执行程序上注册一个 BlockManager。 删除可能已在 deadExecutorStorageStatus 内部注册表中较早地为执行程序注册的任何其他 BlockManager。 |
| `onBlockManagerRemoved` | 从 executorIdToStorageStatus 内部注册表中删除执行程序，并将删除的 StorageStatus 添加到 deadExecutorStorageStatus 内部注册表中。 当 deadExecutorStorageStatus 中的条目数大于 spark.ui.retainedDeadExecutors 时，删除最旧的 StorageStatus。 |
| `onBlockUpdated` | 更新 executorIdToStorageStatus 内部注册表中执行程序的 StorageStatus，即删除 NONE 存储级别的块，否则更新。 |



