# Abstract

我们提供弹性分布式数据集（RDDs），分布式内存抽象，允许程序员以容错方式在大型集群上执行内存中计算。 RDD 是由当前计算框架无效处理的两种类型的应用程序激励的：迭代算法和交互式数据挖掘工具。在这两种情况下，将数据保存在内存中可以提高性能一个数量级。为了有效地实现容错，RDD 提供了一种限制形式的共享内存，基于粗粒变换，而不是对共享状态的细粒度更新。然而，我们表明 RDD 足够表达以捕获大量的计算，包括最近的专门的程序设计迭代作业的模型，如 Pregel，以及这些模型没有捕获的新应用程序。我们在一个称为 Spark 的系统中实现了 RDD，我们通过各种用户应用程序和基准来评估。

# 1. Introduction

集群计算框架如 MapReduce \[10\] 和 Dryad \[19\] 已被广泛应用于大规模数据分析。这些系统允许用户使用一组高级操作符来编写并行计算，而不必担心工作分布和容错。

虽然当前框架提供了访问集群的计算资源的大量抽象，但是它们缺乏利用分布式存储器的抽象。这使得它们对于一类重要的新兴应用程序无效：那些在多个计算中重用中间结果的应用程序。数据重用在许多迭代机器学习和图形算法中是常见的，包括 PageRank，K-means 聚类和逻辑回归。另一个引人注目的用例是交互式数据挖掘，其中用户在数据的相同子集上运行多个自组织查询。不幸的是，在大多数当前框架中，在计算之间（例如，在两个 MapReduce 作业之间）重用数据的唯一方式是将其写入外部稳定存储系统，例如分布式文件系统。这在由于数据复制而导致的大量开销中，磁盘 I / O 和序列化，这可以支配应用程序执行时间。

意识到这个问题，研究人员为需要数据重用的某些应用程序开发了专门的框架。例如，Pregel \[22\] 是一个用于在存储器中保留中间数据的迭代图计算的系统，而 HaLoop \[7\] 提供了一个迭代的 MapReduce 接口。但是，这些框架仅支持特定的计算模式（例如，循环一系列 MapReduce 步骤），并为这些模式隐式执行数据共享。它们不提供用于更一般再使用的抽象，例如，以使用户将若干数据集加载到存储器中并跨它们运行即席查询。

在本文中，我们提出了一种称为弹性分布式数据集（RDDs）的新抽象，可以在广泛的应用程序中有效地重用数据。 RDD 是容错的并行数据结构，它允许用户将中间结果明确地保存在内存中，控制它们的分区以优化数据放置，并使用丰富的运算符操作它们。

设计 RDD 的主要挑战是定义一个可以有效提供容错的编程接口。内存存储群集的现有抽象（如分布式共享内存\[24\]，键值存储\[25\]，数据库和 Piccolo \[27\]）基于对可变状态（例如，表中的单元格）的细粒度更新提供了一个接口。使用此接口，提供容错的唯一方法是跨机器复制数据或跨机器记录更新。这两种方法对于数据密集型工作负载都是昂贵的，因为它们需要复制大量的数据以转移集群的网络，其带宽远低于RAM的带宽，并且它们招致大量的存储开销。

与这些系统相反，RDD 提供基于对许多数据项应用相同操作的粗粒变换（例如，map，filter 和 join）的接口。这允许它们通过记录用于构建数据集（其血统）而不是实际数据的变换来高效地提供容错。如果 RDD 的分区丢失，则 RDD 具有关于如何从其他 RDD 存储仅计算该分区。因此，丢失的数据可以被恢复，通常相当快速，而不需要昂贵的复制。

虽然基于粗粒度变换的接口可能最初看起来有限，但 RDD 对于许多并行应用程序是很好的，因为这些应用程序自然对多个数据项应用相同的操作。实际上，我们表明，RDDs 可以有效地表达许多具有提出分离系统的集群编程模型，包括 MapReduce，DryadLINQ，SQL，Pregel 和 HaLoop，以及这些系统不捕获的新应用程序，如交互式数据挖掘。 RDD 满足计算需求的能力，以前只是通过引入新的框架是我们相信，RDD 抽象的力量的最可靠的证据。

实现了 RDD 系统调用，其用于在加州大学伯克利分校和几家公司的研究和生产应用。 Spark 提供了一种方便的语言集成编程接口，类似于在编程语言\[2\]中的 DryadLINQ \[31\]。此外，Spark 可以交互式地用于从 Scala 解释器查询大数据集。我们相信 Spark 是第一个系统，允许通用编程语言以交互速度用于集群上的内存中数据挖掘。

我们通过微基准和用户应用的测量来评估 RDD 和 Spark。我们显示 Spark 的迭代应用程序的速度比 Hadoop 快20倍，速度supareal世界数据分析报告由40×，并可以积极地使用5-7s延迟扫描1 TB数据集。更基本的是，为了说明RDD的通用性，我们在Spark之上实现了Pregel和HaLoop编程模型，包括它们使用的放置优化，作为相对较小的库（每行200行代码）。

我们通过微基准和用户应用的测量来评估 RDD 和 Spark。我们显示 Spark 对于迭代应用程序的速度比 Hadoop 快20倍，可以将现实世界的数据分析报告加速40倍，并且可以主动使用5-7秒的延迟扫描1 TB 数据集。更基本的是，为了说明 RDD 的通用性，我们在 Spark 之上实现了 Pregel 和 HaLoop 编程模型，包括它们使用的放置优化，作为相对较小的库（每行200行代码）。

本文首先概述了 RDDs（§2）和 Spark（§3）。然后讨论 RDDs 的内部表示（§4），我们的实现（§5）和实验结果（§6）。最后，我们讨论 RDD 如何捕获几个现有的群集编程模型（§7），调查相关工作（§8），并得出结论。

# 2. Resilient Distributed Datasets \(RDDs\)

本节提供 RDD 的概述。我们首先定义 RDD（§2.1），并在 Spark 中描述程序接口（§2.2）。然后我们比较 RDDs 与自适应共享内存抽象（§2.3）。最后，我们讨论 RDD 模型的限制（§2.4）。

# 2.1 RDD Abstraction

形式上，RDD 是只读的，分区的记录集合。 RDD 只能通过（1）稳定存储中的数据或（2）其他 RDD 中的确定性操作来创建。这些操作变换将它们与 RDD 上的其他操作区分开来。转换的示例包括map，filter 和 join。

RDD 不需要在任何时候都实现。相反，RDD 有足够的信息来说明它是如何从其他数据集（其血统）中导出的，以便从稳定存储中的数据计算其分区。这是一个强大的属性：本质上，程序不能引用它不能在失败后重建的 RDD。

最后，用户可以控制 RDD 的其他两个方面：持久性和分区。用户可以指示它们将重用哪些 RDD，并为它们选择存储策略（例如，内存存储）。他们还可以要求 RDD 的元素基于每个记录中的密钥跨机器分区。这对于放置优化很有用，例如确保将以相同方式对将要连接在一起的两个数据集进行哈希分区。

### 2.2 Spark Programming Interface

Spark 通过类似于 DryadLINQ \[31\] 和 Flume Java \[8\] 的语言集成 API 来公开 RDD，其中每个数据集表示为一个对象，并且使用这些对象上的方法调用转换。

程序员通过对稳定存储（例如，map 和 filter）中的数据进行变换来定义一个或多个 RDD。然后，他们可以在操作中使用这些 RDD，这些操作是向应用程序返回值或将数据导出到存储系统的操作。操作的示例包括 count（返回数据集中的元素数），collect（返回元素本身）和 save（将数据集输出到存储系统）。像 DryadLINQ 一样，Spark 首次在一个动作中使用 RDD，因此它可以延迟计算 RDD，以便它可以管道转换。

