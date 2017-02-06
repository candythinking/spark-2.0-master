## ClientDistributedCacheManager {#_clientdistributedcachemanager}

ClientDistributedCacheManager 只是一个包装器，用于保存缓存相关资源条目 CacheEntry（作为 distCacheEntries）的集合，以便向要分发的文件中更新 Spark 配置以及稍后更新 Spark 配置。

| Caution | FIXME What is a resource? Is this a file only? |
| :--- | :--- |


### Adding Cache-Related Resource \(addResource method\) {#__a_id_addresource_a_adding_cache_related_resource_addresource_method}

```
addResource(
  fs: FileSystem,
  conf: Configuration,
  destPath: Path,
  localResources: HashMap[String, LocalResource],
  resourceType: LocalResourceType,
  link: String,
  statCache: Map[URI, FileStatus],
  appMasterOnly: Boolean = false): Unit
```

### Updating Spark Configuration with Resources to Distribute \(updateConfiguration method\) {#__a_id_updateconfiguration_a_updating_spark_configuration_with_resources_to_distribute_updateconfiguration_method}

```
updateConfiguration(conf: SparkConf): Unit
```

updateConfiguration 在输入 conf Spark 配置中设置以下内部 Spark 配置设置：

* spark.yarn.cache.filenames 

* spark.yarn.cache.sizes 

* spark.yarn.cache.timestamps 

* spark.yarn.cache.visibilities 

* spark.yarn.cache.types

它使用带资源的内部 distCacheEntries 来分发。

| Note | 它以后在 ApplicationMaster 中准备本地资源时使用。 |
| :---: | :--- |














