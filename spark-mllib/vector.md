## Vector {#_vector}

向量密封的trait表示值的数值向量（Double类型）和它们的索引（Int类型）。

它属于org.apache.spark.mllib.linalg包。

| Note | 对于Scala和Java开发人员： Spark MLlib中的矢量类属于org.apache.spark.mllib.linalg包。 它不是Scala或Java中的Vector类型。训练你的眼睛看到两种类型的同名。你被警告了。 |
| :---: | :--- |


Vector对象知道它的大小。

Vector对象可以转换为：

* Array \[Double\]使用toArray。

* 一个密集的向量作为DenseVector使用toDense。

* 一个稀疏矢量作为SparseVector使用toSparse。 

* （1.6.0）使用toJson的JSON字符串。 

* （内部）微风矢量作为BV \[双\]使用toBreeze。

有两个可用的Vector sealed trait实现（也属于org.apache.spark.mllib.linalg包）：

* `DenseVector`

* `SparseVector`

| Tip | Use`Vectors`factory object to create vectors, be it`DenseVector`or`SparseVector`. |
| :---: | :--- |


```
import org.apache.spark.mllib.linalg.Vectors

// You can create dense vectors explicitly by giving values per index
val denseVec = Vectors.dense(Array(0.0, 0.4, 0.3, 1.5))
val almostAllZeros = Vectors.dense(Array(0.0, 0.4, 0.3, 1.5, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0))

// You can however create a sparse vector by the size and non-zero elements
val sparse = Vectors.sparse(10, Seq((1, 0.4), (2, 0.3), (3, 1.5)))

// Convert a dense vector to a sparse one
val fromSparse = sparse.toDense

scala> almostAllZeros == fromSparse
res0: Boolean = true
```

| Note | 工厂对象称为Vectors（复数）。 |
| :---: | :--- |


```
import org.apache.spark.mllib.linalg._

// prepare elements for a sparse vector
// NOTE: It is more Scala rather than Spark
val indices = 0 to 4
val elements = indices.zip(Stream.continually(1.0))
val sv = Vectors.sparse(elements.size, elements)

// Notice how Vector is printed out
scala> sv
res4: org.apache.spark.mllib.linalg.Vector = (5,[0,1,2,3,4],[1.0,1.0,1.0,1.0,1.0])

scala> sv.size
res0: Int = 5

scala> sv.toArray
res1: Array[Double] = Array(1.0, 1.0, 1.0, 1.0, 1.0)

scala> sv == sv.copy
res2: Boolean = true

scala> sv.toJson
res3: String = {"type":0,"size":5,"indices":[0,1,2,3,4],"values":[1.0,1.0,1.0,1.0,1.0]}
```















