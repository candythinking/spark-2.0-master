# ABSTRACT

Spark SQL 是 Apache Spark 中的一个新模块，它将相关处理与 Spark 的函数式编程 API 相集成。基于我们对 Shark 的经验，Spark SQL 使 Spark 程序员可以利用关系处理（例如声明性查询和优化存储）的优势，并让 SQL 用户在 Spark 中调用复杂的分析库（例如机器学习）。与以前的系统相比，Spark SQL 有两个主要的补充。首先，它通过与过程化 Spark 代码集成的声明式 DataFrame API，提供关系和过程处理之间更紧密的集成。第二，它包括一个高度可扩展的优化器，Catalyst，使用 Scala 编程语言的特性构建，使得容易添加可组合的规则，控制代码生成和定义扩展点。使用 Catalyst，我们为现代数据分析的复杂需求构建了各种功能（例如，对 JSON 的模式推断，机器学习类型和对外部数据库的查询联合）。我们将 Spark SQL 看作是 SQL-on-Spark 和 Spark it- self 的演进，提供更丰富的 API 和优化，同时保持 Spark 编程模型的优势。

# Categories and Subject Descriptors

H.2 \[Database Management\]: Systems

# Keywords

Databases; Data Warehouse; Machine Learning; Spark; Hadoop

# 1. Introduction

大数据应用程序需要混合处理技术，数据源和存储格式。为这些工作负载设计的最早的系统，如 MapReduce，为用户提供了一个强大的，但低级的过程式编程接口。编程这样的系统是繁重的，并且需要由用户手动优化以实现高性能。因此，多个新系统试图通过提供大数据的关系接口来提供更有效的用户体验。像 Pig，Hive，Dremel 和 Shark \[29,36,25,38\] 的系统都利用声明性查询来提供更丰富的自动优化。

虽然关系系统的普及表明用户通常更喜欢写声明性查询，但关系方法对于许多大数据应用程序来说是不够的。首先，用户想要从可能是半结构化或非结构化的各种数据源执行 ETL，需要自定义代码。其次，用户希望执行高级分析，例如机器学习和图形处理，这在关系系统中是很难表达的。在实践中，我们已经观察到，大多数数据流水线将理想地用关系查询和复杂过程算法的组合表示。不幸的是，这两类系统 - 关系和程序 - 到现在为止仍然大部分不相交，迫使用户选择一种范式或另一种范式。

本文介绍了我们在 Spark SQL 中组合这两个模型的努力，Spark SQL 是 Apache Spark 的一个主要的新组件\[39\]。 Spark SQL 基于我们早期的 SQL-on-Spark 工作，称为 Shark。但是，不是强制用户在关系 API 或过程 API 之间进行选择，而是 Spark SQL 允许用户无缝地混合两者。

Spark SQL 通过两个贡献桥接了两个模型之间的差距。首先，Spark SQL 提供了一个 DataFrame API，可以对外部数据源和 Spark 的内置分布式集合执行关系操作。这个 API 类似于 R \[32\] 中广泛使用的数据框架概念，但是延迟评估操作，以便它可以执行关系优化。第二，为了支持大数据中的各种数据源和算法，Spark SQL 引入了一种名为 Catalyst 的新型可扩展优化器。 Catalyst 可以方便地为域（如机器学习）添加数据源，优化规则和数据类型。

DataFrame API 在 Spark 程序中提供丰富的关系/过程集成。 DataFrames 是结构化记录的集合，可以使用 Spark 的过程化 API 或使用允许更丰富的优化的新关系 API 来操作。它们可以直接从 Spark 的内置分布式 Java / Python 对象集合中创建，从而在现有的 Spark 程序中实现关系处理。其他 Spark 组件，如机器学习库，也可以生成和生成 DataFrames。在许多常见的情况下，DataFrames 比 Spark 的过程化 API 更加方便和高效。例如，它们使得使用 SQL 语句在一个传递中计算多个聚合变得容易，这在传统功能 API 中很难表达。它们还以比 Java / Python 对象显着更紧凑的列格式自动存储数据。最后，与 R 和 Python 中的现有数据框架 API 不同，Spark SQL 中的 DataFrame 操作通过关系优化器 Catalyst。

为了支持 Spark SQL 中的各种数据源和分析工作负载，我们设计了一个名为 Catalyst 的可扩展查询优化器。 Catalyst 使用 Scala 编程语言的特性，如模式匹配，在图灵完备语言中表达可组合的规则。它提供了一个用于转换树的通用框架，我们用它来执行分析，规划和运行时代码生成。通过这个框架，Catalyst 还可以扩展新的数据源，包括半结构化数据，如 JSON 和“智能”数据存储，可以推送过滤器（例如 HBase）;具有用户定义的功能;和用户定义的类型（如机器学习）。功能语言已知是非常适合构建编译器\[37\]，所以也许不奇怪，他们很容易构建一个可扩展的优化器。我们确实发现 Catalyst 有效地使我们能够快速添加功能到 Spark SQL，并且从它的发布以来，我们已经看到外部贡献者很容易添加它们。

Spark SQL 于2014年5月发布，现在是 Spark 中最积极开发的组件之一。在撰写本文时，Apache Spark 是用于大数据处理的最活跃的开源项目，在过去一年中有超过400个贡献者。 Spark SQL 已经部署在非常大规模的环境中。例如，一家大型互联网公司使用 Spark SQL 构建数据管道，并在具有超过 100 PB 数据的8000节点集群上运行查询。每个单独的查询定期运行几十个字节。此外，许多用户采用 Spark SQL 不仅仅用于 SQL 查询，而是在与程序处理相结合的程序中。例如，运行 Spark 的托管服务 Databricks Cloud 的2/3客户在其他编程语言中使用 Spark SQL。在性能方面，我们发现 Spark SQL 在关系型查询的 Hadoop 上仅具有 SQL 的系统具有竞争力。它也比在 SQL 中可表达的计算中纯粹的Spark代码快达 10 x 和更高的内存效率。

更一般地，我们将 Spark SQL 看作是核心 Spark API 的重要演变。虽然 Spark 的原始功能编程 API 是相当普遍的，但它只提供有限的自动优化的机会。 SparkSQL 同时允许更多用户访问 Spark，并改进现有的用户的优化。在 Spark 中，社区现在将 Spark SQL 合并到更多的 API 中：DataFrames 是新的 “MLpipeline”API 标准学习的标准数据表示，并将其扩展到其他组件，如 GraphX 和流。

我们以 Spark 的背景和 Spark SQL 的目标开始本文。然后我们描述 DataFrame API（§3），Catalyst 优化器（§4）和我们在Catalyst（§5）上构建的高级功能。我们在§6中评估 Spark SQL。我们在§7中描述了基于 Catalyst 的外部研究。最后，§8包括相关工作。

# 2. Background and Goals

### 2.1 Spark Overview

Apache Spark 是一个通用的集群计算引擎，包括 Scala，Java 和 Python 中的 API 以及用于流，图形处理和机器学习的库\[6\]。发布于2010年，我们的知识是最广泛使用的一个具有类似于 DryadLINQ \[20\] 的“语言集成” API 的系统，并且是用于大数据处理的最活跃的开源项目。 Spark 在2014年有超过400个贡献者，并由多个供应商打包。

Spark 提供了一个类似于其他系统的函数式编程 API \[20，11\]，用户在这里操作分布式集合，称为弹性分布式数据集（RDDs）\[39\]。每个 RDD 是跨群集分区的 Java 或 Python 对象的集合。可以通过 map，filter 和 reduce 等操作来操作 RDD，这些操作使用编程语言中的函数，并将它们发送到集群上的节点。例如，下面的 Scala 代码计算文本文件中以 “ERROR” 开头的行：

    lines = spark.textFile\("hdfs://..."\) 

    errors = lines.filter\(s =&gt; s.contains\("ERROR"\)\) 

    println\(errors.count\(\)\)

此代码通过读取 HDFS 文件创建一个称为行的字符串的 RDD，然后使用过滤器对其进行转换以获取另一个 RDD 错误。然后它对此数据执行计数。

RDD 是容错的，因为系统可以使用 RDD 的沿袭血统图恢复丢失的数据（通过重新运行诸如上述过滤器的操作来重建缺失的分区）。它们还可以显式地缓存在内存或磁盘上以支持迭代\[39\]。

