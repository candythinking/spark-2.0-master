## Spark Submit — `spark-submit`shell script {#_spark_submit_code_spark_submit_code_shell_script}

spark-submit shell 脚本允许您管理 Spark 应用程序。

您可以将 Spark 应用程序提交到 Spark 部署环境以执行，停止或请求 Spark 应用程序的状态。

您可以在Spark发行版的bin目录中找到spark-submit脚本。

```
$ ./bin/spark-submit
Usage: spark-submit [options] <app jar | python file> [app arguments]
Usage: spark-submit --kill [submission ID] --master [spark://...]
Usage: spark-submit --status [submission ID] --master [spark://...]
Usage: spark-submit run-example [options] example-class [example args]
...
```

执行时，sp​​ark-submit 脚本首先检查是否设置了 SPARK\_HOME 环境变量，如果没有设置，则将其设置为包含 bin / spark-submit shell 脚本的目录。然后它执行 spark 类 shell 脚本以运行 SparkSubmit 独立应用程序。

| Caution | 将 Cluster Manager 和 Deploy Mode 添加到下表（请参阅选项值） |
| :---: | :--- |


表1.命令行选项，Spark 属性和环境变量（来自 SparkSubmitArguments 的 loadEnvironmentArguments 和 handle）

| Command-Line Option | Spark Property | Environment Variable | Description | Internal Property |
| :--- | :--- | :--- | :--- | :--- |
| `action` |  |  | Defaults to`SUBMIT` |  |
| `--archives` |  |  |  | `archives` |
| `--conf` |  |  |  | `sparkProperties` |
| `--deploy-mode` | `spark.submit.deployMode` | `DEPLOY_MODE` | Deploy mode | `deployMode` |
| `--driver-class-path` | `spark.driver.extraClassPath` |  | The driver’s class path | `driverExtraClassPath` |
| `--driver-java-options` | `spark.driver.extraJavaOptions` |  | The driver’s JVM options | `driverExtraJavaOptions` |
| `--driver-library-path` | `spark.driver.extraLibraryPath` |  | The driver’s native library path | `driverExtraLibraryPath` |
| `--driver-memory` | `spark.driver.memory` | `SPARK_DRIVER_MEMORY` | The driver’s memory | `driverMemory` |
| `--driver-cores` | `spark.driver.cores` |  |  | `driverCores` |
| `--exclude-packages` | `spark.jars.excludes` |  |  | `packagesExclusions` |
| `--executor-cores` | `spark.executor.cores` | `SPARK_EXECUTOR_CORES` | The number of executor CPU cores | `executorCores` |
| `--executor-memory` | `spark.executor.memory` | `SPARK_EXECUTOR_MEMORY` | An executor’s memory | `executorMemory` |
| `--files` | `spark.files` |  |  | `files` |
| `ivyRepoPath` | `spark.jars.ivy` |  |  |  |
| `--jars` | `spark.jars` |  |  | `jars` |
| `--keytab` | `spark.yarn.keytab` |  |  | `keytab` |
| `--kill` |  |  | `submissionToKill`and`action`set to`KILL` |  |
| `--master` | `spark.master` | `MASTER` | Master URL. Defaults to`local[*]` | `master` |
| `--class` |  |  |  | `mainClass` |
| `--name` | `spark.app.name` | `SPARK_YARN_APP_NAME`\(YARN only\) | Uses`mainClass`or the directory off`primaryResource`when no other ways set it | `name` |
| `--num-executors` | `spark.executor.instances` |  |  | `numExecutors` |
| `--packages` | `spark.jars.packages` |  |  | `packages` |
| `--principal` | `spark.yarn.principal` |  |  | `principal` |
| `--properties-file` | `spark.yarn.principal` |  |  | `propertiesFile` |
| `--proxy-user` |  |  |  | `proxyUser` |
| `--py-files` |  |  |  | `pyFiles` |
| `--queue` |  |  |  | `queue` |
| `--repositories` |  |  |  | `repositories` |
| `--status` |  |  | `submissionToRequestStatusFor`and`action`set to`REQUEST_STATUS` |  |
| `--supervise` |  |  |  | `supervise` |
| `--total-executor-cores` | `spark.cores.max` |  |  | `totalExecutorCores` |
| `--verbose` |  |  |  | `verbose` |
| `--version` |  |  | `SparkSubmit.printVersionAndExit()` |  |
| `--help` |  |  | `printUsageAndExit(0)` |  |
| `--usage-error` |  |  | `printUsageAndExit(1)` |  |

