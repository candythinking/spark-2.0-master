## Deploy Mode {#_deploy_mode}

部署模式指定驱动程序在部署环境中执行的位置。

部署模式可以是以下选项之一：

* `client`\(default\) - 该驱动程序在启动Spark应用程序的计算机上运行。

* `cluster`- 驱动程序在集群中的随机节点上运行。

| Note | 集群部署模式仅适用于非本地集群部署。 |
| :--- | :--- |


您可以使用 spark-submit 的 --deploy-mode 命令行选项或 spark.submit.deployMode Spark 属性控制 Spark 应用程序的部署模式。

| Note | `spark.submit.deployMode`setting can be`client`or`cluster`. |
| :--- | :--- |


### Client Mode {#__a_id_client_a_client_mode}

| Caution | FIXME |
| :--- | :--- |


### Cluster Mode {#__a_id_cluster_a_cluster_mode}

| Caution | FIXME |
| :--- | :--- |


### spark.submit.deployMode {#__a_id_spark_submit_deploymode_a_spark_submit_deploymode}

`spark.submit.deployMode`\(default:`client`\) can be`client`or`cluster`.