关于 API 的最后一个注意事项是 RDD 被延迟评估。每个 RDD 表示计算数据集的“逻辑计划”，但是 Spark 会等待某些输出操作（例如 count）启动计算。这允许引擎进行一些简单的查询优化，例如管道操作。例如，在上面的示例中，Spark 将通过应用过滤器和计算运行计数从 HDFS 文件中读取行，以便它不需要实现中间行和错误结果。虽然这种优化是非常有用的，但是它也受到限制，因为引擎不理解 RDD 中的数据结构（其是任意 Java / Python 对象）或用户函数（其包含任意代码）的语义。

### 2.2 Previous Relational Systems on Spark

我们在 Spark 上构建关系接口的第一个努力是 Shark \[38\]，它修改 Apache Hive 系统以在 Spark 上运行，并通过 Spark 引擎实现传统的 RDBMS 优化，例如列处理。虽然 Shark 表现出良好的表现和与 Spark 程序集成的良好机会，但它有三个重要的挑战。首先，Shark只能用于查询存储在 Hive 目录中的外部数据，因此对于 Spark 程序内的数据的关系查询（例如，对上面手动创建的错误 RDD）没有用。第二，从 Spark 程序调用 Shark 的唯一方法是将一个 SQL 字符串组合在一起，这在一个模块化程序中是不方便和容易出错的。最后，Hive 优化器针对 MapReduce 量身定制，难以扩展，使得难以构建新功能，如用于机器学习的数据类型或支持新的数据源。

### 2.3 Goals for Spark SQL

根据 Shark 的经验，我们想扩展关系处理以覆盖 Spark 中的本地 RDD 和更广泛的数据源。我们为 Spark SQL 设置了以下目标：

1. 支持使用 Spark 程序（对原生 RDD）和外部数据源的关系处理，使用友好的程序员 API。

2. 使用已建立的 DBMS 技术提供高性能。

3. 轻松支持新的数据源，包括半结构化数据和适合查询联合的外部数据库。

4. 使用高级分析算法（如图形处理和机器学习）启用扩展。

# 3. Programming Interface

Spark SQL 作为 Spark 上的库运行，如图1所示。它暴露了可以通过 JDBC / ODBC 或通过命令行控制台访问的 SQL 接口，以及 DataFrame API 集成到 Spark 支持的编程语言中。我们从覆盖 DataFrame API 开始，它允许用户混合过程和关系代码。但是，高级函数也可以通过 UDF 暴露在 SQL 中，允许通过商业智能工具调用它们。我们在3.7节讨论 UDF。

![](/img/spark_paper/2015 Spark SQL Relational Data Processing in Spark/figure1.png)

### 3.1 DataFrame API

Spark SQL 的 API 中的主要抽象是一个 DataFrame，它是一个具有同构模式的分布式集合。 一个 DataFrame 等价于关系数据库中的表，并且也可以类似于 Spark（RDDs）中的“本地”分布式集合的方式进行操作。 与 RDDs 不同，DataFrames 跟踪它们的模式并支持导致更优化执行的各种关系操作。

DataFrames 可以从系统数据库中的表（基于外部数据源）或本机 Java / Python 对象的现有 RDD（第3.5节）构建。一旦构造，它们可以用各种关系运算符操作，例如 where 和 groupBy，它们采用类似于 R 和 Python 中的数据帧的特定于域的语言（DSL）中的表达式\[32,30\]。每个 DataFrame 也可以被视为 Row 对象的 RDD，允许用户调用过程性的 Spark API，如 map。

最后，与传统的数据框架 API 不同，Spark DataFrames 是懒惰的，每个 DataFrame 对象表示一个计算数据集的逻辑计划，但是直到用户调用一个特殊的“输出操作”（如保存）才会执行。这使得可以在用于构建 DataFrame 的所有操作之间进行丰富的优化。

为了说明，下面的 Scala 代码从 Hive 中的表中定义了一个 DataFrame，基于它派生另一个，并打印一个结果：

    ctx = new HiveContext\(\) 

    users = ctx.table\("users"\) 

    young = users.where\(users\("age"\) &lt; 21\) 

    println\(young.count\(\)\)

在这段代码中，用户和young是DataFrames。代码段 users（“age”）&lt;21 是数据帧 DSL 中的表达式，它作为抽象语法树捕获，而不是像传统 Spark API 中那样表示 Scala 函数。最后，每个 DataFrame 只是表示一个逻辑计划 \(i.e., read the users table and filter for age &lt; 21\)。当用户调用 count（这是一个输出操作）时，Spark SQL 构建一个物理计划来计算最终结果。这可能包括优化，例如只要扫描数据的 “age” 列，如果其存储格式为列，或者甚至使用数据源中的索引来计算匹配的行。

我们接下来介绍DataFrame API的详细信息。

### 3.2 Data Model

Spark SQL 对表和 DataFrames 使用基于 Hive \[19\] 的嵌套数据模型。它支持所有主要的 SQL 数据类型，包括布尔值，整数，双精度，十进制，字符串，日期和时间戳，我们选择了 DataFrame，因为它类似于 R 和 Python 中的结构化数据库，并且设计我们的 API。这些 Row 对象是即时构建的，不一定代表数据的内部存储格式，通常为柱形。以及复杂（即非原子）数据类型：结构，数组，映射和联合。复杂数据类型也可以嵌套在一起以创建更强大的类型。与许多传统的 DBMS 不同，Spark SQL 为查询语言和 API 中的复杂数据类型提供一流的支持。此外，Spark SQL 还支持用户定义的类型，如第4.4.2节所述。

使用这种类型的系统，我们能够准确地对来自各种源和格式的数据进行建模，包括 Hive，关系数据库，JSON 和 Java / Scala / Python 中的本地对象。

### 3.3 DataFrame Operations

用户可以使用类似于 R 数据帧\[32\]和 Python Pandas \[30\] 的特定于域的语言（DSL）在 DataFrames 上执行关系操作。 DataFrames 支持所有常见的关系运算符，包括投影（选择），过滤器（其中），连接和聚合（groupBy）。这些运算符都在有限的 DSL 中使用表达式对象，使 Spark 捕获表达式的结构。例如，以下代码计算每个部门中的女性员工人数。

    employees .join\(dept, employees\("deptId"\) === dept\("id"\)\) .where\(employees\("gender"\) === "female"\).groupBy\(dept\("id"\),dept\("name"\)\) .agg\(count\("name"\)\)

这里，employees 是一个 DataFrame，employees（“deptId”）是一个表示 deptId 列的表达式。表达式对象有许多返回新表达式的运算符，包括通常的比较运算符（例如，===用于等于测试，&gt; 用于大于）和算术运算符（+， - 等）。它们还支持集合，例如count（“name”）。所有这些运算符构建表达式的抽象语法树（AST），然后将其传递给 Catalyst 进行优化。这与本地 Spark API 不同，后者使用包含任意 Scala / Java / Python 代码的函数，这些代码对运行时引擎来说是不透明的。有关 API 的详细列表，请参阅Spark的官方文档\[6\]。

除了关系 DSL，DataFrames 可以注册为系统目录中的临时表，并使用 SQL 查询。下面的代码显示了一个示例：

    users.where\(users\("age"\) &lt; 21\) .registerTempTable\("young"\) ctx.sql\("SELECT count\(\*\), avg\(age\) FROM young"\)

SQL 有时便于简明地计算多个聚合，并且还允许程序通过 JDBC / ODBC 公开数据集。目录中注册的 DataFrames 仍然是非实例化视图，因此可以跨 SQL 和原始 DataFrame 表达式进行优化。但是，DataFrames 也可以实现，我们在第3.6节中讨论。

### 3.4 DataFrames versus Relational Query Languages

虽然表面上，DataFrames 提供与关系查询语言（如 SQL 和 Pig \[29\]）相同的操作，但我们发现，由于他们在完整的编程语言中的集成，他们可以更容易地为用户工作。例如，用户可以将其代码分解成 Scala，Java 或 Python 函数，这些函数在它们之间传递 DataFrame 以构建逻辑计划，并且当他们运行输出操作时，仍然会受益于整个计划的优化。同样，开发人员可以使用如 if 语句和循环的控制结构来构造其工作。一个用户说，DataFrame API 是“简洁和声明性的像 SQL，除了我可以命名中间结果”，指的是如何更容易结构计算和调试中间步骤。

