## Spark Properties and spark-defaults.conf Properties File {#_spark_properties_and_spark_defaults_conf_properties_file}

Spark 属性是调整 Spark 应用程序的执行环境的方法。

默认的 Spark 属性文件是 $ SPARK\_HOME / conf / spark-defaults.conf，可以使用 spark-submit 的 --properties-file 命令行选项覆盖。

Table 1. Environment Variables

| Environment Variable | Default Value | Description |
| :--- | :--- | :--- |
| `SPARK_CONF_DIR` | `${SPARK_HOME}/conf` | Spark’s configuration directory \(with`spark-defaults.conf`\) |

| Tip | Read the official documentation of Apache Spark on [Spark Configuration](http://spark.apache.org/docs/latest/configuration.html). |
| :--- | :--- |


### `spark-defaults.conf` — Default Spark Properties File {#__a_id_spark_defaults_conf_a_code_spark_defaults_conf_code_default_spark_properties_file}

spark-defaults.conf（在 SPARK\_CONF\_DIR 或 $ SPARK\_HOME / conf 下）是带有 Spark 应用程序的 Spark 属性的默认属性文件。

| Note | spark-defaults.conf 由 AbstractCommandBuilder 的 loadPropertiesFile 内部方法加载。 |
| :--- | :--- |


### Calculating Path of Default Spark Properties — `Utils.getDefaultPropertiesFile`method {#__a_id_getdefaultpropertiesfile_a_calculating_path_of_default_spark_properties_code_utils_getdefaultpropertiesfile_code_method}

```
getDefaultPropertiesFile(env: Map[String, String] = sys.env): String
```

getDefaultPropertiesFile 计算 spark-defaults.conf 属性文件的绝对路径，该文件可以位于由 SPARK\_CONF\_DIR 环境变量或 $ SPARK\_HOME / conf 目录指定的目录中。

| Note | `getDefaultPropertiesFile`is a part of`private[spark]org.apache.spark.util.Utils`object. |
| :--- | :--- |


### Environment Variables {#_environment_variables}



