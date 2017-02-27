## Latent Dirichlet Allocation \(LDA\) {#_latent_dirichlet_allocation_lda}

| Note | Information here are based almost exclusively from the blog post [Topic modeling with LDA: MLlib meets GraphX](https://databricks.com/blog/2015/03/25/topic-modeling-with-lda-mllib-meets-graphx.html). |
| :--- | :--- |


**Topic modeling **是一种在识别文档中的隐藏主题结构时非常有用的模型类型。广义地说，它旨在在非结构化文档集合中找到结构。一旦结构被“发现”，您可以回答以下问题：

* 什么是文档X？

* 文档X和Y有多相似？

* 如果我对主题Z感兴趣，我应该先阅读哪些文档？

Spark MLlib为Latent Dirichlet Allocation（LDA）提供了开箱即用的支持，这是基于GraphX构建的第一个MLlib算法。

主题模型\(**Topic models**\)自动推断文档集合中讨论的主题。

### Example {#__a_id_example_a_example}

| Caution | Use Tokenizer, StopWordsRemover, CountVectorizer, and finally LDA in a pipeline. |
| :--- | :--- |


























