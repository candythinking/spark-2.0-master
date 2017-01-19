## YarnClusterScheduler - TaskScheduler for Cluster Deploy Mode {#__a_id_yarnclusterscheduler_a_yarnclusterscheduler_taskscheduler_for_cluster_deploy_mode}

YarnClusterScheduler 是集群部署模式下 YARN 上 Spark 的 TaskScheduler。

它是一个自定义的 YarnScheduler，它确保执行适当的 ApplicationMaster 初始化，即 SparkContext 被初始化和停止。

While being created, you should see the following INFO message in the logs:

```
INFO YarnClusterScheduler: Created YarnClusterScheduler
```

Enable`INFO`logging level for`org.apache.spark.scheduler.cluster.YarnClusterScheduler`to see what happens inside`YarnClusterScheduler`.

Add the following line to`conf/log4j.properties`:

```
log4j.logger.org.apache.spark.scheduler.cluster.YarnClusterScheduler=INFO
```

### postStartHook {#__a_id_poststarthook_a_poststarthook}

postStartHook 在父类的 postStartHook 之前调用 ApplicationMaster.sparkContextInitialized。

You should see the following INFO message in the logs:

```
INFO YarnClusterScheduler: YarnClusterScheduler.postStartHook done
```

### Stopping YarnClusterScheduler \(stop method\) {#__a_id_stop_a_stopping_yarnclusterscheduler_stop_method}

`stop`calls the parent’s`stop`followed by ApplicationMaster.sparkContextStopped

.