为了简化 DataFrames 中的编程，我们还使API分析逻辑计划热切（即，识别列名称是否使用表示在下面的列表，以及它们的数据类型是否适当），即使查询结果被延迟计算。因此，Spark SQL 报告错误，只要用户键入无效的代码行，而不是等待直到执行。这比大型 SQL 语句更容易使用。

### 3.5 Querying Native Datasets

现实世界的管道通常从异构源中提取数据，并从不同的编程库运行各种各样的算法。为了与过程 Spark 代码交互操作，Spark SQL 允许用户直接针对编程语言本地的对象的 RDD 构建 DataFrames。 Spark SQL 可以使用反射自动推断这些对象的模式。在 Scala 和 Java 中，类型信息是从语言的类型系统（从 JavaBeans 和 Scala case 类）中提取的。在 Python 中，Spark SQL 对数据集进行抽样，以便执行由于动态类型系统的模式推断。

例如，下面的 Scala 代码从 User 对象的 RDD 中定义了一个 DataFrame。 Spark SQL 自动检测列的名称（“name”和“age”）和数据类型（string 和 int）。

    case class User\(name: String, age: Int\) 

    // Create an RDD of User objects 

    usersRDD = spark.parallelize\( List\(User\("Alice", 22\), User\("Bob", 19\)\)\) 

    // View the RDD as a DataFrame 

    usersDF = usersRDD.toDF

在内部，Spark SQL 创建一个指向 RDD 的逻辑数据扫描运算符。这被编译成访问本地对象的字段的物理操作符。重要的是注意这与传统的对象关系映射（ORM）非常不同。 ORM 通常会产生昂贵的转换，将整个对象转换为不同的格式。相比之下，Spark SQL 就地访问本机对象，只提取每个查询中使用的字段。

查询本地数据集的功能允许用户在现有的 Spark 程序中运行优化的配置操作。此外，它使 RDD 与外部结构化数据的组合变得更加简单。例如，我们可以使用 Hive 中的表来加入用户 RDD：

    views = ctx.table\("pageviews"\) 

    usersDF.join\(views, usersDF\("name"\) === views\("user"\)\)

### 3.6 In-Memory Caching

与之前的 Shark 一样，Spark SQL 可以使用列存储实现（通常称为“缓存”）内存中的热数据，与 Spark 的本地缓存（它只是将数据存储为 JVM 对象）相比，柱形缓存可以减少内存占用缓存对于交互式查询和机器学习中常见的迭代算法特别有用，它可以通过在 DataFrame 上调用 cache（）来调用。

### 3.7 User-Defined Functions

用户定义的函数（UDF）一直是数据库系统的重要扩展点。例如，MySQL 依靠 UDF 为 JSON 数据提供基本支持。更高级的例子是 MADLib 使用 UDF 来实现 Postgres 和其他数据库系统的机器学习算法\[12\]。然而，数据库系统通常需要在与主查询接口不同的单独的编程环境中定义 UDF。 Spark SQL 的 DataFrame API 支持 UDF 的内联定义，而不需要在其他数据库系统中发现复杂的打包和注册过程。这个特性对于采用 API 已经被证明是非常重要的。

在 Spark SQL 中，可以通过传递 Scala，Java 或 Python 函数（可以内部使用完整的 Spark API）来内联注册 UDF。例如，给定用于机器学习模型的模型对象，可以将其预测函数注册为 UDF：

    val model: LogisticRegressionModel = ... 

    ctx.udf.register\("predict", \(x: Float, y: Float\) =&gt; model.predict\(Vector\(x, y\)\)\) 

    ctx.sql\("SELECT predict\(age, weight\) FROM users"\)

一旦注册，UDF 也可以通过 JDBC / ODBC 接口由商业智能工具使用。除了对像这里的标量值操作的 UDF 之外，可以定义通过获取其名称来操作整个表的 UDF，如在 MADLib \[12\] 中，并在其中使用分布式 Spark API，从而暴露高级分析功能给 SQL 用户。最后，因为 UDF 定义和查询执行被压缩使用通用语言（例如，Scala 或 Python），用户可以使用标准工具调试或配置整个程序。

上面的示例演示了许多管道中的常见用例，即使用关系运算符和高级分析方法（在 SQL 中表达很麻烦）。 DataFrame API 让开发人员无缝地混合这些方法。

# 4. Catalyst Optimizer

为了实现 Spark SQL，我们设计了一个新的可扩展优化器 Catalyst，基于 Scala 中的函数式编程结构。 Catalyst 的可扩展设计有两个目的。首先，我们希望为 Spark SQL 添加新的优化技术和特性，特别是为了解决我们特别关注的“大数据”（例如半结构化数据和高级分析）所遇到的各种问题。其次，我们希望外部开发人员能够扩展优化器，例如，通过添加数据源特定规则，可以将过滤或聚合推送到外部存储系统或支持新的数据类型。 Catalyst 支持基于规则和基于成本的优化。

虽然可扩展优化器在过去已被提出，他们通常需要复杂域特定语言到规范，和“优化器编译器”到透明的代码到可编程代码\[17,16\]。这导致显着的学习曲线和维护负担。相比之下，Catalyst 使用 Scala 编程语言的标准特性，如模式匹配\[14\]，让开发人员使用完整的编程语言，同时仍然使规则易于指定。函数式语言被部分地设计成构建编译器，所以我们发现 Scala 非常适合这个任务。尽管如此，据我们所知，Catalyst 是基于这种语言构建的第一个生产质量查询优化器。

在其核心，Catalyst 包含一个通用库，用于表示树和应用规则来操作它们。在这个框架之上，我们构建了专用于关系查询处理（例如表达式，逻辑查询计划）的库以及处理查询执行的不同阶段的几组规则：分析，逻辑优化，物理规划和代码生成将部分查询编译为 Java 字节码。对于后者，我们使用另一个 Scala 特性，quasiquotes \[34\]，这使得它很容易从可组合表达式在运行时生成代码。最后，Catalyst 提供了几个公共扩展点，包括外部数据源和用户定义的类型。

### 4.1 Trees

Catalyst 中的主要数据类型是由节点对象组成的树。每个节点都有一个节点类型和零个或多个子节点。新节点类型在 Scala 中定义为 TreeNode 类的子类。这些对象是不可变的，并且可以使用函数变换来操作，如下一小节所讨论的。

作为一个简单的例子，假设我们有一个非常简单的表达式语言的以下三个节点类：

* Literal（value：Int）：一个常量值

* Attribute\(name: String\) : anattribute from an input row, e.g.,“x”

* Add\(left: TreeNode, right: TreeNode\) : sum of two expressions.

这些类可以用来建立树木;例如，如图2所示的表达式x +（1 + 2）的树将在Scala代码中表示如下：

Add\(Attribute\(x\), Add\(Literal\(1\), Literal\(2\)\)\)

![](/img/spark_paper/2015 Spark SQL Relational Data Processing in Spark/figure2.png)

### 4.2 Rules

树可以使用规则来操纵，规则是从树到另一个树的函数。虽然规则可以在其输入树上运行任意代码（假定该树只是一个 Scala 对象），但最常见的方法是使用一组模式匹配函数来查找和替换特定结构的子树。

模式匹配是许多函数式语言的一个特性，允许从潜在的嵌套数据类型结构中提取值。在 Catalyst 中，树提供了一种转换方法，该方法在树的所有节点上递归地应用模式匹配函数，将匹配每个模式的那些转换为结果。例如，我们可以实现一个在常量之间折叠 Add 操作的规则，如下所示：

    tree.transform { 

        case Add\(Literal\(c1\), Literal\(c2\)\) =&gt; Literal\(c1+c2\) 

    }

在图2中将这应用于树的x +（1 + 2）将产生新的树x + 3。这里的 case 关键字是 Scala 的标准模式匹配语法\[14\]，可以用于匹配对象的类型，以及给名字提取的值（这里的 c1 和 c2）。

传递给 transform 的模式匹配表达式是一个部分函数，​​这意味着它只需要匹配所有可能的输入树的子集。 Catalyst 将测试给定规则适用的树的哪些部分，自动跳过和下降到不匹配的子树。这种能力意味着规则只需要关于应用给定优化的树，而不是那些不匹配的树。因此，当向系统添加新类型的运算符时，不需要修改规则。

