## `BlockStatusListener`Spark Listener {#__a_id_blockstatuslistener_a_code_blockstatuslistener_code_spark_listener}

BlockStatusListener 是一个 SparkListener，用于跟踪 Web 管理器和 Web UI 中的存储选项卡的块。

表1. BlockStatusListener注册表

| Registry | Description |
| :--- | :--- |
| `blockManagers` | 每个 BlockManagerId 的 BlockId 和 BlockUIData 的集合的查找表。 |

| Caution | When are the events posted? |
| :--- | :--- |




表2. BlockStatusListener事件处理程序

| Event Handler | Description |
| :--- | :--- |
| `onBlockManagerAdded` | 在 blockManagers 内部注册表中注册一个 BlockManager（没有块）。 |
| `onBlockManagerRemoved` | 从 blockManagers 内部注册表中删除一个 BlockManager。 |
| `onBlockUpdated` | 在 BlockManager 内部注册表中将 BlockManager 的 BlockId 的更新的 BlockUIData。 忽略未注册的 BlockManagers 或非 StreamBlockIds 的更新。 对于无效的 StorageLevel（即它们不使用内存或磁盘或不使用复制），将删除该块。 |



