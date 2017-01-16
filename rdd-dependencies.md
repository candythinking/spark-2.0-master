## RDD Dependencies {#__a_id_dependency_a_rdd_dependencies}

`Dependency`类是用于建模两个或多个 RDD 之间的依赖关系的基础（抽象）类。

`Dependency`关系具有单个方法 rdd 来访问在依赖之后的 RDD。

```
def rdd: RDD[T]
```

每当您将 transformation（例如，map，flatMap）应用于 RDD 时，您将构建所谓的 RDD 沿袭血统图。依赖性表示沿袭血统图中的边。

| Note | NarrowDependency 和 ShuffleDependency 是 Dependency 抽象类的两个顶级子类。 |
| :---: | :--- |


Table 1. Kinds of Dependencies

| Name | Description |
| :--- | :--- |
| NarrowDependency |  |
| ShuffleDependency |  |
| OneToOneDependency |  |
| PruneDependency |  |
| RangeDependency |  |

RDD 的依赖关系可以使用依赖关系方法。

```
// A demo RDD
scala> val myRdd = sc.parallelize(0 to 9).groupBy(_ % 2)
myRdd: org.apache.spark.rdd.RDD[(Int, Iterable[Int])] = ShuffledRDD[8] at groupBy at <console>:24

scala> myRdd.foreach(println)
(0,CompactBuffer(0, 2, 4, 6, 8))
(1,CompactBuffer(1, 3, 5, 7, 9))

scala> myRdd.dependencies
res5: Seq[org.apache.spark.Dependency[_]] = List(org.apache.spark.ShuffleDependency@27ace619)

// Access all RDDs in the demo RDD lineage
scala> myRdd.dependencies.map(_.rdd).foreach(println)
MapPartitionsRDD[7] at groupBy at <console>:24
```

您使用 toDebugString 方法以用户友好的方式打印 RDD 谱系血统。

```
scala> myRdd.toDebugString
res6: String =
(8) ShuffledRDD[8] at groupBy at <console>:24 []
 +-(8) MapPartitionsRDD[7] at groupBy at <console>:24 []
    |  ParallelCollectionRDD[6] at parallelize at <console>:24 []
```