| Tip | 设置 SPARK\_PRINT\_LAUNCH\_COMMAND 环境变量以将完整的 Spark 命令打印到控制台，例如                                                $ SPARK\_PRINT\_LAUNCH\_COMMAND=1 ./bin/spark-shell    Spark Command: /Library/Ja...                                                请参阅 Spark 脚本的打印启动命令（或 org.apache.spark.launcher.Main 独立应用程序，其中实际使用此环境变量）。   |
| :--- | :--- |


| Tip | 避免在 Scala 中使用 scala.App trait 作为 Spark 应用程序的主类，如 SPARK-4170 中所报告的运行 Scala 应用程序“扩展应用程序”时的关闭问题。                                                                 请参阅本文档中的执行 Main - runMain 内部方法。 |
| :--- | :--- |


### Preparing Submit Environment — `prepareSubmitEnvironment`Internal Method {#__a_id_preparesubmitenvironment_a_preparing_submit_environment_code_preparesubmitenvironment_code_internal_method}

```
prepareSubmitEnvironment(args: SparkSubmitArguments)
  : (Seq[String], Seq[String], Map[String, String], String)
```

prepareSubmitEnvironment 创建一个4元素元组，即（childArgs，childClasspath，sysProps，childMainClass）。

表2. prepareSubmitEnvironment 的四元素返回元组

| Element | Description |
| :--- | :--- |
| `childArgs` | Arguments |
| `childClasspath` | Classpath elements |
| `sysProps` | Spark properties |
| `childMainClass` | Main class |

`prepareSubmitEnvironment`uses`options`to…​

| Caution | FIXME |
| :--- | :--- |


| Note | `prepareSubmitEnvironment`is used in`SparkSubmit`object. |
| :--- | :--- |


| Tip | 使用 --verbose 命令行选项查看返回元组的元素。 |
| :--- | :--- |


### Custom Spark Properties File — `--properties-file`command-line option {#__a_id_properties_file_a_custom_spark_properties_file_code_properties_file_code_command_line_option}

```
--properties-file [FILE]
```

--properties-file 命令行选项设置 Spark 加载额外 Spark 属性的文件 FILE 的路径。

| Tip | Spark默认使用 conf / spark-defaults.conf。 |
| :--- | :--- |


### Driver Cores in Cluster Deploy Mode — `--driver-cores`command-line option {#__a_id_driver_cores_a_driver_cores_in_cluster_deploy_mode_code_driver_cores_code_command_line_option}

```
--driver-cores NUM
```

--driver-cores 命令行选项将集群部署模式下驱动程序的核心数设置为 NUM。

| Note | --driver-cores 开关仅适用于集群模式（对于独立，Mesos 和 YARN）。 |
| :--- | :--- |


| Note | 它对应于 spark.driver.cores 设置。 |
| :--- | :--- |


| Note | 它在详细模式下打印到标准错误输出。 |
| :--- | :--- |


### Additional JAR Files to Distribute — `--jars`command-line option {#__a_id_jars_a_additional_jar_files_to_distribute_code_jars_code_command_line_option}

```
--jars JARS
```

--jars 是一个以逗号分隔的本地 jar 的列表，包含在驱动程序和执行程序的类路径上。

| Caution | FIXME |
| :--- | :--- |


### Additional Files to Distribute`--files`command-line option {#__a_id_files_a_additional_files_to_distribute_code_files_code_command_line_option}

```
--files FILES
```

| Caution | FIXME |
| :--- | :--- |


