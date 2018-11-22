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


