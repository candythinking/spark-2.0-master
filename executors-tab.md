## Executors Tab {#__a_id_executorstab_a_executors_tab}

Executors tab in web UI shows …​

![](/img/mastering-apache-spark/spark core-tools/figure31.png)

Executors Tab 使用 ExecutorsListener 收集有关 Spark 应用程序中执行器的信息。

选项卡的标题是Executors。

您可以访问/ executors URL 下的 Executors 选项卡，例如 http：// localhost：4040 / executors。

### ExecutorsPage {#__a_id_executorspage_a_executorspage}

| Caution | FIXME |
| :--- | :--- |


### ExecutorThreadDumpPage {#__a_id_executorthreaddumppage_a_executorthreaddumppage}

ExecutorThreadDumpPage 使用 spark.ui.threadDumpsEnabled 设置启用或禁用。

### Settings {#__a_id_settings_a_settings}

#### spark.ui.threadDumpsEnabled {#__a_id_spark_ui_threaddumpsenabled_a_spark_ui_threaddumpsenabled}

`spark.ui.threadDumpsEnabled`\(default:`true`\) is to enable \(`true`\) or disable \(`false`\)ExecutorThreadDumpPage.