### Additional Archives to Distribute — `--archives`command-line option {#__a_id_archives_a_additional_archives_to_distribute_code_archives_code_command_line_option}

```
--archives ARCHIVES
```

| Caution | FIXME |
| :--- | :--- |


### Specifying YARN Resource Queue — `--queue`command-line option {#__a_id_queue_a_specifying_yarn_resource_queue_code_queue_code_command_line_option}

```
--queue QUEUE_NAME
```

使用 --queue，您可以选择 YARN资源队列来提交 Spark 应用程序。默认队列名称为 default。

| Caution | What is a`queue`? |
| :--- | :--- |


| Note | 它对应 spark.yarn.queue Spark 的设置。 |
| :--- | :--- |


| Tip | 它在详细模式下打印到标准错误输出。 |
| :--- | :--- |


### Actions {#__a_id_actions_a_actions}

#### Submitting Applications for Execution — `submit`method {#__a_id_submit_a_submitting_applications_for_execution_code_submit_code_method}

spark-submit 脚本的默认操作是将 Spark 应用程序提交到部署环境以供执行。

| Tip | 使用 --verbose 命令行开关知道要执行的主类，参数，系统属性和类路径（以确保正确处理命令行参数和开关）。 |
| :--- | :--- |


When executed,`spark-submit`executes`submit`method.

```
submit(args: SparkSubmitArguments): Unit
```

If`proxyUser`is set it will…​

| Caution | Review why and when to use`proxyUser`. |
| :--- | :--- |


它将执行传递给 runMain。

##### Executing Main — `runMain`internal method {#__a_id_runmain_a_executing_main_code_runmain_code_internal_method}

```
runMain(
  childArgs: Seq[String],
  childClasspath: Seq[String],
  sysProps: Map[String, String],
  childMainClass: String,
  verbose: Boolean): Unit
```

runMain 是一个内部方法，用于构建执行环境并调用已提交执行的 Spark 应用程序的 main 方法。

| Note | 它在提交执行应用程序时专用。 |
| :---: | :--- |


当启用详细输入标志（即 true）时，runMain 打印所有输入参数，即 childMainClass，childArgs，sysProps 和 childClasspath（按此顺序）。

```
Main class:
[childMainClass]
Arguments:
[childArgs one per line]
System properties:
[sysProps one per line]
Classpath elements:
[childClasspath one per line]
```

| Note | 使用 spark-submit 的 --verbose 命令行选项可以启用详细标志。 |
| :---: | :--- |


runMain 根据 spark.driver.userClassPathFirst 标志构建上下文类加载器（作为加载器）。

| Caution | 描述 spark.driver.userClassPathFirst |
| :---: | :--- |


它将 childClasspath 输入参数中指定的 jar 添加到上下文类加载器（稍后负责加载 childMainClass 主类）。

| Note | 如果在客户端部署模式中指定，childClasspath 输入参数对应于具有主资源的 --jars 命令行选项。 |
| :---: | :--- |


它设置在 sysProps 输入参数中指定的所有系统属性（使用 Java 的 System.setProperty 方法）。

它创建一个 childMainClass 主类的实例（作为 mainClass）。

| Note | childMainClass 是 spark-submit 的主类被调用。 |
| :---: | :--- |


| Tip | 避免在 Scala 中使用 scala.App trait 作为 Spark 应用程序的主类，如运行 Scala 应用程序“扩展应用程序”时 SPARK-4170 关闭问题。 |
| :---: | :--- |


如果对主类使用 scala.App，则应在日志中看到以下警告消息：

```
Warning: Subclasses of scala.App may not work correctly. Use a main() method instead.
```

最后，runMain 执行 Spark 应用程序的 main 方法传递 childArgs 参数。

任何 SparkUserAppException 异常导致 System.exit，而其他的只是重新抛出。

##### Adding Local Jars to ClassLoader — `addJarToClasspath`internal method {#__a_id_addjartoclasspath_a_adding_local_jars_to_classloader_code_addjartoclasspath_code_internal_method}

