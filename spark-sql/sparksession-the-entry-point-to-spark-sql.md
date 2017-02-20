## `SparkSession` — The Entry Point to Spark SQL {#__a_id_sparksession_a_code_sparksession_code_the_entry_point_to_spark_sql}

SparkSession是Spark SQL的入口点。这是您必须创建的第一个对象，以使用完全类型化的Dataset（和无类型的DataFrame）数据抽象开始开发Spark SQL应用程序。

| Note | SparkSession在Spark 2.0.0中的一个对象中合并了SQLContext和HiveContext。 |
| :---: | :--- |


您可以使用SparkSession.builder方法创建SparkSession的实例。





























