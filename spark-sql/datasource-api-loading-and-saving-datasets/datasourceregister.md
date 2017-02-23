## DataSourceRegister {#_datasourceregister}

DataSourceRegister是一个接口，用于在其shortName别名下注册DataSources（稍后查找它们）。

```
package org.apache.spark.sql.sources

trait DataSourceRegister {
  def shortName(): String
}
```

它允许用户使用数据源别名作为完全限定类名称的格式类型。