此外，程序员可以调用持久方法来指示他们在未来操作中要重用哪些 RDD。 Spark 会默认在内存中保留 RDD，但如果没有足够的 RAM，它可能会将它们溢出到磁盘.Userscanals可以用于存储策略，例如将RDD存储在磁盘上或通过标记保存在机器上。最后，用户可以在每个RDD上设置持久性优先级，以指定哪些内存中的数据首先溢出到磁盘。

此外，程序员可以调用 persist 方法来指示他们在未来操作中要重用哪些 RDD。默认情况下，Spark 会在内存中保留持久 RDD，但如果没有足够的 RAM，它会将它们溢出到磁盘。用户还可以请求其他持久性策略，例如将 RDD 仅存储在磁盘上或通过机器上的标记来保存 RDD。最后，用户可以在每个 RDD 上设置持久性优先级，以指定哪些内存中的数据首先溢出到磁盘。

### 2.2.1 Example: Console Log Mining

假设 Web 服务遇到错误，并且操作员想要搜索 Hadoop 文件系统（HDFS）中的兆字节的日志以找到原因。使用 Spark，主机可以将来自日志的错误消息仅加载到一组节点中的 RAM 中并查询它们交互式。她首先键入以下 Scala 代码：

![](/img/spark_paper/Resilient Distributed Datasets A Fault Tolerant Abstraction for In-Memory Cluster Computing/figure1.png)

第1行定义由 HDFS 文件支持的 RDD（作为文本行的集合），而第2行从其导出过滤的 RDD。然后，第3行要求错误在内存中持久化，以便可以在查询之间共享。请注意，filter 的参数是一个闭包的 Scala 语法。

在这一点上，没有对集群执行任何工作。然而，用户现在可以在 action 中使用 RDD，例如，计数消息的数量：

errors.count\(\)

用户还可以对 RDD 执行进一步的转换，并使用它们的结果，如下面几行：

![](/img/spark_paper/Resilient Distributed Datasets A Fault Tolerant Abstraction for In-Memory Cluster Computing/code1.png)

在涉及错误运行的第一个操作之后，Spark 将错误的分区存储在内存中，大大加快了其上的后续计算。请注意，基本 RDD，行，不会加载到 RAM 中。这是可取的，因为错误消息可能只是数据的一小部分（足够小以适合存储器）。

最后，为了说明我们的模型如何实现故障容错，我们在图1的第三个查询中显示 RDDs 的沿袭血统图。在这个查询中，我们从错误开始，在线上的滤波器的结果，并应用毛皮 - 进行过滤和映射之前运行一个收集。 Spark 调度器将管道后两个转换，并发送一组任务来计算它们到持有缓存的错误分区的节点。此外，如果错误的分区丢失，Spark 将通过仅对行的相应分区应用过滤器来重建它。

最后，为了说明我们的模型如何实现容错，我们在图1中的第三个查询中显示 RDD 的沿袭血统图。在这个查询中，我们从错误开始，在将每行中过滤的结果进行 collect 之前先执行 filter 和 map 操作。 Spark 调度器将管道后两个转换，并发送一组任务来计算它们到持有缓存的错误分区的节点。此外，如果错误的分区丢失，Spark 将通过仅对行的相应分区应用 filter 来重建它。

### 2.3 Advantages of the RDD Model

为了理解 RDD 作为分布式内存抽象的好处，我们将它们与表1中的分布式共享内存（DSM）进行比较。在 DSM 系统中，应用程序读取和写入全局地址空间中的任意位置。在下面的定义中，我们不仅包括传统的共享内存系统\[24\]，还有其他应用程序对共享状态进行细粒度写入的系统，包括提供共享 DHT 的 Piccolo \[27\] 和分布式数据库。 DSM 是一个非常一般的抽象，但是这种一般性使得更难以有效和容错的方式在商品集群上实现。

![](/img/spark_paper/Resilient Distributed Datasets A Fault Tolerant Abstraction for In-Memory Cluster Computing/table1.png)

RDDs 和 DSM 之间的主要区别是 RDD 只能通过粗粒度转换创建（“写入”），而 DSM 允许对每个存储器位置进行读取和写入。 这将 RDD 限制为执行批量写入的应用程序，但允许更有效的容错。特别地，RDD 不需要引起检查点的开销，因为它们可以使用沿袭血统来恢复。 此外，只有 RDD 的丢失分区需要在故障时重新计算，并且它们可以在不同节点上并行重新计算，而不必回滚整个程序。

RDDs 的第二个好处是，它们的不可变纳税系统通过运行慢任务的备份副本来减少系统丢失（stragglers），如 MapReduce \[10\]。使用 DSM 难以实现备份任务，因为任务的两个副本将访问相同的内存位置并干扰对方的更新。

最后，RDD 提供了另外两个优于 DSM 的好处。首先，在对 RDD 的批量操作中，运行时可以基于数据位置来调度任务，以提高性能。第二，当没有足够的内存来存储它们时，RDD 会正常降级，只要它们只用于基于扫描的操作。不适合在 RAM 中的分区可以存储在磁盘上，并且将提供与当前数据并行系统类似的性能。

### 2.4 Applications Not Suitable for RDDs

如简介中所讨论的，RDD 最适合于对数据集的所有元素应用相同操作的批处理应用程序。在这些情况下，RDD 可以有效地将每个变换记录在血统图中的一个步骤，并可以重新分配以分配记录大量数据。 RDD 将不太适合于对共享状态进行异步细粒度更新的应用程序，例如 Web 应用程序或增量 Web 搜寻器的存储系统。对于这些应用程序，使用执行传统更新记录和数据检查点的系统（例如数据库， RAMCloud \[25\]，Percolator \[26\] 和 Piccolo \[27\]）更为有效。我们的目标是为批处理分析提供一个高效的编程模型，并将这些异步应用程序留给专用系统。

# 3. Spark Programming Interface

Spark 通过类似于 Scala \[2\] 中 DryadLINQ \[31\] 的语言集成 API 提供 RDD 抽象，这是 Java VM 的静态类型的函数式编程语言。我们选择 Scala，因为它结合简洁（便于交互式使用）和效率（由于静态类型）。然而，没有关于 RDD 抽象需要一个功能语言。

要使用 Spark，开发人员编写一个连接到工作线程集群的驱动程序，如图2所示。驱动程序定义一个或多个 RDD 并调用它们。驱动程序上的 Spark 代码也跟踪 RDDs 的血统。工作程序是长期的进程，可以在操作中在 RAM 中存储 RDD 分区。

![](/img/spark_paper/Resilient Distributed Datasets A Fault Tolerant Abstraction for In-Memory Cluster Computing/figure2.png)

正如我们在第2.2.1节的日志挖掘示例中所展示的，用户通过传递闭包（函数文字）向 RDD 操作提供参数，如 map。 Scala 将每个闭包表示为 Java 对象，这些对象可以序列化并加载到另一个节点上，以在网络上传递闭包。 Scala 还将在闭包中绑定的任何变量保存为 Java 对象中的字段。例如，可以编写类似 var x = 5 的代码; rdd.map（\_ + x）为 RDD 的每个元素添加5。

RDD 本身是由元素类型参数化的静态类型对象。例如，RDD \[Int\] 是整数的 RDD。但是，我们的大多数示例省略了类型，因为 Scala 支持类型推断。

