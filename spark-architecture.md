## Spark Architecture {#_spark_architecture}

Spark 使用 master/worker 体系结构。有一个驱动程序与一个名为 master 的单个协调器通信，它管理 executors 运行的 worker。

![](/im/mastering-apache-spark/spark core-architecture/figure1.png)

驱动程序和执行程序在自己的 Java 进程中运行。您可以在同一个（水平集群）或单独的机器（垂直集群）或混合机器配置中运行它们。

![](/img/mastering-apache-spark/spark core-architecture/figure2.png)

物理机称为主机或节点。

