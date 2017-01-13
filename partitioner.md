## Partitioner {#__a_id_partitioner_a_partitioner}

分区器在输出处捕获数据分布。调度器可以基于此优化未来的操作。

```
val partitioner: Option[Partitioner] specifies how the RDD is partitioned.
```

分区器的合同确保给定密钥的记录必须驻留在单个分区上。

### `numPartitions`Method {#__a_id_numpartitions_a_code_numpartitions_code_method}

| Caution | FIXME |
| :--- | :--- |


### `getPartition`Method {#__a_id_getpartition_a_code_getpartition_code_method}

| Caution | FIXME |
| :--- | :--- |










