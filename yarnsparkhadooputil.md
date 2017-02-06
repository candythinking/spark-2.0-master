## YarnSparkHadoopUtil {#__a_id_yarnsparkhadooputil_a_yarnsparkhadooputil}

## YarnSparkHadoopUtil {#__a_id_yarnsparkhadooputil_a_yarnsparkhadooputil}

`YarnSparkHadoopUtil`is…​FIXME

`YarnSparkHadoopUtil`can only be created when SPARK\_YARN\_MODE flag is enabled.

| Note | `YarnSparkHadoopUtil`belongs to`org.apache.spark.deploy.yarn`package. |
| :--- | :--- |


| Tip | Enable`DEBUG`logging level for`org.apache.spark.deploy.yarn.YarnSparkHadoopUtil`logger to see what happens inside.Add the following line to`conf/log4j.properties`:log4j.logger.org.apache.spark.deploy.yarn.YarnSparkHadoopUtil=DEBUG                                                                                                    Refer to Logging. |
| :--- | :--- |


### `startCredentialUpdater`Method {#__a_id_startcredentialupdater_a_code_startcredentialupdater_code_method}

| Caution | FIXME |
| :--- | :--- |


### Getting YarnSparkHadoopUtil Instance — `get`Method {#__a_id_get_a_getting_yarnsparkhadooputil_instance_code_get_code_method}

| Caution | FIXME |
| :--- | :--- |


### `addPathToEnvironment`Method {#__a_id_addpathtoenvironment_a_code_addpathtoenvironment_code_method}

```
addPathToEnvironment(env: HashMap[String, String], key: String, value: String): Unit
```

| Caution | FIXME |
| :--- | :--- |


### `startExecutorDelegationTokenRenewer` {#__a_id_startexecutordelegationtokenrenewer_a_code_startexecutordelegationtokenrenewer_code}

| Caution | FIXME |
| :--- | :--- |


### `stopExecutorDelegationTokenRenewer` {#__a_id_stopexecutordelegationtokenrenewer_a_code_stopexecutordelegationtokenrenewer_code}

| Caution | FIXME |
| :--- | :--- |


### `getApplicationAclsForYarn`Method {#__a_id_getapplicationaclsforyarn_a_code_getapplicationaclsforyarn_code_method}

| Caution | FIXME |
| :--- | :--- |


### MEMORY\_OVERHEAD\_FACTOR {#__a_id_memory_overhead_factor_a_memory_overhead_factor}

MEMORY\_OVERHEAD\_FACTOR 是一个常量，等于内存开销的 10％。

### MEMORY\_OVERHEAD\_MIN {#__a_id_memory_overhead_min_a_memory_overhead_min}

MEMORY\_OVERHEAD\_MIN 是一个常量，等于 384M 内存开销。

### Resolving Environment Variable — `expandEnvironment`Method {#__a_id_expandenvironment_a_resolving_environment_variable_code_expandenvironment_code_method}

```
expandEnvironment(environment: Environment): String
```

expandEnvironment 使用 YARN 的Environment.$ 或 Environment.$$ 方法（取决于使用的 Hadoop 版本）来解析环境变量。

### Computing YARN’s ContainerId — `getContainerId`Method {#__a_id_getcontainerid_a_computing_yarn_s_containerid_code_getcontainerid_code_method}

```
getContainerId: ContainerId
```

getContainerId 是一个私有的 \[spark\] 方法，它从 YARN 环境变量 ApplicationConstants.Environment.CONTAINER\_ID 获取 YARN 的 ContainerId，并使用 YARN 的 ConverterUtils.toContainerId 将其转换为返回对象。

### Calculating Initial Number of Executors — `getInitialTargetExecutorNumber`Method {#__a_id_getinitialtargetexecutornumber_a_calculating_initial_number_of_executors_code_getinitialtargetexecutornumber_code_method}

```
getInitialTargetExecutorNumber(conf: SparkConf, numExecutors: Int = 2): Int
```

getInitialTargetExecutorNumber 计算 YARN 上 Spark 的初始执行器数。它取决于是否启用动态分配。

| Note | 执行程序的默认数量（又名 DEFAULT\_NUMBER\_EXECUTORS）为2。 |
| :---: | :--- |


启用动态分配后，如果其他未定义，getInitialTargetExecutorNumber 为 spark.dynamicAllocation.initialExecutors 或 spark.dynamicAllocation.minExecutors 以回退为 0。

禁用动态分配时，getInitialTargetExecutorNumber 是 spark.executor.instances 属性或 SPARK\_EXECUTOR\_INSTANCES 环境变量的值，或缺省值（输入参数 numExecutors）的值 2。

| Note | getInitialTargetExecutorNumber 用于计算 totalExpectedExecutors 以在客户端或集群模式下在 YARN 上启动 Spark。 |
| :---: | :--- |