虽然我们在 Scala 中暴露 RDD 的方法在概念上很简单，但是我们必须使用反射解决 Scala 的闭包对象的问题\[33\]。我们还需要更多的工作使 Spark 可以从 Scala 解释器中使用，我们将在第5.2节中讨论。尽管如此，我们不必修改 Scala 编译器。

### 3.1 RDD Operations in Spark

表2列出了 Spark 中可用的主要 RDD 转换和操作。我们给出每个操作的签名，在方括号中显示类型参数。请注意，转换是定义新 RDD 的延迟操作，而动作启动计算以将值返回到程序或将数据写入外部存储器。

![](/img/spark_paper/Resilient Distributed Datasets A Fault Tolerant Abstraction for In-Memory Cluster Computing/table2.png)

注意，一些操作，如 join，只能在键值对的 RDD 上使用。此外，我们的函数名称被选择为匹配 Scala 和其他函数式语言中的其他 API;例如，map 是一对一映射，而 flatMap 将每个输入值映射到一个或多个输出（类似于 MapReduce 中的映射）。

除了这些运算符，用户可以要求 RDD 持久化。此外，用户可以获得 RDD 的分区顺序，由分区器类表示，并根据它分区另一个数据集。操作如 groupByKey，reduceByKey 和自动排序导致散列或范围分区的 RDD。

### 3.2 Example Applications

我们使用两个迭代应用程序来补充2.2.1节中的数据挖掘示例：逻辑回归和 PageRank。后者还展示了如何控制 RDDs 的分区可以提高性能。

### 3.2.1 Logistic Regression

许多机器学习算法本质上是迭代的，因为它们运行迭代优化过程，例如梯度下降，以最大化函数。因此，通过将其数据保存在内存中，它们可以运行得更快。

作为示例，以下程序实现了人为回归\[14\]，一种常见的分类算法，搜索最佳分离两组点（例如，垃圾邮件和非垃圾邮件）的超平面 w。算法使用梯度下降：它以随机值开始 w，并且在每次迭代中，它将 w 的函数与数据相加以在改进它的方向上移动 w。

![](/img/spark_paper/Resilient Distributed Datasets A Fault Tolerant Abstraction for In-Memory Cluster Computing/code2.png)

我们首先定义一个称为点的持久 RDD 作为对文本文件进行地图变换的结果，将每行文本解析为 Point 对象。然后我们重复地运行 map和 reduce on points，通过对当前 w 的函数求和来计算每个步骤的梯度。在内存中跨越迭代保持点可以产生20倍的加速，如我们在第6.1节中所示。

### 3.2.2 PageRank

更复杂的数据共享模式发生在 PageRank \[6\] 中。该算法通过将链接到它的文档的贡献相加来迭代地更新每个文档的排名。在每次迭代时，每个文档向其邻居发送 r n 的贡献，其中 r 是其秩，n 是其邻居的数目。然后它将其等级更新为 α/ N +（1-α）Σci，其中总和超过其接收的贡献，N 是文档的总数。我们可以在 Spark 中写如下的 PageRank：

    // Load graph as an RDD of \(URL, outlinks\) pairs

    val links = spark.textFile\(...\).map\(...\).persist\(\)

    var ranks = // RDD of \(URL, rank\) pairs

    for \(i &lt;- 1 to ITERATIONS\) {

        // Build an RDD of \(targetURL, float\) pairs

        // with the contributions sent by each page

        val contribs = links.join\(ranks\).flatMap {

        \(url, \(links, rank\)\) =&gt; links.map\(dest =&gt; \(dest, rank/links.size\)\)

    }

    // Sum contributions by URL and get new ranks

    ranks = contribs.reduceByKey\(\(x,y\) =&gt; x+y\).mapValues\(sum =&gt; a/N + \(1-a\)\*sum\)

    }

这个程序导致图3中的 RDD 谱系血统图。在每次迭代中，我们基于来自上一次迭代和静态链接数据集的贡献和排名创建一个新的秩数据集。这个图的一个有趣的特点是它随着迭代次数的增长而增长。因此，在具有许多迭代的作业中，可能有必要可靠地复制等级的一些版本以减少故障恢复时间\[20\]。用户可以使用 RELIABLE 标志调用 persist，这样做。但是，请注意，链接数据集不需要复制，因为它的分区可以通过在输入文件的块上重新运行映射来有效地重建。这个数据集通常比排名要大得多，因为每个文档都有许多链接，但只有一个数字作为排名，因此使用沿袭恢复它可以节省时间，超过检查程序的整个内存状态的系统。

![](/img/spark_paper/Resilient Distributed Datasets A Fault Tolerant Abstraction for In-Memory Cluster Computing/figure3.png)

最后，我们可以通过控制 RDD 的分区来优化 PageRank 中的通信。如果我们为链接指定分区（例如，通过跨节点的 URL 对链接列表进行哈希分区），我们可以以相同的方式分区排名，并确保链接和排名之间的联接操作不需要通信（因为每个 URL 的排名将在与其链接列表相同的机器上）。我们还可以编写一个自定义 Partitioner 类来将彼此链接到一起的页面分组（例如，按域名分区 URL）。当我们定义链接时，通过调用 partitionBy 可以显示两种优化：

    links = spark.textFile\(...\).map\(...\) .partitionBy\(myPartFunc\).persist\(\)

在这个初始调用之后，链接和排名之间的联接操作将自动将每个 URL 的贡献聚合到其链接列表所在的计算机上，计算其新排序，并将其与其链接相加。这种类型的跨迭代的一致分区是 Pregel 等专门框架中的主要优化之一。 RDDs 让用户直接表达这个目标。

# 4. Representing RDDs

提供 RDDs 作为抽象的挑战之一是为它们选择一种能够在宽范围的变换中跟踪谱系的表示。理想地，实现 RDD 的系统应当提供尽可能丰富的一组变换算子（例如，表2中的变换算子），并且允许用户以任意方式组合它们。我们提出了一个简单的基于图形的 RDDs 表示，以促进这些目标。我们在 Spark 中使用这种表示形式来支持各种各样的转换，而不需要为每个转换器添加特殊的逻辑，这大大简化了系统设计。

简而言之，我们建议通过公开五个信息的公共接口来表示每个 RDD：一组分区，其是数据集的原子部分;一组对父 RDD 的依赖关系;用于基于其子项计算数据集的函数;以及关于其分区方案和数据放置的元数据。例如，表示 HDFS 文件的 RDD 具有用于文件的每个块的分区，并且知道每个块在哪个机器上。同时，此 RDD 上的映射的结果具有相同的分区，但是当计算其元素时将映射函数应用于父数据。我们在表3中总结了这个接口。

![](/img/spark_paper/Resilient Distributed Datasets A Fault Tolerant Abstraction for In-Memory Cluster Computing/table3.png)

在设计这个接口时最有趣的问题是如何表示 RDD 之间的依赖关系。我们发现这对分类依赖性类型是足够和有用的：窄的依赖性，父 RDD 的任何分区被子 RDD 的至多一个部分使用，宽依赖性，其中多个子分区可能依赖于它。例如，map 导致一个狭窄的依赖，而 join 导致宽依赖（除非父对象被哈希分区）。图4显示了其他示例。

![](/img/spark_paper/Resilient Distributed Datasets A Fault Tolerant Abstraction for In-Memory Cluster Computing/figure4.png)

