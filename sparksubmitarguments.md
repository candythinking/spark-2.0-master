## `SparkSubmitArguments` — spark-submit’s Command-Line  {#__a_id_sparksubmitarguments_a_code_sparksubmitarguments_code_spark_submit_s_command_line_argument_parser}

## Argument Parser {#__a_id_sparksubmitarguments_a_code_sparksubmitarguments_code_spark_submit_s_command_line_argument_parser}

SparkSubmitArguments 是一个自定义 SparkSubmitArgumentsParser，用于处理动作（即提交，杀死和状态）用于执行（可能使用显式 env 环境）的 spark-submit 脚本的命令行参数。

| Note | SparkSubmitArguments 在启动 spark-submit 脚本时创建，只有 args 被传入，并且稍后用于以详细模式打印参数。 |
| :---: | :--- |


### Calculating Spark Properties — `loadEnvironmentArguments`internal method {#__a_id_loadenvironmentarguments_a_calculating_spark_properties_code_loadenvironmentarguments_code_internal_method}

```
loadEnvironmentArguments(): Unit
```

loadEnvironmentArguments 计算当前执行 spark-submit 的 Spark 属性。 

loadEnvironmentArguments 读取命令行选项，然后首先跟踪 Spark 属性和系统的环境变量。

| Note | Spark 配置属性以 spark 开头。前缀并且可以使用 --conf \[key = value\] 命令行选项设置。 |
| :---: | :--- |


### `handle`Method {#__a_id_handle_a_code_handle_code_method}

```
protected def handle(opt: String, value: String): Boolean
```

`handle`解析输入 opt 参数，并返回 true 或当发现未知的 opt 时抛出 IllegalArgumentException。

handle 设置表命令行选项，Spark 属性和环境变量中的内部属性。

### `mergeDefaultSparkProperties`Internal Method {#__a_id_mergedefaultsparkproperties_a_code_mergedefaultsparkproperties_code_internal_method}

```
mergeDefaultSparkProperties(): Unit
```

mergeDefaultSparkProperties 将 Spark 属性从默认 Spark 属性文件（即 spark-defaults.conf）与通过 --conf 命令行选项指定的属性合并。

