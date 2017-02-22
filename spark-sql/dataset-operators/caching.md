## Caching {#_caching}

| Caution | FIXME |
| :--- | :--- |


您可以使用CACHE TABLE \[tableName\]来缓存内存中的tableName表。这是一个渴望的操作，只要语句被执行就执行。

```
sql("CACHE TABLE [tableName]")
```

你可以使用LAZY关键字使缓存延迟。

```
// Cache Dataset -- it is lazy
scala> val df = spark.range(1).cache
df: org.apache.spark.sql.Dataset[Long] = [id: bigint]

// Trigger caching
scala> df.show
+---+
| id|
+---+
|  0|
+---+

// Visit http://localhost:4040/storage to see the Dataset cached. It should.

// You may also use queryExecution or explain to see InMemoryRelation
// InMemoryRelation is used for cached queries
scala> df.queryExecution.withCachedData
res0: org.apache.spark.sql.catalyst.plans.logical.LogicalPlan =
InMemoryRelation [id#0L], true, 10000, StorageLevel(disk, memory, deserialized, 1 replicas)
   +- *Range (0, 1, step=1, splits=Some(8))

// Use the cached Dataset in another query
// Notice InMemoryRelation in use for cached queries
scala> df.withColumn("newId", 'id).explain(extended = true)
== Parsed Logical Plan ==
'Project [*, 'id AS newId#16]
+- Range (0, 1, step=1, splits=Some(8))

== Analyzed Logical Plan ==
id: bigint, newId: bigint
Project [id#0L, id#0L AS newId#16L]
+- Range (0, 1, step=1, splits=Some(8))

== Optimized Logical Plan ==
Project [id#0L, id#0L AS newId#16L]
+- InMemoryRelation [id#0L], true, 10000, StorageLevel(disk, memory, deserialized, 1 replicas)
      +- *Range (0, 1, step=1, splits=Some(8))

== Physical Plan ==
*Project [id#0L, id#0L AS newId#16L]
+- InMemoryTableScan [id#0L]
      +- InMemoryRelation [id#0L], true, 10000, StorageLevel(disk, memory, deserialized, 1 replicas)
            +- *Range (0, 1, step=1, splits=Some(8))

// Clear in-memory cache using SQL
// Equivalent to spark.catalog.clearCache
scala> sql("CLEAR CACHE").collect
res1: Array[org.apache.spark.sql.Row] = Array()

// Visit http://localhost:4040/storage to confirm the cleaning
```

### Caching Dataset — `cache`Method {#__a_id_cache_a_caching_dataset_code_cache_code_method}

```
cache(): this.type
```

缓存只执行无参数持久方法。

```
val ds = spark.range(5).cache
```

### Persisting Dataset — `persist`Methods {#__a_id_persist_a_persisting_dataset_code_persist_code_methods}

```
persist(): this.type
persist(newLevel: StorageLevel): this.type
```

持久化使用默认存储级别MEMORY\_AND\_DISK或newLevel高速缓存数据集并返回它。

在内部，persist请求CacheManager缓存查询（可以通过当前SparkSession的SharedState访问）。

| Caution | FIXME |
| :--- | :--- |


### Unpersisting Dataset — `unpersist`Method {#__a_id_unpersist_a_unpersisting_dataset_code_unpersist_code_method}

```
unpersist(blocking: Boolean): this.type
```

unpersist可能通过阻止调用来高速缓存数据集。

在内部，unpersist请求CacheManager重新缓存查询。









