## ML Pipeline Components — Transformers {#_ml_pipeline_components_transformers}

变换器是将DataFrame映射（也称为变换）到另一个DataFrame（两者都称为数据集）的函数对象。

```
transformer: DataFrame =[transform]=> DataFrame
```

变换器准备用于机器学习算法的数据集。它们也非常有助于一般地转换DataFrames（甚至在机器学习空间之外）。

变换器是org.apache.spark.ml.Transformer抽象类的实例，提供变换族方法：

```
transform(dataset: DataFrame): DataFrame
transform(dataset: DataFrame, paramMap: ParamMap): DataFrame
transform(dataset: DataFrame, firstParamPair: ParamPair[_], otherParamPairs: ParamPair[_]*): DataFrame
```

`Transformer`是PipelineStage，因此可以是Pipeline的一部分。

Transformer的一些可用实现：

* StopWordsRemover

* Binarizer

* SQLTransformer

* VectorAssembler — a feature transformer that assembles \(merges\) multiple columns into a \(feature\) vector column.

* UnaryTransformer

  * Tokenizer

  * RegexTokenizer

  * NGram

  * HashingTF

  * OneHotEncoder

* Model

See Custom UnaryTransformer section for a custom`Transformer`implementation.

### StopWordsRemover {#__a_id_stopwordsremover_a_stopwordsremover}

StopWordsRemover是一个机器学习功能变换器，它接受一个字符串数组列，并输出一个字符串数组列，并删除所有定义的停用词。transformer附带一组标准的英语停用词作为默认值（与scikit-learn用法相同，即来自[the Glasgow Information Retrieval Group](http://ir.dcs.gla.ac.uk/resources/linguistic_utils/stop_words)\)。

| Note | 它的工作原理就像是一个UnaryTransformer，但它尚未迁移到扩展类。 |
| :---: | :--- |


StopWordsRemover类属于org.apache.spark.ml.feature包。

```
import org.apache.spark.ml.feature.StopWordsRemover
val stopWords = new StopWordsRemover
```

它接受以下参数：

```
scala> println(stopWords.explainParams)
caseSensitive: whether to do case-sensitive comparison during filtering (default: false)
inputCol: input column name (undefined)
outputCol: output column name (default: stopWords_9c2c0fdd8a68__output)
stopWords: stop words (default: [Ljava.lang.String;@5dabe7c8)
```