这种区分是有用的有两个原因。首先，窄依赖性允许在一个集群节点上进行流水线执行，该节点可以计算所有父分区。例如，可以在逐个元素的基础上应用 map 后面接着用 filter。相反，宽依赖关系要求所有父分区的数据可用，并使用类似 MapReduce 的操作在节点之间进行重排。第二，节点故障后的恢复对于窄依赖性更有效，因为只有丢失的父分区需要重新计算，并且它们可以在不同节点上并行重新计算。相比之下，在具有宽依赖性的沿袭血统图中，单个故障节点可能导致从 RDD 的所有祖先中丢失一些分区，从而需要完全重新执行。

RDD 的这种通用接口使得可以在少于20行的代码中实现 Spark 中的大多数变换。事实上，即使新的 Spark 用户在不知道调度器的细节的情况下实现了新的转换（例如，采样和各种类型的联接）。我们在下面描述一些 RDD 实现。

**HDFS files：**我们的样本中的输入 RDD 是 HDFS 中的文件。对于这些 RDD，分区为文件的每个块返回一个分区（块的偏移存储在每个分区对象中），首选 Locations 给出块所在的节点，迭代器读取块。

**map：**在任何 RDD 上调用映射都会返回 MappedRDD 对象。此对象具有与其父对象相同的分区和首选位置，但在其 iterator 方法中应用传递的函数映射到父对象的记录。

**union：**在两个 RDD 上调用联合会返回一个 RDD，它的分区是父节点的联合。每个子分区通过对相应父项的窄依赖性计算。

**sample：**抽样类似于映射，除了 RDD 存储每个分区的随机数生成器种子以确定性地对父记录进行抽样。

**join：**加入两个 RDD 可能导致两个依赖关系（如果它们用相同的分区器分区），两个宽依赖关系或混合（如果一个父代有分区器，一个没有分区器）。在任一情况下，输出 RDD 具有分区器（从父节点继承的分区器或默认哈希分区器）。

# 5. Implementation

我们在大约14000行 Scala 中实现了 Spark。系统运行在 Mesos 集群管理器\[17\]，允许它与 Hadoop，MPI 和其他应用程序共享资源。每个 Spark 程序作为一个单独的 Mesos 应用程序运行，具有自己的驱动程序（masler）和工作程序，这些应用程序之间的资源共享由 Mesos 处理。

Spark 可以使用 Hadoop 现有的输入插件 API 从任何 Hadoop 输入源（例如 HDFS 或 HBase）读取数据，并在未修改的 Scala 版本上运行。

我们现在绘制系统的几个技术上有趣的部分：我们的作业调度器（§5.1），我们的 Spark 解释器允许交互式使用（§5.2），内存管理（§5.3）和支持检查点（§5.4）。

### 5.1 Job Scheduling

Spark 的调度器使用我们的 RDDs 表示，描述在第4节。

总的来说，我们的调度器类似于 Dryad 的\[19\]，但它另外考虑了持久 RDD 的哪些分区在内存中可用。每当用户在 RDD 上运行动作（例如，计数或保存）时，调度器检查 RDD 的沿袭血统图以构建要执行的阶段的 DAG，如图5所示。每个阶段包含具有窄依赖性的多个流水线变换可能。阶段的边界是宽依赖性所需的随机操作，或任何已经计算的可以使父 RDD 的计算短路的分区。然后，调度程序启动任务以计算每个阶段中缺少的分区，直到计算出目标 RDD。

我们的调度程序使用延迟调度将任务分配给基于数据位置的机器\[32\]。如果任务需要处理节点上的内存中可用的分区，那么我们将其发送到该节点。否则，如果任务处理包含 RDD 提供优先位置（例如，HDFS 文件）的分区，则将其发送到那些。

对于宽依赖（即，随机依赖），我们当前在保存父分区的节点上实现中间记录，以简化故障恢复，就像 MapReduce 实现映射输出一样。

如果任务失败，我们在另一个节点上重新运行它，只要它的父节点仍然可用。如果某些阶段变得不可用（例如，因为 shuffle 的“map side”的输出丢失），则提交任务以并行计算缺失的分区。我们还不能容忍调度程序失败，虽然复制 RDD 谱系血统图将是直接的。

最后，虽然 Spark 中的所有计算都响应于驱动程序中调用的 action 而运行，但我们还在尝试让集群中的任务（例如，maps）调用 lookup 操作，这提供了对 hash-按键分区的 RDD。在这种情况下，任务将需要告诉调度程序计算所需的分区，如果它缺失。

### 5.2 Interpreter Integration

Scala 包括一个类似于 Ruby 和 Python 的交互式 shell。考虑到内存中数据获得的低延迟，我们希望让用户从解释器中积极地运行 Spark 来查询大数据集。

Scala 解释器通常通过为用户键入的每一行编译一个类，将它加载到 JVM 并调用一个函数。这个类包括一个单例对象，该对象包含该行上的变量或函数，并在初始化方法中运行该行的代码。例如，如果用户键入 var x = 5，后跟 println（x），解释器定义一个包含 x 的 Line1 类，并使第二行编译为 println（Line1.getInstance（）.x）。

我们对 Spark 中的解释器做了两个更改：

1. Class shipping（类运输）：为了让工作节点获取在每行上创建的类的字节码，我们使解释器通过 HTTP 提供这些类。

2. 修改代码生成：通常，为每行代码创建的单例对象通过对应类的静态方法访问。这意味着当我们序列化一个引用在上一行定义的变量的闭包（例如上面的例子中的 Line1.x）时，Java 将不会通过对象图来传递绕过 x 的 Line1 实例。因此，工作节点将不会接收 x。我们修改代码生成逻辑直接引用每个 line 对象的实例。

图6显示了解释器如何将用户输入的一组行转换为 Java 对象。

![](/img/spark_paper/Resilient Distributed Datasets A Fault Tolerant Abstraction for In-Memory Cluster Computing/figure6.png)

我们发现 Spark 解释器在处理作为我们研究和 HDFS 中存储的数据集的一部分的大轨迹时非常有用。我们还计划使用以交互方式运行更高级别的查询语言，例如 SQL。

### 5.3 Memory Management

Spark 为永久 RDD 的存储提供了三个选项：内存存储作为反序列化的 Java 对象，内存存储作为序列化数据，以及磁盘存储。第一个选项提供最快的性能，因为 Java VM 可以本地访问每个 RDD 元素。第二个选项允许用户在空间有限时选择比 Java 对象图更高的内存高效表示，但代价是性能降低。 第三个选项对于太大而不能保存在 RAM 中但对每次使用重新计算成本高昂的 RDD 非常有用。 

为了管理有限的可用内存，我们在 RDD 级别使用 LRU 逐出策略。当计算新的 RDD 分区但没有足够的空间来存储它时，我们从最近访问的 RDD 逐出一个分区，除非这是与具有新分区的 RDD 相同的 RDD。在这种情况下，我们保留旧的分区在内存中，以防止循环分区从相同的 RDD 进出。这是很重要的，因为大多数操作都将在整个 RDD 上运行任务，因此很有可能未来将需要已经在内存中的分区。我们发现这个默认策略在所有的应用程序中运行良好，但是我们还通过每个 RDD 的“持久性优先级”为用户提供进一步的控制。

最后，集群上的每个 Spark 实例都有自己独立的内存空间。在未来的工作中，我们计划通过统一的内存管理器调查跨 Spark 实例的 RDD 共享。

### 5.4 Support for Checkpointing

尽管沿袭血统可以总是用于在故障后恢复 RDD，但是对于具有长谱系血统链的 RDD，这种恢复可能是耗时的。因此，将某些 RDD 检查点稳定存储是有帮助的。

