## DataStreamReader

DataStreamReader是用于从具有指定格式，模式和选项的数据源读取DataFrame中的流式数据的接口。

DataStreamReader提供对内置格式的支持：json，csv，parquet，text。 parquet格式是使用spark.sql.sources.default配置的默认数据源设置。

DataStreamReader可以使用SparkSession.readStream方法。

```
val spark: SparkSession = ...
val schema = spark.read
.format("csv")
.option("header", true)
.option("inferSchema", true)
.load("csv-logs/*.csv")
.schema
val df = spark.readStream
.format("csv")
.schema(schema)
.load("csv-logs/*.csv")
```

## format

```
format(source: String): DataStreamReader
```

format指定流式数据源的源格式。

## schema

```
schema(schema: StructType): DataStreamReader
```

模式\(schema\)指定流式数据源的模式。

## option Methods

```
option(key: String, value: String): DataStreamReader
option(key: String, value: Boolean): DataStreamReader
option(key: String, value: Long): DataStreamReader
option(key: String, value: Double): DataStreamReader
```

选项族方法为流式数据源指定其他选项。

为了用户方便，支持String，Boolean，Long和Double类型的值，并在内部转换为String类型。

| Note | 您还可以使用options方法批量设置选项。你必须自己做类型转换。 |
| :---: | :--- |


## options

```
options(options: scala.collection.Map[String, String]): DataStreamReader
```

options方法允许指定流输入数据源的一个或多个选项。

| Note | 您也可以使用选项方法逐个设置选项。 |
| :---: | :--- |


## load Methods

```
load(): DataFrame
load(path: String): DataFrame (1)
```

在将调用传递给load（）之前指定路径选项

load 加载流输入数据作为DataFrame。

在内部，load从当前SparkSession和StreamingRelation（基于模式，格式和选项的DataSource）创建一个DataFrame。

## Built-in Formats

```
json(path: String): DataFrame
csv(path: String): DataFrame
parquet(path: String): DataFrame
text(path: String): DataFrame
```

DataStreamReader可以从以下格式的数据源加载流式数据：

* json
* csv
* parquet
* text

这些方法简单地将调用传递给格式，后跟load（path）







