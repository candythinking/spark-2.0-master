## Datasets vs DataFrames vs RDDs {#_datasets_vs_dataframes_vs_rdds}

许多人可能已经问自己为什么他们应该使用数据集，而不是使用案例类的所有Spark - RDDs的基础。

This document collects advantages of`Dataset`vs`RDD[CaseClass]`to answer [the question Dan has asked on twitter](https://twitter.com/danosipov/status/704421546203308033):

> "In \#Spark, what is the advantage of a DataSet over an RDD\[CaseClass\]?"

### Saving to or Writing from Data Sources {#_saving_to_or_writing_from_data_sources}

在Datasets中，读取或写入可以适当地使用SQLContext.read或SQLContext.write方法。

### Accessing Fields / Columns {#_accessing_fields_columns}

您可以在数据集中选择列，而不必担心列的位置。

在RDD中，您必须通过名称对案例类和访问字段执行额外的跳转。

