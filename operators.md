## Operators - Transformations and Actions {#_operators_transformations_and_actions}

RDD有两种类型的操作：transformations 和 actions。

| Note | Operators are also called**operations**. |
| :--- | :--- |


### Gotchas - things to watch for {#_gotchas_things_to_watch_for}

即使你不明确地访问它，它也不能在闭包内被引用，因为它被序列化并且跨越执行器。

See [https://issues.apache.org/jira/browse/SPARK-5063](https://issues.apache.org/jira/browse/SPARK-5063)









