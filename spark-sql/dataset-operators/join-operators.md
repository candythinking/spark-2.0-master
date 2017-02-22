## Join Operators {#_join_operators}

Table 1. Joins

| SQL | JoinType | Alias / joinType |
| :--- | :--- | :--- |
| `CROSS` | `Cross` | `cross` |
| `INNER` | `Inner` | `inner` |
| `FULL OUTER` | `FullOuter` | `outer`,`full`,`fullouter` |
| `LEFT ANTI` | `LeftAnti` | `leftanti` |
| `LEFT OUTER` | `LeftOuter` | `leftouter`,`left` |
| `LEFT SEMI` | `LeftSemi` | `leftsemi` |
| `RIGHT OUTER` | `RightOuter` | `rightouter`,`right` |
| `NATURAL` | `NaturalJoin` | Special case for`Inner`,`LeftOuter`,`RightOuter`,`FullOuter` |
| `USING` | `UsingJoin` | Special case for`Inner`,`LeftOuter`,`LeftSemi`,`RightOuter`,`FullOuter`,`LeftAnti` |

| Tip | 别名不区分大小写，可以在任何位置使用下划线（\_），即left\_anti和LEFT\_ANTI是可以接受的。 |
| :---: | :--- |


您可以将连接表达式用作连接运算符的一部分，或者将其忽略并使用where运算符进行描述。

```
df1.join(df2, $"df1Key" === $"df2Key")
df1.join(df2).where($"df1Key" === $"df2Key")
```

### `join`Methods {#__a_id_join_a_code_join_code_methods}

```
join(right: Dataset[_]): DataFrame (1)
join(right: Dataset[_], usingColumn: String): DataFrame (2)
join(right: Dataset[_], usingColumns: Seq[String]): DataFrame (3)
join(right: Dataset[_], usingColumns: Seq[String], joinType: String): DataFrame (4)
join(right: Dataset[_], joinExprs: Column): DataFrame (5)
join(right: Dataset[_], joinExprs: Column, joinType: String): DataFrame (6)
```

1. Inner join

2. Inner join

3. Inner join

4. Equi-join with explicit join type

5. Inner join

6. Join with explicit join type+

`join`joins two`Dataset`s.

```
val left = Seq((0, "zero"), (1, "one")).toDF("id", "left")
val right = Seq((0, "zero"), (2, "two"), (3, "three")).toDF("id", "right")

// Inner join
scala> left.join(right, "id").show
+---+----+-----+
| id|left|right|
+---+----+-----+
|  0|zero| zero|
+---+----+-----+

scala> left.join(right, "id").explain
== Physical Plan ==
*Project [id#50, left#51, right#61]
+- *BroadcastHashJoin [id#50], [id#60], Inner, BuildRight
   :- LocalTableScan [id#50, left#51]
   +- BroadcastExchange HashedRelationBroadcastMode(List(cast(input[0, int, false] as bigint)))
      +- LocalTableScan [id#60, right#61]

// Full outer
scala> left.join(right, Seq("id"), "fullouter").show
+---+----+-----+
| id|left|right|
+---+----+-----+
|  1| one| null|
|  3|null|three|
|  2|null|  two|
|  0|zero| zero|
+---+----+-----+

scala> left.join(right, Seq("id"), "fullouter").explain
== Physical Plan ==
*Project [coalesce(id#50, id#60) AS id#85, left#51, right#61]
+- SortMergeJoin [id#50], [id#60], FullOuter
   :- *Sort [id#50 ASC NULLS FIRST], false, 0
   :  +- Exchange hashpartitioning(id#50, 200)
   :     +- LocalTableScan [id#50, left#51]
   +- *Sort [id#60 ASC NULLS FIRST], false, 0
      +- Exchange hashpartitioning(id#60, 200)
         +- LocalTableScan [id#60, right#61]

// Left anti
scala> left.join(right, Seq("id"), "leftanti").show
+---+----+
| id|left|
+---+----+
|  1| one|
+---+----+

scala> left.join(right, Seq("id"), "leftanti").explain
== Physical Plan ==
*BroadcastHashJoin [id#50], [id#60], LeftAnti, BuildRight
:- LocalTableScan [id#50, left#51]
+- BroadcastExchange HashedRelationBroadcastMode(List(cast(input[0, int, false] as bigint)))
   +- LocalTableScan [id#60]
```

在内部，join使用Join逻辑运算符（在当前SparkSession中）创建一个DataFrame。

### `joinWith`Method {#__a_id_joinwith_a_code_joinwith_code_method}

```
joinWith[U](other: Dataset[U], condition: Column): Dataset[(T, U)]  (1)
joinWith[U](other: Dataset[U], condition: Column, joinType: String): Dataset[(T, U)]
```

1. Inner join

| Caution | FIXME |
| :--- | :--- |


### `crossJoin`Method {#__a_id_crossjoin_a_code_crossjoin_code_method}

```
crossJoin(right: Dataset[_]): DataFrame
```

| Caution | FIXME |
| :--- | :--- |


### Broadcast Join \(aka Map-Side Join\) {#__a_id_broadcast_join_a_broadcast_join_aka_map_side_join}

| Caution | FIXME: Review`BroadcastNestedLoop`. |
| :--- | :--- |


在连接运算符中使用时，可以使用广播函数来标记要广播的数据集。

| Note | 根据Spark中的Map-Side Join，广播连接也称为复制连接（在分布式系统社区中）或映射端连接（在Hadoop社区中）。 |
| :---: | :--- |


| Note | 最后！我一直想知道什么是地图侧的连接，它似乎我接近揭露真相！ |
| :---: | :--- |


稍后在Spark中的Map-Side Join中，您可以发现，通过广播连接，您可以非常有效地连接具有相对较小的表（维度）的大型表（事实），即执行星型模式连接避免通过网络发送大表的所有数据。

CanBroadcast对象匹配LogicalPlan，输出小到足以广播加入。

| Note | 目前仅支持Hive Metastore表的统计信息，其中已运行命令ANALYZE TABLE \[tableName\] COMPUTE STATISTICS noscan。 |
| :---: | :--- |


它使用spark.sql.autoBroadcastJoinThreshold设置来控制将在执行连接时向所有工作节点广播的表的大小。