```
addJarToClasspath(localJar: String, loader: MutableURLClassLoader)
```

addJarToClasspath 是一个内部方法，用于将文件或本地 jar（作为 localJar）添加到加载器类加载器。

在内部，addJarToClasspath 解析 localJar 的 URI。如果URI 是文件或本地，并且由 localJar 表示的文件存在，则将 localJar 添加到加载器。否则，将向日志中打印以下警告：

```
Warning: Local jar /path/to/fake.jar does not exist, skipping.
```

对于所有其他URI，会向日志中打印以下警告：

```
Warning: Skip remote jar hdfs://fake.jar.
```

| Note | 当 localJar 没有指定 URI 时，addJarToClasspath 假定文件URI。 /path/to/local.jar。 |
| :---: | :--- |


| Caution | 什么是 URI fragment？这个更改如何改变 YARN 分布式缓存？请参见 Utils＃resolveURI。 |
| :---: | :--- |


#### Killing Applications — `--kill`command-line option {#__a_id_kill_a_killing_applications_code_kill_code_command_line_option}

`--kill`

#### Requesting Application Status — `--status`command-line option {#__a_id_status_a_a_id_requeststatus_a_requesting_application_status_code_status_code_command_line_option}

`--status`

### Command-line Options {#__a_id_command_line_options_a_command_line_options}

执行spark-submit --help了解支持的命令行选项。

```
➜  spark git:(master) ✗ ./bin/spark-submit --help
Usage: spark-submit [options] <app jar | python file> [app arguments]
Usage: spark-submit --kill [submission ID] --master [spark://...]
Usage: spark-submit --status [submission ID] --master [spark://...]
Usage: spark-submit run-example [options] example-class [example args]

Options:
  --master MASTER_URL         spark://host:port, mesos://host:port, yarn, or local.
  --deploy-mode DEPLOY_MODE   Whether to launch the driver program locally ("client") or
                              on one of the worker machines inside the cluster ("cluster")
                              (Default: client).
  --class CLASS_NAME          Your application's main class (for Java / Scala apps).
  --name NAME                 A name of your application.
  --jars JARS                 Comma-separated list of local jars to include on the driver
                              and executor classpaths.
  --packages                  Comma-separated list of maven coordinates of jars to include
                              on the driver and executor classpaths. Will search the local
                              maven repo, then maven central and any additional remote
                              repositories given by --repositories. The format for the
                              coordinates should be groupId:artifactId:version.
  --exclude-packages          Comma-separated list of groupId:artifactId, to exclude while
                              resolving the dependencies provided in --packages to avoid
                              dependency conflicts.
  --repositories              Comma-separated list of additional remote repositories to
                              search for the maven coordinates given with --packages.
  --py-files PY_FILES         Comma-separated list of .zip, .egg, or .py files to place
                              on the PYTHONPATH for Python apps.
  --files FILES               Comma-separated list of files to be placed in the working
                              directory of each executor.

  --conf PROP=VALUE           Arbitrary Spark configuration property.
  --properties-file FILE      Path to a file from which to load extra properties. If not
                              specified, this will look for conf/spark-defaults.conf.

  --driver-memory MEM         Memory for driver (e.g. 1000M, 2G) (Default: 1024M).
  --driver-java-options       Extra Java options to pass to the driver.
  --driver-library-path       Extra library path entries to pass to the driver.
  --driver-class-path         Extra class path entries to pass to the driver. Note that
                              jars added with --jars are automatically included in the
                              classpath.

  --executor-memory MEM       Memory per executor (e.g. 1000M, 2G) (Default: 1G).

  --proxy-user NAME           User to impersonate when submitting the application.
                              This argument does not work with --principal / --keytab.

  --help, -h                  Show this help message and exit.
  --verbose, -v               Print additional debug output.
  --version,                  Print the version of current Spark.

 Spark standalone with cluster deploy mode only:
  --driver-cores NUM          Cores for driver (Default: 1).

 Spark standalone or Mesos with cluster deploy mode only:
  --supervise                 If given, restarts the driver on failure.
  --kill SUBMISSION_ID        If given, kills the driver specified.
  --status SUBMISSION_ID      If given, requests the status of the driver specified.

 Spark standalone and Mesos only:
  --total-executor-cores NUM  Total cores for all executors.

 Spark standalone and YARN only:
  --executor-cores NUM        Number of cores per executor. (Default: 1 in YARN mode,
                              or all available cores on the worker in standalone mode)

 YARN-only:
  --driver-cores NUM          Number of cores used by the driver, only in cluster mode
                              (Default: 1).
  --queue QUEUE_NAME          The YARN queue to submit to (Default: "default").
  --num-executors NUM         Number of executors to launch (Default: 2).
  --archives ARCHIVES         Comma separated list of archives to be extracted into the
                              working directory of each executor.
  --principal PRINCIPAL       Principal to be used to login to KDC, while running on
                              secure HDFS.
  --keytab KEYTAB             The full path to the file that contains the keytab for the
                              principal specified above. This keytab will be copied to
                              the node running the Application Master via the Secure
                              Distributed Cache, for renewing the login tickets and the
                              delegation tokens periodically.
```

