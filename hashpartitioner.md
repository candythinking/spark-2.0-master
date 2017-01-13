## HashPartitioner {#__a_id_hashpartitioner_a_hashpartitioner}

HashPartitioner 是一个分区程序，它使用分区可配置的分区数来对数据进行洗牌（shuffle）。

Table 1.`HashPartitioner`Attributes and Method

| Property | Description |
| :--- | :--- |
| `numPartitions` | Exactly`partitions`number of partitions |
| `getPartition` | 0用于 null keys，Java 的 Object.hashCode 用于 non-null keys（模数分区数为分区数，或为负数散列为0）。 |
| `equals` | `true`for`HashPartitioner`s with`partitions`number of partitions. Otherwise,`false`. |
| `hashCode` | Exactly`partitions`number of partitions |

| Note | `HashPartitioner`is the default`Partitioner`for coalesce transformation with`shuffle`enabled, e.g. calling repartition. |
| :--- | :--- |


尽管键值 k 的所有记录已经在单个 Spark 执行器上（即，块管理器是精确的），但是可以重新洗牌数据。当 k1 的 HashPartitioner 的结果为3时，键值 k1 将转到第三个执行器。