规则（和一般的 Scala 模式匹配）可以在同一个变换调用中匹配多个模式，使得它非常简洁，可以一次实现多个变换：

    tree.transform { 

        case Add\(Literal\(c1\), Literal\(c2\)\) =&gt; Literal\(c1+c2\) case Add\(left, Literal\(0\)\) =&gt; left case Add\(Literal\(0\), right\) =&gt; right 

    }

在实践中，规则可能需要执行多次以完全变换树。 Catalyst 组将规则分为批处理，并执行每个批处理，直到达到固定点，即直到树在应用其规则后停止更改。将规则运行到固定点意味着每个规则可以是简单和自包含的，但仍然最终对树具有更大的全局效果。在上面的例子中，重复应用会不断折叠更大的树，例如（x + 0）+（3 + 3）。作为另一示例，第一批可以分析表达式以将类型分配给所有属性，而第二批可以使用这些类型来进行常数折叠。在每个批次之后，开发人员还可以对新树执行健全性检查（例如，查看所有属性都已分配类型），通常也通过递归匹配写入。

最后，规则条件及其主体可以包含任意的 Scala 代码。这使 Catalyst 比优化器的域特定语言更强大，同时保持简洁的规则。

在我们的经验中，对不可变树的功能转换使得整个优化器很容易推理和调试。它们还在优化器中启用并行化，虽然我们还没有利用这一点。

### 4.3 Using Catalyst in Spark SQL

我们在四个阶段使用 Catalyst 的通用树转换框架，如图3所示：（1）分析逻辑计划以解析引用，（2）逻辑计划优化，（3）物理规划，以及（4）查询到 Java 字节码。在物理规划阶段，Catalyst 可以生成多个计划并根据成本进行比较。所有其他阶段都是纯粹基于规则的。每个阶段使用不同类型的树节点; Catalyst 包括表达式，数据类型以及逻辑和物理运算符的节点库。我们现在描述每个阶段。

![](/img/spark_paper/2015 Spark SQL Relational Data Processing in Spark/figure3.png)

### 4.3.1 Analysis

Spark SQL 以要通过 SQL 解析器返回的抽象语法树（AST）或使用 API​​ 构建的 DataFrame 对象计算的关系开始。在这两种情况下，关系可能包含未解析的属性引用或关系：例如，在SQL查询 SELECT col FROM sales 中，col 的类型，甚至是否是有效的列名，直到我们查找表销售。如果一个属性不知道其类型或没有与输入表（或别名）匹配，则该属性被称为未解析。 Spark SQL 使用 Catalyst 规则和跟踪所有数据源中的表的目录对象来解析这些属性。它首先构建具有未绑定属性和数据类型的“未解析逻辑计划”树，然后应用执行以下操作的规则：

* 从目录中按名称查找关系。

* 将指定的属性（如 col）映射到提供给操作员的孩子的输入。

* 确定哪些属性引用相同的值，为它们提供唯一的 ID（稍后允许优化表达式，如 col = col）。

* 通过表达式传播和强制类型：例如，我们不能知道 1 + col 的返回类型，直到我们已经解析 col 并且可能将其子表达式转换为兼容类型。

总的来说，分析仪的规则约为1000行代码。

### 4.3.2 Logical Optimization

