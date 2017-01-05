## `SparkSubmitCommandBuilder`Command Builder {#__a_id_sparksubmitcommandbuilder_a_code_sparksubmitcommandbuilder_code_command_builder}

SparkSubmitCommandBuilder 用于构建一个 spark-submit 和 SparkLauncher 用来启动 Spark 应用程序的命令。

SparkSubmitCommandBuilder 使用第一个参数来区分shell：

1. `pyspark-shell-main`

2. `sparkr-shell-main`

3. `run-example`

| Caution | Describe`run-example` |
| :--- | :--- |


SparkSubmitCommandBuilder 使用 OptionParser（这是一个 SparkSubmitOptionParser）解析命令行参数。 OptionParser 自带了以下方法：

1. `handle`来处理已知的选项（见下表）。它设置 master，deployMode，propertiesFile，conf，mainClass，sparkArgs 的内部属性。

2. `handleUnknown`可处理通常导致无法识别的选项错误消息的无法识别的选项。

3. `handleExtraArgs`用于处理被视为 Spark 应用程序参数的额外参数。

| Note | 对于 spark-shell，它假定应用程序参数在 spark-submit 的参数之后。 |
| :--- | :--- |


### `SparkSubmitCommandBuilder.buildCommand`/`buildSparkSubmitCommand` {#__a_id_buildcommand_a_code_sparksubmitcommandbuilder_buildcommand_code_code_buildsparksubmitcommand_code}

```
public List<String> buildCommand(Map<String, String> env)
```

| Note | buildCommand 是 AbstractCommandBuilder 公共 API 的一部分。 |
| :---: | :--- |


SparkSubmitCommandBuilder.buildCommand 只是将调用传递给 buildSparkSubmitCommand 私有方法（除非它对 pyspark 或 sparkr 脚本执行，我们在本文档中不感兴趣）。

#### `buildSparkSubmitCommand`Internal Method {#__a_id_buildsparksubmitcommand_a_code_buildsparksubmitcommand_code_internal_method}

```
private List<String> buildSparkSubmitCommand(Map<String, String> env)
```

buildSparkSubmitCommand 通过构建所谓的有效配置启动。在客户端模式下，

uildSparkSubmitCommand 将 spark.driver.extraClassPath 添加到结果 Spark 命令。

| Note | 使用 spark-submit 使 spark.driver.extraClassPath 生效。 |
| :---: | :--- |


buildSparkSubmitCommand 构建 Java 命令的第一部分，传递额外的类路径（仅适用于客户端部署模式）。

| Caution | 添加 isThriftServer 案例。 |
| :---: | :--- |


buildSparkSubmitCommand 附加了 SPARK\_SUBMIT\_OPTS 和 SPARK\_JAVA\_OPTS 环境变量。 （仅适用于客户端部署模式）...

| Caution | 详细说明客户端 deply 模式的情况。 |
| :---: | :--- |


`addPermGenSizeOpt`case…​elaborate

| Caution | 详细说明 addPermGenSizeOpt |
| :---: | :--- |


buildSparkSubmitCommand 附加 org.apache.spark.deploy.SparkSubmit 和命令行参数（使用 buildSparkSubmitArgs）。

#### `buildSparkSubmitArgs`method {#__a_id_buildsparksubmitargs_a_code_buildsparksubmitargs_code_method}

```
List<String> buildSparkSubmitArgs()
```

buildSparkSubmitArgs 构建 spark-submit 的命令行参数列表。

buildSparkSubmitArgs 使用 SparkSubmitOptionParser 添加 spark-submit 可识别的命令行参数（稍后执行它时，使用完全相同的 SparkSubmitOptionParser 解析器来解析命令行参数）。

表1. SparkSubmitCommandBuilder 属性和对应的 SparkSubmitOptionParser 属性

| `SparkSubmitCommandBuilder `Property | `SparkSubmitOptionParser`Attribute |
| :--- | :--- |
| `verbose` | `VERBOSE` |
| `master` | `MASTER [master]` |
| `deployMode` | `DEPLOY_MODE [deployMode]` |
| `appName` | `NAME [appName]` |
| `conf` | `CONF [key=value]*` |
| `propertiesFile` | `PROPERTIES_FILE [propertiesFile]` |
| `jars` | `JARS [comma-separated jars]` |
| `files` | `FILES [comma-separated files]` |
| `pyFiles` | `PY_FILES [comma-separated pyFiles]` |
| `mainClass` | `CLASS [mainClass]` |
| `sparkArgs` | `sparkArgs`\(passed straight through\) |
| `appResource` | `appResource`\(passed straight through\) |
| `appArgs` | `appArgs`\(passed straight through\) |