* `--class`

* `--conf`or`-c`

* `--deploy-mode`\(see Deploy Mode\)

* `--driver-class-path`\(see`--driver-class-path`command-line option\)

* `--driver-cores`\(see Driver Cores in Cluster Deploy Mode\)

* `--driver-java-options`

* `--driver-library-path`

* `--driver-memory`

* `--executor-memory`

* `--files`

* `--jars`

* `--kill`for Standalone cluster mode only

* `--master`

* `--name`

* `--packages`

* `--exclude-packages`

* `--properties-file`\(see Custom Spark Properties File\)

* `--proxy-user`

* `--py-files`

* `--repositories`

* `--status`for Standalone cluster mode only

* `--total-executor-cores`

List of switches, i.e. command-line options that do not take parameters:

* `--help`or`-h`

* `--supervise`for Standalone cluster mode only

* `--usage-error`

* `--verbose`or`-v`\(see Verbose Mode\)

* `--version`\(see Version\)

YARN-only options:

* `--archives`

* `--executor-cores`

* `--keytab`

* `--num-executors`

* `--principal`

* `--queue`\(see Specifying YARN Resource Queue \(--queue switch\)\)

### `--driver-class-path`command-line option {#__a_id_driver_class_path_a_code_driver_class_path_code_command_line_option}

--driver-class-path 命令行选项设置应添加到驱动程序 JVM 的额外类路径条目（例如 jars 和目录）。

| Tip | 您应该在客户端部署模式（不是 SparkConf）中使用 --driver-class-path，以确保使用条目设置 CLASSPATH。客户端部署模式使用与 spark-submit 相同的 JVM 作为驱动程序。 |
| :---: | :--- |


--driver-class-path 设置内部 driverExtraClassPath 属性（当调用 SparkSubmitArguments.handle 时）。

它适用于所有集群管理器和部署模式。

如果未在命令行中设置 driverExtraClassPath，则使用 spark.driver.extraClassPath 设置。

| Note | 命令行选项（例如 --driver-class-path）的优先级高于 Spark 属性文件中的相应 Spark 设置（例如 spark.driver.extraClassPath）。因此，您可以通过使用命令行选项在命令行上覆盖 Spark 设置来控制最终设置。 |
| :---: | :--- |


表3. Spark 属性文件和命令行中的 Spark 设置

| Setting / System Property | Command-Line Option | Description |
| :---: | :---: | :---: |
| spark.driver.extraClassPath | --driver-class-path | 额外的类路径条目（例如 jars 和目录）传递到驱动程序的 JVM。 |

