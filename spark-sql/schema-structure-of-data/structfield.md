## StructField {#_structfield}

StructField描述了StructType中的单个字段。它具有名称，类型以及是否为空，以及可选的元数据和注释。

注释是注释键下元数据的一部分，用于构建Hive列或描述表时。

```
scala> schemaTyped("a").getComment
res0: Option[String] = None

scala> schemaTyped("a").withComment("this is a comment").getComment
res1: Option[String] = Some(this is a comment)
```

























