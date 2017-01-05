## Fair Scheduler Pool Details Page {#__a_id_poolpage_a_fair_scheduler_pool_details_page}

Fair Scheduler 池详细信息页面显示有关可调度池的信息，仅当 Spark 应用程序使用 FAIR 调度模式（由 spark.scheduler.mode 设置控制）时可用。

![](/img/mastering-apache-spark/spark core-tools/figure27.png)

PoolPage 在/ pool URL 下呈现页面，并且需要一个请求参数 poolname，它是要显示的池的名称，例如。 http：// localhost：4040 / stages / pool /？poolname = production。它由两个表组成：摘要（具有池的详细信息）和活动阶段（在池中的活动阶段）。

它是StagesTab的一部分。

PoolPage 使用父的 SparkContext 来访问池中的活动阶段的池和 JobProgressListener 的信息（按照默认情况下降序的 submissionTime 排序）。

### Summary Table {#__a_id_pooltable_a_a_id_pool_summary_a_summary_table}

摘要表显示可调度池的详细信息。

![](/img/mastering-apache-spark/spark core-tools/figure28.png)

It uses the following columns:

* **Pool Name**

* **Minimum Share**

* **Pool Weight**+

* **Active Stages**- the number of the active stages in a`Schedulable`pool.

* **Running Tasks**

* **SchedulingMode**

所有列都是 Schedulable 的属性，但是使用池的活动阶段列表（来自父类的 JobProgressListener）计算的活动阶段数。

### Active Stages Table {#__a_id_stagetablebase_a_a_id_active_stages_a_active_stages_table}

“活动阶段”表显示池中的活动阶段。

![](/img/mastering-apache-spark/spark core-tools/figure29.png)

It uses the following columns:

* **Stage Id**

* \(optional\)**Pool Name**- only available when in FAIR scheduling mode.

* **Description**+

* **Submitted**

* **Duration**

* **Tasks: Succeeded/Total**

* **Input** — Bytes and records read from Hadoop or from Spark storage.

* **Output** — Bytes and records written to Hadoop.

* **Shuffle Read** — Total shuffle bytes and records read \(includes both data read locally and data read from remote executors\).

* **Shuffle Write** — Bytes and records written to disk in order to be read by a shuffle in a future stage.

该表使用 JobProgressListener 来获取池中每个阶段的信息。

### Request Parameters {#__a_id_parameters_a_request_parameters}

#### poolname {#__a_id_poolname_a_poolname}

poolname 是要在页面上显示的调度程序池的名称。它是一个强制的请求参数。

