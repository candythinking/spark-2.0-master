## Jobs Tab {#__a_id_jobstab_a_jobs_tab}

作业选项中显示 Spark 应用程序中所有 Spark 作业的状态（即 SparkContext）。

![](/img/mastering-apache-spark/spark core-tools/figure2.png)Figure 2  Jobs Tab

“作业”选项位于/ jobs URL（即http：// localhost：4040 / jobs）下。

![](/img/mastering-apache-spark/spark core-tools/figure3.png)

Figure 3 Event Timeline in Jobs Tab

Jobs tab 由两个页面组成，即作业页面的所有作业和详细信息。

在内部，Jobs tab 由 JobsTab 类表示，该类是具有作业前缀的自定义 Spark UI Tab。

提示

Jobs tab 使用 JobProgressListener 访问 Spark 应用程序中要显示的作业执行的统计信息。

### Showing All Jobs — `AllJobsPage`Page {#__a_id_alljobspage_a_showing_all_jobs_code_alljobspage_code_page}

AllJobsPage 是一个页面（在“作业”选项卡中），用于呈现摘要，事件时间轴以及 Spark 应用程序的活动，完成和失败作业。

提示

当作业的数量大于0时，显示作业（处于任何状态）。

AllJobsPage 显示“摘要”部分，其中包含当前 Spark 用户，总运行时间，计划模式和每个状态的作业数。

注意

AllJobsPage 使用 JobProgressListener 作为调度模式

![](/img/mastering-apache-spark/spark core-tools/figure4.png)

Figure 4 Summary Section in Jobs Tab

在摘要部分下是事件时间线部分。

![](/img/mastering-apache-spark/spark core-tools/figure5.png)

Figure 5 Event Timeline in Jobs Tab

注意

AllJobsPage 使用 ExecutorsListener 构建事件

活动作业，完成作业和失败作业部分如下。

![](/img/mastering-apache-spark/spark core-tools/figure6.png)

Figure 6 Job Status Section in Jobs Tab

作业是可点击的，即您可以点击作业以查看其中的任务阶段的信息。

当您将鼠标悬停在事件时间轴中的某个作业上时，不仅您会看到作业图例，还会在“摘要”部分中突出显示该作业。

![](/img/mastering-apache-spark/spark core-tools/figure7.png)Figure 7 在事件时间线中悬停在作业上突出显示状态部分中的作业

提示

在 SparkContext 中使用可编程动态分配来管理执行程序以用于演示目的。

### Details for Job — `JobPage`Page {#__a_id_jobpage_a_details_for_job_code_jobpage_code_page}

当您在 AllJobsPage 页面中单击作业时，将看到“作业详细信息”页面。

![](/img/mastering-apache-spark/spark core-tools/figure8.png)Figure 9 Details for Job Page

JobPage 是一个自定义 WebUIPage，显示给定作业的统计信息和阶段列表。

作业页面的详细信息在/作业 URL（即 http：// localhost：4040 / jobs / job /？id = 0）下注册，并接受一个强制性 id 请求参数作为作业标识符。

当找不到作业 ID 时，您应该看到“没有要显示的作业 ID 信息”消息。

![](/img/mastering-apache-spark/spark core-tools/figure10.png)Figure 10 作业页面的详细信息中不显示“作业的信息”

JobPage 显示作业的状态，group（如果可用）和每个状态的阶段：活动，挂起，完成，跳过和失败。

注意

作业可以处于运行，成功，失败或未知状态。

![](/img/mastering-apache-spark/spark core-tools/figure11.png)Figure 11 具有活动和待处理阶段的作业页面的详细信息

![](/img/mastering-apache-spark/spark core-tools/figure12.png)

Figure 12 有四个阶段的作业页的详细信息



