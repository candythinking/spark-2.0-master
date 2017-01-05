## `SparkSubmitOptionParser` — spark-submit’s Command-Line Parser {#__a_id_sparksubmitoptionparser_a_code_sparksubmitoptionparser_code_spark_submit_s_command_line_parser}

SparkSubmitOptionParser 是 spark-submit 的命令行选项的解析器。

表1. spark-submit 命令行选项

| Command-Line Option | Description |
| :--- | :--- |
| `--archives` |  |
| `--class` | 主类运行（作为 mainClass 的内部属性）。 |
| `--conf [prop=value]`or`-c [prop=value]` | All =分隔的值在conf中可能覆盖现有设置。命令行命令。 |
| `--deploy-mode` | deployMode内部属性 |
| `--driver-class-path` | spark.driver.extraClassPath in conf - 驱动程序类路径 |
| `--driver-cores` |  |
| `--driver-java-options` | `spark.driver.extraJavaOptions`in`conf` — the driver VM options |
| `--driver-library-path` | `spark.driver.extraLibraryPath`in`conf` — the driver native library path |
| `--driver-memory` | `spark.driver.memory`in`conf` |
| `--exclude-packages` |  |
| `--executor-cores` |  |
| `--executor-memory` |  |
| `--files` |  |
| `--help`or`-h` | The option is added to`sparkArgs` |
| `--jars` |  |
| `--keytab` |  |
| `--kill` | The option and a value are added to`sparkArgs` |
| `--master` | `master`internal property |
| `--name` |  |
| `--num-executors` |  |
| `--packages` |  |
| `--principal` |  |
| `--properties-file [FILE]` | `propertiesFile`internal property. Refer to Custom Spark Properties File — `--properties-file`command-line option. |
| `--proxy-user` |  |
| `--py-files` |  |
| `--queue` |  |
| `--repositories` |  |
| `--status` | The option and a value are added to`sparkArgs` |
| `--supervise` |  |
| `--total-executor-cores` |  |
| `--usage-error` | The option is added to`sparkArgs` |
| `--verbose`or`-v` |  |
| `--version` | The option is added to`sparkArgs` |

### SparkSubmitOptionParser Callbacks {#__a_id_callbacks_a_sparksubmitoptionparser_callbacks}

SparkSubmitOptionParser 应该被覆盖以下功能（作为回调）。

表2.回调

| Callback | Description |
| :--- | :--- |
| `handle` | Executed when an option with an argument is parsed. |
| `handleUnknown` | Executed when an unrecognized option is parsed. |
| `handleExtraArgs` | Executed for the command-line arguments that`handle`and`handleUnknown`callbacks have not processed.+ |

`SparkSubmitOptionParser`belongs to`org.apache.spark.launcher`Scala package and`spark-launcher`Maven/sbt module.

| Note | `org.apache.spark.launcher.SparkSubmitArgumentsParser`is a custom`SparkSubmitOptionParser`. |
| :---: | :--- |


### Parsing Command-Line Arguments — `parse`Method {#__a_id_parse_a_parsing_command_line_arguments_code_parse_code_method}

```
final void parse(List<String> args)
```

`parse`解析命令行参数列表。

每当发现一个已知的命令行选项或开关（没有参数的命令行选项）时，`parse`调用`handle`回调。它为无法识别的命令行选项调用 handleUnknown 回调。

parse 会继续处理命令行参数，直到 handle 或 knownUnknown 回调返回 false 或所有命令行参数都已消耗。

最终，`parse`调用`handleExtraArgs`回调。







