## Introduction to Hadoop YARN {#_introduction_to_hadoop_yarn}

Apache Hadoop 2.0 引入了一个用于作业调度和集群资源管理和协商的框架，称为 Hadoop YARN（又一个资源协商器）。

YARN 是用于分布式应用程序的通用应用程序调度框架，最初旨在改进 MapReduce 作业管理，但是迅速转变为支持非 MapReduce 应用程序，例如 Spark on YARN。

YARN有两个组件 - ResourceManager 和 NodeManager - 在自己的机器上运行。

* ResourceManager 是与 YARN 客户端通信的主守护程序，跟踪群集上的资源（在 NodeManager 上），并通过将任务分配给 NodeManager 来协调工作。它协调 ApplicationMasters 和 NodeManager 的工作。

* NodeManager 是一个工作进程，它提供资源（内存和 CPU）作为资源容器。它启动并跟踪在其上产生的进程。

* **Containers **运行任务，包括 ApplicationMasters。 YARN 提供容器分配。

YARN 当前定义了两个资源：vcore 和 内存。 vcore 是 CPU 内核的使用率。

YARN ResourceManager 跟踪集群的资源，而 NodeManager 跟踪本地主机的资源。

它可以选择使用两个其他组件：

* **History Server**for job history

* **Proxy Server**for viewing application status and logs from outside the cluster.

YARN ResourceManager 接受应用程序提交，调度它们，并跟踪它们的状态（通过 ApplicationMasters）。 YARN NodeManager 向 ResourceManager 注册，并提供其本地 CPU 和内存以进行资源协商。

在真正的 YARN 集群中，有一个 ResourceManager（两个用于高可用性）和多个 NodeManager。

### YARN ResourceManager {#__a_id_resourcemanager_a_yarn_resourcemanager}

YARN ResourceManager 管理计算资源对应用程序的全局分配，例如。内存，cpu，磁盘，网络等。

### YARN NodeManager {#__a_id_nodemanager_a_yarn_nodemanager}

每个 NodeManager 跟踪其自己的本地资源，并将其资源配置传达给 ResourceManager，ResourceManager 保持集群的可用资源的运行总计。

通过跟踪总数，ResourceManager 知道如何在请求时分配资源。

### YARN ApplicationMaster {#__a_id_applicationmaster_a_yarn_applicationmaster}

YARN ResourceManager 管理计算资源对应用程序的全局分配，例如。内存，cpu，磁盘，网络等。

* 应用程序是由一个或多个任务组成的YARN客户端程序。

* 对于每个正在运行的应用程序，一个称为 ApplicationMaster 的特殊代码有助于协调 YARN 集群上的任务。 ApplicationMaster 是应用程序启动后运行的第一个进程。

* YARN中的应用程序包括三个部分：

  * 应用程序客户端，这是程序在集群上运行的方式。

  * ApplicationMaster，它为YARN提供代表应用程序执行分配的能力。

  * 一个或多个任务，在由 YARN。分配的容器中执行实际工作（在进程中运行）

* 在 YARN 集群上运行任务的应用程序由以下步骤组成：

  * 应用程序启动并与集群的 ResourceManager（在主服务器上运行）通信。

  * ResourceManager 代表应用程序发出单个容器请求。

  * ApplicationMaster 开始在该容器中运行。

  * ApplicationMaster 从 ResourceManager 请求分配用于为应用程序运行任务的后续容器。这些任务与 ApplicationMaster 进行大多数状态通信。

  * 所有任务完成后，ApplicationMaster 退出。最后一个容器从集群中取消分配。

  * 应用程序客户端退出。 （在容器中启动的 ApplicationMaster 更具体地称为托管 AM）。

* ResourceManager，NodeManager 和 ApplicationMaster 一起工作以管理集群的资源，并确保任务以及相应的应用程序完全完成。

### YARN’s Model of Computation \(aka YARN components\) {#_yarn_s_model_of_computation_aka_yarn_components}

