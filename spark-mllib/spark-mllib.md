## Spark MLlib {#_spark_mllib}

| Caution | 我是新机器学习作为一门学科，Spark MLlib，特别是这个文档中的错误被认为是一个规范（不是例外）。 |
| :---: | :--- |


Spark MLlib是Apache Spark的一个模块（库/扩展），在Spark的RDD抽象之上提供分布式机器学习算法。其目标是简化大规模机器学习的开发和使用。

您可以在MLlib中找到以下类型的机器学习算法：

* Classification

* Regression

* Frequent itemsets \(via FP-growth Algorithm\)

* Recommendation

* Feature extraction and selection

* Clustering

* Statistics

* Linear Algebra+

You can also do the following using MLlib:

* Model import and export

* Pipelines

| Note | Spark MLlib中有两个机器学习库：org.apache.spark.mllib用于基于RDD的机器学习和org.apache.spark.ml中的更高级别的API，用于基于DataFrame的机器学习与管道。 |
| :---: | :--- |


机器学习使用大型数据集来识别（推断）模式并做出决策（也称为预测）。自动决策是机器学习如此吸引人的原因。您可以从数据集中教授系统，并让系统自行预测未来。

数据量（以TB或PB测量）是什么使得Spark MLlib特别重要，因为人类不可能在短时间内从数据集中提取很多价值。

Spark处理数据分配，并通过RDDs，DataFrames和最近的数据集提供巨大的数据。

用例为机器学习（因此星火MLlib附带适当的算法）：

* 安全监控和欺诈检测

* 操作优化

* 产品建议或（更广泛）营销优化

* 广告投放和优化

### Concepts {#__a_id_concepts_a_concepts}

本节介绍机器学习的概念以及如何在Spark MLlib中建模。

#### Observation {#__a_id_observation_a_observation}

观察用于了解或评估（即得出关于）观察项目的目标值的结论。

Spark将观察结果建模为DataFrame中的行。

#### Feature {#__a_id_feature_a_feature}

feature特征（又叫做维度或变量）是观察的属性。它是一个独立变量。

将Spark模型特征用作DataFrame中的列（每个特征或一组特征）。

| Note | 最终，一个算法需要一列或多列特征。 |
| :---: | :--- |


有两类特征：

* 使用离散值分类，即可能值的集合是有限的，并且可以在从一到几千的范围内。没有暗示的顺序，因此值是不可比的。

* 带有定量值的数字，即可以彼此比较的任何数值。您可以进一步将它们分类为离散和连续的特征。

#### Label {#__a_id_label_a_label}

标签是机器学习系统学习预测分配给观察的变量。

有**categorical**分类和**numerical**数字标签。

标签是依赖于其他依赖或独立变量（如特征）的因变量。

### FP-growth Algorithm {#__a_id_fp_growth_algorithm_a_fp_growth_algorithm}

Spark 1.5在用于关联规则生成和顺序模式挖掘的新算法的频繁模式挖掘能力上有显着改进。

使用Parallel FP-growth算法进行频繁项集挖掘（**Frequent Itemset Mining**）（从Spark 1.3开始）

1. [ ] [Frequent Pattern Mining in MLlib User Guide](https://spark.apache.org/docs/latest/mllib-frequent-pattern-mining.html)

2. [ ] **frequent pattern mining**

3. 揭示了特定时期最常访问的网站

4. 找到在特定区域中产生最多流量的流行路由路径

5. [ ] 将其输入建模为一组事务\(**transactions**\)，例如节点的路径。

6. [ ] 事务是一组项目\(**items**\)，例如网络节点。

7. [ ] 该算法寻找在交易中出现的项目的公共子集，例如经常被遍历的网络的子路径。

8. [ ] 一个朴素的解决方案：生成所有可能的项集并计算它们的出现

9. [ ] 当子集出现在所有事务的最小比例 - 支持中时，它被认为是模式。

10. [ ] 事务中的项是无序的

11. [ ] 分析网络日志中的流量模式

12. [ ] 该算法找到所有频繁项集，而不生成和测试所有候选项

由过滤事务构建和生成的后缀树（FP-树）

也可在Mahout，但较慢。

分布式生成关联规则（[association rules](https://en.wikipedia.org/wiki/Association_rule_learning)）（从Spark 1.5开始）。

* [ ] 在零售商的交易数据库中，具有置信度值0.8的规则{牙刷，牙线}⇒{牙膏}将指示80％的购买牙刷和牙线的顾客也在同一交易中购买牙膏。然后零售商可以使用这些信息，将牙刷和牙线卖出去，但提高牙膏的价格以提高整体利润。

* [ ] [FPGrowth](http://spark.apache.org/docs/latest/mllib-frequent-pattern-mining.html#fp-growth) model

并行顺序模式挖掘（**parallel sequential pattern mining**）（从Spark 1.5开始）

* [ ] PrefixSpan算法与修改并行化Spark的算法。

* [ ] 提取频繁的顺序模式，如路由更新，激活失败和广播超时，这可能会导致客户投诉，并主动接触到客户时发生。

### Power Iteration Clustering（迭代聚类） {#_power_iteration_clustering}

从Spark 1.3开始

无监督学习包括聚类

识别用户或网络群集之间的类似行为

MLlib中的迭代聚类（PIC），一种简单和可扩展的图聚类方法

* [ ] [PIC in MLlib User Guide](https://spark.apache.org/docs/latest/mllib-clustering.html#power-iteration-clustering-pic)

* [ ] org.apache.spark.mllib.clustering.PowerIterationClustering

* [ ] 图形算法

* [ ] 基于GraphX的第一个MLlib算法。

* [ ] 采用具有在边界上定义的相似性的无向图，并且在节点上输出聚类分配

* [ ] 使用截断的幂迭代来找到节点的非常低维的嵌入，并且该嵌入导致有效的图聚类。

* [ ] 将归一化相似度矩阵存储为具有被定义为边缘特性的归一化相似度的图

* [ ] 边缘属性被缓存并且在功率迭代期间保持静态。

* [ ] 节点的嵌入被定义为同一图形拓扑上的节点属性。

* [ ] 通过幂迭代更新嵌入，其中aggregateMessages用于计算矩阵向量乘法，幂乘法中的基本运算

* [ ] k-means用于使用嵌入对集群节点进行聚类。

* [ ] 能够清楚地区分相似度 - 由点之间的欧几里得距离表示 - 即使它们的关系是非线性的

### Further reading or watching {#__a_id_i_want_more_a_further_reading_or_watching}

* [Improved Frequent Pattern Mining in Spark 1.5: Association Rules and Sequential Patterns](https://databricks.com/blog/2015/09/28/improved-frequent-pattern-mining-in-spark-1-5-association-rules-and-sequential-patterns.html)+

* [New MLlib Algorithms in Spark 1.3: FP-Growth and Power Iteration Clustering](https://databricks.com/blog/2015/04/17/new-mllib-algorithms-in-spark-1-3-fp-growth-and-power-iteration-clustering.html)

* \(video\)[GOTO 2015 • A Taste of Random Decision Forests on Apache Spark • Sean Owen](https://youtu.be/ObiCMJ24ezs)



