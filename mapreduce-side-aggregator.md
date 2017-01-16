## Map/Reduce-side Aggregator {#__a_id_aggregator_a_map_reduce_side_aggregator}

`Aggregator`是用于聚合分布式数据集的一组函数：

```
createCombiner: V => C
mergeValue: (C, V) => C
mergeCombiners: (C, C) => C
```

| Note | 聚合器在 combineByKeyWithClassTag transformations 中创建，以创建 ShuffledRDD，并最终传递到 ShuffleDependency。它也用于 ExternalSorter。 |
| :---: | :--- |


### `updateMetrics`Internal Method {#__a_id_updatemetrics_a_code_updatemetrics_code_internal_method}

| Caution | FIXME |
| :--- | :--- |


### `combineValuesByKey`Method {#__a_id_combinevaluesbykey_a_code_combinevaluesbykey_code_method}

| Caution | FIXME |
| :--- | :--- |


### `combineCombinersByKey`Method {#__a_id_combinecombinersbykey_a_code_combinecombinersbykey_code_method}

| Caution | FIXME |
| :--- | :--- |


















