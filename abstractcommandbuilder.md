## AbstractCommandBuilder {#__a_id_abstractcommandbuilder_a_abstractcommandbuilder}

AbstractCommandBuilder 是 SparkSubmitCommandBuilder 和 SparkClassCommandBuilder 专用命令构建器的基本命令构建器。

`AbstractCommandBuilder`expects that command builders define`buildCommand`.

表1. AbstractCommandBuilder方法

| Method | Description |
| :--- | :--- |
| `buildCommand` | 子类必须定义的唯一抽象方法。 |
| [buildJavaCommand](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/spark-AbstractCommandBuilder.html#buildJavaCommand) |  |
| [getConfDir](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/spark-AbstractCommandBuilder.html#getConfDir) |  |
| [loadPropertiesFile](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/spark-AbstractCommandBuilder.html#loadPropertiesFile) | 加载 Spark 应用程序的配置文件，无论是用户指定的属性文件还是 Spark 配置目录下的 spark-defaults.conf 文件。 |

### `buildJavaCommand`Internal Method {#__a_id_buildjavacommand_a_code_buildjavacommand_code_internal_method}

```
List<String> buildJavaCommand(String extraClassPath)
```

buildJavaCommand 为 Spark 应用程序（这是一个具有 java 可执行文件路径，来自 java-opts 文件的 JVM 选项以及类路径的元素的集合）构建 Java 命令。

如果设置了 javaHome，buildJavaCommand 会将 JavaHome / bin / java 添加到结果 Java 命令中。否则，它使用 JAVA\_HOME，或者，当没有早期检查成功时，将落入 java.home Java 的系统属性。

| Caution | 谁设置了 javaHome 内部属性，又是在什么时候呢？ |
| :---: | :--- |


如果文件存在，buildJavaCommand 从配置目录中的 java-opts 文件加载额外的 Java 选项，并将它们添加到结果 Java 命令中。

最终，buildJavaCommand 构建类路径（如果非空则带有额外的类路径），并将其作为 -cp 添加到结果 Java 命令中。

### `buildClassPath`method {#__a_id_buildclasspath_a_code_buildclasspath_code_method}

```
List<String> buildClassPath(String appClassPath)
```

buildClassPath 构建 Spark 应用程序的类路径。

| Note | 目录总是在它们的路径末尾有特定的操作系统的文件分隔符。 |
| :---: | :--- |


`buildClassPath`adds the following in that order:

1. `SPARK_CLASSPATH`environment variable

2. The input`appClassPath`

3. The configuration directory

4. \(only with`SPARK_PREPEND_CLASSES`set or`SPARK_TESTING`being`1`\) Locally compiled Spark classes in`classes`,`test-classes`and Core’s jars.

   | Caution | Elaborate on "locally compiled Spark classes". |
   | :--- | :--- |

5. \(only with`SPARK_SQL_TESTING`being`1`\) …​

   | Caution | Elaborate on the SQL testing case |
   | :--- | :--- |

6. `HADOOP_CONF_DIR`environment variable

7. `YARN_CONF_DIR`environment variable

8. `SPARK_DIST_CLASSPATH`environment variable

| Note | `childEnv`is queried first before System properties. It is always empty for`AbstractCommandBuilder`\(and`SparkSubmitCommandBuilder`, too\). |
| :--- | :--- |


### Loading Properties File — `loadPropertiesFile`Internal Method {#__a_id_loadpropertiesfile_a_loading_properties_file_code_loadpropertiesfile_code_internal_method}

```
Properties loadPropertiesFile()
```

loadPropertiesFile 是 AbstractCommandBuilder 私有 API 的一部分，它从属性文件（在命令行中指定）或配置目录中的 spark-defaults.conf 加载 Spark 设置。

It loads the settings from the following files starting from the first and checking every location until the first properties file is found:

1. `propertiesFile`\(if specified using`--properties-file`command-line option or set by`AbstractCommandBuilder.setPropertiesFile`\).+

2. `[SPARK_CONF_DIR]/spark-defaults.conf`

3. `[SPARK_HOME]/conf/spark-defaults.conf`

| Note | `loadPropertiesFile`reads a properties file using`UTF-8`. |
| :--- | :--- |


### Spark’s Configuration Directory — `getConfDir`Internal Method {#__a_id_getconfdir_a_a_id_configuration_directory_a_spark_s_configuration_directory_code_getconfdir_code_internal_method}

AbstractCommandBuilder 使用 getConfDir 来计算 Spark 应用程序的当前配置目录。

它使用 SPARK\_CONF\_DIR（从 childEnv 始终为空或作为环境变量），并落到 \[SPARK\_HOME\] / conf（使用 SPARK\_HOME 从 getSparkHome 内部方法）。

### Spark’s Home Directory — `getSparkHome`Internal Method {#__a_id_getsparkhome_a_a_id_home_directory_a_spark_s_home_directory_code_getsparkhome_code_internal_method}

AbstractCommandBuilder 使用 getSparkHome 来为 Spark 应用程序计算 Spark 的主目录。

它使用SPARK\_HOME（从childEnv始终为空或作为环境变量）。

If`SPARK_HOME`is not set, Spark throws a`IllegalStateException`:

```
Spark home not found; set it explicitly or use the SPARK_HOME environment variable.
```



