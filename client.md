## Client {#_client}

客户端（包：org.apache.spark.deploy.yarn）是 YARN 集群部署 ApplicationMaster（用于部署到 YARN 集群的 Spark 应用程序）的句柄\(handle\)。

![](/img/mastering-apache-spark/spark on yarn/figure2.png)

根据部署模式，它使用 ApplicationMaster 或 ApplicationMaster 的包装器 ExecutorLauncher，通过它们在 ContainerLaunchContext（表示 YARN NodeManager 启动容器所需的所有信息）的类名。

它最初用作一个独立的应用程序来提交 Spark 应用程序到 YARN 集群，但目前被认为已过时。

为 org.apache.spark.deploy.yarn.Client 记录器启用 INFO 或 DEBUG 日志记录级别，以查看内部发生了什么。

将以下行添加到 conf / log4j.properties：

```
log4j.logger.org.apache.spark.deploy.yarn.Client=DEBUG
```

Refer to Logging.

### Creating Client Instance {#__a_id_creating_instance_a_creating_client_instance}

创建Client的实例执行以下操作：

* 创建一个成为 yarnClient 的 YarnClient 的内部实例（使用 YarnClient.createYarnClient）。

* 创建一个内部实例的 YarnConfiguration（使用 YarnConfiguration 和输入 hadoopConf），变成 yarnConf。

* 设置内部 isClusterMode，说明 spark.submit.deployMode 是否为集群部署模式。

* 当 isClusterMode 启用时，将内部 amMemory 设置为 spark.driver.memory，否则设置 spark.yarn.am.memory。

* 当 isClusterMode 启用时，将内部 amMemoryOverhead 设置为 spark.yarn.driver.memoryOverhead，否则设置为spark.yarn.am.memoryOverhead。如果两者都不可用，则选择 amMemory 的 10％ 和 384m 的最大值。

* 当 isClusterMode 启用时，将内部 amCore 设置为 spark.driver.cores，否则设置 spark.yarn.am.cores。

* 将内部 executorMemory 设置为 spark.executor.memory。

* 将内部 executorMemoryOverhead 设置为 spark.yarn.executor.memoryOverhead。如果不可用，则设置为 executorMemory 的10％ 和 384m 的最大值。

* 创建 ClientDistributedCacheManager 的内部实例（作为 distCacheMgr）。

* 设置变量：loginFromKeytab 为 false，其中 principal，keytab 和 credentials 为 null。

* 创建 LauncherBackend 的内部实例（如 launcherBackend）。

* 将内部 fireAndForget 标志设置为 isClusterMode 的结果，而不是 spark.yarn.submit.waitAppCompletion。

* 将内部变量 appId 设置为 null。

* 将内部 appStagingBaseDir 设置为 spark.yarn.stagingDir 或 Hadoop 的主目录。

### Submitting Spark Application to YARN \(submitApplication method\) {#__a_id_submitapplication_a_submitting_spark_application_to_yarn_submitapplication_method}

当 YarnClientSchedulerBackend 启动时，它创建一个新的 Client 实例并执行 submitApplication。

```
submitApplication(): ApplicationId
```

submitApplication 将一个 Spark 应用程序（由 ApplicationMaster 表示）提交到 YARN 集群（即，到 YARN ResourceManager），并返回应用程序的 ApplicationId。

| Note | submitApplication 也用于当前已弃用的 Client.run 中。 |
| :---: | :--- |


在内部，它首先执行 LauncherBackend.connect，然后执行 Client.setupCredentials 设置未来调用的 credentials。

然后它进入内部 yarn 客户端（与内部 yarnConf）并启动它。所有这一切都发生在使用 Hadoop API。

| Caution | FIXME How to configure`YarnClient`? What is YARN’s`YarnClient.getYarnClusterMetrics`? |
| :--- | :--- |


You should see the following INFO in the logs:

```
INFO Client: Requesting a new application from cluster with [count] NodeManagers
```

然后，YarnClient.createApplication（）在 YARN 中创建一个新的应用程序并获取应用程序 ID。 LauncherBackend 实例使用应用程序 ID 将状态更改为 SUBMITTED。

| Caution | FIXME Why is this important? |
| :--- | :--- |


submitApplication 验证集群是否具有 ApplicationManager 的资源（使用 verifyClusterResources）。

然后创建 YARN ContainerLaunchContext，然后创建 YARN ApplicationSubmissionContext。

您应该在日志中看到以下 INFO 消息：

```
INFO Client: Submitting application [appId] to ResourceManager
```

submitApplication 将 ApplicationMaster 的新 YARN ApplicationSubmissionContext 提交到 YARN（使用 Hadoop 的 YarnClient.submitApplication）。

