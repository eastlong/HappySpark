Structed Streaming教程 
=========== 
# 1 基础编程指南
## 1.1 概述
Structured Streaming （结构化流）是一种基于 Spark SQL 引擎构建的可扩展且容错的 stream processing engine （流处理引擎）。
您可以以静态数据表示批量计算的方式来表达 streaming computation （流式计算）。 Spark SQL 引擎将随着 streaming data 持续到达而增量地持续地运行，
并更新最终结果。  

## 1.2 quickStart
### 1 创建会话
```
import org.apache.spark.sql.functions._
import org.apache.spark.sql.SparkSession

val spark = SparkSession
  .builder
  .appName("StructuredNetworkWordCount")
  .getOrCreate()
  
import spark.implicits._
```
接下来，我们创建一个 streaming DataFrame ，它表示从监听 localhost:9999 的服务器上接收的 text data （文本数据），
并且将 DataFrame 转换以计算 word counts 。

```
// 创建表示从连接到 localhost:9999 的输入行 stream 的 DataFrame
val lines = spark.readStream
  .format("socket")
  .option("host", "localhost")
  .option("port", 9999)
  .load()

// 将 lines 切分为 words
val words = lines.as[String].flatMap(_.split(" "))

// 生成正在运行的 word count
val wordCounts = words.groupBy("value").count()
```


这个 lines DataFrame 表示一个包含包含 streaming text data （流文本数据） 的无边界表。此表包含了一列名为 “value” 的 strings ，并且 streaming text data 中的每一 line （行）都将成为表中的一 row （行）。请注意，这并不是正在接收的任何数据，因为我们只是设置 transformation （转换），还没有开始。接下来，我们使用 .as[String] 将 DataFrame 转换为 String 的 Dataset ，以便我们可以应用 flatMap 操作将每 line （行）切分成多个 words 。所得到的 words Dataset 包含所有的 words 。最后，我们通过将 Dataset 中 unique values （唯一的值）进行分组并对它们进行计数来定义 wordCounts DataFrame 。请注意，这是一个 streaming DataFrame ，它表示 stream 的正在运行的 word counts 。


我们现在已经设置了关于 streaming data （流数据）的 query （查询）。剩下的就是实际开始接收数据并计算 counts （计数）。为此，我们将其设置为在每次更新时将完整地计数（由 outputMode("complete") 指定）发送到控制台。然后使用 start() 启动 streaming computation （流式计算）。