一般来说，检查点使用对包含宽依赖性的线性图形的 RDD，例如我们的 PageRank 示例中的秩数据集（§3.2.2）。在这些情况下，集群中的节点故障可能导致从每个父 RDD 丢失一些数据片，需要进行完全重新计算\[20\]。相反，对于对稳定存储中的数据具有窄依赖性的 RDD，例如我们的逻辑回归示例（§3.2.1）中的点和 PageRank 中的链接列表，检查点可能永远是不值得的。如果一个节点发生故障，这些 RDD 中的丢失分区可以在其他节点上并行重新计算，这只是复制整个 RDD 的一小部分成本。

Spark 当前提供了一个用于检查点的 API（REPLICATE 标志为持久化），但将哪个数据的检查点决定给用户。然而，我们还在调查如何执行自动检查点。由于每个数据库的调度以及首次计算所花费的时间，它应该能够选择一个最优的 RDD 集合作为检查点，以最小化系统恢复时间\[30\]。

最后，请注意，RDD 的只读性质使得它们比一般共享内存更容易检查点。因为一致性不是一个问题，RDD 可以在后台写出，而不需要程序暂停或分布式快照方案。

# 6. Evaluation

我们通过 Amazon EC2 上的一系列实验以及用户应用程序的基准评估了 Spark 和 RDDs。总的来说，我们的结果显示如下：

* Spark 在迭代机器学习和图形应用中的性能优于 Hadoop 高达20倍。加速是通过将数据作为 Java 对象存储在内存中来避免 I / O 和反序列化成本。

* 我们的用户编写的应用程序执行和扩展良好。特别是，我们使用 Spark 加速了一个在 Hadoop 上运行40倍的分析报告。

* 当节点失败时，Spark 可以通过仅重建丢失的 RDD 分区来快速恢复。

* Spark可用于以5-7秒的延迟相互查询 1 TB 数据集。

我们首先介绍针对 Hadoop 的迭代机器学习应用程序（§6.1）和 PageRank（§6.2）的基准。然后我们评估 Spark 中的故障恢复（§6.3）和当数据集不适合存储器时的行为（§6.4）。最后，我们讨论用户应用程序（§6.5）和交互式数据挖掘（§6.6）的结果。

除非另有说明，我们的测试使用了具有4个内核和 15 GB RAM 的 m1.xlarge EC2 节点。我们使用 HDFS 存储，256 MB 块。在每次测试之前，我们清除 OS 缓冲区高速缓存以精确测量 IO 成本。

### 6.1 Iterative Machine Learning Applications

我们实现了两个迭代机器学习应用程序，逻辑回归和k均值，以比较以下系统的性能：

* Hadoop：Hadoop 0.20.2稳定版本。

* HadoopBinMem：一种 Hadoop 部署，它在第一次迭代中将输出转换为高于开销的二进制格式，以消除后面的文本解析，并将其存储在内存中 HDFS 实例中。

* Spark：我们实现的RDDs。

我们使用 25-100 台机器在100 GB 数据集上运行这两种算法进行10次迭代。两个应用程序之间的关键区别是每个字节数据执行的计算量。 k-均值的迭代时间由计算控制，而逻辑回归更少计算密集，因此对反序列化和 I / O 花费的时间更加敏感。

由于典型的学习算法需要数十个迭代来收敛，我们分别报告第一次迭代和后续迭代的时间。我们发现，通过 RDD 共享数据大大加快了未来的迭代。

### First Iterations

所有三个系统在第一次迭代中从 HDFS 读取文本输入。如图7中的光条所示，Spark 在实验中比 Hadoop 快一点。这种差异是由于 Hadoop 的心跳协议中的 master 和 worker 之间的信令开销。 HadoopBinMem 是最慢的，因为它运行一个额外的 MapReduce 作业将数据转换为二进制，它不得不通过网络将此数据写入复制的内存中 HDFS 实例。

![](/img/spark_paper/Resilient Distributed Datasets A Fault Tolerant Abstraction for In-Memory Cluster Computing/figure7.png)

### Subsequent Iterations

图7还显示了后续迭代的平均运行时间，而图8显示了这些随簇大小的变化。对于自然回归，spark 分别比 Hadoop 和HadoopBinMem 分别在100台机器上快 25.3倍 和 20.7 倍。对于更多的计算密集型 k 均值应用，Spark 仍然实现了 1.9 倍到 3.2 倍的加速。

![](/img/spark_paper/Resilient Distributed Datasets A Fault Tolerant Abstraction for In-Memory Cluster Computing/figure8.png)

### Understanding the Speedup

我们惊讶地发现，Spark 甚至超过 Hadoop，内存存储的二进制数据（HadoopBinMem）20倍的余量。在 HadoopBinMem 中，我们使用了 Hadoop 的标准二进制格式（SequenceFile）和 256 MB 的大块大小，我们迫使 HDFS 的数据目录位于内存文件系统中。但是，由于以下几个因素，Hadoop 仍然运行较慢：

1. Hadoop 软件堆栈的最小开销

2.  提供数据时 HDFS 的开销

3. 将二进制记录转换为可用的内存中 Java 对象的反序列化成本。

我们依次调查了这些因素。为了测量（1），我们运行了无操作 Hadoop 作业，并且发现这些作业至少花费了25个开销，以完成作业设置，启动任务和清理的最低要求。关于（2），我们发现 HDFS 执行多个内存副本和一个校验和服务每个块。

最后，为了测量（3），我们在单台机器上运行微基准，以 256 MB 输入以各种格式运行逻辑回归计算。特别是，我们比较了HDFS（HDFS 堆栈中的开销将会显示）和内存中的本地文件（内核可以非常有效地将数据传递给程序）中处理文本和二进制输入的时间。

我们在图9中显示了这些测试的结果。内存 HDFS 和本地文件之间的差异表明，通过 HDFS 读取导致了2秒的开销，即使数据在本地机器上的内存中。文本和二进制输入之间的差异表明解析开销为7秒。最后，即使从内存文件读取，将预解析的二进制数据转换为 Java 对象需要3秒，这仍然几乎与逻辑回归本身一样昂贵。通过将 RDD 元素直接作为 Java 对象存储在内存中，Spark 避免了所有这些开销。

![](/img/spark_paper/Resilient Distributed Datasets A Fault Tolerant Abstraction for In-Memory Cluster Computing/figure9.png)

### 6.2 PageRank

我们使用54 GB维基百科转储数据就 PageRank 的计算进行将Spark的性能与Hadoop进行了比较。我们运行了10次迭代的PageRank算法来处理大约400万篇文章的链接图。图10展示了单独的内存存储为 Spark 提供了一个在30个节点上 Hadoop 的2.4倍速加速。此外，如第3.2.2节所述，控制 RDD 的分区以使其在迭代中保持一致，将加速提高到7.4倍。结果也近似线性扩展到60个节点。

![](/img/spark_paper/Resilient Distributed Datasets A Fault Tolerant Abstraction for In-Memory Cluster Computing/figure10.png)

我们还评估了一个版本的 PageRank，使用我们的 Pregel 在 Spark 中的实现，我们在7.1节中描述。迭代次数类似于图10中的迭代次数，但是更长时间约4秒，因为 Pregel 在每次迭代运行一个额外的操作，让顶点“投票”是否完成作业。

### 6.3 Fault Recovery

我们评估了在 k 平均应用中节点故障后使用沿袭血统重建 RDD 分区的成本。图11比较了在正常操作情况下75节点集群上 k-means 的10次运行时间，其中节点在第6次迭代开始时失败。没有任何失败，每次迭代包括工作在 100 GB 数据的400个任务。 