它返回 Spark 应用程序的 YARN ApplicationId（由 ApplicationMaster 表示）。

### loginFromKeytab {#__a_id_loginfromkeytab_a_loginfromkeytab}

### Creating YARN ApplicationSubmissionContext \(createApplicationSubmissionContext method\) {#__a_id_createapplicationsubmissioncontext_a_creating_yarn_applicationsubmissioncontext_createapplicationsubmissioncontext_method}

```
createApplicationSubmissionContext(
  newApp: YarnClientApplication,
  containerContext: ContainerLaunchContext): ApplicationSubmissionContext
```

createApplicationSubmissionContext 创建 YARN 的 ApplicationSubmissionContext。

| Note | YARN 的 ApplicationSubmissionContext 表示 YARN ResourceManager 为 Spark 应用程序启动 ApplicationMaster 所需的所有信息。 |
| :---: | :--- |


createApplicationSubmissionContext 使用 YARN 的 YarnClientApplication（作为输入 newApp）创建 ApplicationSubmissionContext。

createApplicationSubmissionContext 在 ApplicationSubmissionContext 中设置以下信息：

| The name of the Spark application | spark.app.name configuration setting or`Spark`if not set |
| :--- | :--- |
| Queue \(to which the Spark application is submitted\) | spark.yarn.queue configuration setting |
| `ContainerLaunchContext`\(that describes the`Container`with which the`ApplicationMaster`for the Spark application is launched\) | the input`containerContext` |
| Type of the Spark application | `SPARK` |
| Tags for the Spark application | spark.yarn.tags configuration setting |
| Number of max attempts of the Spark application to be submitted. | spark.yarn.maxAppAttempts configuration setting |
| The`attemptFailuresValidityInterval`in milliseconds for the Spark application | spark.yarn.am.attemptFailuresValidityInterval configuration setting |
| Resource Capabilities for ApplicationMaster for the Spark application | See Resource Capabilities for ApplicationMaster — Memory and Virtual CPU Cores section below |
| Rolled Log Aggregation for the Spark application | See Rolled Log Aggregation Configuration for Spark Application section below |

You will see the DEBUG message in the logs when the setting is not set:

```
DEBUG spark.yarn.maxAppAttempts is not set. Cluster's default value will be used.
```

#### Resource Capabilities for ApplicationMaster — Memory and Virtual CPU Cores {#__a_id_resource_a_resource_capabilities_for_applicationmaster_memory_and_virtual_cpu_cores}

| Note | YARN 的资源模型集群中的一组计算机资源。目前，YARN 仅支持具有 memory 和 virtual CPU cores 功能的资源。 |
| :---: | :--- |


Spark 应用程序的 ApplicationMaster 请求的 YARN 资源是 amMemory 和 amMemoryOverhead 的内存以及 virtual CPU cores 的amCore的总和。

此外，如果设置 spark.yarn.am.nodeLabelExpression，则会创建一个新的 YARN ResourceRequest（对于 ApplicationMaster 容器），包括：

| Resource Name | `*`\(star\) that represents no locality. |
| :--- | :--- |
| Priority | `0` |
| Capability | The resource capabilities as defined above. |
| Number of containers | `1` |
| Node label expression | spark.yarn.am.nodeLabelExpression configuration setting |
| ResourceRequest of AM container | spark.yarn.am.nodeLabelExpression configuration setting |

It sets the resource request to this new YARN`ResourceRequest`detailed in the table above.

#### Rolled Log Aggregation for Spark Application {#__a_id_logaggregationcontext_a_rolled_log_aggregation_for_spark_application}

| Note | YARN 的 LogAggregationContext 表示 YARN NodeManager 处理应用程序日志所需的所有信息。 |
| :---: | :--- |


如果定义了 spark.yarn.rolledLog.includePattern，它将创建具有以下模式的 YARN LogAggregationContext：

| Include Pattern | spark.yarn.rolledLog.includePattern configuration setting |
| :--- | :--- |
| Exclude Pattern | spark.yarn.rolledLog.excludePattern configuration setting |

#### Verifying Maximum Memory Capability of YARN Cluster \(verifyClusterResources private method\) {#__a_id_verifyclusterresources_a_verifying_maximum_memory_capability_of_yarn_cluster_verifyclusterresources_private_method}

```
verifyClusterResources(newAppResponse: GetNewApplicationResponse): Unit
```

verifyClusterResources 是一个私有助手方法，submitApplication 用来确保 Spark 应用程序（作为一组 ApplicationMaster 和执行器）不会请求超过 YARN 集群的最大内存能力。如果是这样，它会抛出 IllegalArgumentException。

