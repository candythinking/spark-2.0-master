## YarnClusterManager — ExternalClusterManager for YARN {#__a_id_yarnclustermanager_a_yarnclustermanager_externalclustermanager_for_yarn}

YarnClusterManager 是 Spark 中当前唯一已知的 ExternalClusterManager。它为 YARN 创建一个 TaskScheduler 和一个 SchedulerBackend。

### `canCreate`Method {#__a_id_cancreate_a_code_cancreate_code_method}

`YarnClusterManager`can handle the`yarn`master URL only.

### `createTaskScheduler`Method {#__a_id_createtaskscheduler_a_code_createtaskscheduler_code_method}

createTaskScheduler 为集群部署模式和 YarnScheduler 客户端部署模式创建 YarnClusterScheduler。

它为未知部署模式抛出 SparkException。

```
Unknown deploy mode '[deployMode]' for Yarn
```

### `createSchedulerBackend`Method {#__a_id_createschedulerbackend_a_code_createschedulerbackend_code_method}

createSchedulerBackend 为客户端部署模式创建集群部署模式的 YarnClusterSchedulerBackend 和 YarnClientSchedulerBackend。

它为未知部署模式抛出 SparkException。

```
Unknown deploy mode '[deployMode]' for Yarn
```

### Initializing YarnClusterManager — `initialize`Method {#__a_id_initialize_a_initializing_yarnclustermanager_code_initialize_code_method}

initialize 只是初始化输入 TaskSchedulerImpl。



