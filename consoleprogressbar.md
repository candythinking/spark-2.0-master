## ConsoleProgressBar {#__a_id_consoleprogressbar_a_consoleprogressbar}

ConsoleProgressBar 显示活动阶段到标准错误的进程，即 stderr。它使用 SparkStatusTracker 周期性轮询阶段的状态，并打印多个任务的活动阶段。它一直覆盖自己，一次保持在一行，最多3个第一并发阶段。

```
[Stage 0:====>          (316 + 4) / 1000][Stage 1:>                (0 + 0) / 1000][Stage 2:>                (0 + 0) / 1000]]]
```

进度包括阶段 ID，完成，活动和总任务的数量。

| Tip | 当你 ssh 到工人，并希望看到活动阶段的进度时，ConsoleProgressBar 可能是有用的。 |
| :---: | :--- |


当 SparkContext 启动 spark.ui.showConsoleProgress 并且 org.apache.spark.SparkContext 记录器的日志记录级别为 WARN 或更高（即打印出更少的消息，因此 ConsoleProgressBar 有一个“空间”）时，将创建 ConsoleProgressBar。

```
import org.apache.log4j._
Logger.getLogger("org.apache.spark.SparkContext").setLevel(Level.WARN)
```

要打印进度很好 ConsoleProgressBar 使用 COLUMNS 环境变量来了解终端的宽度。它假设80列。

进度条打印出每个 spark.ui.consoleProgress.update.interval 毫秒阶段至少运行500毫秒后的状态。

| Note | 初始延迟500毫秒之前，ConsoleProgressBar 显示进度是不可配置的。 |
| :---: | :--- |


使用以下命令查看 Spark shell 中的进度条：

```
$ ./bin/spark-shell --conf spark.ui.showConsoleProgress=true  (1)

scala> sc.setLogLevel("OFF")  (2)

import org.apache.log4j._
scala> Logger.getLogger("org.apache.spark.SparkContext").setLevel(Level.WARN)  (3)

scala> sc.parallelize(1 to 4, 4).map { n => Thread.sleep(500 + 200 * n); n }.count  (4)
[Stage 2:>                                                          (0 + 4) / 4]
[Stage 2:==============>                                            (1 + 3) / 4]
[Stage 2:=============================>                             (2 + 2) / 4]
[Stage 2:============================================>              (3 + 1) / 4]
```

1. Make sure`spark.ui.showConsoleProgress`is`true`. It is by default.

2. Disable \(`OFF`\) the root logger \(that includes Spark’s logger\)

3. Make sure`org.apache.spark.SparkContext`logger is at least`WARN`.

4. Run a job with 4 tasks with 500ms initial sleep and 200ms sleep chunks to see the progress bar.

| Tip | [Watch the short video](https://youtu.be/uEmcGo8rwek) that show ConsoleProgressBar in action. |
| :--- | :--- |


您可能想要使用以下示例来看到完整的进度条 - 控制台中的所有3个并发阶段（从注释到 \[SPARK-4017\] 显示控制台 ＃3029 中的进度条）：

```
> ./bin/spark-shell
scala> val a = sc.makeRDD(1 to 1000, 10000).map(x => (x, x)).reduceByKey(_ + _)
scala> val b = sc.makeRDD(1 to 1000, 10000).map(x => (x, x)).reduceByKey(_ + _)
scala> a.union(b).count()
```

### Creating`ConsoleProgressBar`Instance {#__a_id_creating_instance_a_creating_code_consoleprogressbar_code_instance}

ConsoleProgressBar 需要一个 SparkContext。

创建时，ConsoleProgressBar 读取 spark.ui.consoleProgress.update.interval Spark 属性以设置终端宽度（或假定为80列）的更新间隔和 COLUMNS 环境变量。

ConsoleProgressBar 启动内部定时器刷新进度，刷新并显示进度。

| Note | 当 SparkContext 启动，spark.ui.showConsoleProgress 启用时，创建 ConsoleProgressBar，org.apache.spark.SparkContext 记录器的日志记录级别为 WARN 或更高（即打印输出的消息较少，因此 ConsoleProgressBar 有一个“空格”） 。 |
| :---: | :--- |


| Note | 一旦创建，ConsoleProgressBar 在内部可用作 \_progressBar。 |
| :---: | :--- |


### `refresh`Method {#__a_id_refresh_a_code_refresh_code_method}

| Caution | FIXME |
| :--- | :--- |


### `finishAll`Method {#__a_id_finishall_a_code_finishall_code_method}

| Caution | FIXME |
| :--- | :--- |


### `stop`Method {#__a_id_stop_a_code_stop_code_method}

```
stop(): Unit
```

`stop`cancels \(stops\) the internal timer.

| Note | `stop`is executed when`SparkContext`stops. |
| :--- | :--- |


### SparkStatusTracker {#__a_id_sparkstatustracker_a_sparkstatustracker}

SparkStatusTracker 需要一个 SparkContext 才能工作。它是作为 SparkContext 初始化的一部分创建的。

### Settings {#__a_id_settings_a_settings}

Table 1. Spark Properties

| Spark Property | Default Value | Description |
| :--- | :--- | :--- |
| `spark.ui.showConsoleProgress` | `true` | Controls whether to create`ConsoleProgressBar`\(`true`\) or not \(`false`\). |
| `spark.ui.consoleProgress.update.interval` | `200`\(ms\) | Update interval, i.e. how often to show the progress. |