![](/img/spark_paper/Resilient Distributed Datasets A Fault Tolerant Abstraction for In-Memory Cluster Computing/figure11.png)

直到第5次迭代结束，迭代次数约为58秒。在第6次迭代中，其中一台机器被杀死，导致该机器上运行的任务和存储在那里的 RDD 分区丢失。 Spark 在其他机器上并行重新执行这些任务，在那里他们重新读取相应的输入数据并通过沿袭血统重建 RDD，从而将计算时间增加到80秒。一旦丢失的 RDD 分区被重建，迭代时间回到58s。

请注意，使用基于检查点的故障恢复机制，恢复可能需要重新运行至少几次迭代，这取决于检查点的频率。此外，系统将需要在网络上复制应用程序的 100 GB 工作集（文本输入数据转换为二进制），并且将消耗两倍的 Spark 内存以将其复制到 RAM 中，或者必须等待写入 100 GB 到磁盘。相比之下，我们的示例中的 RDDs 的沿袭血统图的大小都小于 10 KB。

### 6.4 Behavior with Insufficient Memory

到目前为止，我们确保集群中的每台机器都有足够的内存来存储迭代中的所有 RDD。一个自然的问题是如果没有足够的内存来存储作业的数据，Spark 是如何运行的。在这个实验中，我们配置 Spark 不使用超过一定百分比的内存来在每台机器上存储 RDD。我们给出了用于逻辑回归的各种量的存储空间的结果，如图12所示。我们看到性能以较少的空间优雅地降低。

![](/img/spark_paper/Resilient Distributed Datasets A Fault Tolerant Abstraction for In-Memory Cluster Computing/figure12.png)

### 6.5 User Applications Built with Spark

### In-Memory Analytics

Conviva 公司，一家视频分销公司，使用 Spark 加速了一些以前运行 Hadoop 的数据分析报告。例如，一个报告作为一系列 Hive \[1\] 查询运行，这些查询计算了客户的各种统计信息。这些查询都对同一数据子集（与客户提供的过滤器匹配的记录）进行了操作，但是在不同的分组字段上执行聚合（平均值，百分位数和 COUNT DISTINCT），需要单独的 MapReduce 作业。通过在 Spark 中实现查询并将它们共享的数据子集加载到 RDD 中，公司能够将报表加速40倍。在 Hadoop 集群上花费20小时的 200 GB 压缩数据的报告现在仅使用两台 Spark 计算机在30分钟内运行。此外，Spark 程序只需要 96 GB 的 RAM，因为它只在 RDD 中存储与客户过滤器匹配的行和列，而不是整个解压缩文件。

### Traffic Modeling

Berkeley 的 Mobile Millennum 项目的研究者并行化了一个学习算法，用于从道路汽车 GPS 测量推断道路交通拥堵。源数据是用于大城市区域的 10,000 个链路公路网络，以及对于装备 GPS 的汽车的点对点旅行时间的 600,000 个采样（每个路径的旅行时间可以包括多个道路链路）。使用交通模型，系统可以估计在个别道路链路上行驶所花费的时间。研究人员使用期望最大化（EM）算法训练此模型，该算法重复地分别重复两个 map 和 reduceByKey 步骤。应用程序从20个节点几乎线性扩展到80个节点，每个节点有4个核心，如图13（a）所示。

![](/img/spark_paper/Resilient Distributed Datasets A Fault Tolerant Abstraction for In-Memory Cluster Computing/figure13.png)

### Twitter Spam Classification（Twitter 垃圾邮件分类）

Berkeley 的 Monarch 项目\[29\]使用 Spark 来识别 Twitter 消息中的垃圾邮件。他们在 Spark 的顶部实现了一个逻辑回归分类器，类似于6.1节中的例子，但是他们使用分布式 reduceByKey 来并行求和梯度向量。在图13（b）中，我们示出了在 50GB 数据子集上训练分类器的缩放结果：与每个 URL 的页面的网络和内容属性相关的 250,000 个 URL 和 10 7 个特征/维度。由于每次迭代的固定通信成本较高，因此缩放不像线性。

### 6.6 Interactive Data Mining

为了演示 Spark 的交互式查询大数据集的能力，我们使用它来分析 1TB 的维基百科页面查看日志（2年的数据）。对于这个实验，我们使用 100 m2.4xlarge EC2 实例，每个 8 核和 68 GB RAM。我们运行查询以查找（1）所有页面，（2）标题与给定字词完全匹配的页面的总视图，以及（3）标题与部分字词匹配的页面。每个查询扫描整个输入数据。

图 14 显示了查询对完整数据集以及数据的一半和十分之一的响应时间。即使在 1 TB 的数据，查询 Spark 花了5-7秒。这比使用磁盘数据要快一个数量级;例如，从磁盘查询 1 TB 文件需要 170 秒。这说明 RDDs 使 Spark 成为用于交互式数据挖掘的强大工具。

![](/img/spark_paper/Resilient Distributed Datasets A Fault Tolerant Abstraction for In-Memory Cluster Computing/figure14.png)

# 7. Discussion

尽管 RDD 由于其不可变的性质和粗粒度的变换而似乎提供了有限的编程接口，我们发现它们适用于广泛的应用。特别地，RDD 可以表达令人惊讶的数量的群集编程模型，其迄今为止被提出作为单独的框架，允许用户在一个程序中组合这些模型（例如，运行 MapReduce 操作以构建图形，然后 run Pregel on it） 。本文讨论了 RDD 可以表达的编程模型以及它们为什么如此广泛适用（§7.1）。此外，我们讨论沿袭血统信息在 RDDs 中的另一个好处，我们正在追求，这是为了便于跨这些模型的调试（§7.2）。

### 7.1 Expressing Existing Programming Models

RDDs 可以有效地表示迄今为止独立提出的许多聚类规划模型。通过“有效地”，我们意味着 RDD 不仅可以用于产生与这些模型中编写的程序相同的输出，而且 RDD 还可以捕获这些框架执行的优化，例如将特定数据保存在内存中，将其分区最小化通信，有效地从故障中恢复。使用 RDD 可表达的模型包括：

**MapReduce：**这个模型可以使用 spark 中的 flatMap 和 groupByKey 操作来表示，或者如果存在组合器，则使用 reduce-ByKey 来表示。

**DryadLINQ：**DryadLINQ 系统提供比 MapReduce 更广泛的操作符，而不是更一般的 Dryad 运行时，但这些都是直接对应于 Spark（map，groupByKey，join 等）中可用的 RDD 转换的批量操作符。

**SQL：**像 DryadLINQ 表达式一样，SQL 查询对记录集执行数据并行操作。

**Pregel：**Google 的 Pregel \[22\] 是迭代图应用程序的专用模型，它最初看起来与其他系统中面向集合的编程模型非常不同。在 Pregel 中，程序作为一系列协调的“超级步”运行。在每个超级步中，图中的每个顶点都运行一个用户函数，可以更新与顶点相关的状态，更改图拓扑，并发送消息其他顶点用于下一个超级步。该模型可以表示许多图算法，包括最短路径，二分匹配和 PageRank。

使用 RDD 实现这个模型的关键观察是，Pregel 将相同的用户函数应用于所有的透视单元。因此，我们可以存储 RDD 中每个迭代的顶点状态，并执行批量变换（flatMap）以应用此函数并生成消息的 RDD 。然后我们可以将这个 RDD 与顶点状态相结合来执行消息交换。同样重要的是，RDDs 允许我们像 Pregel 那样在存储器中保持 Vertex 状态，通过控制它们的分区来最小化通信，并支持失败时的部分恢复。我们已经在 Spark 顶部实现了 Pregel 作为一个200行的库，更多的细节请参考\[33\]。

