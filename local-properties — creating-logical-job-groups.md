## Local Properties — Creating Logical Job Groups {#_local_properties_creating_logical_job_groups}

本地属性概念的目的是通过属性（不管用于提交作业的线程）创建逻辑作业组，使得从不同线程启动的单独作业属于单个逻辑组。

您可以设置将影响从线程提交的 Spark 作业的本地属性，例如 Spark fair 调度程序池。您可以使用自己的自定义属性。属性传播到 worker 任务，并可以通过 TaskContext.getLocalProperty 访问。

| Note | 当 SparkContext 被请求运行或提交 Spark 作业，然后将它们传递给 DAGScheduler 时，将局部属性传播给 worker。 |
| :---: | :--- |


| Note | 本地属性用于在 FAIR 作业调度程序中通过 spark.scheduler.pool 每线程属性和 SQLExecution.withNewExecutionId 辅助方法将作业分组到池中 |
| :---: | :--- |


本地属性概念的一个常见用例是在线程中设置一个本地属性，例如 spark.scheduler.pool，之后在线程中提交的所有作业将被分组，通过 FAIR 作业调度程序写入池中。

```
val rdd = sc.parallelize(0 to 9)

sc.setLocalProperty("spark.scheduler.pool", "myPool")

// these two jobs (one per action) will run in the myPool pool
rdd.count
rdd.collect

sc.setLocalProperty("spark.scheduler.pool", null)

// this job will run in the default pool
rdd.count
```

### Local Properties — `localProperties`Property {#__a_id_localproperties_a_local_properties_code_localproperties_code_property}

```
localProperties: InheritableThreadLocal[Properties]
```

localProperties 是 SparkContext 的受保护的 \[spark\] 属性，它是您可以通过其创建逻辑作业组的属性。

| Tip | 阅读 Java 的 java.lang.InheritableThreadLocal。 |
| :---: | :--- |


### Setting Local Property — `setLocalProperty`Method {#__a_id_setlocalproperty_a_setting_local_property_code_setlocalproperty_code_method}

```
setLocalProperty(key: String, value: String): Unit
```

setLocalProperty 将关键本地属性设置为值。

| Tip | 当 value 为 null 时，key 属性从 localProperties 中删除。 |
| :---: | :--- |


### Getting Local Property — `getLocalProperty`Method {#__a_id_getlocalproperty_a_getting_local_property_code_getlocalproperty_code_method}

```
getLocalProperty(key: String): String
```

getLocalProperty 通过此线程中的键获取本地属性。如果缺少键，则返回 null。

### Getting Local Properties — `getLocalProperties`Method {#__a_id_getlocalproperties_a_getting_local_properties_code_getlocalproperties_code_method}

```
getLocalProperties: Properties
```

getLocalProperties 是一个私有的 \[spark\] 方法，可以访问 localProperties。

### `setLocalProperties`Method {#__a_id_setlocalproperties_a_code_setlocalproperties_code_method}

```
setLocalProperties(props: Properties): Unit
```

setLocalProperties 是一个私有 \[spark\] 方法，将 props 设置为 localProperties。















