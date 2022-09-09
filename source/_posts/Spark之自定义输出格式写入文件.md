---
title: Spark 之自定义输出格式写入文件
date: 2022-09-08 18:17:00
categories:
- 大数据
tags: 
- Spark
toc: true
---
## Spark 常用的保存文件方式

1. RDD 保存至文本文件
```scala
rdd.saveAsTextFile("path/result")
```
2. RDD 以指定 Hadoop 输出格式保持至文件，仅支持 (key,value) 格式的 RDD
```scala
rdd.saveHadoopFile("path/result",classOf[T],classOf[T],classOf[outputFormat])
```
3. DataFrame 以指定格式保持至文件
```scala
df.write.mode("overwrite").option("header","true").format("csv").save("path/result")
```
以上都简单的，最普遍的保存文件的方式，但有时候是不能够满足我们的需求，使用上述的文件保存方式保存之后，文件名通常是 `part-00000` 的方式保存在输出文件夹中，并且还包含数据校验和文件 `part-00000.crc` 和 `.SUCCESS` 文件，其中 `part-00000.crc` 用来校验数据的完整性，`.SUCCESS` 文件用来表示本次输出任务成功完成。
<!--more-->
## 自定义保存文件
### 创建自定义 FileoutputFormat 类
继承 `MultipleTextOutputFormat` 类并复写以下方法：
```scala
import org.apache.hadoop.fs.{FileSystem, Path}
import org.apache.hadoop.mapred.{FileOutputFormat, JobConf}
import org.apache.hadoop.mapred.lib.MultipleTextOutputFormat
import org.apache.hadoop.io.NullWritable

class CustomOutputFormat() extends MultipleTextOutputFormat[Any, Any] {

  override def generateFileNameForKeyValue(key: Any, value: Any, name: String): String = {
    //这里的key和value指的就是要写入文件的rdd对
    key.asInstanceOf[String] + ".csv"
  }

  override def generateActualKey(key: Any, value: Any): String = {
    //输出文件中只保留value 故 key 返回为空
    NullWritable.get()
  }

  override def checkOutputSpecs(ignored: FileSystem, job: JobConf): Unit = {
    val outDir: Path = FileOutputFormat.getOutputPath(job)
    if (outDir != null) {
      //相同文件名的文件自动覆盖
      //避免第二次运行分区数少于第一次,历史数据覆盖失败,直接删除已经存在的目录
      try {
        ignored.delete(outDir, true)
      } catch {
        case _: Throwable => {}
      }
      FileOutputFormat.setOutputPath(job, outDir)
    }
  }
}
```
### 将 RDD 映射为 PairRDD
```scala
val pair_rdd = rdd.map(x=>(x.split(",")(0),x)).partitionBy(new HashPartitioner(50))
```
### 调用 saveAsHadoopFile 输出
```scala
pair_rdd.saveAsHadoopFile(output, classOf[String], classOf[String], classOf[CustomOutputFormat])
```