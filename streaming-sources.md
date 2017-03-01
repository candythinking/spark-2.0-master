# Streaming Sources

流式源\(streaming source\)代表用于流式查询的连续数据流。它为给定的开始和结束偏移量生成DataFrame的批次。对于容错，源必须能够在给定起始偏移的情况下重放数据。

流源应该能够使用一定范围的偏移来重放流中的过去数据的任意序列。像Apache Kafka和Amazon Kinesis这样的流式源（以及它们的记录偏移量）很好地适合这个模型。这是假设这样的结构化流可以实现端到端的一次性保证。

流源由org.apache.spark.sql.execution.streaming包中的Source contract描述。

```
import org.apache.spark.sql.execution.streaming.Source
```

有以下源实现可用：

1. FileStreamSource

2. MemoryStream

3. TextSocketSource

## Source Contract

源合同\(source contract\)要求流源提供以下功能：

1. The schema of the datasets using  schema  method.

2. The maximum offset \(of type  Offset  \) using  getOffset  method.

3. A batch for start and end offsets \(of type DataFrame\).

