verifyClusterResources 查询输入 GetNewApplicationResponse（作为 newAppResponse）以获取最大内存。

```
INFO Client: Verifying our application has not requested more than the maximum memory capability of the cluster ([maximumMemory] MB per container)
```

如果最大内存容量高于所需的执行程序或 ApplicationMaster 内存，则应在日志中看到以下 INFO 消息：

```
INFO Client: Will allocate AM container, with [amMem] MB memory including [amMemoryOverhead] MB overhead
```

然而，如果执行器内存（作为 spark.executor.memory 和 spark.yarn.executor.memoryOverhead 设置的总和）超过最大内存功能，verifyClusterResources 将抛出 IllegalArgumentException 并显示以下消息：

```
Required executor memory ([executorMemory]+[executorMemoryOverhead] MB) is above the max threshold 
([maximumMemory] MB) of this cluster! Please check the values of 'yarn.scheduler.maximum-allocation-mb' 
and/or 'yarn.nodemanager.resource.memory-mb'.
```

如果 ApplicationMaster 所需的内存大于最大内存功能，verifyClusterResources 将抛出 IllegalArgumentException 并显示以下消息：

```
Required AM memory ([amMemory]+[amMemoryOverhead] MB) is above the max threshold ([maximumMemory] MB) of 
this cluster! Please increase the value of 'yarn.scheduler.maximum-allocation-mb'.
```

#### Creating Hadoop YARN’s ContainerLaunchContext for Launching ApplicationMaster \(createContainerLaunchContext private method\) {#__a_id_createcontainerlaunchcontext_a_creating_hadoop_yarn_s_containerlaunchcontext_for_launching_applicationmaster_createcontainerlaunchcontext_private_method}

当 Spark 应用程序提交到 YARN 时，它会调用私有帮助器方法 createContainerLaunchContext，该方法创建 YARN ContainerLaunchContext 请求，以便 YARN NodeManager 启动 ApplicationMaster（在容器中）。

```
createContainerLaunchContext(newAppResponse: GetNewApplicationResponse): ContainerLaunchContext
```

| Note | 输入 newAppResponse 是 Hadoop YARN 的 GetNewApplicationResponse。 |
| :---: | :--- |


When called, you should see the following INFO message in the logs:

```
INFO Setting up container launch context for our AM
```

它在应用程序 ID（从输入 newAppResponse）。

它计算应用程序的暂存目录的路径。

| Caution | FIXME What’s`appStagingBaseDir`? |
| :--- | :--- |


它为 Python 应用程序执行自定义步骤。

它设置一个环境以启动 ApplicationMaster 容器和 prepareLocalResources。使用环境和本地资源创建 ContainerLaunchContext 记录。

The JVM options are calculated as follows:

* `-Xmx`\(that was calculated when the Client was created\)

* `-Djava.io.tmpdir=`-FIXME:`tmpDir`

  | Caution | FIXME`tmpDir`? |
  | :--- | :--- |

* Using`UseConcMarkSweepGC`when`SPARK_USE_CONC_INCR_GC`is enabled.

  | Caution | FIXME`SPARK_USE_CONC_INCR_GC`? |
  | :--- | :--- |

* In cluster deploy mode, …​FIXME

* In client deploy mode, …​FIXME

  | Caution | FIXME |
  | :--- | :--- |

