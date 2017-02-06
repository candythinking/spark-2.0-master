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

* 














