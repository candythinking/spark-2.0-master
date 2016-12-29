# 摘要

许多“大数据”应用程序必须实时处理数据。以越来越大的规模运行这些应用程序需要自动处理故障和分离器的并行平台。不幸的是，当前的分布式流处理模型以昂贵的方式提供故障恢复，需要热复制或长的恢复时间，并且不处理分离器。我们提出了一种新的处理模型，离散流（D-Streams），克服了这些挑战。 D-Streams 支持并行恢复机制，提高了传统复制和备份方案的效率，并允许分离器。我们表明，它们支持一组丰富的操作员，同时获得类似于单节点系统的高每节点吞吐量，线性扩展到100个节点，亚秒级延迟和亚秒级故障恢复。最后，D-Streams 可以轻松地使用批处理和交互式查询模型（如 MapReduce）进行组合，从而实现组合这些模式的丰富应用程序。我们在称为 Spark Streaming 的系统中实现 D-Streams。

# 1. Introduction

许多“大数据”被实时接收，并且在其到达时是最有价值的。例如，社交网络可能希望在几分钟内检测趋势对话主题;搜索站点可能希望建模哪些用户访问新页面;并且服务操作者可能希望监视程序日志以便在数秒内检测故障。为了启用这些低延迟处理应用程序，需要一种能够透明地扩展标量聚类的流计算模型，同时也提供了 MapReduce 简化离线处理等批处理模型。

然而，设计这样的模型是具有挑战性的，因为最大的应用（例如，实时处理机器学习）所需的规模可以节省节点。在这个规模，两个主要的问题是故障和stragglers（慢节点）。这两个问题在大型集群中是不可避免的\[12\]，所以流应用程序必须迅速恢复。快速恢复在流式传输中比在批处理作业中更重要：从故障或分离器恢复30秒的延迟在批处理设置中是一个麻烦，它可能意味着失去在一个关键决定中的机会流设置。

不幸的是，现有的流传输系统具有有限的故障和分段容差。大多数分布式流系统，包括 Storm \[37\]，TimeStream \[33\]，MapReduce在线\[11\]和流数据库\[5,9,10\]，是基于连续操作模型，其中长 - 运行，状态运算符接收每个记录，更新内部状态，并发送新记录。虽然这个模型是很自然的，它使它很难处理故障和 stragglers（慢节点）。

具体来说，给定连续运算符模型，系统通过两种方法执行恢复\[20\]：复制，其中每个节点\[5,34\]有两个副本或上游备份，其中节点缓冲发送的消息并将它们重播到新的副本的故障节点\[33，11，37\]。这两种方法在大型集群中都不具有吸引力：复制成本是硬件的2倍，而上游备份需要很长时间来恢复，因为整个系统必须等待一个新节点通过运行操作员连续重建故障节点的状态。两种方法都不能处理错误：在上游备份中，分步器必须被视为故障，导致昂贵的恢复步骤，而复制系统使用像 Flux 这样的同步协议来协调副本，因此分裂器将减慢两个副本。

本文提出了一种新的流处理模型，离散流（D-Streams），克服了这些挑战。代替管理长期运算符，D-Streams 中的想法是将流式计算结构化为在小的时间间隔上的一系列无状态，确定性批量计算。例如，我们可以将每秒（或每100ms）接收的数据放入一个间隔，并对每个间隔运行 MapReduce 操作以计算计数。类似地，我们可以通过将每个间隔中的新计数添加到旧结果来运行多个间隔的滚动计数。通过以这种方式构造计算，D-Stream 使得（1）在给定输入数据的情况下每个时间步骤的状态是完全确定的，放弃对同步协议的需要，以及（2）在细粒度上可见的该状态和旧数据之间的依赖性。我们表明，这使得强大的恢复机制（类似于批处理系统中的机制）胜过复制和上游备份。

实现D-Stream模型有两个挑战。第一个是使延迟（间隔粒度）低。传统的批处理系统（如 Hadoop）在这里不足，因为它们在作业之间在复制的磁盘存储系统中保持状态。相反，我们使用一种称为弹性分布式数据集（RDDs）\[43\]的数据结构，它将数据保存在内存中，并且可以通过跟踪用于构建它的操作的行计划图来重建。使用RDD，我们显示我们可以达到次秒的端到端延迟。我们认为这对于许多现实世界的大数据应用是足够的，其中跟踪的事件的时间尺度（例如，社交媒体中的趋势）高得多。

第二个挑战是迅速从故障和stragglers中恢复。在这里，我们使用 D-Streams 的确定性来提供在以前的流系统中没有存在的新的恢复机制：并发恢复丢失的节点的状态。当节点发生故障时，集群中的每个节点都将重新计算丢失节点的 RDD 的一部分，从而导致比上游备份快得多的恢复速度，而无需复制成本。并行恢复难以在连续处理系统中执行，因为即使对于基本复制（例如，Flux \[34\]）1所需的复杂状态同步协议，但使用完全确定的D流模型变得简单。以类似的方式，D-Streams 可以使用推测执行从破解者恢复\[12\]，而以前的流系统不处理它们。

我们在基于 Spark 引擎的 Spark Streaming 系统中实现了 D-Streams \[43\]。系统可以在亚秒级延迟上在100个节点上处理超过6000万条记录/秒，并且可以在亚秒的时间内从故障和分离器中恢复。 Spark Streaming 的每节点吞吐量可与商业流数据库相媲美，同时为100个节点提供线性可扩展性，比开源 Storm 和 S4 系统快2-5倍，同时提供缺少的故障恢复保证。除了它的性能，我们通过两个实际应用的端口说明了 Spark Streaming 的表现力：一个视频分配监控系统和一个在线机器学习系统。

最后，因为 D-Streams 使用与批处理作业相同的处理模型和数据结构（RDDs），所以我们的模型的一个优势是，流查询可以无缝地与批处理和交互式计算结合。我们利用 Spark Streaming 中的此功能，让用户使用 Spark 对流运行即席查询，或者连接具有作为 RDD 计算的历史数据的流。这在实践中是一个强大的功能，给用户一个单独的 API 来结合以前不同的计算。我们描述了我们如何在应用程序中使用它来模糊实时和离线处理之间的界限。

# 2. Goals and Background

许多重要的应用程序处理大量实时到达的数据流。我们的工作目标是应用程序需要几十个机器，并延迟几秒钟的延迟。一些例子是：

* **Site activity statistics（网站活动统计信息）：**Facebook 建立了一个名为 Puma 的分布式聚合系统，使广告商可以统计用户在10-30秒内点击其网页并处理10的6次方个事件\[35\]。
* **Cluster monitoring（集群监控）：**数据中心操作员收集和挖掘程序日志以检测问题，在数百个节点上使用类似 Flume \[3\] 的系统\[17\]。
* **Spam detection（垃圾邮件检测）：**诸如 Twitter 的社交网络可能希望使用统计学习算法实时识别新的垃圾邮件活动\[39\]。

对于这些应用，我们认为 D-Streams 的0.5-2秒延迟是足够的，因为它远低于监测的趋势的时间范围。我们故意不针对延迟需求低于几百毫秒的应用程序，例如高频交易。

### 2.1 Goals

为了在大规模运行这些应用程序，我们寻求一个系统设计，以满足四个目标：

1.可扩展到数百个节点。

2.除了基本处理之外的最小成本 - 例如，我们不希望支付2×复制开销。

3.二级延迟。

4.从故障和 stragglers（分离器）的第二规模恢复。

据我们所知，以前的系统不能满足这些目标：复制的系统具有高开销，而基于备份的系统可能需要几十秒来恢复丢失的状态\[33,41\]，也不能容忍分散程序。

### 2.2 Previous Processing Models

虽然在分布式流处理方面已经有了大量的工作，但是大多数以前的系统使用相同的连续运算符模型。在这个模型中，流计算被分为一组长寿命状态运算符，每个运算符通过更新内部状态（例如，表跟踪页面视图在一个窗口上计数）来处理记录，并发送新的记录作为响应\[10\]。图1（a）说明。

![](/img/spark_paper/2013 Discretized Streams Fault-Tolerant Streaming Computation at Scale/figure1.png)