* `-Dspark.yarn.app.container.log.dir=`…​[FIXME](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/GLOSSARY.html#fixme)

* Perm gen size option…​FIXME

`--class`is set if in cluster mode based on`--class`command-line argument.

| Caution | FIXME |
| :--- | :--- |


If`--jar`command-line argument was specified, it is set as`--jar`

在集群部署模式下，在客户端部署模式下创建 org.apache.spark.deploy.yarn.ApplicationMaster，它是 org.apache.spark.deploy.yarn.ExecutorLauncher。

If`--arg`command-line argument was specified, it is set as`--arg`.

--properties 文件的路径基于 YarnSparkHadoopUtil.expandEnvironment（Environment.PWD），LOCALIZED\_CONF\_DIR，SPARK\_CONF\_FILE 构建。

整个 ApplicationMaster 参数行（如 amArgs）的形式如下：

```
[amClassName] --class [userClass] --jar [userJar] --arg [userArgs] --properties-file [propFile]
```

整个命令行的形式：

```
[JAVA_HOME]/bin/java -server [javaOpts] [amArgs] 1> [LOG_DIR]/stdout 2> [LOG_DIR]/stderr
```

| Caution | FIXME`prefixEnv`? How is`path`calculated?`ApplicationConstants.LOG_DIR_EXPANSION_VAR`? |
| :--- | :--- |


启动 ApplicationMaster 的命令行设置为 ContainerLaunchContext 记录（使用 setCommands）。

You should see the following DEBUG messages in the logs:

```
DEBUG Client: ===============================================================================
DEBUG Client: YARN AM launch context:
DEBUG Client:     user class: N/A
DEBUG Client:     env:
DEBUG Client:         [launchEnv]
DEBUG Client:     resources:
DEBUG Client:         [localResources]
DEBUG Client:     command:
DEBUG Client:         [commands]
DEBUG Client: ===============================================================================
```

将创建 SecurityManager 并将其设置为应用程序的 ACL。

| Caution | FIXME`setApplicationACLs`? Set up security tokens? |
| :--- | :--- |


#### prepareLocalResources method {#__a_id_preparelocalresources_a_preparelocalresources_method}

| Caution | FIXME |
| :--- | :--- |


```
prepareLocalResources(
  destDir: Path,
  pySparkArchives: Seq[String]): HashMap[String, LocalResource]
```

`prepareLocalResources`is…​FIXME

| Caution | FIXME Describe`credentialManager` |
| :--- | :--- |


When called,`prepareLocalResources`prints out the following INFO message to the logs:

```
INFO Client: Preparing resources for our AM container
```

| Caution | FIXME What’s a delegation token? |
| :--- | :--- |


prepareLocalResources 然后从 credential \(凭证\)提供程序获取安全令牌，并获取下次更新的最近时间（用于可更新 credentials \(凭据\)）。

After all the security delegation tokens are obtained and only when there are any, you should see the following DEBUG message in the logs:

```
DEBUG Client: [token1]
DEBUG Client: [token2]
...
DEBUG Client: [tokenN]
```

| Caution | FIXME Where is`credentials`assigned? |
| :--- | :--- |


如果使用 keytab 登录，并且下次更新的最近时间是将来的时间，prepareLocalResources 将设置续订和更新安全性令牌的内部 spark.yarn.credentials.renewalTime 和 spark.yarn.credentials.updateTime 时间。

它获取复制因子（使用 spark.yarn.submit.file.replication 设置）或返回到输入 destDir 的默认值。

| Note | 复制因子仅用于以后的 copyFileToRemote。也许这里不应该提到（？） |
| :---: | :--- |


它使用 0700 权限（rwx ------）创建输入 destDir（在 HDFS 兼容的文件系统上），即除了其所有者和超级用户之外的所有人都不可访问，所以所有者只能读，写和执行。它使用 Hadoop 的 Path.getFileSystem 来访问拥有 destDir 的 Hadoop 的文件系统（使用构造函数的 hadoopConf - Hadoop 的配置）。

| Tip | See [org.apache.hadoop.fs.FileSystem](https://hadoop.apache.org/docs/current/api/org/apache/hadoop/fs/FileSystem.html) to know a list of HDFS-compatible file systems, e.g. [Amazon S3](http://aws.amazon.com/s3/) or [Windows Azure](https://azure.microsoft.com/). |
| :--- | :--- |


If a keytab is used to log in, …​FIXME

| Caution | FIXME`if (loginFromKeytab)` |
| :--- | :--- |


如果设置包含 Spark jar（spark.yarn.archive）的单个归档的位置，则将其分发（如 ARCHIVE）到 spark\_libs。

Else if the location of the Spark jars \(spark.yarn.jars\)is set, …​FIXME

| Caution | FIXME Describe`case Some(jars)` |
| :--- | :--- |


If neither spark.yarn.archive nor spark.yarn.jars is set, you should see the following WARN message in the logs:

```
WARN Client: Neither spark.yarn.jars nor spark.yarn.archive is set, falling back to uploading libraries under SPARK_HOME.
```

然后，它将在 SPARK\_HOME 下找到包含 jar 文件的目录（使用 YarnCommandBuilderUtils.findJarsDir）。

并且所有的 jars 被压缩到临时存档，例如。 spark\_libs2944590295025097383.zip，它作为 ARCHIVE 分发到 spark\_libs（仅当它们不同时）。

如果在命令行中指定了用户 jar（--jar），则该 jar 将作为 FILE 分发到 app.jar。

然后它会为应用程序分配 SparkConf 中指定的其他资源，即 jar（在 spark.yarn.dist.jars 下），文件（在 spark.yarn.dist.files 下）和归档（在 spark.yarn.dist.archives 下）。

| Note | 要分发的其他文件可以使用 spark-submit 使用命令行选项 --jars，--files 和 --archives 来定义。 |
| :---: | :--- |


| Caution | 描述一下分发机制 |
| :---: | :--- |


它为具有本地化路径（非本地路径）或其路径（对于本地路径）的 jar 设置 spark.yarn.secondary.jars。

它更新 Spark 配置（使用内部 distCacheMgr 参考的内部配置设置）。

| Caution | 他们在哪里使用？看起来它们是 ApplicationMaster 在准备本地资源时所必需的，但是通向 ApplicationMaster 的调用顺序是什么？ |
| :---: | :--- |


它将 spark\_conf.zip 上传到输入 destDir 并设置 spark.yarn.cache.confArchive

它创建配置归档和 copyFileToRemote（destDir，localConfArchive，replication，force = true，destName = Some（LOCALIZED\_CONF\_ARCHIVE））。

| Caution | `copyFileToRemote(destDir, localConfArchive, replication, force = true, destName = Some(LOCALIZED_CONF_ARCHIVE))` | ? |
| :---: | :--- | :--- |


它添加了一个缓存相关资源（使用内部 distCacheMgr）。

| Caution | 什么 resources？哪里？为什么需要这个？ |
| :---: | :--- |


最终，它清除缓存相关的内部配置设置 - spark.yarn.cache.filenames，spark.yarn.cache.sizes，spark.yarn.cache.timestamps，spark.yarn.cache.visibilities，spark.yarn.cache.type，spark.yarn.cache.confArchive - 来自 SparkConf 配置，因为它们是内部的，不应该“污染” Web UI 的环境页面。

返回 localResources。

| Caution | localResources 如何计算？ |
| :---: | :--- |


| Note | 它专用于客户端创建 ContainerLaunchContext 以启动 ApplicationMaster 容器时。 |
| :---: | :--- |


#### Creating \_\_spark\_conf\_\_.zip Archive With Configuration Files and Spark Configuration \(createConfArchive private method\) {#__a_id_createconfarchive_a_creating_spark_conf_zip_archive_with_configuration_files_and_spark_configuration_createconfarchive_private_method}

```
createConfArchive(): File
```

createConfArchive 是一个私有助手方法，prepareLocalResources 用于使用本地配置文件 - log4j.properties 和 metrics.properties 创建一个存档（在分发它之前，以及用于 ApplicationMaster 和执行器在 YARN 集群上使用的其他文件）。

归档还将包含 HADOOP\_CONF\_DIR 和 YARN\_CONF\_DIR 环境变量（如果已定义）下的所有文件。

此外，归档包含 spark\_conf.properties 和当前 Spark 配置。

存档是具有 spark\_conf 前缀和 .zip 扩展名与上述文件的临时文件。

#### Copying File to Remote File System \(copyFileToRemote helper method\) {#__a_id_copyfiletoremote_a_copying_file_to_remote_file_system_copyfiletoremote_helper_method}

```
copyFileToRemote(
  destDir: Path,
  srcPath: Path,
  replication: Short,
  force: Boolean = false,
  destName: Option[String] = None): Path
```

copyFileToRemote 是一个私有 \[yarn\] 方法，用于将 srcPath 复制到远程文件系统 destDir（如果需要），并返回在符号链接和 挂载点后解析的目标路径。

| Note | 它专用于 prepareLocalResources。 |
| :---: | :--- |


除非强制启用（默认情况下禁用），copyFileToRemote 只会在源（srcPath）和目标（destDir）文件系统相同时复制 srcPath。

You should see the following INFO message in the logs:

```
INFO Client: Uploading resource [srcPath] -> [destPath]
```

copyFileToRemote 将 srcPath 复制到 destDir 并设置 644 权限，即所有人可读和所有者可写。

If`force`is disabled or the files are the same,`copyFileToRemote`will only print out the following INFO message to the logs:

```
INFO Client: Source and destination file systems are the same. Not copying [srcPath]
```

最终，copyFileToRemote 返回在符号链接和挂载点后解析的目标路径。

### Populating CLASSPATH for ApplicationMaster and Executors \(populateClasspath method\) {#__a_id_populateclasspath_a_populating_classpath_for_applicationmaster_and_executors_populateclasspath_method}

```
populateClasspath(
  args: ClientArguments,
  conf: Configuration,
  sparkConf: SparkConf,
  env: HashMap[String, String],
  extraClassPath: Option[String] = None): Unit
```

populateClasspath 是一个私有 \[yarn\] 帮助程序方法，用于填充 CLASSPATH（用于 ApplicationMaster 和执行程序）。

| Note | 在为 ExecutorRunnable 准备环境和为客户端准备构造函数的 args 时，输入 args 为 null。 |
| :---: | :--- |


它只是将以下条目添加到输入 env 中的 CLASSPATH key：

1. 可选的 extraClassPath（首先更改为包括 YARN 群集计算机上的路径）。

   | Note | `extraClassPath`corresponds\(对应\) to spark.driver.extraClassPath for the driver and spark.executor.extraClassPath for executors. |
   | :---: | :--- |

2. YARN’s own`Environment.PWD`

3. `__spark_conf__`directory under YARN’s`Environment.PWD`

4. If the _deprecated _spark.yarn.user.classpath.first is set, …​FIXME

   | Caution | FIXME |
   | :--- | :--- |

5. `__spark_libs__/*`under YARN’s`Environment.PWD`

6. （除非定义了可选的 spark.yarn.archive）spark.yarn.jars 中的所有本地 jar（首先更改为 YARN 群集计算机上的路径）。

7. 来自 YARN 的 yarn.application.classpath 或 YarnConfiguration.DEFAULT\_YARN\_APPLICATION\_CLASSPATH 的所有条目（如果 yarn.application.classpath 未设置）

8. 来自 YARN 的 mapreduce.application.classpath 或 MRJobConfig.DEFAULT\_MAPREDUCE\_APPLICATION\_CLASSPATH 的所有条目（如果未设置 mapreduce.application.classpath）。

9. SPARK\_DIST\_CLASSPATH（首先更改为包括 YARN 群集计算机上的路径）

在为 org.apache.spark.deploy.yarn.Client 记录器启用 DEBUG 日志记录级别时，应该会看到执行 populateClasspath 的结果，即

```
DEBUG Client:     env:
DEBUG Client:         CLASSPATH -> <CPS>/__spark_conf__<CPS>/__spark_libs__/*<CPS>$HADOOP_CONF_DIR<CPS>$HADOOP_COMMON_HOME/share/hadoop/common/*<CPS>$HADOOP_COMMON_HOME/share/hadoop/common/lib/*<CPS>$HADOOP_HDFS_HOME/share/hadoop/hdfs/*<CPS>$HADOOP_HDFS_HOME/share/hadoop/hdfs/lib/*<CPS>$HADOOP_YARN_HOME/share/hadoop/yarn/*<CPS>$HADOOP_YARN_HOME/share/hadoop/yarn/lib/*<CPS>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*<CPS>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*
```

#### Changing Path to be YARN NodeManager-aware \(getClusterPath method\) {#__a_id_getclusterpath_a_changing_path_to_be_yarn_nodemanager_aware_getclusterpath_method}

```
getClusterPath(conf: SparkConf, path: String): String
```

getClusterPath 将 spark 中的 spark.yarn.config.gatewayPath 的所有出现替换为 spark.yarn.config.replacementPath 的值。

#### Adding CLASSPATH Entry to Environment \(addClasspathEntry method\) {#__a_id_addclasspathentry_a_adding_classpath_entry_to_environment_addclasspathentry_method}

```
addClasspathEntry(path: String, env: HashMap[String, String]): Unit
```

addClasspathEntry 是一个私有助手方法，用于在输入 env 中添加输入路径到 CLASSPATH key。

#### Distributing Files to Remote File System \(distribute private method\) {#__a_id_distribute_a_distributing_files_to_remote_file_system_distribute_private_method}

```
distribute(
  path: String,
  resType: LocalResourceType = LocalResourceType.FILE,
  destName: Option[String] = None,
  targetDir: Option[String] = None,
  appMasterOnly: Boolean = false): (Boolean, String)
```

distribute 是一个内部辅助方法，prepareLocalResources 用于确定输入路径是否为 local：URI 方案，并返回非本地路径的本地化路径，或简单地为本地路径的输入路径。

distribute 返回一对，其中第一个元素是用于输入路径是本地或非本地的标志，而本地或本地化路径的另一个元素。

对于尚未分发的本地路径，将输出路径复制到远程文件系统（如果需要），并将路径添加到应用程序的分布式缓存中。

### Joining Path Components using Path.SEPARATOR \(buildPath method\) {#__a_id_buildpath_a_joining_path_components_using_path_separator_buildpath_method}

```
buildPath(components: String*): String
```

buildPath 是一个辅助方法，用于使用目录分隔符连接所有路径组件，即 [org.apache.hadoop.fs.Path.SEPARATOR](https://hadoop.apache.org/docs/current/api/org/apache/hadoop/fs/Path.html#SEPARATOR)。

### isClusterMode Internal Flag {#__a_id_isclustermode_a_isclustermode_internal_flag}

isClusterMode 是一个内部标志，说明 Spark 应用程序是以集群还是客户端部署模式运行。该标志对集群部署模式启用，即 true。

| Note | 由于 Spark 应用程序需要在每个部署模式下使用不同的设置，因此 isClusterMode 标志在每个部署模式下有效地“拆分”客户端两个部分 - 一个负责客户端，另一个负责集群部署模式。 |
| :---: | :--- |


| Caution | 将下面使用的内部字段替换为其真实含义。 |
| :---: | :--- |


Table 1. Internal Attributes of`Client`per Deploy Mode \(`isClusterMode`flag\)

| Internal attribute | cluster deploy mode | client deploy mode |
| :--- | :--- | :--- |
| `amMemory` | spark.driver.memory | spark.yarn.am.memory |
| `amMemoryOverhead` | spark.yarn.driver.memoryOverhead | spark.yarn.am.memoryOverhead |
| `amCores` | spark.driver.cores | spark.yarn.am.cores |
| `javaOpts` | spark.driver.extraJavaOptions | spark.yarn.am.extraJavaOptions |
| `libraryPaths` | spark.driver.extraLibraryPath and spark.driver.libraryPath | spark.yarn.am.extraLibraryPath |
| `--class`command-line argument for`ApplicationMaster` | `args.userClass` |  |
| Application master class | org.apache.spark.deploy.yarn.ApplicationMaster | org.apache.spark.deploy.yarn.ExecutorLauncher |

当启用 isClusterMode 标志时，对 YARN 的 YarnClient 的内部引用用于停止应用程序。

当 isClusterMode 标志被启用（并且 spark.yarn.submit.waitAppCompletion 被禁用）时，fireAndForget 内部标志也是如此。

### launcherBackend value {#__a_id_launcherbackend_a_launcherbackend_value}

`launcherBackend`…​FIXME

### SPARK\_YARN\_MODE Flag {#__a_id_spark_yarn_mode_a_spark_yarn_mode_flag}

`SPARK_YARN_MODE`is a flag that says whether…​FIXME.

| Note | Any environment variable with the`SPARK_`prefix is propagated to all \(remote\) processes. |
| :--- | :--- |


| Caution | FIXME Where is`SPARK_`prefix rule enforced? |
| :--- | :--- |


| Note | `SPARK_YARN_MODE`is a system property \(i.e. available using`System.getProperty`\) and a environment variable \(i.e. available using`System.getenv`or`sys.env`\). See YarnSparkHadoopUtil. |
| :--- | :--- |


当客户端设置一个环境以启动 ApplicationMaster 容器（以及当前被视为不推荐，将 Spark 应用程序部署到 YARN 集群中）时，在客户端部署模式下在 YARN 上为 Spark 创建 SparkContext 时启用（即 true）。

访问 YarnSparkHadoopUtil 或 SparkHadoopUtil 时，将检查 SPARK\_YARN\_MODE 标志。 当客户端被请求停止时，它被清除。

### Setting Up Environment to Launch ApplicationMaster Container \(setupLaunchEnv method\) {#__a_id_setuplaunchenv_a_setting_up_environment_to_launch_applicationmaster_container_setuplaunchenv_method}

| Caution | FIXME |
| :--- | :--- |


### Internal LauncherBackend \(launcherBackend value\) {#__a_id_launcherbackend_a_internal_launcherbackend_launcherbackend_value}

| Caution | FIXME |
| :--- | :--- |


### Internal Hadoop’s YarnClient \(yarnClient value\) {#__a_id_yarnclient_a_internal_hadoop_s_yarnclient_yarnclient_value}

```
val yarnClient = YarnClient.createYarnClient
```

yarnClient 是对 Hadoop 的 YarnClient 的私有内部引用，客户端用于创建和提交 YARN 应用程序（对于您的 Spark 应用程序）killApplication。

客户端将 Spark 应用程序提交到 YARN 群集时启动并启动 yarnClient。

客户端停止时，yarnClient 停止。

### main {#__a_id_main_a_main}

`main`方法在将 Spark 应用程序部署到 YARN 集群时调用。

| Note | 它由 spark-submit 与 --master yarn 命令行参数执行。 |
| :---: | :--- |


当您在启动 Client 独立应用程序时启动 main 方法，例如使用 ./bin/spark-class org.apache.spark.deploy.yarn.Client，您将在日志中看到以下 WARN 消息，除非您设置 SPARK\_SUBMIT 系统属性。

```
WARN Client: WARNING: This client is deprecated and will be removed in a future version of Spark. Use ./bin/spark-submit with "--master yarn"
```

`main`turns SPARK\_YARN\_MODE flag on.

然后它实例化 SparkConf，解析命令行参数（使用 ClientArguments），并将调用传递给 Client.run 方法。

### stop {#__a_id_stop_a_stop}

```
stop(): Unit
```

停止关闭内部 LauncherBackend 并停止内部 yarnClient。

它还清除 SPARK\_YARN\_MODE 标志（以允许在集群类型之间切换）。

### run {#__a_id_run_a_run}

`run`将 Spark 应用程序提交到 YARN ResourceManager（RM）。

如果 LauncherBackend 未连接到 RM，即 LauncherBackend.isConnected 返回 false，并且 fireAndForget 已启用，... FIXME

| Caution | FIXME When could`LauncherBackend`lost the connection since it was connected in submitApplication? |
| :--- | :--- |


| Caution | FIXME What is`fireAndForget`? |
| :--- | :--- |


否则，当 LauncherBackend 已连接或 fireAndForget 已禁用时，将调用 monitorApplication。它返回一对 yarnApplicationState 和finalApplicationStatus，它们针对三个不同的状态对进行检查并抛出 SparkException：

* YarnApplicationState.KILLED 或 FinalApplicationStatus.KILLED 导致 SparkException 与消息 “Application \[appId\] 被杀死”。

* YarnApplicationState.FAILED 或 FinalApplicationStatus.FAILED 导致 SparkException 消息“应用程序 \[appId\] 完成失败的状态”。

* FinalApplicationStatus.UNDEFINED 导致 SparkException 与消息“应用程序的最后状态 \[appId\] 未定义”。

| Caution | FIXME What are`YarnApplicationState`and`FinalApplicationStatus`statuses? |
| :--- | :--- |


### monitorApplication {#__a_id_monitorapplication_a_monitorapplication}

```
monitorApplication(
  appId: ApplicationId,
  returnOnRunning: Boolean = false,
  logApplicationReport: Boolean = true): (YarnApplicationState, FinalApplicationStatus)
```

monitorApplication 持续报告 Spark 应用程序 appId 的状态（每个 spark.yarn.report.interval），直到应用程序状态为以下 YarnApplicationState 之一：

* `RUNNING`\(when`returnOnRunning`is enabled\)

* `FINISHED`

* `FAILED`

* `KILLED`

| Note | It is used in run,YarnClientSchedulerBackend.waitForApplicationand`MonitorThread.run`. |
| :--- | :--- |


它从 YARN ResourceManager 获取应用程序的报告，以获取 ApplicationMaster 的 YarnApplicationState。

| Tip | It uses Hadoop’s`YarnClient.getApplicationReport(appId)`. |
| :--- | :--- |


Unless`logApplicationReport`is disabled, it prints the following INFO message to the logs:

```
INFO Client: Application report for [appId] (state: [state])
```

If`logApplicationReport`and DEBUG log level are enabled, it prints report details every time interval to the logs:

```
16/04/23 13:21:36 INFO Client:
	 client token: N/A
	 diagnostics: N/A
	 ApplicationMaster host: N/A
	 ApplicationMaster RPC port: -1
	 queue: default
	 start time: 1461410495109
	 final status: UNDEFINED
	 tracking URL: http://japila.local:8088/proxy/application_1461410200840_0001/
	 user: jacek
```

对于 INFO 日志级别，仅当应用程序状态更改时才打印报告详细信息。

当应用程序状态更改时，会通知 LauncherBackend（使用 LauncherBackend.setState）。

| Note | The application state is an instance of Hadoop’s`YarnApplicationState`. |
| :--- | :--- |


对于状态 FINISHED，FAILED 或 KILLED，调用 cleanupStagingDir，并且该方法通过返回一对当前状态和最终应用程序状态来结束。

如果启用了 returnOnRunning（默认情况下禁用），并且应用程序状态变为 RUNNING，则该方法返回一对当前状态 RUNNING 和最终应用程序状态。

| Note | 当启用 returnOnRunning 并且应用程序转为 RUNNING 时，将不会调用 cleanupStagingDir。我想这可能是一个剩余，因为客户端现在已被弃用。 |
| :---: | :--- |


记录当前状态以用于将来检查（在循环中）。

### cleanupStagingDir {#__a_id_cleanupstagingdir_a_cleanupstagingdir}

cleanupStagingDir清除应用程序的暂存目录。

| Note | 它在 submitApplication 中使用，当应用程序完成并且方法退出时，有一个异常和 monitorApplication。 |
| :---: | :--- |


它使用 spark.yarn.stagingDir 设置或回退到暂存目录的用户主目录。如果启用清除，则会删除应用程序的整个暂存目录。

You should see the following INFO message in the logs:

```
INFO Deleting staging directory [stagingDirPath]
```

### reportLauncherState {#__a_id_reportlauncherstate_a_reportlauncherstate}

```
reportLauncherState(state: SparkAppHandle.State): Unit
```

`reportLauncherState`merely passes the call on to`LauncherBackend.setState`.

| Caution | What does`setState`do? |
| :--- | :--- |


### ClientArguments {#__a_id_clientarguments_a_clientarguments}