**Iterative MapReduce：**最近提出的几个系统，包括 HaLoop \[7\] 和 Twister \[11\]，提供了一个迭代的 MapReduce 模型，用户给系统一系列 MapReduce 作业循环。系统保持跨迭代一致的数据分区，并且 Twister 也可以将其保存在内存中。这两个优化都很容易用 RDD 表达，我们能够使用 Spark 实现 HaLoop 作为一个 200 行的库。

**Batched Stream Processing：**研究人员最近提出了几个增量处理系统，用于周期性地用新数据更新结果的应用\[21,15,4\]。例如，每15分钟更新一次有关广告点击次数的统计信息的应用程序应该能够将来自前 15 分钟窗口的中间状态与来自新日志的数据合并。这些系统执行类似于 Dryad 的批量操作，但是在分布式文件系统中的存储应用状态。将中间状态置于 RDD 中将加快其处理速度。

### Explaining the Expressivity of RDDs

为什么 RDD 能够表达这些不同的编程模型？原因是对 RDD 的限制在许多并行应用中几乎没有影响。特别地，尽管 RDD 只能通过批量转换来创建，但是许多并行程序自然地对许多记录应用相同的操作，使得它们易于表达。类似地，RDDs 的不变性不是障碍，因为相同数据集的多个 RDD 存在表示。事实上，今天的许多 MapReduce 应用程序运行在不允许更新文件的文件系统上，例如 HDFS。

最后一个问题是，以前的框架为什么没有提供同样的一般性。我们认为这是因为这些系统探讨了 MapReduce 和 Dryad 不能很好处理的特定问题，例如迭代，而没有观察到这些问题的常见原因是缺乏数据共享抽象。

### 7.2 Leveraging RDDs for Debugging

虽然我们最初设计的 RDD 是确定性的重计算容错，这个属性也方便调试。特别地，通过记录在作业期间创建的 RDD 的谱系血统，可以（1）稍后重建这些 RDD，并让用户以交互方式查询它们，以及（2）在单过程调试器中从作业重新运行任何任务，通过重新计算它依赖的 RDD 分区。与传统的分布式系统的重放调试器\[13\]不同，后者必须捕获或推断多个节点上事件的顺序，这种方法增加了几乎零记录开销，因为只需要记录 RDD 谱系血统图。 我们正在开发基于这些想法的 Spark 调试器\[33\]。

# 8. Related Work

### Cluster Programming Models

集群编程模型中的相关工作分为几类。首先，数据流模型如 MapReduce \[10\]，Dryad \[19\] 和 Ciel \[23\] 支持一组丰富的运算符处理数据，但通过稳定的存储系统共享它。 RDD 表示比稳定存储更高效的数据共享抽象，因为它们避免了数据复制，I / O 和序列化的成本。

第二，用于数据流系统的几个高级编程接口，包括 DryadLINQ \[31\] 和 Flume Java \[8\]，提供了语言集成的 API，用户通过操作符（如map 和 join）操作“并行集合”。然而，在这些系统中，并行集合表示用于表达查询计划的磁盘上的文件或短暂数据集。尽管系统将在同一查询中的操作符（例如，后面跟着另一个 map 的 map）上管理数据，但是它们不能在查询之间有效地共享数据。我们基于 Spark 的 API 在并行收集模型上由于它的方便，并且不声称新颖的语言集成接口，但通过提供 RDD 作为这个接口背后的存储抽象，我们允许它支持一个更广泛的类别的应用程序。

第三类系统为需要数据共享的特定类别的应用提供高级接口。例如，Pregel \[22\] 支持迭代图应用，而 Twister \[11\] 和 HaLoop \[7\] 是迭代 MapReduce 运行时。然而，这些框架隐含地为它们支持的计算模式执行数据共享，并且不提供用户可以用来在她选择的操作中共享她选择的数据的一般摘要。例如，用户不能使用 Pregel 或 Twister 将数据集加载到内存中，然后决定要对其运行什么查询。 RDD 显式地提供分布式存储抽象，并且因此可以支持这些专用系统不捕获的应用，诸如交互式数据挖掘。

最后，一些系统暴露共享的可变状态以允许用户执行内存计算。例如，Piccolo \[27\] 允许用户运行读取和更新分布式哈希表中的单元格的并行函数。分布式共享内存（DSM）系统\[24\]和像 RAMCloud \[25\] 这样的键值存储提供了一个类似的模型。 RDD 与这些系统在两个方面不同。首先，RDD 基于 map，sort 和 join等运算符提供更高级别的编程接口，而 Piccolo 和 DSM 中的接口只是对表单元格的读取和更新。第二，Piccolo 和 DSM 系统通过检查点和回滚实现恢复，这在许多应用中比 RDD 的基于沿袭血统的策略更昂贵。最后，如第2.3节所述，RDD 还提供了与 DSM 相比的其他优点，例如缓解分块。

### Caching Systems

Nectar \[12\] 可以通过程序分析识别常见的子表达式来重用 DryadLINQ 作业中的中间结果\[16\]。这种能力将被迫增加到基于 RDD 的系统。但是，Nectar 不提供内存缓存（它将数据放置在分布式文件系统中），也不允许用户明确地控制哪些数据集保留以及如何对其进行分区。 Ciel \[23\] 和 Flume Java \[8\] 可以同样缓存任务结果，但不提供内存缓存或对哪些数据进行缓存的显式控制。

Ananthanarayanan et al. 建议在分布式文件系统中增加一个内存中缓存来利用数据访问的时间和空间局部性\[3\]。虽然此解决方案提供对已经在文件系统中的数据的更快访问，但是它不像在 RDD 中在应用程序中共享中间结果的方法那样高效，因为它仍然需要应用程序将这些结果写入文件系统阶段。

### Lineage

捕获数据的谱系血统或来源信息长期以来一直是科学计算和数据库中的研究课题，用于解释结果，允许其他人再现，以及在工作流中发现错误时重新计算数据，或者如果数据集丢失。我们引用读者\[5\]和\[9\]来调查这项工作。 RDD 提供了并行编程模型，其中细粒化谱系捕获成本低，使得其可以用于故障恢复。

我们的基于沿袭血统的恢复机制也类似于 MapReduce 和 Dryad 中的计算（作业）中使用的恢复机制，它跟踪 DAG 任务之间的依赖关系。然而，在这些系统中，沿袭血统信息在 job 结束后丢失，需要使用复制存储系统来跨计算共享数据。相比之下，RDD 应用沿袭血统来跨计算高效地保持内存中的数据，而不需要复制和磁盘 I / O 的成本。

### Relational Databases

RDD 在概念上类似于数据库中的视图，持久 RDD 类似物化视图\[28\]。但是，与 DSM 系统一样，数据库通常允许对所有记录进行细粒度读写访问，需要记录操作和数据以实现容错和额外开销以保持一致性。 RDD 的粗粒度变换模型不需要这些开销。

# 9. Conclusion

我们提出了弹性分布式数据集（RDDs），一种高效的，通用的和容错的抽象，用于在集群应用程序中共享数据。 RDD 可以表达广泛的并行应用程序，包括已经提出用于迭代计算的许多专用编程模型以及这些模型未捕获的新应用程序。与现有的需要数据复制以实现容错的存储抽象不同，RDDs 提供了一个基于粗粒度转换的 API，可以使用 lineage 有效地恢复数据。我们已经在一个称为 Spark 的系统中实现了 RDD，在迭代应用程序中性能优于 Hadoop，可以交互使用查询数百 GB 的数据。

