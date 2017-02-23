## CSVFileFormat {#__a_id_csvfileformat_a_csvfileformat}

CSVFileFormat是一个TextBasedFileFormat，它注册名为csv的DataSources。

```
spark.read.csv("people.csv")

// or the same as above in a more verbose way
spark.read.format("csv").load("people.csv")
```



