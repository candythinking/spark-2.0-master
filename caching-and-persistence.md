## RDD Caching and Persistence {#_rdd_caching_and_persistence}

缓存或持久性是（迭代和交互）Spark 计算的优化技术。它们有助于节省临时部分结果，因此可以在后续阶段重复使用。因此，作为 RDD 的这些中间结果被保存在存储器（默认）或更加坚固的存储器（如磁盘和/或 replicated）

可以使用高速缓存操作来高速缓存 RDD。它们也可以使用持久操作来持久化。

缓存和持久操作之间的区别是纯粹的语法。 cache 是​​持久性或持久性（MEMORY\_ONLY）的同义词，即，高速缓存只是以默认存储级别 MEMORY\_ONLY 持久化。

| Note | 由于缓存和 RDDs 的持久性之间的非常小的和纯粹的句法差异，这两个术语通常可以互换使用，我将遵循这里的“模式”。 |
| :---: | :--- |


RDD 还可以是 unpersisted，以从诸如存储器和/或磁盘的永久存储器移除 RDD。

### Caching RDD — `cache`Method {#__a_id_cache_a_caching_rdd_code_cache_code_method}

```
cache(): this.type = persist()
```

cache 是​​具有 MEMORY\_ONLY 存储级别的持久性的同义词。

### Persisting RDD — `persist`Method {#__a_id_persist_a_persisting_rdd_code_persist_code_method}

```
persist(): this.type
persist(newLevel: StorageLevel): this.type
```

persist 使用 newLevel 存储级别标记用于持久性的 RDD。

您只能更改一次存储级别或抛出 UnsupportedOperationException 异常：

```
Cannot change storage level of an RDD after it was already assigned a level
```

| Note | 只有当存储级别与当前分配的存储级别相同时，才能假装更改具有已分配存储级别的 RDD 的存储级别。 |
| :---: | :--- |


如果 RDD 第一次被标记为持久化，则 RDD 被注册到 ContextCleaner（如果可用）和 SparkContext。

内部 storageLevel 属性设置为输入的 newLevel 存储级别。

### Unpersisting RDDs \(Clearing Blocks\) — `unpersist`Method {#__a_id_unpersist_a_unpersisting_rdds_clearing_blocks_code_unpersist_code_method}

```
unpersist(blocking: Boolean = true): this.type
```

调用时，unpersist 会在日志中输出以下 INFO 消息：

```
INFO [RddName]: Removing RDD [id] from persistence list
```

然后调用 SparkContext.unpersistRDD（id，blocking），并将 NONE 存储级别设置为当前存储级别。