虽然连续处理使等待时间最小化，但是操作者的有状态性与由在网络上的记录交错产生的非计数性相结合，使得难以有效地提供容错。具体来说，主要的恢复挑战是重建失去或缓慢的节点上的运营商的状态。先前的系统使用复制和上游备份两种方案之一\[20\]，它们在成本和恢复时间之间提供了一个明显的权衡。

在复制中，这在数据库系统中很常见，有两个处理图的副本，并且输入记录被发送到这两个副本。然而，简单地复制节点是不够的;系统还需要运行同步协议，如 Flux \[34\] 或 Borealis 的 DPC \[5\]，以确保每个操作符的两个副本以相同的顺序查看来自上游父进程的消息。例如，输出两个父流的联合（在任一个上接收的所有记录的序列）的算子需要以相同的顺序看到父流，以产生相同的输出流，因此该算子的两个副本需要坐标。因此，复制是昂贵的，尽管它从故障中快速恢复。

在上游备份中，每个节点保留自某些检查点后发送的消息的副本。当节点故障时，备用计算机接管其角色，父节点向该备用节点重播消息以重建其状态。因此，这种方法导致高的恢复时间，因为单个节点必须通过在串行有状态的运算符代码中运行数据来重新计算丢失的状态。 TimeStream \[33\] 和 MapReduce Online \[11\] 使用这个模型。像 Storm \[37\] 等热门消息队列系统也使用这种方法，但通常只提供“至少一次”消息传递，依靠用户的代码来处理状态恢复。

更重要的是，复制和上游备份都不会处理 stragglers。如果节点在复制模型中运行缓慢，则整个系统会受到影响，因为同步需要复制副本以相同的顺序接收消息。在上游备份中，缓解分离器的唯一方法是将其作为故障处理，这需要经历上述慢速状态恢复过程，并且对于可能是暂时的问题是沉重的。因此，虽然传统的流方法在较小的规模下工作良好，但它们在大型商品集群中面临实质性问题。

# 3. Discretized Streams \(D-Streams\)

D-Streams 通过将计算结构化为一组短的，无状态的，确定性的任务而不是连续的有状态运算符来避免传统流处理的问题。然后，它们将状态作为容错数据结构（RDD）存储在内存中，可以重新计算确定性。将计算分解为短任务在细粒度上暴露依赖性，并且允许诸如并行恢复和推测的功能强大的恢复技术。除了故障容错之外，D-Stream 模型还提供了其他优点，例如强大的统一与批处理。

### 3.1 Computation Model

我们将流计算视为在小时间间隔上的一系列确定性批量计算。在每个间隔中接收的数据可靠地存储在集群中以形成该间隔的输入数据集。一旦时间间隔完成，通过确定性并行操作（例如 map，reduce 和 groupBy）处理此数据集，以产生表示程序输出或中间状态的新数据集。在先前的情况下，结果可以以分布式方式推送到外部系统。在后一种情况下，中间状态被存储为弹性分布式数据集（RDDs）\[43\]，这是一种快速存储抽象，通过使用沿袭血统恢复来避免复制，我们将在此解释。然后可以与下一批输入数据一起处理该状态数据集，以产生更新的中间状态的新数据集。图1（b）显示了我们的模型。

我们基于这个模型实现了我们的系统，Spark Streaming。我们使用 Spark \[43\] 作为每批数据的批处理引擎。图2示出了在 Spark Streaming 的上下文中的计算模型的高级草图。这将在后面更详细地解释。

![](/img/spark_paper/2013 Discretized Streams Fault-Tolerant Streaming Computation at Scale/figure2.png)

在我们的API中，用户通过操作称为离散流（D-Streams）的对象来定义程序。 D 流是一个不可变的，分区数据集（RDDs）的序列，可以通过确定性变换进行操作。这些转换产生新的 D-Stream，并且可以以 RDD 的形式创建中间状态。

我们用一个 Spark Streaming 程序说明这个想法，该程序通过 URL 计算视图事件的运行计数。 Spark Streaming 通过类似于 Scala 编程语言中的 LINQ \[42,2\] 的功能 API 来展示 D-Streams。 我们的程序的代码是：

```
pageViews = readStream（“http：// ...”，“1s”）

ones = pageViews.map（event =&gt;（event.url，1））

counts = ones.runningReduce（（a，b） b）
```

此代码通过 HTTP 读取事件流创建一个称为 pageViews 的 D 流，并将这些流分组为1秒间隔。然后，它转换事件流以获得称为“1”的（D，D）流的（D，D）流，并且通过有状态的运行 Reduce 变换来执行这些流的运行计数。map 和运行 Reduce 的参数是 Scala 函数文字。

为了执行这个程序，Spark Streaming 将接收数据流，将其划分为一秒钟，并将它们作为 RDD 存储在 Spark 的内存中（见图2）。此外，它将调用 RDD 转换（如 map 和 reduce）来处理 RDD。为了执行这些转换，Spark 将首先启动 map 任务来处理事件并生成 url-one 对。然后，它将启动 reduce 任务，这些任务将 map 的结果和上一个时间间隔的缩减结果存储在 RDD 中。这些任务将生成带有更新计数的新 RDD。因此，程序中的每个 D-Stream 变成一系列 RDD。

最后，为了从故障和分离器中恢复，D-Streams 和 RDD 跟踪它们的血统，即用于构建它们的确定性操作的图\[43\]。 Spark 在每个分布式数据集中的分区级别跟踪此信息，如图3所示。当节点发生故障时，它会通过重新运行从原始输入数据中可靠存储的任务来重新计算其上的 RDD 分区在集群中。系统还周期性地检查状态 RDD（例如，通过异步复制每第十个 RDD）以防止无限重新计算，但这不需要对所有数据进行，因为恢复通常很快：可以并行重新计算丢失的分区在单独的节点上。以类似的方式，如果一个节点分裂，我们可以在其他节点上指定执行其任务的副本\[12\]，因为它们将产生相同的结果。

![](/img/spark_paper/2013 Discretized Streams Fault-Tolerant Streaming Computation at Scale/figure3.png)

我们注意到，在 D-Streams 中可用于恢复的并行性高于上游备份，即使每个节点运行多个运算符。 D-Streams 在操作符和时间的两个分区上显示并行性：

1. 很像批处理系统对每个节点运行多个任务，变换的每个时间步长可以为每个节点创建多个 RDD 分区（例如，100个核心集群上的1000个 RDD 分区）。当节点故障时，我们可以在其他节点上并行重新计算其分区
2. 沿袭血统图通常使得能够并行重建来自不同时间步长的数据。例如，在图3中，如果一个节点失败，我们可能会丢失每个时间步的一些 map 输出;来自不同时间步长的 map 可以并行重新运行，这在连续运算符系统中是不可能的，这种系统假设每个运算符的串行执行。

由于这些属性，即使每30秒进行一次检查点，D-Streams 也可以平行分隔多达几十英尺和两秒钟。

在本节的其余部分，我们更详细地描述 D-Streams 的保证和编程接口。然后我们回到第4节中的实现。

### 3.2 Timing Considerations

请注意，D-Streams 根据每条记录到达系统的时间将记录放置到输入数据集中。这是必要的，以确保系统总是能够及时地开始新的批次，并且在记录在与流式传输节目相同的位置中生成的应用中，例如由相同数据中心中的服务生成，对于语义而言没有问题。然而，在其他应用中，开发者可能希望基于事件发生时的外部时间戳（例如，当用户点击链接时，以及记录可能乱序到达）来对记录进行分组。 D-Streams 提供了两种方法来处理这种情况：

1. 系统在开始处理每个批次之前等待“松弛时间”。