ApplicationMaster 是一个轻量级进程，协调应用程序的任务执行，并向 ResourceManager 请求任务的资源容器。

它监视任务，重新启动失败的任务等。它可以运行任何类型的任务，无论是 MapReduce 任务还是 Spark 任务。

ApplicationMaster 就像一个皇后蜂开始在 YARN 集群中创建工作者蜂（在自己的容器中）。

### Others {#_others}

* 主机是计算机（也称为节点，YARN 术语）的 Hadoop术语。

* 集群是由高速本地网络连接的两个或更多个主机。

  * 它在技术上也可以是用于调试和简单测试的单个主机。

  * Master 主机是保留用于控制集群其余部分的少量主机。工作程序主机是集群中的非主控主机。

  * Master 主机是客户端程序的通信点。Master 主机将工作发送到集群的其余部分，该集群由 workers 主机组成。

* YARN 配置文件是包含属性的 XML 文件。此文件放置在集群中每个主机上的已知位置，用于配置 ResourceManager 和 NodeManager。默认情况下，此文件命名为 yarn-site.xml。

* YARN 中的容器保存 YARN 集群上的资源。

  * 容器保持请求由 vcore 和内存组成。

* 一旦在主机上授予保留，NodeManager 将启动一个称为任务的进程。

* 应用程序 jar 文件的分布式缓存。

* 抢占（适用于高优先级应用）

* 队列和嵌套队列

* 通过 Kerberos 进行用户验证

### Hadoop YARN {#_hadoop_yarn}

* YARN 可以被认为是用于大分布式数据的 Hadoop OS（操作系统）的基石，HDFS 作为存储，YARN 作为进程调度程序。

* YARN 本质上是一个容器系统和调度程序，主要设计用于基于 Hadoop 的集群。

* YARN 中的容器能够运行各种类型的任务。

* Resource manager, node manager, container, application master, jobs

* 专注于数据存储和离线批处理分析

* Hadoop 是存储和计算平台：

  * MapReduce 是计算部分。

  * HDFS 是存储部分。

* Hadoop 是一个资源和集群管理器（YARN）

* Spark 在 YARN 集群上运行，可以从 HDFS 读取数据并将数据保存到 HDFS。S.

  * 利用数据本地化。

* Spark 需要分布式文件系统和 HDFS（或 Amazon S3，但速度较慢）是一个伟大的选择。

* HDFS 允许数据本地化。

* 当 Spark 和 Hadoop 都分布在同一个（YARN 或 Mesos）集群节点上时，具有出色的吞吐量。

* HDFS 提供（对于初始加载数据很重要）：

  * high data locality

  * high throughput when co-located with Spark

  * low latency because of data locality

  * very reliable because of replication

* 当从 HDFS 读取数据时，每个 InputSplit 都映射到一个 Spark 分区。

* HDFS 在数据节点上分发文件并在文件系统上存储文件，它将被分割成多个分区。

### ContainerExecutors {#_containerexecutors}

* LinuxContainerExecutor and Docker

* WindowsContainerExecutor

#### LinuxContainerExecutor and Docker {#__a_id_linuxcontainerexecutor_docker_a_linuxcontainerexecutor_and_docker}

[YARN-3611 Support Docker Containers In LinuxContainerExecutor](https://issues.apache.org/jira/browse/YARN-3611)is an umbrella JIRA issue for Hadoop YARN to support Docker natively.

### Further reading or watching {#__a_id_i_want_more_a_further_reading_or_watching}

* [Introduction to YARN](http://www.ibm.com/developerworks/library/bd-yarn-intro/index.html)

* [Untangling Apache Hadoop YARN, Part 1](http://blog.cloudera.com/blog/2015/09/untangling-apache-hadoop-yarn-part-1/)

* [Quick Hadoop Startup in a Virtual Environment](https://dzone.com/articles/quick-hadoop-startup-in-a-virtual-environment)

* \(video\)[HUG Meetup Apr 2016: The latest of Apache Hadoop YARN and running your docker apps on YARN](https://youtu.be/1jv0x8a9c3E)