# Acknowledgements

我们感谢第一批 Spark 用户，包括 Tim Hunter，Lester Mackey，Dilip Joseph 和 Jibin Zhan，在他们的实际应用中尝试我们的系统，提供了许多好的建议，并指出了一些研究挑战。我们也感谢我们的牧羊人，夜总会和我们的审稿人的反馈。这项研究得到了DARPA（合同 ＃FA8650-11-C-7136）部分支持的 Berkeley AMP 实验室赞助 Google，SAP，Amazon Web Services，云时代，华为，IBM，Intel，Microsoft，NEC，NetApp 和 VMWare， ，通过 Google 博士研究生奖学金，以及加拿大自然科学和工程研究理事会。

# References

\[1\] Apache Hive. http://hadoop.apache.org/hive.

\[2\] Scala. http://www.scala-lang.org.

\[3\] G. Ananthanarayanan, A. Ghodsi, S. Shenker, and I. Stoica.

Disk-locality in datacenter computing considered irrelevant. In

HotOS ’11, 2011.

\[4\] P. Bhatotia, A. Wieder, R. Rodrigues, U. A. Acar, and

R. Pasquin. Incoop: MapReduce for incremental computations.

In ACM SOCC ’11, 2011.

\[5\] R. Bose and J. Frew. Lineage retrieval for scientific data

processing: a survey. ACM Computing Surveys, 37:1–28, 2005.

\[6\] S. Brin and L. Page. The anatomy of a large-scale hypertextual

web search engine. In WWW, 1998.

\[7\] Y. Bu, B. Howe, M. Balazinska, and M. D. Ernst. HaLoop:

efficient iterative data processing on large clusters. Proc. VLDB

Endow., 3:285–296, September 2010.

\[8\] C. Chambers, A. Raniwala, F. Perry, S. Adams, R. R. Henry,

R. Bradshaw, and N. Weizenbaum. FlumeJava: easy, efficient

data-parallel pipelines. In PLDI ’10. ACM, 2010.

\[9\] J. Cheney, L. Chiticariu, and W.-C. Tan. Provenance in

databases: Why, how, and where. Foundations and Trends in

Databases, 1\(4\):379–474, 2009.

\[10\] J. Dean and S. Ghemawat. MapReduce: Simplified data

processing on large clusters. In OSDI, 2004.

\[11\] J. Ekanayake, H. Li, B. Zhang, T. Gunarathne, S.-H. Bae, J. Qiu,

and G. Fox. Twister: a runtime for iterative mapreduce. In

HPDC ’10, 2010.

\[12\] P. K. Gunda, L. Ravindranath, C. A. Thekkath, Y. Yu, and

L. Zhuang. Nectar: automatic management of data and

computation in datacenters. In OSDI ’10, 2010.

\[13\] Z. Guo, X. Wang, J. Tang, X. Liu, Z. Xu, M. Wu, M. F.

Kaashoek, and Z. Zhang. R2: an application-level kernel for

record and replay. OSDI’08, 2008.

\[14\] T. Hastie, R. Tibshirani, and J. Friedman. The Elements of

Statistical Learning: Data Mining, Inference, and Prediction.

Springer Publishing Company, New York, NY, 2009.

\[15\] B. He, M. Yang, Z. Guo, R. Chen, B. Su, W. Lin, and L. Zhou.

Comet: batched stream processing for data intensive distributed

computing. In SoCC ’10.

\[16\] A. Heydon, R. Levin, and Y. Yu. Caching function calls using

precise dependencies. In ACM SIGPLAN Notices, pages

311–320, 2000.

\[17\] B. Hindman, A. Konwinski, M. Zaharia, A. Ghodsi, A. D.

Joseph, R. H. Katz, S. Shenker, and I. Stoica. Mesos: A platform

for fine-grained resource sharing in the data center. In NSDI ’11.

\[18\] T. Hunter, T. Moldovan, M. Zaharia, S. Merzgui, J. Ma, M. J.

Franklin, P. Abbeel, and A. M. Bayen. Scaling the Mobile

Millennium system in the cloud. In SOCC ’11, 2011.

\[19\] M. Isard, M. Budiu, Y. Yu, A. Birrell, and D. Fetterly. Dryad:

distributed data-parallel programs from sequential building

blocks. In EuroSys ’07, 2007.

\[20\] S. Y. Ko, I. Hoque, B. Cho, and I. Gupta. On availability of

intermediate data in cloud computations. In HotOS ’09, 2009.

\[21\] D. Logothetis, C. Olston, B. Reed, K. C. Webb, and K. Yocum.

Stateful bulk processing for incremental analytics. SoCC ’10.

\[22\] G. Malewicz, M. H. Austern, A. J. Bik, J. C. Dehnert, I. Horn,

N. Leiser, and G. Czajkowski. Pregel: a system for large-scale

graph processing. In SIGMOD, 2010.

\[23\] D. G. Murray, M. Schwarzkopf, C. Smowton, S. Smith,

A. Madhavapeddy, and S. Hand. Ciel: a universal execution

engine for distributed data-flow computing. In NSDI, 2011.

\[24\] B. Nitzberg and V. Lo. Distributed shared memory: a survey of

issues and algorithms. Computer, 24\(8\):52 –60, Aug 1991.

\[25\] J. Ousterhout, P. Agrawal, D. Erickson, C. Kozyrakis,

J. Leverich, D. Mazières, S. Mitra, A. Narayanan, G. Parulkar,

M. Rosenblum, S. M. Rumble, E. Stratmann, and R. Stutsman.

The case for RAMClouds: scalable high-performance storage

entirely in DRAM. SIGOPS Op. Sys. Rev., 43:92–105, Jan 2010.

\[26\] D. Peng and F. Dabek. Large-scale incremental processing using

distributed transactions and notifications. In OSDI 2010.

\[27\] R. Power and J. Li. Piccolo: Building fast, distributed programs

with partitioned tables. In Proc. OSDI 2010, 2010.

\[28\] R. Ramakrishnan and J. Gehrke. Database Management

Systems. McGraw-Hill, Inc., 3 edition, 2003.

\[29\] K. Thomas, C. Grier, J. Ma, V. Paxson, and D. Song. Design and

evaluation of a real-time URL spam filtering service. In IEEE

Symposium on Security and Privacy, 2011.

\[30\] J. W. Young. A first order approximation to the optimum

checkpoint interval. Commun. ACM, 17:530–531, Sept 1974.

\[31\] Y. Yu, M. Isard, D. Fetterly, M. Budiu, ´ U. Erlingsson, P. K.

Gunda, and J. Currey. DryadLINQ: A system for

general-purpose distributed data-parallel computing using a

high-level language. In OSDI ’08, 2008.

\[32\] M. Zaharia, D. Borthakur, J. Sen Sarma, K. Elmeleegy,

S. Shenker, and I. Stoica. Delay scheduling: A simple technique

for achieving locality and fairness in cluster scheduling. In

EuroSys ’10, 2010.

\[33\] M. Zaharia, M. Chowdhury, T. Das, A. Dave, J. Ma,

M. McCauley, M. Franklin, S. Shenker, and I. Stoica. Resilient

distributed datasets: A fault-tolerant abstraction for in-memory

cluster computing. Technical Report UCB/EECS-2011-82,

EECS Department, UC Berkeley, 2011.