#### `getEffectiveConfig`Internal Method {#__a_id_geteffectiveconfig_a_code_geteffectiveconfig_code_internal_method}

```
Map<String, String> getEffectiveConfig()
```

getEffectiveConfig 内部方法构建与安装的 Spark 属性 conf 配合的 effectiveConfig（使用 loadPropertiesFile 内部方法）跳过已经加载的键（当在 handle 方法中解析命令行选项时）。

| Note | 命令行选项（例如 --driver-class-path）的优先级高于 Spark 属性文件中的相应 Spark 设置（例如 spark.driver.extraClassPath）。因此，您可以通过使用命令行选项在命令行上覆盖 Spark 设置来控制最终设置。字符集并修剪值周围的空格。 |
| :---: | :--- |


#### `isClientMode`Internal Method {#__a_id_isclientmode_a_code_isclientmode_code_internal_method}

```
private boolean isClientMode(Map<String, String> userProps)
```

isClientMode 首先检查主（从命令行选项）然后 spark.master Spark 属性。与 deployMode 和 spark.submit.deployMode 相同。

| Caution | 查看 master 和 deployMode。它们是如何设置的？ |
| :---: | :--- |


当没有明确设置显式主控和客户端部署模式时，isClientMode 响应为正。

### OptionParser {#__a_id_optionparser_a_optionparser}

OptionParser 是 SparkSubmitCommandBuilder 用于解析命令行参数的自定义 SparkSubmitOptionParser。它定义所有 SparkSubmitOptionParser 回调，即 handle，handleUnknown 和 handleExtraArgs，用于命令行参数处理。

#### OptionParser’s`handle`Callback {#__a_id_optionparser_handle_a_optionparser_s_code_handle_code_callback}

```
boolean handle(String opt, String value)
```

OptionParser 提供了一个自定义句柄回调（来自 SparkSubmitOptionParser 回调）

表2. handle方法

| Command-Line Option | Property / Behaviour |
| :--- | :--- |
| `--master` | `master` |
| `--deploy-mode` | `deployMode` |
| `--properties-file` | `propertiesFile` |
| `--driver-memory` | Sets`spark.driver.memory`\(in`conf`\) |
| `--driver-java-options` | Sets`spark.driver.extraJavaOptions`\(in`conf`\) |
| `--driver-library-path` | Sets`spark.driver.extraLibraryPath`\(in`conf`\) |
| `--driver-class-path` | Sets`spark.driver.extraClassPath`\(in`conf`\) |
| `--conf` | Expects a`key=value`pair that it puts in`conf` |
| `--class` | Sets`mainClass`\(in`conf`\).It may also set`allowsMixedArguments`and`appResource`if the execution is for one of the special classes, i.e.[spark-shell](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/spark-shell.html),`SparkSQLCLIDriver`, or[HiveThriftServer2](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/spark-sql-thrift-server.html). |
| `--kill`\|`--status` | Disables`isAppResourceReq`and adds itself with the value to`sparkArgs`. |
| `--help`\|`--usage-error` | Disables`isAppResourceReq`and adds itself to`sparkArgs`. |
| `--version` | Disables`isAppResourceReq`and adds itself to`sparkArgs`.+ |
| _anything else_ | Adds an element to`sparkArgs` |

#### OptionParser’s`handleUnknown`Method {#__a_id_optionparser_handleunknown_a_optionparser_s_code_handleunknown_code_method}

```
boolean handleUnknown(String opt)
```

如果 allowedMixedArguments 被启用，handleUnknown 只是简单地将输入 opt 添加到 appArgs，并允许进一步解析参数列表。

| Caution | 在哪里允许 MixedArguments 启用？ |
| :---: | :--- |


如果 isExample 被启用，handleUnknown 将 mainClass 设置为 org.apache.spark.examples。\[opt\]（除非输入 opt 已经有包前缀），并停止进一步解析参数列表。 

| Caution | Where’s`isExample`enabled? |
| :--- | :--- |


否则，handleUnknown 会设置 appResource 并停止进一步解析参数列表。

#### OptionParser’s`handleExtraArgs`Method {#__a_id_optionparser_handleextraargs_a_optionparser_s_code_handleextraargs_code_method}

```
void handleExtraArgs(List<String> extra)
```

handleExtraArgs 将所有额外的参数添加到 appArgs。

