## Tokenizer {#__a_id_tokenizer_a_tokenizer}

Tokenizer是一个一元变换器，将字符串值列转换为小写，然后用空格分隔。

```
import org.apache.spark.ml.feature.Tokenizer
val tok = new Tokenizer()

// dataset to transform
val df = Seq(
  (1, "Hello world!"),
  (2, "Here is yet another sentence.")).toDF("id", "sentence")

val tokenized = tok.setInputCol("sentence").setOutputCol("tokens").transform(df)

scala> tokenized.show(truncate = false)
+---+-----------------------------+-----------------------------------+
|id |sentence                     |tokens                             |
+---+-----------------------------+-----------------------------------+
|1  |Hello world!                 |[hello, world!]                    |
|2  |Here is yet another sentence.|[here, is, yet, another, sentence.]|
+---+-----------------------------+-----------------------------------+
```