2. 用户程序可以在应用程序级别更正延迟记录。例如，假设应用程序希望计算时间 t 和 t + 1 之间的广告点击次数。使用具有一秒间隔大小的 D-Streams，应用程序可以提供在 t 和 t + 1 之间接收的点击的计数，只要时间 t + 1 通过。然后，在将来的时间间隔中，应用可以收集具有在 t 和 t + 1 之间的外部时间戳的任何进一步的事件，并且计算更新的结果。例如，它可以基于在 t 和 t + 5 之间接收的该间隔的记录，在时间 t + 5 输出针对时间间隔 \[t，t + 1）的新计数。可以使用有效的增量缩减操作来执行该计算，该操作将从 t + 1 计算的旧计数添加到自那以后的新记录的计数，从而避免浪费工作。这种方法类似于顺序无关的处理\[23\]。

这些时序问题是流处理的固有问题，因为任何系统都必须处理外部延迟。他们在数据库中进行了详细的研究\[23,36\]。通常，任何这样的技术可以通过 D-Streams 通过以小批量（分批运行相同的逻辑）“离散化”其计算来实现。因此，我们不在本文中进一步探讨这些方法。

### 3.3 D-Stream API

因为 D-Streams 主要是执行策略（描述如何将计算分解为步骤），它们可以用于在流系统中实现许多标准操作，例如滑动窗口和增量处理\[10,4\]通过简单地将它们的执行分批为小的时间步长。为了说明，我们描述了 Spark Streaming 中的操作，尽管也可以支持其他接口（例如，SQL）。

在 Spark Streaming 中，用户使用功能API注册一个或多个流。程序可以从外部定义输入流，其通过使节点在端口上监听或通过从存储系统（例如，HDFS）周期性地加载它来接收数据。然后它可以对这些流应用两种类型的操作：

* **Transformations**：其从一个或多个父流创建新的 D 流。这些可以是无状态的，在每个时间间隔中分别在 RDD 上应用，或者它们可以跨间隔产生状态。
* **Output operations**：它让程序将数据写入外部系统。例如，保存操作将 D-Stream 中的每个 RDD 输出到数据库。

D-Streams 支持典型的批处理框架\[12，42\]中可用的相同的无状态转换，包括 map，reduce，groupBy 和 join。我们提供 Spark 的所有操作\[43\]。例如，程序可以使用以下代码在字串的每个时间间隔上运行规范的 MapReduce 字计数：

```
pairs = words.map（w =&gt;（w，1））

counts = pairs.reduceByKey（（a，b）=&gt; a + b）
```

此外，D-Streams 基于诸如滑动窗口的标准流处理技术为跨越多个间隔的计算提供了若干状态转换\[10,4\]。这些包括：

**Windowing：**窗口操作将来自过去时间间隔的滑动窗口的所有记录分组到一个 RDD 中。例如，在上面的代码中调用words.window（“5s”）产生包含区间\[0,5），\[1,6\]，\[2,7）等中的字的 RDD 的 D 流。

**Incremental aggregation：**对于在滑动窗口上计算聚合（如计数或最大值）的常见用例，D-Streams 具有增量 reduceByWindow 操作的几个变体。最简单的只需要一个关联合并函数来组合值。例如，在上面的代码中，可以写：

```
pairs.reduceByWindow（“5s”，（a，b）=&gt; a + b）
```

这将计算每个时间间隔的每间隔计数一次，但必须重复添加过去五秒的计数，如图4（a）所示。如果聚合函数也是可逆的，一个更高效的版本还采用“减”值的函数，并且以增量方式维持状态（图4（b））：

![](/img/spark_paper/2013 Discretized Streams Fault-Tolerant Streaming Computation at Scale/figure4.png)

**State tracking：**通常，应用程序必须响应于指示改变的事件流来跟踪各种对象的状态。例如，对在线视频递送的视图监视可能希望跟踪活动会话的数量，其中当系统接收到“加入”事件，并且当它接收到“退出”事件时结束。然后它可以询问诸如“有多少会话具有高于 X 的比特率”的问题. D-Streams 提供基于三个参数将（Key，Event）记录的流转换成（Key，State）记录的流的跟踪操作：

* 用于从新密钥的第一个事件创建状态的初始化函数。

* 用于返回给定旧状态的新状态和用于其键的事件的更新函数。

* 用于删除旧状态的超时。

例如，可以对来自（ClientID，Event）对的流的活动会话进行计数，如下所示：

```
sessions = events.track\( 

    \(key, ev\) =&gt; 1, // initialize function 

    \(key, st, ev\) =&gt; // update function 

    ev == Exit ? null : 1,

    "30s"\) // timeout

counts = sessions.count\(\) // a stream of ints
```

此代码将每个客户端的状态设置为1（如果它处于活动状态），并在离开时通过从更新返回 null 来将其删除。因此，会话包含每个活动客户端的（ClientID，1）元素，并且计数计数会话。

这些运算符都是使用 Spark 中的批处理运算符实现的，通过将它们应用到父流中的不同时间的 RDD。例如，图5显示了由轨道构建的 RDD，其通过对每个间隔的旧状态和新事件进行分组来工作。

![](/img/spark_paper/2013 Discretized Streams Fault-Tolerant Streaming Computation at Scale/figure5.png)

最后，用户调用输出运算符以将结果从 Spark Streaming 发送到外部系统（例如，用于在仪表板上显示）。我们提供两个这样的运算符：save，它将 D-Stream 中的每个 RDD 写入存储系统（例如 HDFS 或 HBase），以及 foreachRDD，它在每个 RDD 上运行用户代码片段（任何Spark代码）。例如，用户可以使用 counts.foreachRDD（rdd =&gt; print（rdd.top（K）））打印前K个计数

### 3.4 Consistency Semantics

D-Streams 的一个好处是它们提供了干净的一致性语义。节点状态的一致性在处理每个记录的流式系统中可能是一个问题。例如，考虑一个按国家计算网页浏览量的系统，其中每个网页浏览事件发送到负责汇总其国家统计信息的不同节点。如果负责英国的节点落在法国的节点之后（例如由于负载），则其状态的快照将是不一致的：英格兰的计数将反映比法国的计数更旧的流的前缀，以及通常会更低，对事件的混淆推论。一些系统，如 Borealis \[5\]，同步节点以避免此问题，而其他像 Storm 一样忽略它。

使用 D-Streams，一致性语义是清楚的，因为时间自然地离散成间隔，并且每个间隔的输出 RDD 反映在该间隔和先前间隔中接收的所有输入。这是真的，不管输出和状态 RDD 是否分布在集群中 - 用户不需要担心节点是否落在彼此之后。特别地，每个输出 RDD 中的结果在计算时与在先前间隙中的所有批处理作业都在锁步中运行相同，并且没有分段和失败，这仅仅是由于计算的确定性和从不同间隔单独命名数据集。因此，D-Streams 在集群中提供一致的，“一次”处理。

### 3.5 Unification with Batch & Interactive Processing

因为 D-Streams 遵循相同的处理模型，数据结构（RDDs）和容错机制作为批处理系统，所以两者可以无缝地组合。

Spark Streaming 提供了几个强大的功能来统一流式处理和批处理。

首先，D-Streams 可以与使用标准 Spark 作业计算的静态 RDDs 组合。例如，可以针对预先投放的垃圾邮件过滤器加入消息事件流，或者将其与历史数据进行比较。

第二，用户可以使用“批处理模式”对先前的历史数据运行 D-Stream 程序。这使得可以容易地计算关于过去数据的新的流报告。

第三，用户通过将 Scala 控制台连接到 SparkStreaming 程序并对 RDD 上的 RDD 运行任意 Spark 操作来在 D-Streams 上互动地执行即时查询。例如，用户可以通过键入以下内容来查询时间范围中的最流行的单词：

```
counts.slice\("21:00", "21:05"\).topK\(10\)
```

与编写离线（基于 Hadoop 的）和在线处理应用程序的开发人员的讨论表明，这些功能具有重要的实用价值。只需在同一代码库中使用这些程序所使用的数据类型和函数就可以节省大量的开发时间，因为流和批处理系统当前有单独的 API。能够以交互方式查询流系统中的状态的能力甚至更具吸引力：它使得调试运行的计算变得简单，或者在定义流作业中的聚合时询问不期望的查询，例如，问题与网站。没有这种能力，用户通常需要等待几十分钟，数据才能进入批处理集群，即使所有相关状态都在流处理节点上的内存中。

### 3.6 Summary

为了结束我们对 D-Streams 的概述，我们将它们与表1中的连续运算符系统进行比较。主要区别是 D-Streams 将工作划分为小批量操作的小型任务。这提高了他们的最小延迟，但让他们采用高效的恢复技术。事实上，一些连续的运算符系统，如 TimeStream 和Borealis \[33,5\]，也为了确定性地执行具有多个上游父节点的运算符（通过等待流中的周期性“标点符号”）以及提供一致性。这提高了它们的延迟超过毫秒级和 D-Streams 的第二级。

![](/img/spark_paper/2013 Discretized Streams Fault-Tolerant Streaming Computation at Scale/table1.png)

# 4. System Architecture

我们已经在一个名为 Spark Streaming 的系统中实现了 D-Streams，基于 Spark 处理引擎的修改版本\[43\]。 Spark Streaming 由三个组件组成，如图6所示：

* 跟踪 D-Stream 沿袭血统图并计划任务以计算新的 RDD 分区的主服务器。

* 接收数据，存储输入和计算的 RDD 的分区，以及执行任务的工作节点。

* 用于将数据发送到系统的客户端库。

![](/img/spark_paper/2013 Discretized Streams Fault-Tolerant Streaming Computation at Scale/figure6.png)

如图所示，Spark Streaming 重用了 Spark 的许多组件，但我们还修改并添加了多个组件以启用流式传输。我们在第4.2节讨论这些变化。

从架构的角度来看，Spark Streaming 和传统流系统的主要区别在于，Spark Streaming 将其计算划分为短的，无状态的，确定性的任务，每个任务可以在集群中的任何节点上运行， - 节点。不同于传统系统中的刚性拓扑，将计算的一部分移动到另一台机器是一个主要任务，这种方法可以简单地在集群之间平衡负载，对故障做出反应或启动慢任务的推测副本。它与批处理系统中使用的方法（如MapReduce）相同，原因相同。然而，Spark Streaming 中的任务由于在内存中 RDD 上运行而短得多，通常只有50-200 ms。

Spark Streaming 中的所有状态都存储在容错数据结构（RDD）中，而不是像以前的系统中那样是长时间运行的运算符进程的一部分。 RDD 分区可以驻留在任何节点上，甚至可以在多个节点上计算，因为它们是定义计算的。系统尝试放置状态和任务以最大化数据局部性，但这种基础灵活性使得推测和并行恢复成为可能。

这些好处自然来自在批量平台（Spark）上运行，但是我们还必须做出重大改变以支持流式传输。在介绍这些更改之前，我们将更详细地讨论作业执行。

### 4.1 Application Execution

Spark Streaming 应用程序从定义一个或多个输入流开始。系统可以通过直接从客户端接收记录，或通过从外部存储系统（例如HDFS）定期加载数据来加载流，在此可能会被日志收集系统放置\[3\]。在前一种情况下，我们确保在向客户端库发送确认之前，跨两个工作节点复制新数据，因为 D-Streams 要求输入数据可靠地存储以重新计算结果。如果worker失败，客户端库会将未确认的数据发送给另一个worker。

所有数据由每个工作线程上的块存储管理，主节点上有一个跟踪器，让节点查找块的位置。因为我们从它们计算的输入块和 RDD 分区是不可变的，所以保持块的前后紧接 - 每个块被简单地给定唯一的 ID，并且具有该 ID 的任何节点可以服务它（例如，如果多个节点计算它）。块存储器将新块保存在存储器中，但是如下所述以 LRU 方式丢弃它们。

为了决定何时开始处理新的间隔，我们假设节点的时钟通过 NTP 同步，并且每个节点向主机发送它在每个间隔中接收到的块 ID 的列表，当它结束时。主机然后开始启动任务以计算该时间间隔的输出 RDD，而不需要任何进一步的同步。像其他批处理调度器\[22\]，它只是启动每个任务，当它的父进程完成。

Spark Streaming 依赖于每个时间步长内的 Spark 现有的批处理调度器\[43\]，并在像 DryadLINQ \[42\] 这样的系统中执行许多优化：

* 它管道可以被分组为单个任务的操作符，例如 map 后面跟着另一个 map。

* 它根据数据位置放置任务。

* 它控制 RDD 的分区，以避免整个网络中的数据丢失。例如，在 reduceByWindow 操作中，每个间隔的任务需要从当前间隔“添加”新的部分结果（例如，每个页面的点击计数），并且从几个时间间隔“结束”结果。调度器以相同的方式对不同间隔的状态 RDD进行分区，使得每个密钥（例如，页面）的数据在整个时间步长上始终位于同一节点上。更多细节在\[43\]中给出。

### 4.2 Optimizations for Stream Processing

虽然 Spark Streaming 基于 Spark 构建，但我们还必须对此批处理引擎进行重大的优化和更改，以支持流式处理。这些包括：

### Network communication（网络通信）

我们重写了 Spark 的数据平面，使用异步 I / O 让任务与远程输入，如 reduce 任务，更快地提取它们。

### Timestep pipelining（时间步骤流水线）

因为每个时步内的任务可能无法完美地利用集群（例如，在时步结束时，可能只有几个任务运行），我们修改了 Spark 的调度程序，以允许在当前时间之前从下一个时步提交任务一个已经完成。例如，考虑我们在图3中的第一个 map + running Reduce 作业。因为每个步骤的映射是独立的，我们可以在 timestep 1 的 reduce 完成之前开始运行时间步2的 map。

### Task Scheduling（任务调度）

我们对 Spark 的任务调度器进行了多次优化，例如手动调整控制消息的大小，以便能够每几百毫秒启动数百个任务的并行作业。

### Storage layer（存储层）

我们重写了 Spark 的存储层，以支持 RDD 的异步检查点并提高性能。因为 RDD 是不可变的，所以它们可以在网络上检查点，而不阻塞对它们的计算并减慢作业。新的存储层在可能的情况下也使用零拷贝 I / O。

### Lineage cutoff（血统截止）

因为 D 流中的 RDD 之间的沿袭血统图可以无限地增长，所以我们修改了调度器以便在检查点之后的 RDD 上进行删除，使得它的状态不会任意增长。类似地，Spark 中的其他数据结构不受限制地进行周期性清理过程。

### Master recovery（master恢复）

因为流应用程序需要运行24/7，我们添加了恢复 Spark master 的状态，如果失败（第5.3节）的支持。

有趣的是，流处理的优化也提高了 Spark 的性能基准测试标准的2倍。这是使用相同的引擎进行流和批处理的一个强大的好处。

### 4.3 Memory Management

在我们当前的 Spark Streaming 实现中，每个节点的块存储以 LRU 方式管理 RDD 分区，如果没有足够的内存，则将数据丢弃到磁盘。此外，用户可以设置最大历史超时，之后系统将会简单地忘记旧块而不执行磁盘 I / O（此超时必须大于检查点间隔）。我们发现在许多应用程序中，Spark Streaming 所需的内存并不繁重，因为计算中的状态通常比输入数据（许多应用程序计算聚合统计信息）小得多，并且任何可靠的流系统都需要复制接收到的数据网络到多个节点，就像我们一样。然而，我们也计划探讨优先使用记忆的方法。

# 5. Fault and Straggler Recovery（故障和分段恢复）

D-Streams 的确定性性质使得可能对工作者状态使用两种强大的恢复技术，这在传统流式系统中很难应用：并行恢复和推测执行。此外，它简化了 master 恢复，我们也将讨论。

### 5.1 Parallel Recovery

当节点故障时，D-Streams 允许在其他节点上并行重新计算节点上的状态 RDD 分析以及当前运行的所有任务。系统通过异步地将它们复制到其他工作节点来周期性地检查一些状态 RDD。 例如，在计算页面查看的运行计数的程序中，系统可以选择每分钟检查点数。然后，当节点失败时，系统检测所有丢失的 RDD 分区，并启动任务以从上一个检查点重新计算它们。许多任务可以同时启动以计算不同的 RDD 分区，从而允许整个集群参与恢复。如第3节所述，D-Streams 在每个时间步长中跨越 RDD 的分区并且在不依赖操作的时间步长（例如，初始map）中利用并行性，因为沿袭血统图以细粒度捕获依赖性。

为了展示并行恢复的优势，图7使用简单的分析模型将其与单节点上游备份进行比较。该模型假设系统正在从一个分钟的检查点恢复。

![](/img/spark_paper/2013 Discretized Streams Fault-Tolerant Streaming Computation at Scale/figure7.png)

在上游备份线中，单个空闲机器执行所有恢复，然后开始处理新记录。在高负载下需要很长时间才能赶上，因为它的新记录在重建旧状态时继续到达。实际上，假设故障前的负载为λ。然后在每一分钟的恢复期间，备份节点可以进行1分钟的工作，但接收λ分钟的新工作。因此，它从故障节点从上一个检查点以来的λ个工作单元完全恢复， t up·1 =λ+ t up·λ，其中t up =λ/\(1-λ\)。

在其他行，所有的机器都进行恢复，同时还处理新的记录。假设在故障之前在集群中存在N个机器，则剩余的N-1个机器现在都必须恢复λ/ N工作，而且还以\(N/N-1\)λ的速率接收新数据。它们赶上到达的流的时间t par满足t par·1 =λ/N + t par·\(N/N-1\)λ，其给出

t par =  
λ/N  
/\(1−  
\(N  
/N−1\)λ\)  
≈  
λ  
/N\(1−λ\) .

因此，对于更多的节点，并行重新循环与到达流相比，比上游备份快得多。

### 5.2 Straggler Mitigation（交错缓解）

除了失败，大集群中的另一个问题是 stragglers \[12\]。幸运的是，D-Streams 还允许我们通过运行慢任务的指定备份副本来缓解批处理系统所做的那样。这种猜测在连续的操作系统中将是困难的，因为它将需要启动节点的新副本，填充其状态，并超越慢拷贝。事实上，流处理的复制算法，如 Flux 和 DPC \[34,5\]，重点是同步两个副本。

在我们的实现中，我们使用一个简单的阈值来检测stragglers：每当任务运行比中间任务在其工作阶段超过1.4倍，我们标记为慢。也可以使用更精细的算法，但我们表明，这种方法仍然可以工作得足够好，以在一秒钟内从stragglers恢复。

### 5.3 Master Recovery

运行 Spark Streaming 24/7的最后一个要求是容忍 Spark 的 master 的失败。我们这样做：（1）​​在启动每个时间步时可靠地写入计算状态，（2）使工作者连接到一个新的主节点，并在老主节点故障时向它报告它们的 RDD 分区。简化恢复的 D-Streams 的一个关键方面是，如果给定的 RDD 被计算两次，没有问题。因为操作是确定性的，这种结果与从故障中恢复类似。 这意味着在主服务器重新连接时丢失一些正在运行的任务是很好的，因为它们可以重做。

我们当前的实现在 HDFS 中存储 D-Stream 元数据，写入（1）用户的 D-Streams 和 Scala 函数对象的图形，表示用户代码，（2）最后一个检查点的时间，以及（3） RDDs，因为 HDFS 文件中的检查点是通过每个时间步上的原子重命名更新的。恢复后，新主节点读取该文件以查找它在哪里停止，并重新连接到工作线程以确定每个 RDD 分区在内存中的哪些 RDD 分区。然后它恢复处理每个错过的时间步。虽然我们还没有优化恢复过程，但是速度相当快，100节点集群的恢复工作在12秒内完成。

# 6. Evaluation

我们使用几个基准应用程序和移植两个真正的应用程序来评估 Spark Streaming：一个商业视频分布监控系统和一个机器学习算法，用于从汽车 GPS 数据估计交通状况\[19\]。这些后面的应用程序还利用 D-Streams 的统一与批处理，我们将讨论。

### 6.1 Performance

我们使用增加复杂性的三个应用程序测试系统的性能：Grep，其找到匹配模式的输入字符串的数量;字计数，执行滑动窗口计数超过30秒;和 TopKCount，它找到过去30秒内的 k 个最频繁的字。后两个应用程序使用增加的 reduceByWindow 操作符。我们首先报告 Spark Streaming 的原始扩展性能，然后将其与两个广泛使用的流系统进行比较，从 Yahoo! S4 和 Storm 从 Twitter \[29,37\]。我们在 Amazon EC2 上的 “m1.xlarge” 节点上运行这些应用程序，每个节点具有4个内核和 15 GB RAM。

图8报告了 Spark Streaming 可以维持的最大吞吐量，同时保持端到端的稳定性低于给定的目标。 “端到端延迟”是指从记录发送到系统到合并结果出现的时间。因此，事件包括等待新输入批次开始的时间。对于1秒延迟目标，我们使用500毫秒输入间隔，而对于2秒目标，我们使用1秒间隔。在这两种情况下，我们使用100字节的输入记录。

![](/img/spark_paper/2013 Discretized Streams Fault-Tolerant Streaming Computation at Scale/figure8.png)

我们看到，Spark Streaming 几乎线性扩展到100个节点，并且对于 Grep 在100个节点上的亚秒级延迟可以处理高达 6GB / s（64Mrecords / s），而对于另一个节点可以处理高达 2.3GB / s（25Mrecords / s）更多的 CPU 密集型作业。 允许较大的延迟稍微提高吞吐量，但即使在亚秒级延迟的性能也很高。

### Comparison with Commercial Systems（与商业系统的比较）

Spark Streaming 针对 Grep 的每节点吞吐量为640,000条记录，对于4核节点上的 TopKCount 为25万条记录/ s，这与商业单节点流系统报告的速度相当。例如，Oracle CEP 在16核机器上报告了100万条记录/秒的吞吐量，Stream Base 在8个核心上报告了245,000条记录/秒\[40\]，Esper 在4个核上报告了500,000条记录/ s \[13\] \]。虽然没有理由期望 D-Streams 在每个节点上更慢或更快，但主要优点是 Spark Streaming 几乎线性扩展到100个节点。

### Comparison with S4 and Storm

我们还将 Spark Streaming 与两个开源的分布式流系统 S4 和 Storm 进行了比较。两者都是连续的运算符系统，不能在节点之间提供一致性，并且具有有限的容错保证（S4 没有，而 Storm 保证至少一次传送记录）。我们在两个系统中实现了我们的三个应用程序，但是发现 S4 在每个节点可以处理的记录数量/秒（对于 Grep 最多为7500记录/秒，对于 WordCount 最多为1000），这使得它几乎慢10倍火花和风暴。因为 Storm 更快，我们还在30节点集群上测试了它，使用100字节和1000字节的记录。

我们将 Storm 与 Spark Streaming 进行比较，如图9所示，报告吞吐量 Spark 达到亚秒级。我们看到，Storm 仍然受到较小的记录大小的不利影响，对于100字节记录，Grep 为 115k 记录/秒/节点，而 Spark 为 670k。这是尽管在我们的 Storm 实现中采取了几个预防措施来提高性能，包括从 Grep 每100个输入记录发送“批量”更新，并且 Word-Count 和 TopK 中的 “reduce” 节点每秒只发送新的计数，而不是每次计数改变。 Storm 使用1000字节的记录更快，但仍然比 Spark 慢2倍。

![](/img/spark_paper/2013 Discretized Streams Fault-Tolerant Streaming Computation at Scale/figure9.png)

### 6.2 Fault and Straggler Recovery（故障和分段恢复）

我们使用 WordCount 和 Grep 应用程序在各种条件下评估故障恢复。我们使用具有驻留在 HDFS 中的输入数据的1秒批次，并将数据速率设置为用于 WordCount 的 20MB / s /节点和用于 Grep 的 80MB / s /节点，这导致大约相等的每间隔处理时间 0.58 s 为 WordCount 和 0.54s 为 Grep。因为 WordCount 作业执行增量 reduceByKey，它的沿袭血统图形无限增长（因为每个区间减去过去的30秒的数据），所以我们给它一个10秒的检查点间隔。使用150个 map 作业和10 个 reduce 作业的任务。

我们首先在这些基本条件下报告恢复时间，如图10所示。该图显示了1或2个并发故障之前的故障之前，故障间隔期间和之后的3秒期间内的1秒钟的平均处理时间。 （这些后期的处理被延迟，同时恢复故障间隔的数据，所以我们显示系统如何重新稳定。）我们看到恢复是快速的，延迟最多1秒，即使两个故障和10秒检查点间隔。 WordCount 的恢复需要更长的时间，因为它必须重新计算数据远远回来，而 Grep 只是失去四个任务在每个失败的节点。

![](/img/spark_paper/2013 Discretized Streams Fault-Tolerant Streaming Computation at Scale/figure10.png)

### Varying the Checkpoint Interval（不同的检查点间隔）

图11显示了更改 WordCount 的检查点间隔的影响。即使每隔30秒进行检查点操作，结果也会延迟最多3.5秒。使用2s检查点，系统恢复仅0.15秒，而仍然支付少于完全复制。

![](/img/spark_paper/2013 Discretized Streams Fault-Tolerant Streaming Computation at Scale/figure11.png)

### Varying the Number of Nodes（不同节点的数量）

为了看到并行性的效果，我们还尝试了40个节点上的 WordCount 应用程序。如图12所示，将节点加倍将恢复时间减少一半。虽然考虑到 WordCount 中的滑动 reduceByWindow 操作符的线性依赖链，可能似乎很高的并行性，但并行性是因为每个时步上的局部聚合可以并行完成（参见图4） ，这些都是大部分的工作。

![](/img/spark_paper/2013 Discretized Streams Fault-Tolerant Streaming Computation at Scale/figure12.png)

### Straggler Mitigation

最后，我们尝试通过启动一个重载 CPU 的60线程进程来减慢其中一个节点而不是杀死它。图13显示了没有分段器的每个间隔的处理时间，其中分段器但是推测执行（备份任务）被禁用，并且分段器和推测被启用。投机显着改善了响应时间。注意，我们当前的实现并不试图记住跨时间的分段节点，因此尽管在慢节点上重复地启动新任务，但是这些改进发生。这表明，即使意外的 stragglers 可以快速处理。完全实现会将慢节点列入黑名单。

![](/img/spark_paper/2013 Discretized Streams Fault-Tolerant Streaming Computation at Scale/figure13.png)

### 6.3 Real Applications

我们通过移植两个真正的应用程序来评估 D 流的表现力。这两个应用程序都显着地比目前为止所展示的测试程序更复杂，并且除了流式处理外，还利用 D-Streams 执行批处理或交互式处理。

### 6.3.1 Video Distribution Monitoring

Conviva为互联网上的视频分发提供了一个商业管理平台。此平台的一个特点是能够跟踪不同地理区域，CDN，客户端设备和 ISP 的性能，这允许广播公司快速识别和响应交付问题。系统从视频播放器接收事件，并使用它们计算超过五十个度量，包括复杂度量（例如唯一的观看者）和会话级度量（如缓冲比率）。

当前应用程序在两个系统中实现：定制的用于实时数据的分布式流传输系统，以及用于历史数据和即席查询的 Hadoop / Hive 实现。拥有实时数据和历史数据至关重要，因为客户通常希望及时回到调试问题，但在这两个独立系统上实施应用程序会产生重大挑战。首先，这两个实现必须以相同的方式在计算机上进行计算。第二，在数据通过一系列 Hadoop 导入作业到达准备好临时查询的形式之前，存在几分钟的滞后。

我们通过包装 Hadoop 版本中实现的 map 和 reduce 将应用程序移植到 D-Streams。使用500行的 Spark Streaming 程序和另外一个在 Spark 中执行 Hadoop 作业的700行包装程序，我们能够以2秒的间隔计算所有度量（一个2阶段 MapReduce 作业）。我们的代码使用第3.3节中描述的跟踪操作符为每个客户端 ID 构建一个会话状态对象，并在事件到达时更新它，随后是一个滑动恢复 BeyKey 来聚合会话上的度量。

我们测量了应用程序的扩展性能，发现在64个四核 EC2 节点上，它可以处理足够的事件来支持380万个并发观众，这超过了Conviva目前为止遇到的峰值负载。图14（a）示出了缩放。

![](/img/spark_paper/2013 Discretized Streams Fault-Tolerant Streaming Computation at Scale/figure14.png)

此外，我们使用 D-Streams 添加了原始应用程序中不存在的新功能：即时流状态的即席查询。如图14（b）所示，Spark Streaming 可以在不到一秒钟内从 Scala shell 运行表示会话状态的 RDD 的即席查询。我们的集群可以轻松地在 RAM 中保留十分钟的数据，缩短历史和实时处理之间的差距，并允许单个代码库执行这两种操作。

### 6.3.2 Crowdsourced Traffic Estimation（众包流量估算）

我们将 D 流应用于 Mobile Millennium 交通信息系统\[19\]，这是一个基于机器学习的项目，用于估计城市的汽车交通状况。虽然由于专用传感器测量高速公路的交通是直接的，但是主干道路（城市中的道路）缺乏这样的基础设施。移动千禧通过使用来自装备 GPS 的汽车（例如，出租车）和运行移动应用的手机的众包 GPS 数据来攻克这个问题。

来自 GPS 数据的流量估计是具有挑战性的，因为数据嘈杂（由于高建筑物附近的 GPS 不准确）和稀疏（系统仅每分钟从每个汽车接收一个测量）。移动 Millenium 使用高度计算密集型期望最大化（EM）算法来推断条件，使用马尔可夫链蒙特卡罗和一个流量模型来估计每个道路链路的旅行时间分布。之前的实现\[19\]是 Spark 中的一个迭代批处理作业，运行超过30分钟的数据窗口。

我们使用 EM 算法的在线版本将此应用程序移植到 Spark Streaming，该算法每5秒合并一次新数据。实现是260行的 Spark Streaming 代码，并且在离线程序中包装了现有的 map 和 reduce 函数。另外，我们发现只有使用实时数据可能会导致过拟合，因为在五秒内接收的数据非常稀疏。我们利用 D-Streams 在过去十天内从历史数据中提取数据以解决这个问题。

图15显示了该算法在多达80个四核 EC2 节点上的性能。该算法几乎完美地缩放，因为它是 CPU 限制的，并且提供比批处理版本快10倍的答案。

# ![](/img/spark_paper/2013 Discretized Streams Fault-Tolerant Streaming Computation at Scale/figure15.png) 7. Discussion

我们已经提出了离散化流（D-Streams），一个用于集群的新的流处理模型。通过将计算分解为短的，确定性的任务和存储基于沿袭的数据结构（RDD）中的状态，D-Streams 可以使用强大的恢复机制，类似于批处理系统中的处理故障和分离器。

也许 D-Streams 的主要限制是它们由于批处理数据而具有固定的最小等待时间。然而，我们已经显示总延迟仍然可以低至1-2秒，这对于许多真实世界的使用情况是足够的。有趣的是，即使一些连续操作系统，如 Borealis 和 TimeStream \[5,33\]，也增加了延迟以确保确定性：Borealis 的 SUnion 运算符和 TimeStream 的 HashPartition 等待在 “heartbeat” 边界批量处理数据，以确定性顺序。因此， D 流的时空是相似的范围系统，同时提供显着更有效的恢复。

除了它们的恢复效益，我们认为 D-Streams 最重要的方面是它们表明流，批处理和交互式计算可以在同一平台上统一。由于“大”数据成为某些应用程序可以操作的唯一数据大小（例如，大型网站上的垃圾邮件检测），组织将需要这些工具来编写使用此数据的低延迟应用程序和更具交互性的应用程序，而不仅仅是到目前为止使用的周期性批处理作业。 D-Streams 在深层次上集成了这些计算模式，因为它们不仅遵循类似的 API，而且遵循与批作业相同的数据结构和容错模型。这使得丰富的功能，如组合流与离线数据或流状态运行专门查询。

最后，虽然我们提出了 D-Streams 的基本实现，但有几个领域可供未来工作：

**Expressiveness（表现力）：**一般来说，由于 D-Stream 抽象主要是一个执行策略，应该可以在其中运行大多数流算法，通过简单的“批处理”算法的执行到步骤和发出状态。这将是有趣的端口语言，如流 SQL \[4\] 或复杂事件处理模型\[14\]。

**Setting the batch interval（设置批处理间隔）：**给定任何应用程序，设置适当的批处理间隔非常重要，因为它直接决定了端到端延迟和流工作负载的吞吐量之间的权衡。目前，开发人员必须探索这种折衷并手动确定间隔。系统可以自动调整它。

**Memory usage：**我们的状态流处理模型生成新的 RDD 来存储每个操作员的状态，每批处理后的数据。在我们目前的实现中，这种高速连续运算符具有可变状态。存储不同版本的状态 RDD 对于系统执行基于沿袭血统的故障恢复至关重要。然而，可以通过仅存储这些状态 RDD 之间的增量来减少存储器使用。

**Approximate results（大致结果）：**除了重新计算丢失的工作之外，处理故障的另一种方法是返回最近的部分结果。 D-Streams通过在父节点全部完成之前简单地启动任务来提供计算部分结果的机会，并且提供沿袭血统数据以知道哪些父节点丢失。

# 8. Related Work

### Streaming Databases

诸如 Aurora，Telegraph，Borealis 和 STREAM 等流数据库是最早研究流媒体的学术系统，并开创了诸如窗口和增量运算符等概念。然而，分布式流数据库（如 Borealis）使用复制或上游备份进行恢复\[20\]。我们对他们做出两个贡献。

首先，D-Streams 提供了一种更有效的恢复机制，并行恢复，运行速度比上游备份更快，无需复制成本。并行恢复是可行的，因为 D-Streams 将计算离散化为无状态的确定性任务。相比之下，流数据库使用有状态的连续运算符模型，并且需要用于复制（例如， Borealis 的 DPC \[5\] 或 Flux \[34\]）和上游备份\[20\]的复杂协议。 Hwang 等人\[21\]唯一的并行恢复协议，只能容忍一个节点故障，不能处理 stragglers。

第二，D-Streams 也允许使用推理执行的 stragglers \[12\]。在连续运算符模型中，解决缓冲区很困难，因为每个节点都有可变状态，不能在另一个节点上重建，而不需要昂贵的串行重放过程。

### Large-scale Streaming

虽然几个最近的系统支持与 D-Stream 类似的高级 API 的流计算，但它们也缺乏离散流模型的故障和分离器恢复优势。

TimeStream \[33\] 在集群上的 Microsoft StreamInsight \[2\] 中运行连续的，有状态的操作符。它使用类似于上游备份的恢复机制，跟踪每个操作员依赖的上游数据，并通过操作员的新副本连续重放。恢复因此发生在每个操作员的单个节点上，并且花费与该操作者的处理窗口成比例的时间（例如，30秒的滑动窗口为30秒）\[33\]。相比之下，D-Streams 使用无状态转换并明确地将状态置于数据结构（RDDs）中，可以（1）异步地对绑定的恢复时间进行检查点，（2）并行重建，跨越数据分区和时间步在亚秒级恢复。 D-Streams 也可以处理 stragglers，而 TimeStream 不行。

Naiad \[27,28\] 自动递增写在 LINQ 中的数据流计算，并且是唯一能够递增迭代计算的。然而，它使用传统的同步检查点来实现容错，并且不能响应 stragglers。

MillWheel \[1\] 使用事件驱动的API 运行状态计算，但通过将所有状态写入复制存储系统（如 BigTable）来处理可靠性。

MapReduce Online\[11\] 是一个运行时的 hadoop 流，它在 maps 和 reduces 之间推送记录，并使用上游备份确保可靠性。但是，它不能恢复具有长期状态的 reduce 任务（用户必须手动将这种状态检查点放入外部系统中），并且不处理 stragglers。Meteor Shower\[41\] 也使用上游备份，并可能需要几十秒来恢复状态。 iMR \[25\] 提供了一个用于日志处理的 MapReduce API，但是在失败时可能丢失数据。 Percolator \[32\] 使用触发器运行增量计算，但不提供高级操作符，如 map 和 join。

最后，据我们所知，几乎没有一个系统支持将流与批处理和即席查询相结合，就像 D-Streams 一样。一些流数据库支持组合表和流\[15\]。

### Message Queueing Systems

类似 Storm，S4 和 Flume \[37,29,3\] 的系统提供了一个消息传递模型，其中用户写有状态代码来处理记录，但它们通常具有有限的容错保证。例如，Storm 确保在源处使用上游备份“至少一次”传递消息，但需要用户手动处理状态恢复，例如通过将所有状态保持在复制数据库中\[38\]。 Trident \[26\] 提供了一个类似于 Storm 上的 LINQ 的功能 API，它自动管理状态。然而，Trident 通过将所有状态存储在复制的数据库中以提供容错，这是昂贵的。

### Incremental Processing

CBP \[24\] 和 Comet \[18\] 通过每几分钟对新数据运行 MapReduce 作业，在传统的 MapReduce 平台上提供“批量增量处理”。虽然这些系统受益于 MapReduce 在每个时间步长内的可扩展性和错误/错误容限，但它们将所有状态存储在跨时间步长的复制磁盘文件系统中，从而导致高额开销和几十秒到几分钟的延迟。相比之下，D-Streams 可以使用 RDD 在内存中保持状态不被复制，并且可以使用沿袭在时间步长上恢复它，从而产生数量级的更低等待时间。 Incoop \[6\] 修改 Hadoop 以支持在输入文件更改时对作业输出进行增量重新计算，并且还包括用于分离器恢复的机制，但它仍然在时间步之间使用复制的磁盘上存储，并且不提供显式流 - 与诸如 windows 之类的概念的接口。

### Parallel Recovery

一个最近的将并行恢复添加到流运算符的系统是 SEEP \[8\]，它允许连续运算符通过标准 API 公开和拆分它们的状态。然而，SEEP 要求针对该 API 对每个操作符进行侵入式重写，并且不扩展到 stragglers。

我们的并行恢复机制也类似于 MapReduce，GFS 和 RAMCloud 中的技术\[12,16,30\]，所有这些都针对故障进行分区恢复工作。我们的贡献是展示如何构造一个流计算，允许跨数据分区和时间使用这种机制，并表明它可以在足够小的时间内实现流。

# 9. Conclusion

我们已经提出了 D-Streams，一种用于分布式流计算的新模型，其能够实现从故障和分离器的快速（通常是次秒）恢复，而没有复制的开销。 D-Streams 通过将数据分成小的时间步长来放弃传统的流传输智慧。这实现了强大的恢复机制，利用跨数据分区和时间的并行性。我们表明，D-Streams 可以支持广泛的运营商，可以实现高每节点吞吐量，线性扩展到100个节点，亚秒级延迟和亚秒故障恢复。最后，因为 D-Streams 使用与批处理平台相同的执行模型，所以它们与批处理和交互式查询无缝组合。我们在 Spark Streaming 中使用了此功能，以便让用户以强大的方式组合这些模型，并展示了如何为两个真实应用程序添加丰富的功能。

Spark Streaming 是开源的，现在包含在 Spark 的 [http://spark-project.org](http://spark.apache.org/)

# 10. Acknowledgements

我们感谢 SOSP 审稿人和我们的牧羊人的详细反馈。这项研究得到 NSF CISE Expeditions 奖 CCF-1139158 和 DARPA XData 奖 FA8750-12-2-0331，Google 博士奖学金以及来自 Amazon Web Services，Google，SAP，Cisco，Clearstory Data，Cloudera，Ericsson，Facebook，FitWave，通用电气，Hortonworks，华为，英特尔，微软，NetApp，甲骨文，三星，Splunk，VMware，WANdisco 和雅虎！

# References

\[1\] T. Akidau, A. Balikov, K. Bekiroglu, S. Chernyak,

J. Haberman, R. Lax, S. McVeety, D. Mills,

P. Nordstrom, and S. Whittle. MillWheel: Fault-

tolerant stream processing at internet scale. In

VLDB, 2013.

\[2\] M. H. Ali, C. Gerea, B. S. Raman, B. Sezgin,

T. Tarnavski, T. Verona, P. Wang, P. Zabback,

A. Ananthanarayan, A. Kirilov, M. Lu, A. Raiz-

man, R. Krishnan, R. Schindlauer, T. Grabs,

S. Bjeletich, B. Chandramouli, J. Goldstein,

S. Bhat, Y. Li, V. Di Nicola, X. Wang, D. Maier,

S. Grell, O. Nano, and I. Santos. Microsoft CEP

serverandonlinebehavioraltargeting. Proc.VLDB

Endow., 2\(2\):1558, Aug. 2009.

\[3\] Apache Flume. [http://incubator.apache.org/flume/](http://incubator.apache.org/flume/).

\[4\] A. Arasu, B. Babcock, S. Babu, M. Datar,

K. Ito, I. Nishizawa, J. Rosenstein, and J. Widom.

STREAM: The Stanford stream data management

system. SIGMOD 2003.

\[5\] M. Balazinska, H. Balakrishnan, S. R. Madden,

and M. Stonebraker. Fault-tolerance in the Borealis

distributed stream processing system. ACM Trans.

Database Syst., 2008.

\[6\] P. Bhatotia, A. Wieder, R. Rodrigues, U. A. Acar,

and R. Pasquin. Incoop: MapReduce for incremen-

tal computations. In SOCC ’11, 2011.

\[7\] D. Carney, U. C¸etintemel, M. Cherniack, C. Con-

vey, S. Lee, G. Seidman, M. Stonebraker, N. Tat-

bul, and S. Zdonik. Monitoring streams: a new

class of data management applications. In VLDB

’02, 2002.

\[8\] R. Castro Fernandez, M. Migliavacca, E. Kaly-

vianaki, and P. Pietzuch. Integrating scale out and

fault tolerance in stream processing using operator

state management. In SIGMOD, 2013.

\[9\] S. Chandrasekaran, O. Cooper, A. Deshpande,

M. J. Franklin, J. M. Hellerstein, W. Hong, S. Kr-

ishnamurthy, S. Madden, V. Raman, F. Reiss, and

M. Shah. TelegraphCQ: Continuous dataflow pro-

cessing for an uncertain world. In CIDR, 2003.

\[10\] M. Cherniack, H. Balakrishnan, M. Balazinska,

D. Carney, U. Cetintemel, Y. Xing, and S. B.

Zdonik. Scalable distributed stream processing. In

CIDR, 2003.

\[11\] T. Condie, N. Conway, P. Alvaro, and J. M. Heller-

stein. MapReduce online. NSDI, 2010.

\[12\] J. Dean and S. Ghemawat. MapReduce: Simplified

data processing on large clusters. In OSDI, 2004.

\[13\] EsperTech. Performance-related information.

[http://esper.codehaus.org/esper/performance/](http://esper.codehaus.org/esper/performance/)

performance.html, Retrieved March 2013.

\[14\] EsperTech. Tutorial. [http://esper.codehaus.org/](http://esper.codehaus.org/)

tutorials/tutorial/tutorial.html, Retrieved March

2013.

\[15\] M. Franklin, S. Krishnamurthy, N. Conway, A. Li,

A. Russakovsky, and N. Thombre. Continuous an-

alytics: Rethinking query processing in a network-

effect world. CIDR, 2009.

\[16\] S. Ghemawat, H. Gobioff, and S.-T. Leung. The

Google File System. In Proceedings of SOSP ’03,

2003.

\[17\] J. Hammerbacher. Who is using flume in produc-

tion? [http://www.quora.com/Flume/Who-is-using-](http://www.quora.com/Flume/Who-is-using-)

Flume-in-production/answer/Jeff-Hammerbacher.

\[18\] B. He, M. Yang, Z. Guo, R. Chen, B. Su, W. Lin,

and L. Zhou. Comet: batched stream processing

for data intensive distributed computing. In SoCC,

2010.

\[19\] T. Hunter, T. Moldovan, M. Zaharia, S. Merzgui,

J. Ma, M. J. Franklin, P. Abbeel, and A. M.

Bayen. Scaling the Mobile Millennium system in

the cloud. In SOCC ’11, 2011.

\[20\] J.-H. Hwang, M. Balazinska, A. Rasin,

U. Cetintemel, M. Stonebraker, and S. Zdonik.

High-availability algorithms for distributed stream

processing. In ICDE, 2005.

\[21\] J. hyon Hwang, Y. Xing, and S. Zdonik. A coop-

erative, self-configuring high-availability solution

for stream processing. In ICDE, 2007.

\[22\] M. Isard, M. Budiu, Y. Yu, A. Birrell, and D. Fet-

terly. Dryad: distributed data-parallel programs

from sequential building blocks. In EuroSys 07,

2007.

\[23\] S. Krishnamurthy, M. Franklin, J. Davis, D. Farina,

P. Golovko, A. Li, and N. Thombre. Continuous

analytics over discontinuous streams. In SIGMOD,

2010.

\[24\] D.Logothetis,C.Olston,B.Reed,K.C.Webb,and

K. Yocum. Stateful bulk processing for incremen-

tal analytics. SoCC, 2010.

\[25\] D. Logothetis, C. Trezzo, K. C. Webb, and

K. Yocum. In-situ MapReduce for log processing.

In USENIX ATC, 2011.

\[26\] N. Marz. Trident: a high-level ab-

straction for realtime computation.

[http://engineering.twitter.com/2012/08/trident-](http://engineering.twitter.com/2012/08/trident-)

high-level-abstraction-for.html.

\[27\] F.McSherry,D.G.Murray,R.Isaacs,andM.Isard.

Differential dataflow. In Conference on Innovative

Data Systems Research \(CIDR\), 2013.

\[28\] D. Murray, F. McSherry, R. Isaacs, M. Isard,

P. Barham, and M. Abadi. Naiad: A timely

dataflow system. In SOSP ’13, 2013.

\[29\] L. Neumeyer, B. Robbins, A. Nair, and A. Kesari.

S4: Distributed stream computing platform. In Intl.

Workshop on Knowledge Discovery Using Cloud

and Distributed Computing Platforms \(KDCloud\),

2010.

\[30\] D. Ongaro, S. M. Rumble, R. Stutsman, J. K.

Ousterhout, and M. Rosenblum. Fast crash recov-

ery in RAMCloud. In SOSP, 2011.

\[31\] Oracle. Oracle complex event processing per-

formance. [http://www.oracle.com/technetwork/](http://www.oracle.com/technetwork/)

middleware/complex-event-processing/overview/

cepperformancewhitepaper-128060.pdf, 2008.

\[32\] D. Peng and F. Dabek. Large-scale incremental

processing using distributed transactions and noti-

fications. In OSDI 2010.

\[33\] Z. Qian, Y. He, C. Su, Z. Wu, H. Zhu, T. Zhang,

L. Zhou, Y. Yu, and Z. Zhang. Timestream: Reli-

able stream computation in the cloud. In EuroSys

’13, 2013.

\[34\] M. Shah, J. Hellerstein, and E. Brewer. Highly

available, fault-tolerant, parallel dataflows. SIG-

MOD, 2004.

\[35\] Z. Shao. Real-time analytics at Face-

book. XLDB 2011, [http://www-conf.slac](http://www-conf.slac).

stanford.edu/xldb2011/talks/xldb2011 tue 0940

facebookrealtimeanalytics.pdf.

\[36\] U. Srivastava and J. Widom. Flexible time man-

agement in data stream systems. In PODS, 2004.

\[37\] Storm. [https://github.com/nathanmarz/storm/wiki](https://github.com/nathanmarz/storm/wiki).

\[38\] Guaranteed message processing \(Storm wiki\).

[https://github.com/nathanmarz/storm/wiki/](https://github.com/nathanmarz/storm/wiki/)

Guaranteeing-message-processing.

\[39\] K. Thomas, C. Grier, J. Ma, V. Paxson, and

D. Song. Design and evaluation of a real-time URL

spam filtering service. In IEEE Symposium on Se-

curity and Privacy, 2011.

\[40\] R. Tibbetts. Streambase performance &

scalability characterization. [http://www](http://www).

streambase.com/wp-content/uploads/downloads/

StreamBase White Paper Performance and

Scalability Characterization.pdf, 2009.

\[41\] H. Wang, L.-S. Peh, E. Koukoumidis, S. Tao, and

M. C. Chan. Meteor shower: A reliable stream

processing system for commodity data centers. In

IPDPS ’12, 2012.

\[42\] Y. Yu, M. Isard, D. Fetterly, M. Budiu,

´

U. Erlings-

son, P. K. Gunda, and J. Currey. DryadLINQ:

A system for general-purpose distributed data-

parallel computing using a high-level language. In

OSDI ’08, 2008.

\[43\] M. Zaharia, M. Chowdhury, T. Das, A. Dave,

J. Ma, M. McCauley, M. Franklin, S. Shenker, and

I. Stoica. Resilient distributed datasets: A fault-

tolerant abstraction for in-memory cluster comput-

ing. In NSDI, 2012.

