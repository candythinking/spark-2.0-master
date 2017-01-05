## SQL Tab {#__a_id_sqltab_a_sql_tab}

Web UI 中的 SQL 选项卡显示每个运算符的累加器值。

| Caution | Intro |
| :--- | :--- |


您可以访问 / SQL URL 下的 SQL 选项卡，例如。 http：// localhost：4040 / SQL /。

默认情况下，它显示所有 SQL 查询执行。但是，在选择查询后，“SQL” 选项卡将显示 SQL 查询执行的详细信息。

### AllExecutionsPage {#__a_id_allexecutionspage_a_allexecutionspage}

AllExecutionsPage 显示每个状态的 Spark 应用程序中的所有 SQL 查询执行，并按提交时间倒序排序。

![](/img/mastering-apache-spark/spark core-tools/figure32.png)

在内部，页面请求 SQLListener 用于运行，完成和失败状态下的查询执行（状态对应于页面上的各个表）。

### ExecutionPage {#__a_id_executionpage_a_executionpage}

ExecutionPage 显示给定查询执行 ID 的 SQL 查询执行详细信息。

注意

id请求参数是必需的。

ExecutionPage 显示包含已提交时间，持续时间，正在运行的作业的可点击标识符，已成功作业和失败作业的摘要。

它还使用可扩展详细信息部分（对应于 SQLExecutionUIData.physicalPlanDescription）显示可视化（使用累加器更新和查询的 SparkPlanGraph）。

![](/img/mastering-apache-spark/spark core-tools/figure33.png)

如果对于给定的查询 id 没有要显示的信息，您应该看到以下页面。

![](/img/mastering-apache-spark/spark core-tools/figure34.png)

在内部，它仅使用 SQLListener 来获取 SQL 查询执行指标。它请求 SQLListener 显示 id 请求参数的 SQL 执行数据。

### Creating SQLTab Instance {#__a_id_creating_instance_a_creating_sqltab_instance}

当使用 Spark History Server 时，SharedState 是或在第一个 SparkListenerSQLExecutionStart 事件时创建 SQLTab。

![](/img/mastering-apache-spark/spark core-tools/figure35.png)

注意

SharedState 表示跨所有活动 SQL 会话的共享状态。



















