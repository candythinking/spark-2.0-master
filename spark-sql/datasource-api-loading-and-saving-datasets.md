## DataSource API — Loading and Saving Datasets {#_datasource_api_loading_and_saving_datasets}

### Reading Datasets {#__a_id_reading_datasets_a_reading_datasets}

Spark SQL可以通过DataFrameReader接口从外部存储系统（如文件，Hive表和JDBC数据库）读取数据。

您使用SparkSession使用读取操作访问DataFrameReader。

```
import org.apache.spark.sql.SparkSession
val spark = SparkSession.builder.getOrCreate

val reader = spark.read
```

DataFrameReader是一个从文件，Hive表或JDBC创建DataFrames（也称为Dataset \[Row\]）的接口。

```
val people = reader.csv("people.csv")
val cities = reader.format("json").load("cities.json")
```

从Spark 2.0开始，DataFrameReader可以使用返回Dataset \[String\]（而不是DataFrames）的textFile方法读取文本文件。

```
spark.read.textFile("README.md")
```

您还可以定义自己的自定义文件格式。

```
val countries = reader.format("customFormat").load("countries.cf")
```

Spark SQL中有两种操作模式，即批处理和流式处理（称为结构化流式处理）。 您可以通过SparkSession.readStream方法访问DataStreamReader以读取流数据集。

```
import org.apache.spark.sql.streaming.DataStreamReader
val stream: DataStreamReader = spark.readStream
```

DataStreamReader中的可用方法与DataFrameReader类似。

### Saving Datasets {#__a_id_saving_datasets_a_saving_datasets}

Spark SQL可以通过DataFrameWriter界面将数据保存到外部存储系统，如文件，Hive表和JDBC数据库。

您在数据集上使用写入方法来访问DataFrameWriter。

```
import org.apache.spark.sql.{DataFrameWriter, Dataset}
val ints: Dataset[Int] = (0 to 5).toDS

val writer: DataFrameWriter[Int] = ints.write
```

DataFrameWriter是一个接口，用于以批量方式将Datasets持久保存到外部存储系统。 

您可以通过Dataset.writeStream方法访问DataStreamWriter以写入流数据集。

```
val papers = spark.readStream.text("papers").as[String]

import org.apache.spark.sql.streaming.DataStreamWriter
val writer: DataStreamWriter[String] = papers.writeStream
```

DataStreamWriter中的可用方法与DataFrameWriter类似。

