### Version — `--version`command-line option {#__a_id_version_a_version_code_version_code_command_line_option}

    $ ./bin/spark-submit --version
    Welcome to
          ____              __
         / __/__  ___ _____/ /__
        _\ \/ _ \/ _ `/ __/  '_/
       /___/ .__/\_,_/_/ /_/\_\   version 2.1.0-SNAPSHOT
          /_/

    Branch master
    Compiled by user jacek on 2016-09-30T07:08:39Z
    Revision 1fad5596885aab8b32d2307c0edecbae50d5bd7a
    Url https://github.com/apache/spark.git
    Type --help for more information.

### Verbose Mode — `--verbose`command-line option {#__a_id_verbose_mode_a_verbose_mode_code_verbose_code_command_line_option}

当使用 --verbose 命令行选项执行 spark-submit 时，它将进入详细模式。

在详细模式下，解析的参数将打印到系统错误输出。

它还打印出属性文件和文件中的属性。

### Deploy Mode — `--deploy-mode`command-line option {#__a_id_deploy_mode_a_deploy_mode_code_deploy_mode_code_command_line_option}

您可以使用 spark-submit 的 --deploy-mode 命令行选项来指定 Spark 应用程序的部署模式。

### Environment Variables {#__a_id_environment_variables_a_environment_variables}

以下是未指定命令行选项时考虑的环境变量的列表：

* `MASTER`for`--master`

* `SPARK_DRIVER_MEMORY`for`--driver-memory`

* `SPARK_EXECUTOR_MEMORY`\(see Environment Variables in the SparkContext document\)

* `SPARK_EXECUTOR_CORES`

* `DEPLOY_MODE`

* `SPARK_YARN_APP_NAME`

* `_SPARK_CMD_USAGE`

### External packages and custom repositories {#_external_packages_and_custom_repositories}

spark-submit 实用程序支持使用 --packages 和自定义存储库使用 --repositories 使用 Maven 坐标指定外部包。

```
./bin/spark-submit \
  --packages my:awesome:package \
  --repositories s3n://$aws_ak:$aws_sak@bucket/path/to/repo
```

### `SparkSubmit`Standalone Application — `main`method {#__a_id_main_a_code_sparksubmit_code_standalone_application_code_main_code_method}

| Tip | 脚本的源代码位于 https://github.com/apache/spark/blob/master/bin/spark-submit。 |
| :---: | :--- |


执行时，sp​​ark-submit 脚本将调用传递给 spark-class，org.apache.spark.deploy.SparkSubmit 类后跟命令行参数。

| Tip | spark-class 使用类名 - org.apache.spark.deploy.SparkSubmit - 来适当地解析命令行参数。 请参阅 org.apache.spark.launcher.Main 独立应用程序 |
| :---: | :--- |


它创建一个SparkSubmitArguments的实例。

如果在详细模式\(verbose mode\)下，它打印出应用程序参数。

然后它将执行中继到操作特定的内部方法（使用应用程序参数）：

* When no action was explicitly given, it is assumed submit action.

* kill\(when`--kill`switch is used\)

* requestStatus\(when`--status`switch is used\)

| Note | 该操作只能有三个可用值之一：SUBMIT，KILL 或 REQUEST\_STATUS。 |
| :--- | :--- |


#### spark-env.sh - load additional environment settings {#__a_id_sparkenv_a_spark_env_sh_load_additional_environment_settings}

* `spark-env.sh`包括为您的站点配置 Spark 的环境设置。

  ```
  export JAVA_HOME=/your/directory/java
  export HADOOP_HOME=/usr/lib/hadoop
  export SPARK_WORKER_CORES=2
  export SPARK_WORKER_MEMORY=1G
  ```

* `spark-env.sh`在 Spark 的命令行脚本启动时加载。

* `SPARK_ENV_LOADED` env var是确保spark-env.sh脚本加载一次。

* `SPARK_CONF_DIR`指向使用 spark-env.sh 或 $ SPARK\_HOME / conf 的目录。

* `spark-env.sh`is executed if it exists.

* `$SPARK_HOME/conf`目录具有 spark-env.sh.template 文件，用作您自己的自定义配置的模板。

在官方文档中查阅环境变量。