逻辑优化阶段将标准的基于规则的优化应用于逻辑计划。这些包括常量折叠，预测下推，投影修剪，空传播，布尔表达式简化和其他规则。一般来说，我们发现为各种各样的情况添加规则非常简单。例如，当我们将固定精度 DECIMAL 类型添加到 Spark SQL 中时，我们希望以小精度优化 DECIMAL 上的聚合，如和和平均;它花了12行代码来编写一个规则，在 SUM 和 AVG 表达式中找到这样的小数，并将它们转换为未缩放的64位 LONG，然后对其进行聚合，然后将结果转换回来。仅优化 SUM 表达式的此规则的简化版本转载如下：

    object DecimalAggregates extends Rule\[LogicalPlan\] { 

        /\*\* Maximum number of decimal digits in a Long \*/ 

        val MAX\_LONG\_DIGITS = 18 

        def apply\(plan: LogicalPlan\): LogicalPlan = { 

            plan transformAllExpressions { 

                case Sum\(e @ DecimalType.Expression\(prec, scale\)\) 

                if prec + 10 &lt;= MAX\_LONG\_DIGITS =&gt; MakeDecimal\(Sum\(UnscaledValue\(e\)\), prec + 10, scale\) 

        } 

    }

作为另一个示例，12行规则使用简单的正则表达式将 LIKE 表达式优化为 String.startsWith 或 String.contains 调用。在规则中使用任意 Scala 代码的自由使这些优化，超越了模式匹配子树的结构，容易表达。总的来说，逻辑优化规则是800行代码。

### 4.3.3 Physical Planning

在物理规划阶段，Spark SQL 使用逻辑计划并使用与 Spark 执行引擎匹配的物理运算符来生成一个或多个物理计划。然后使用成本模型选择计划。目前，基于成本的优化仅用于选择连接算法：对于已知较小的关系，Spark SQL 使用广播连接，使用 Spark 中提供的对等广播功能。 然而，该框架支持更广泛地使用基于成本的优化，因为可以使用规则对整个树进行递归估计成本。因此，我们打算在未来实施更丰富的基于成本的优化。

物理规划执行基于规则的物理优化，例如将投影或过滤器管道化为一个 Spark 映射操作。此外，它可以将操作从逻辑计划推送到支持谓词或投影下推的数据源。我们将在第4.4.1节中描述这些数据源的 API。

总的来说，物理规划规则约有500行代码。

### 4.3.4 Code Generation

查询优化的最后阶段涉及生成在每台机器上运行的Java字节码。因为 Spark SQL 经常运行内存中的数据集，其中处理是 CPU 限制的，所以我们希望支持代码生成以加快执行速度。尽管如此，代码生成引擎通常构建起来很复杂，基本上相当于编译器。 Catalyst 依赖于 Scala 语言的一个特殊功能，quasiquotes \[34\]，使代码生成更简单。 Quasiquotes 允许以 Scala 语言对抽象语法树（AST）进行编程构建，然后可以在运行时将其提供给 Scala 编译器以生成字节码。我们使用 Catalyst 将表示 SQL 中的表达式的树转换为用于 Scala 代码的 AST，以评估该表达式，然后编译并运行生成的代码。

作为一个简单的例子，考虑4.2节中介绍的 Add，Attribute 和 Literal 树节点，它允许我们写出表达式，如（x + y）+1。在没有代码生成的情况下，通过沿着 Add，Attribute 和 Literal 节点的树向下走，必须为每行数据解释这样的表达式。这引入了大量的分支和虚拟函数调用，这会减慢执行速度。使用代码生成，我们可以编写一个函数来将特定的表达式树翻译为 Scala AST，如下所示：

    def compile\(node: Node\): AST = node match { 

        case Literal\(value\) =&gt; q"$value" 

        case Attribute\(name\) =&gt; q"row.get\($name\)" 

        case Add\(left, right\) =&gt; q"${compile\(left\)} + ${compile\(right\)}" 

    }

以 q 开头的字符串是 quasiquote，意思是虽然它们看起来像字符串，它们由 Scala 编译器在 compiletime 和表达式下解析。 Quasiquotes 可以有变量或其他 AST 拼接到它们，用 $ 表示。例如，Literal（1）将成为1的 Scala AST，而 Attribute（“x”）成为row.get（“x”）。最后，像 Add（Literal（1），Attribute（“x”））这样的树成为一个 Scala 表达式的 AST，如 1 + row.get（“x”）。

Quasiquotes 在编译时进行类型检查，以确保只替换适当的 AST 或字面值，使它们比字符串连接显着更有用，它们直接导致在 Scala AST 中，而不是在运行时运行 Scala 解析器。此外，它们是高度可组合的，因为每个节点的代码生成规则不需要知道如何构造其子节点返回的树。最后，如果 Catalyst 缺少表达式优化，Scala 编译器会进一步优化生成的代码。图4显示了 quasiquotes 让我们生成代码，性能类似于手动调整的程序。

![](/img/spark_paper/2015 Spark SQL Relational Data Processing in Spark/figure4.png)

我们发现 quasiquotes 非常直接用于代码生成，我们观察到，即使新的贡献者 Spark SQL 可以快速添加新的表达式类型的规则。 Quasiquotes 也适用于我们在本地 Java 对象上运行的目标：当从这些对象访问字段时，我们可以编码 - 生成对必需字段的直接访问，而不必将对象复制到 Spark SQL 行，并使用 Row 访问器方法。最后，直接将代码生成评估与对我们尚未生成代码的表达式的解释评估结合起来，因为我们编译的 Scala 代码可以直接调用我们的表达式解释器。

总的来说，Catalyst的代码生成器大约有700行代码。

### 4.4 Extension Points

Catalyst 围绕可组合规则的设计使得用户和第三方库易于扩展。开发人员可以在运行时向查询优化的每个阶段添加批次的规则，只要它们遵守每个阶段的合同（例如，确保分析解析所有属性）。但是，为了使添加某些类型的扩展而不了解 Catalyst 规则更简单，我们还定义了两个较窄的公共扩展点：数据源和用户定义的类型。这些仍然依赖于核心引擎中的设施来与剩余的优化器交互。

### 4.4.1 Data Sources

开发人员可以使用多个 API 为 Spark SQL 定义新的数据源，这些 API 暴露了不同程度的可能优化。所有数据源必须实现 createRelation 函数，该函数接受一组键值参数，并为该关系返回一个 BaseRelation 对象（如果可以成功加载）。每个 BaseRelation 包含一个模式和一个可选的估计大小（以字节为单位）。 例如，表示 MySQL 的数据源可以使用表名作为参数，并要求 MySQL 估计表大小。

为了让 Spark SQL 读取数据，BaseRelation 可以实现几个接口之一，让它们暴露不同程度的复杂性。最简单的 TableScan 需要关系来为表中的所有数据返回 Row 对象的 RDD。更高级的 PrunedScan 需要读取列名称数组，并且应该返回只包含这些列的 Rows。第三个接口 PrunedFilteredScan 接受所需的列名和一个 Filter 对象数组，这是 Catalyst 表达式语法的一个子集，允许谓词下推。 建议使用过滤器，即数据源应尝试仅返回通过每个过滤器的行，但允许在无法评估的过滤器的情况下返回假阳性。最后，CatalystScan 接口提供了一个完整的 Catalyst 表达式树序列，用于谓词下推，尽管它们再次是建议性的。

这些接口允许实现各种优化，同时仍然使开发人员能够容易地添加几乎任何类型的简单数据源。我们和其他人已经使用该接口来实现以下数据源：

* CSV文件，它只是扫描整个文件，但允许用户指定模式。

* Avro \[4\]，一种用于嵌套数据的自描述二进制格式。

* Parquet \[5\]，一种柱形文件格式，我们支持列修剪以及过滤器。

* JDBC数据源，从RDBMS并行扫描表的范围，并将过滤器推送到RDBMS以最小化通信。

要使用这些数据源，程序员在 SQL 语句中指定其包名称，传递配置选项的键值对。例如，Avro 数据源获取文件的路径：

    CREATE TEMPORARY TABLE messages 

    USING com.databricks.spark.avro 

    OPTIONS \(path "messages.avro"\)

所有数据源还可以暴露网络位置信息，即，数据的每个分区从哪个机器最有效地读取。这是通过它们返回的 RDD 对象来公开的，因为 RDD 具有用于数据局部性的内置 API \[39\]。

最后，存在用于将数据写入现有表或新表的类似接口。这些更简单，因为 Spark SQL 只提供要写入的 Row 对象的 RDD。

### 4.4.2 User-Defined Types \(UDTs\)

我们想要允许在 Spark SQL 中进行高级分析的一个功能是用户定义的类型。例如，机器学习应用程序可能需要一个向量类型，图形算法可能需要类型代表段，这可能是相关的表\[15\]。然而，添加新类型可能是具有挑战性的，因为数据类型贯穿执行引擎的所有方面。例如，在 Spark SQL 中，内置数据类型以内存中缓存的柱形压缩格式存储（第3.6节），在上一节的数据源 API 中，我们需要公开所有可能的数据类型到数据源作者。

在 Catalyst 中，我们通过将用户定义的类型映射到由 Catalyst 内置类型组成的结构来解决这个问题，如第3.2节所述。要将 Scala 类型注册为 UDT，用户提供从其类的对象到内置类型的 Catalyst 行的映射，以及反向映射。在用户代码中，他们现在可以在他们使用 Spark SQL 查询的对象中使用 Scala 类型，并且它将转换为内置类型。同样，它们可以注册直接对其类型操作的 UDF（参见第3.7节）。

作为示例，假设注册二维点（x，y）asaUDT.Wecan 表示这两个值的两个值。要注册 UDT，可以这样写：

    class PointUDT extends UserDefinedType\[Point\] { 

    def dataType = StructType\(Seq\( // Our native structure 

        StructField\("x", DoubleType\), 

        StructField\("y", DoubleType\) 

    \)\) 

    def serialize\(p: Point\) = Row\(p.x, p.y\) 

    def deserialize\(r: Row\) = 

        Point\(r.getDouble\(0\), r.getDouble\(1\)\) 

    }

注册此类型后，将在内部对象中识别出 Spark SQL 被要求转换为 DataFrames，并将被传递给在 Points 上定义的 UDF。此外，当缓存数据（将 x 和 y 压缩为单独的列）时，Spark SQL 将以柱形格式存储 Points，并且 Points 将可写入所有 Spark SQL 的数据源，这些数据源将它们看作 DOUBLE 对。我们在 Spark 的机器学习库中使用这个功能，如第5.2节所述。

# 5. Advanced Analytics Features

在这一部分中，我们描述了我们添加到 Spark SQL 中的三个特性，专门用于处理“大数据”环境中的挑战首先，在这些环境中，数据通常是非结构化的或半结构化的，尽管程序化地解析这些数据是可能的，为了让用户立即查询数据，Spark SQL 包括一个用于 JSON 和其他半结构化数据的模式推理算法;其次，大规模的处理往往是在数据的聚合和连接到机器学习之上，最后我们描述如何将 Spark SQL 整合到 Spark 的机器学习库\[26\]的新的高级 API 中，数据管道通常组合来自不同存储系统的数据，基于第4.4.1节中的数据源 API，Spark SQL 支持查询联合，允许单个程序有效地查询不同的源。这些特性都建立在 Catalyst 框架上。

### 5.1 Schema Inference for Semistructured Data

半结构化数据在大规模环境中很常见，因为随着时间的推移，易于生成和添加字段。在 Spark 用户中，我们已经看到输入数据的 JSON 使用率非常高。不幸的是，在 Spark 或 MapReduce 这样的过程环境中使用 JSON 是很麻烦的：大多数用户使用 ORM 类库（例如 Jackson \[21\]）将 JSON 结构映射到 Java 对象，或者尝试解析每个输入记录直接与较低级别的库。

在 Spark SQL 中，我们添加了一个 JSON 数据源，它从一组记录中自动推断出一个模式。例如，给定图5中的 JSON 对象，库推断出图6所示的模式。用户可以简单地将JSON文件注册为表，并使用根据路径访问字段的语法查询它，例如：

    SELECT loc.lat, loc.long FROM tweets WHERE text LIKE ’%Spark%’ AND tags IS NOT NULL

![](/img/spark_paper/2015 Spark SQL Relational Data Processing in Spark/figure5-6.png)

我们的模式推理算法在数据的一次传递中工作，并且如果需要也可以在数据的样本上运行。它与 XML 和对象数据库的模式推理的先前工作相关\[9,18,27\]，但是更简单，因为它只推断静态树结构，而不允许在任意深度递归嵌套元素。

具体来说，算法尝试 STRUCT 类型，每个类型可以包含原子，阵列或其他 STRUCT。对于由与根JSON对象（例如，tweet.loc.latitude）的不同路径定义的每个字段，算法找到与该字段的观察到的实例匹配的最具体的 Spark SQL 数据类型。例如，如果该字段的所有出现都是适合32位的整数，它将推断 INT;如果它们较大，它将使用 LONG（64位）或 DECIMAL（任意精度）;如果还有小数值，它将使用 FLOAT。对于显示多个类型的字段，Spark SQL 使用 STRING 作为最通用的类​​型，保留原始的 JSON 表示形式。对于包含数组的字段，它使用相同的“最特定的超类型”逻辑来确定来自所有观察到的元素的元素类型。我们使用对数据的单个 reduce 操作实现这个算法，它从 schemata 类型），并使用关联的“最具体的超类型”函数（它综合了每个字段的类型）来合并它们。这使得算法单通和高效通信，因为在每个节点本地发生高度的降低。

作为一个简短的例子，注意在图5和6中，算法概括了 loc.lat 和 loc.long 的类型。每个字段在一个记录中显示为整数，在另一个记录中显示为浮点数，因此算法返回 FLOAT。还要注意如何为标签字段，算法推断不能为 null 的字符串数组。

在实践中，我们发现这个算法可以很好地与现实世界的 JSON 数据集。例如，它正确地标识了来自 Twitter 的 firehose 的 JSON  tweets 的可用模式，其中包含大约100个不同字段和高度嵌套。多个 Databricks 的客户也已经成功地将其应用于其内部 JSON 格式。

在 SparkSQL 中，用于扩展 Python 对象的 RDDs 的算法（参见第3节），因为 Python 不是统计类型的，所以 RDD 可以包含多个对象类型。将来，我们计划为 CSV 文件和 XML 添加类似的推断。开发人员已经发现能够将这些类型的数据集视为表，并立即查询它们或将它们与其他对其生产力非常有价值的数据相结合。

### 5.2 Integration with Spark’s Machine Learning Library

作为 Spark SQL 在其他 Spark 模块中的实用程序的一个例子，ML-lib，Spark 的机器学习库，介绍了一个使用 DataFrames 的新的高级 API \[26\]。这个新的 API 基于机器学习管道概念，一种在其他高级 ML 库中的抽象，如 SciKit-Learn \[33\]。管道是对数据进行变换的图形，例如特征提取，归一化，降维和模型训练，每个交换数据集。管道是一个有用的抽象，因为 ML 工作流有许多步骤;将这些步骤表示为可组合元素，使得可以容易地更改流水线的部分或在整个工作流程的级别搜索调整参数。

为了在流水线阶段之间交换数据，MLlib 的开发人员需要一种紧凑（因为数据集可以很大）而且灵活的格式，允许为每个记录存储多种类型的字段。例如，用户可以从包含文本字段以及数字字段的记录开始，然后在文本上运行特征化算法（例如 TF-IDF）以将其转换为向量，将其他字段之一归一化，执行维度减少整个特征集等。为了表示数据集，新的 API 使用 DataFrames，其中每个列表示数据的特征。可以在管道中调用的所有算法都采用输入列和输出列的名称，因此可以在字段的任何子集上调用，并生成新的。这使得开发人员可以轻松地构建复杂的管道，同时保留每个记录的原始数据。为了说明 API，图7显示了一个短管道和创建的 DataFrames 的模式。

![](/img/spark_paper/2015 Spark SQL Relational Data Processing in Spark/figure7.png)

MLlib 使用 Spark SQL 的主要工作是为向量创建用户定义的类型。该向量 UDT 可以存储稀疏和稠密向量，并将它们表示为四个原始字段：类型（密集或稀疏）的布尔值，向量的大小，索引数组（用于稀疏坐标）和双值的数组（稀疏向量的非零坐标或其他所有坐标）。除了用于跟踪和操纵列的 DataFrames 的实用程序，我们还发现它们有用的另一个原因：他们使得 MLlib 的新 API 更容易暴露在所有的 Spark 的支持的编程语言。以前，MLlib 中的每个算法都使用对于领域特定概念的对象（例如，用于分类的标记点或者用于推荐的（用户，产品）评级），并且这些类中的每一个必须在各种语言（例如，从 Scala 复制到 Python）。在任何地方使用DataFrames 使得所有语言的所有算法都更加简单，因为我们只需要 Spark SQL 中的数据转换，它们已经存在。这是特别重要的，因为 Spark 为新的编程语言添加了绑定。

最后，使用 DataFrames 存储在 MLlib 中也使得它很容易暴露其所有的 SQL 算法。我们可以简单地定义一个 MADlib 风格的 UDF，如第3.7节所述，它将在内部调用表上的算法。我们还在探索 API 以暴露 SQL 中的管道构造。

### 5.3 Query Federation to External Databases

从异构资源合并的数据路径。例如，推荐流水线可以将流量日志与用户简档数据库和用户的社交媒体流组合。由于这些数据源通常驻留在不同的机器或地理位置，纯粹地查询它们可能是非常昂贵的。 Spark SQL 数据源利用 Catalyst 尽可能地将谓词推入数据源。

例如，以下内容使用 JDBC 数据源和 JSON 数据源将两个表连接在一起，以查找最近注册用户的流量日志。方便地，两个数据源可以自动推断该模式，而无需用户定义它。 JDBC 数据源还会将过滤谓词向下推入 MySQL，以减少传输的数据量。

    CREATE TEMPORARY TABLE users USING jdbc 

    OPTIONS\(driver "mysql" url "jdbc:mysql://userDB/users"\) 



    CREATE TEMPORARY TABLE logs 

    USING json OPTIONS \(path "logs.json"\) 



    SELECT users.id, users.name, logs.message 

    FROM users JOIN logs 

    WHERE users.id = logs.userId AND users.registrationDate &gt; "2015-01-01"

在引擎下，JDBC 数据源使用第4.4.1节中的 PrunedFiltered-Scan 接口，它为这些列提供了请求的列名和简单谓词（等式，比较和 IN 子句）。在这种情况下，JDBC 数据源将在 MySQL 上运行以下查询：

    SELECT users.id, users.name FROM users WHERE users.registrationDate &gt; "2015-01-01"

在未来的 Spark SQL 版本中，我们还希望为键值存储（如 HBase 和 Cassandra）添加预定下推式，这些存储支持有限形式的过滤。

# 6. Evaluation

我们在两个维度上评估 Spark SQL 的性能：SQL 查询处理性能和 Spark 程序性能。特别是，我们演示了 Spark SQL 的可扩展架构不仅能够提供更丰富的功能集，而且相对于以前的基于 Spark 的 SQL 引擎提供了显着的性能改进。此外，对于 Spark 应用程序开发人员，DataFrame API 可以使本地 Spark API 具有更高的速度，同时使 Spark 程序更简洁，更容易理解。最后，组合关系和过程查询的应用程序在集成 Spark SQL 引擎上运行速度比将 SQL 和过程代码作为单独的并行作业运行速度更快。

### 6.1 SQL Performance

我们使用 AMPLab 大数据基准\[3\]比较了 Spark SQL 与 Shark 和 Imapa 的性能\[3\]，它使用 Pavlo 等人开发的网络分析工作负载。 \[31\]。基准包含四种类型的查询，它们具有不同的参数，包括扫描，聚合，联接和基于 UDF 的 MapReduce 作业。我们使用了一个六个 EC2 i2.xlarge 机器（一个主机，五个工人）的集群，每个机器有4个内核，30 GB 内存和一个800 GB SSD，运行 HDFS 2.4，Spark 1.3，Shark 0.9.1 和 Impala 2.1.1 。数据集是使用柱形 Parquet 格式压缩后的 110 GB 数据\[5\]。

图8显示了每个查询的结果，按查询类型分组。查询1-3具有不同的参数，改变它们的选择性，其中 1a，2a 等等是选择性的，并且 1c，2c 等等是最少选择性的并且处理更多的数据。查询4使用了 Impala 中未直接支持的基于 Python 的 Hive UDF，但很大程度上受到 UDF 的 CPU 成本的约束。

![](/img/spark_paper/2015 Spark SQL Relational Data Processing in Spark/figure8.png)

我们看到在所有查询中，Spark SQL 比 Shark 快得多，并且通常与 Impala 具有竞争力。与 Shark 的区别的主要原因是 Catalyst 中的代码生成（第4.4.3节），它减少了 CPU 开销。在许多这些查询中，此功能与 SP 和 C ++ 和基于 LLVM 的 Impala 引擎竞争。与 Impala 最大的差距在查询 3a 中，Impala 选择更好的连接计划，因为查询的选择性使得其中一个表非常小。

### 6.2 DataFrames vs. Native Spark Code

除了运行 SQL 查询之外，Spark SQL 还可以帮助非 SQL 开发人员通过 DataFrame API 编写更简单和更高效的 Spark 代码. Catalyst 可以对 DataFrame 操作执行优化，这些操作很难手写代码。例如谓词下推，流水线和自动连接选择。即使没有这些优化，DataFrame API 也会由于代码生成而导致更高效的执行。对于 Python 应用程序尤其如此，因为 Python 通常比 JVM 慢。

Forth 是评估，我们比较了执行分布式聚合的 Spark 程序的两个实现。数据集由10亿个整数对（a，b）组成，具有100,000个不同的 a 值，在与上一节中相同的五个 worker i2.xlarge 集群上。我们测量计算 a 的每个值的 b 的平均值所花费的时间。首先，我们来看一个使用 Python API for Spark 中的 map 和 reduce 函数计算平均值的版本：

    sum\_and\_count = \ 

        data.map\(lambda x: \(x.a, \(x.b, 1\)\)\) \ 

            .reduceByKey\(lambda x, y: \(x\[0\]+y\[0\], x\[1\]+y\[1\]\)\) \ 

            .collect\(\) 

    \[\(x\[0\], x\[1\]\[0\] / x\[1\]\[1\]\) for x in sum\_and\_count\]

相反，相同的程序可以写成使用 DataFrame API 的简单操作：

    df.groupBy\("a"\).avg\("b"\)

图9显示，DataFrame 版本的代码在 12x 版本的基础上表现出手写的 Python 版本，除了更加简洁外。这是因为在 DataFrame API 中，只有逻辑计划在 Python 中构建，所有物理执行都被编译为本机 Spark 代码作为 JVM 字节码，从而实现更高效的执行。事实上，DataFrame 版本的性能也超过了上面 2x 的 Spark 代码的 Scala 版本。这主要是由于代码生成：DataFrame 版本中的代码避免了在手写 Scala 代码中出现的键值对的昂贵分配。

![](/img/spark_paper/2015 Spark SQL Relational Data Processing in Spark/figure9.png)

### 6.3 Pipeline Performance

DataFrame API 还可以提高结合关系和过程处理的应用程序的性能，通过让开发人员将所有操作写入单个程序并跨关系和过程代码管道计算。作为一个简单的例子，我们考虑一个两阶段流水线，从一个语料库中选择一个子集的文本消息，并计算最常用的单词。虽然非常简单，但它可以模拟一些真实世界的流水线，例如，计算特定人口统计信息中使用的最流行的词汇。

在这个实验中，我们在 HDFS 中生成了10亿条消息的合成数据集。每封邮件平均包含10个来自英语字典的单词。流水线的第一阶段使用关系过滤器来选择大约 90％ 的消息。第二阶段计算字数。

首先，我们使用单独的 SQL 查询，然后是基于 Scala 的 Spark 作业来实现管道，这可能发生在环境关系和过程引擎（例如 Hive 和 Spark）中。然后，我们使用 DataFrame API 实现了一个组合管道，即使用 DataFrame 的关系运算符来执行过滤器，并使用 RDD API 对结果执行字数计算。与第一个管道相比，第二个管道避免了将 SQL 查询的整个结果保存到 HDFS 文件作为中间数据集，然后再将其传递到 Spark 作业中的成本，因为 SparkSQL 使用关系运算符用于过滤。图10比较了两种方法的运行时性能。除了更容易理解和操作之外，基于 DataFrame 的管道还可以将性能提高2倍。

![](/img/spark_paper/2015 Spark SQL Relational Data Processing in Spark/figure10.png)

# 7. Research Applications

除了立即实现的 Spark SQL 的实际生产用例，我们还看到了研究更多实验项目的研究人员的极大兴趣。我们概述了利用 Catalyst 的可扩展性的两个研究项目：一个在近似查询处理中，一个在基因组学中。

### 7.1 Generalized Online Aggregation

Zeng et al. 已经使用 Catalyst 在他们的工作中提高在线聚合的生成性\[40\]。此工作概括了在线聚合的执行，以支持任意嵌套的聚合查询。它允许用户通过查看在总数据的一小部分上计算的结果来查看执行查询的进度。这些部分结果还包括精度测量，允许用户在达到足够的精度时停止查询。

为了在 Spark SQL 中实现此系统，自动化操作符表示断开上下采样的批处理的相关性。在查询计划期间，使用多个查询替换原始完整查询，每个查询对数据的连续采样进行操作。

然而，简单地用样本替换完整数据集不足以以在线方式计算正确答案。标准聚集与标准聚集相比，考虑了当前样本和以前批次的结果。此外，可能基于近似答案过滤出元组的操作必须被替换为可以考虑当前估计误差的版本。

这些转换中的每一个都可以表示为修改运算符树的 Catalyst 规则，直到它产生正确的在线转换。不基于采样数据的树片段由这些规则来管理，并且可以使用标准代码路径执行。通过使用 Spark SQL 作为基础，作者能够在大约2000行代码中实现一个相当完整的原型。

### 7.2 Computational Genomics

计算基因组学中的常见操作包括基于数值偏移检查重叠区域。这个问题可以表示为具有不等式谓词的连接。考虑两个数据集 a 和 b，其模式为（start LONG，end LONG）。范围连接操作可以在 SQL 中表示如下：

    SELECT \* FROM a JOIN b

    WHERE a.start &lt; a.end

        AND b.start &lt; b.end

        AND a.start &lt; b.start

        AND b.start &lt; a.end

没有特殊优化，前面的查询将被许多使用低效算法的系统执行，例如嵌套循环连接。相反，专用系统可以使用间隔树来计算对该连接的答案。 ADAM 项目\[28\]的研究人员能够在一个版本的 Spark SQL 中构建一个特殊的规划规则，以便有效地执行这样的计算，允许他们利用标准的数据操作能力和专门的处理代码。所需的更改大约是100行代码。

# 8. Related Work

### Programming Model

几个系统试图将定位处理与最初用于大型集群的过程处理引擎相结合。其中，Shark \[38\] 最接近 Spark SQL，运行在相同的引擎上，提供了相同的关系查询和高级分析组合。 Spark SQL 通过更丰富和更多的程序员友好的 API（DataFrames）改进了 Shark，其中可以使用主机编程语言中的构造以模块化方式组合查询（参见第3.4节）。它还允许直接在本机 RDD 上运行关系查询，并支持除 Hive 之外的各种数据源。

启发 Spark SQL 设计的一个系统是 DryadLINQ \[20\]，它将 C＃ 中的语言集成查询编译为分布式 DAG 执行引擎。 LINQ 查询也是关系型的，但可以直接对 C＃ 对象进行操作。 Spark SQL 超越了 DryadLINQ，提供了一个类似于通用数据科学库\[32,30\]的 DataFrame 接口，一个用于数据源和类型的API，以及通过在Spark上执行支持迭代算法。

其他系统在内部仅使用关系数据模型，并将过程代码与 UDF 相关。例如，Hive 和 Pig \[36，29\] 提供关系查询语言，但是广泛使用 UDF 接口。 ASTERIX \[8\] 内部有一个半结构化数据模型。 Stratosphere \[2\] 也有一个半结构化模型，但提供了在 Scala 和 Java 的 API，让用户轻松调用 UDF。 PIQL \[7\] 同样提供了 Scala DSL。与这些系统相比，Spark SQL 通过能够直接在用户定义的类（本地 Java / Python 对象）中查询数据，并且允许开发人员以相同的语言混合过程和关系 API，与本地 Spark 应用程序更紧密地集成。此外，通过 Catalyst 优化器，Spark SQL 实现了大多数大型计算框架中不存在的优化（例如代码生成）和其他功能（例如，用于 JSON 和机器学习数据类型的模式推断）。我们相信这些功能对于为大数据提供一个集成，易于使用的环境至关重要。

最后，已经为单个机器\[32,30\]和簇\[13,10\]建立了数据帧 API。与以前的 API 不同，Spark SQL 使用关系优化器优化 DataFrame 计算。

### Extensible Optimizers

Catalyst 优化器与可扩展优化器框架（如 EXODUS \[17\] 和 Cascades \[16\]）具有相似的目标。然而，传统上，优化器框架需要一个特定领域的语言来编写规则，以及一个“优化器编译器”将它们转换为可运行代码。我们的主要改进是使用功能性编程语言的标准特性来构建我们的优化器，它们提供相同（通常更大）的表达性，同时减少维护负担和学习曲线。高级语言特性有助于 Catalyst 的许多领域 - 例如，我们使用 quasiquotes 代码生成的方法（第4.3.4节）是我们知道的最简单和最可组合的方法之一。虽然可扩展性难以定量测量，一个有希望的迹象表明，Spark SQL 在发布后的前8个月有超过50个外部贡献者。

对于代码生成，LegoBase \[22\] 最近提出了一种在 Scala 中使用生成式编程的方法，这将有可能在 Catalyst 中使用而不是 quasiquotes。

### Advanced Analytics

Spark SQL 基于最近的工作，在大型集群上运行高级分析算法，包括迭代算法和图形分析\[15,24\]的平台。与 MADlib \[12\] 共享暴露分析功能的愿望，尽管这种方法是不同的，因为 MADlib 必须使用 Postgres UDF 的有限接口，而 Spark SQL 的 UDF 可以是完全成熟的Spark program。最后，包括新的和不可见的加载\[35,1\]的技术试图提供和优化半结构化数据如 JSON 的查询。我们希望在我们的 JSON 数据源中应用这些技术。

# 9. Conclusion

我们提供了 Spark SQL，Apache Spark 中的一个新模块，提供与关系处理的丰富集成。 Spark SQL 使用声明式 DataFrame API 扩展 Spark，以允许进行相关处理，提供了诸如自动优化的优点，并允许用户编写复杂的管道来混合关系和复杂的分析。它支持针对大规模数据分析定制的各种功能，包括半结构化数据，查询联合和机器学习的数据类型。为了启用这些功能，Spark SQL 基于一个名为 Catlylyst 的可扩展优化器，可以通过嵌入到 Scala 编程语言中轻松添加优化规则，数据源和数据类型。用户反馈和基准显示，Spark  SQL 使得编写数据管道的过程更加简单和高效，这些数据管道混合了关系和过程处理，同时提供了比以前的 SQL-on-Spark 引擎更高的速度。

Spark SQL is open source at http://spark.apache.org 

# 10. Acknowledgments

我们要感谢程浩，TayukaUeshin，TorMyklebust，Daoyuan Wang 和其他 Spark SQL 贡献者。我们还要感谢 John Cieslewicz 和 Google 团队的其他成员，以便早日讨论 Catalyst 优化器。作者 Franklin 和 Kaftan 的工作部分得到了 NSF CISE 探险奖 CCF-1139158，LBNL 奖7076018和 DARPA XData 奖 FA8750-12-2-0331 的支持，以及来自 Amazon Web Services，Google，SAP，The Thomas 和 Stacey Siebel 基金会，Adatao，Adobe，苹果公司，蓝色 Goji，博世，C3Energy，思科，Cray，Cloudera，EMC2，爱立信，Facebook，Guavus，华为，Informatica，英特尔，微软，NetApp，Pivotal，三星，斯伦贝谢，Splunk，Virdata 和 VMware。

# 11. References

\[1\] A. Abouzied, D. J. Abadi, and A. Silberschatz. Invisible

loading: Access-driven data transfer from raw files into

database systems. In EDBT, 2013.

\[2\] A. Alexandrov et al. The Stratosphere platform for big data

analytics. The VLDB Journal, 23\(6\):939–964, Dec. 2014.

\[3\] AMPLab big data benchmark.

https://amplab.cs.berkeley.edu/benchmark.

\[4\] Apache Avro project. http://avro.apache.org.

\[5\] Apache Parquet project. http://parquet.incubator.apache.org.

\[6\] Apache Spark project. http://spark.apache.org.

\[7\] M. Armbrust, N. Lanham, S. Tu, A. Fox, M. J. Franklin, and

D. A. Patterson. The case for PIQL: a performance insightful

query language. In SOCC, 2010.

\[8\] A. Behm et al. Asterix: towards a scalable, semistructured

data platform for evolving-world models. Distributed and

Parallel Databases, 29\(3\):185–216, 2011.

\[9\] G. J. Bex, F. Neven, and S. Vansummeren. Inferring XML

schema definitions from XML data. In VLDB, 2007.

\[10\] BigDF project. https://github.com/AyasdiOpenSource/bigdf.

\[11\] C. Chambers, A. Raniwala, F. Perry, S. Adams, R. R. Henry,

R. Bradshaw, and N. Weizenbaum. FlumeJava: Easy,

efficient data-parallel pipelines. In PLDI, 2010.

\[12\] J. Cohen, B. Dolan, M. Dunlap, J. Hellerstein, and

C. Welton. MAD skills: new analysis practices for big data.

VLDB, 2009.

\[13\] DDF project. http://ddf.io.

\[14\] B. Emir, M. Odersky, and J. Williams. Matching objects with

patterns. In ECOOP 2007 – Object-Oriented Programming,

volume 4609 of LNCS, pages 273–298. Springer, 2007.

\[15\] J. E. Gonzalez, R. S. Xin, A. Dave, D. Crankshaw, M. J.

Franklin, and I. Stoica. GraphX: Graph processing in a

distributed dataflow framework. In OSDI, 2014.

\[16\] G. Graefe. The Cascades framework for query optimization.

IEEE Data Engineering Bulletin, 18\(3\), 1995.

\[17\] G. Graefe and D. DeWitt. The EXODUS optimizer

generator. In SIGMOD, 1987.

\[18\] J. Hegewald, F. Naumann, and M. Weis. XStruct: efficient

schema extraction from multiple and large XML documents.

In ICDE Workshops, 2006.

\[19\] Hive data definition language.

https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL.

\[20\] M. Isard and Y. Yu. Distributed data-parallel computing

using a high-level programming language. In SIGMOD,

2009.

\[21\] Jackson JSON processor. http://jackson.codehaus.org.

\[22\] Y. Klonatos, C. Koch, T. Rompf, and H. Chafi. Building

efficient query engines in a high-level language. PVLDB,

7\(10\):853–864, 2014.

\[23\] M. Kornacker et al. Impala: A modern, open-source SQL

engine for Hadoop. In CIDR, 2015.

\[24\] Y. Low et al. Distributed GraphLab: a framework for

machine learning and data mining in the cloud. VLDB, 2012.

\[25\] S. Melnik et al. Dremel: interactive analysis of web-scale

datasets. Proc. VLDB Endow., 3:330–339, Sept 2010.

\[26\] X. Meng, J. Bradley, E. Sparks, and S. Venkataraman. ML

pipelines: a new high-level API for MLlib.

https://databricks.com/blog/2015/01/07/ml-pipelines-a-new-

high-level-api-for-mllib.html.

\[27\] S. Nestorov, S. Abiteboul, and R. Motwani. Extracting

schema from semistructured data. In ICDM, 1998.

\[28\] F. A. Nothaft, M. Massie, T. Danford, Z. Zhang, U. Laserson,

C. Yeksigian, J. Kottalam, A. Ahuja, J. Hammerbacher,

M. Linderman, M. J. Franklin, A. D. Joseph, and D. A.

Patterson. Rethinking data-intensive science using scalable

analytics systems. In SIGMOD, 2015.

\[29\] C. Olston, B. Reed, U. Srivastava, R. Kumar, and

A. Tomkins. Pig Latin: a not-so-foreign language for data

processing. In SIGMOD, 2008.

\[30\] pandas Python data analysis library.

http://pandas.pydata.org.

\[31\] A. Pavlo et al. A comparison of approaches to large-scale

data analysis. In SIGMOD, 2009.

\[32\] R project for statistical computing. http://www.r-project.org.

\[33\] scikit-learn: machine learning in Python.

http://scikit-learn.org.

\[34\] D. Shabalin, E. Burmako, and M. Odersky. Quasiquotes for

Scala, a technical report. Technical Report 185242, École

Polytechnique Fédérale de Lausanne, 2013.

\[35\] D. Tahara, T. Diamond, and D. J. Abadi. Sinew: A SQL

system for multi-structured data. In SIGMOD, 2014.

\[36\] A. Thusoo et al. Hive–a petabyte scale data warehouse using

Hadoop. In ICDE, 2010.

\[37\] P. Wadler. Monads for functional programming. In Advanced

Functional Programming, pages 24–52. Springer, 1995.

\[38\] R. S. Xin, J. Rosen, M. Zaharia, M. J. Franklin, S. Shenker,

and I. Stoica. Shark: SQL and rich analytics at scale. In

SIGMOD, 2013.

\[39\] M. Zaharia et al. Resilient distributed datasets: a

fault-tolerant abstraction for in-memory cluster computing.

In NSDI, 2012.

\[40\] K. Zeng et al. G-OLA: Generalized online aggregation for

interactive analysis on big data. In SIGMOD, 2015.

