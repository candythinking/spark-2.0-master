## SparkUI {#__a_id_sparkui_a_sparkui}

SparkUI 表示 Spark 应用程序和 Spark History Server 的 Web UI。创建 SparkContext 时（在启用 spark.ui.enabled 时）创建和绑定它。

注意

Spark 应用程序和 Spark 历史服务器的 SparkUI 之间的唯一区别是... FIXME

启动时，SparkUI 将绑定到 appUIAddress 地址，您可以使用 SPARK\_PUBLIC\_DNS 环境变量或 spark.driver.host Spark 属性控制此地址。

提示

启用 org.apache.spark.ui.SparkUI 日志记录器的 INFO 日志记录级别，以查看内部发生了什么。 将以下行添加到 conf / log4j.properties： log4j.logger.org.apache.spark.ui.SparkUI = INFO 请参阅日志记录。

### Creating`SparkUI`Instance {#__a_id_creating_instance_a_creating_code_sparkui_code_instance}

```
class SparkUI (
  val sc: Option[SparkContext],
  val conf: SparkConf,
  securityManager: SecurityManager,
  val environmentListener: EnvironmentListener,
  val storageStatusListener: StorageStatusListener,
  val executorsListener: ExecutorsListener,
  val jobProgressListener: JobProgressListener,
  val storageListener: StorageListener,
  val operationGraphListener: RDDOperationGraphListener,
  var appName: String,
  val basePath: String,
  val startTime: Long)
extends WebUI(securityManager,
  securityManager.getSSLOptions("ui"), SparkUI.getUIPort(conf),
  conf, basePath, "SparkUI")
```

当执行时，SparkUI 创建一个 StagesTab，并在 web UI 中初始化选项卡和处理程序。

创建 SparkContext 时创建 SparkUI（启用 spark.ui.enabled）。 SparkUI 获取对所有 SparkContext 和其他属性的引用，即 SparkConf，LiveListenerBus Event Bus，JobProgressListener，SecurityManager，appName 和 startTime。

### Attaching Tabs and Context Handlers — `initialize`Method {#__a_id_initialize_a_attaching_tabs_and_context_handlers_code_initialize_code_method}

```
initialize(): Unit
```

`initialize`attaches the tabs of the following pages:

1. JobsTab

2. StagesTab

3. StorageTab

4. EnvironmentTab

5. ExecutorsTab

`initialize`also attaches`ServletContextHandler`handlers:

1. / static 从 org / apache / spark / ui / static 目录（在 CLASSPATH 上）提供静态文件。

2. 重定向 /to/ jobs /（所以作业选项卡是打开 Web UI 时的第一个选项卡）。

3. 服务 / api 上下文路径（使用 org.apache.spark.status.api.v1 提供程序包）使用 ApiRootResource。

4. 重定向`/stages/stage/kill`to`/stages/`

注意

initialize 是 WebUI 合同的一部分，在创建 SparkUI 时执行。

### Stopping`SparkUI` — `stop`Method {#__a_id_stop_a_stopping_code_sparkui_code_code_stop_code_method}

```
stop(): Unit
```

stop** **停止 HTTP 服务器并将以下 INFO 消息打印到日志中：

```
INFO SparkUI: Stopped Spark web UI at [appUIAddress]
```

注意

上述 INFO 消息中的 appUIAddress 是 appUIAddress 方法的结果。

### `appUIAddress`Method {#__a_id_appuiaddress_a_code_appuiaddress_code_method}

```
appUIAddress: String
```

appUIAddress 返回 Spark 应用程序的 Web UI 的完整 URL，包括 http：// scheme。 在内部，appUIAddress 使用 appUIHostPort。

### `getSparkUser`Method {#__a_id_getsparkuser_a_code_getsparkuser_code_method}

```
getSparkUser: String
```

getSparkUser 返回 Spark 应用程序运行的用户的名称。

在内部，getSparkUser 请求 user.name 来自 EnvironmentListener 的系统属性 Spark 侦听器。

注意

getSparkUser 仅用于在 Spark Jobs 页面中显示用户名。

### `createLiveUI`Method {#__a_id_createliveui_a_code_createliveui_code_method}

```
createLiveUI(
  sc: SparkContext,
  conf: SparkConf,
  listenerBus: SparkListenerBus,
  jobProgressListener: JobProgressListener,
  securityManager: SecurityManager,
  appName: String,
  startTime: Long): SparkUI
```

createLiveUI 为实时运行的 Spark 应用程序创建一个 SparkUI。

在内部，createLiveUI 只是将调用转发到 create。

注意

createLiveUI 在创建 SparkContext（并启用 spark.ui.enabled）时调用。

### `createHistoryUI`Method {#__a_id_createhistoryui_a_code_createhistoryui_code_method}

| Caution | FIXME |
| :--- | :--- |


### `create`Factory Method {#__a_id_create_a_code_create_code_factory_method}

```
create(
  sc: Option[SparkContext],
  conf: SparkConf,
  listenerBus: SparkListenerBus,
  securityManager: SecurityManager,
  appName: String,
  basePath: String = "",
  jobProgressListener: Option[JobProgressListener] = None,
  startTime: Long): SparkUI
```

create 是一个工厂帮助方法来创建 SparkUI。它负责为 SparkUI 注册 SparkListeners。

注意

create 为运行的 Spark 应用程序和 Spark History Server 创建一个 Web UI。

在内部，使用输入 listenerBus 创建一个 EnvironmentListener，StorageStatusListener，ExecutorsListener，StorageListener 和RDDOperationGraphListener。注册侦听器后，create 将创建 SparkUI 的实例。

### `appUIHostPort`Method {#__a_id_appuihostport_a_code_appuihostport_code_method}

```
appUIHostPort: String
```

appUIHostPort 返回 Spark 应用程序的 Web UI，它是公共主机名和端口，不包括 scheme。

注意

appUIAddress 使用 appUIHostPort 并添加 http：// scheme。

### `getAppName`Method {#__a_id_getappname_a_code_getappname_code_method}

```
getAppName: String
```

getAppName 返回 Spark 应用程序的名称（SparkUI 实例的名称）。

注意

getAppName 在 SparkUITab 请求应用程序的名称时使用。

### `SparkUITab` — Custom`WebUITab` {#__a_id_sparkuitab_a_a_id_appname_a_code_sparkuitab_code_custom_code_webuitab_code}

SparkUITab 是一个私有的 \[spark\] 自定义 WebUITab，仅定义一种方法，即 appName。

```
appName: String
```

`appName`returns the application’s name.



