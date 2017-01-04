## Stages for All Jobs Page {#__a_id_allstagespage_a_stages_for_all_jobs_page}

AllStagesPage 是一个在 Stages 选项卡中注册的网页（节），显示 Spark 应用程序中的所有阶段 - 活动阶段，挂起阶段，完成阶段和失败阶段及其计数。

![](/img/mastering-apache-spark/spark core-tools/figure16.png)

在 FAIR 调度模式下，您可以访问显示调度程序池的表以及每个阶段的池名称。

注意

池名称使用 SparkContext.getAllPools 计算。

在内部，AllStagesPage 是一个 WebUIPage，可以访问父 Stages 选项卡，更重要的是，JobProgressListener 可以访问整个 Spark 应用程序的当前状态。

### Rendering AllStagesPage \(render method\) {#__a_id_render_a_rendering_allstagespage_render_method}

```
render(request: HttpServletRequest): Seq[Node]
```

render 将生成要在 Web 浏览器中显示的 HTML 页面。

它使用父的JobProgressListener来了解：

* active stages \(as activeStages\)

* pending stages \(as pendingStages\)

* completed stages \(as completedStages\)

* failed stages \(as failedStages\)

* the number of completed stages \(as numCompletedStages\)

* the number of failed stages \(as numFailedStages\)

注意

阶段信息作为 StageInfo 对象可用。

| Caution | FIXME StageInfo??? |
| :---: | :---: |


对于阶段的不同状态有4个不同的表 - 活动，等待，完成和失败。仅当在给定状态中存在阶段时才显示它们。

![](/img/mastering-apache-spark/spark core-tools/figure17.png)

您还可以在重试时注意“重试”阶段。

| 警告 | FIXME 屏幕截图 |
| :---: | :---: |




