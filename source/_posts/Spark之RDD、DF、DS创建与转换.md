---
title: Spark 之 RDD、DF、DS 创建与转换
date: 2022-09-08 17:17:48
categories:
- 大数据
tags: 
- Spark
toc: true
---
## Resilient Distributed Datasets（RDD）
`RDD` 是 Resilient Distributed Datasets（弹性分布式数据集）的缩写，是 Spark 中一个重要的抽象概念，它表示跨集群节点且被分区的数据集合，可以并行操作。Spark 为 RDD 提供了丰富的操作算子，可以高效处理数据。
### 创建 RDD
有两种创建 RDD 的方式：并行化驱动程序中的现有集合，或引用外部存储系统中的数据集，例如共享文件系统、HDFS、HBase 或任何提供 Hadoop InputFormat 的数据源。
```scala
// 创建 SparkContext
val conf = new SparkConf().setAppName(appName).setMaster(master)
val sc = new SparkContext(conf)
// 并行化集合
val data = Array(1, 2, 3, 4, 5)
val distData = sc.parallelize(data)
// 外部文件
val distFile = sc.textFile("data.txt")
```
<!--more-->
## Dataset（DS）
`Dataset` 是分布式数据集合。Dataset 是 Spark 1.6 中添加的一个新接口，它提供了 RDD 的优势（强类型化、使用强大 lambda 函数的能力）以及 Spark SQL 优化执行引擎的优势。
## DataFrame（DF）
`DataFrame` 其实是 `Dataset[Row]` 的别名，其中的数据是按照字段组织的，它在概念上等同于关系数据库中的表或 R/Python 中的 `data frame`。
应用程序可以使用 `SparkSession` 从现有的 RDD、Hive 表或 Spark 数据源创建 DataFrame。
```scala
// 创建 SparkSession 
val spark = SparkSession
  .builder()
  .appName("app name")
  .config("spark.sql.warehouse.dir", warehouseLocation)
  .enableHiveSupport()
  .getOrCreate()
// text file（行分割文本）
val text_df = spark.read.text("file.txt")
// json file
val json_df = spark.read.json("file.json")
// csv file
val csv_df = spark.read.csv("file.csv")
// parquet file
val parquet_df = spark.read.csv("file.parquet")
// hive table
val hive_table_df = spark.sql("select * from database_name.table_name")
```
## RDD to DF
### 通过反射推断创建 DataFrame
```scala
val rdd = sc.parallelize(Seq(("Tom", 13),("Lily", 25)))

import spark.implicits._

val df = rdd.toDF("name","age")
```
toDF() 方法定义如下：
```scala
def toDF(colNames: String*): DataFrame
```
用于将强类型数据集合转换为具有重命名列的通用 DataFrame。在从 RDD 转换为具有有意义名称的 DataFrame 时非常方便。
### 通过 StructType 创建 DataFrame
```scala
import org.apache.spark.sql.types.{IntegerType, StringType, StructField, StructType}
import org.apache.spark.sql.Row

val rdd = sc.parallelize(Seq(("Tom", "13"),("Lily", "25")))
//创建 schema
val schema = StructType(
    List(
        StructField("name", StringType, false),
        StructField("age", IntegerType, false)
    )
)
//将 rdd 映射到 rdd[row] 上，并将数据格式化为相应的类型
val rdd_row = rdd.map(x => Row(x._1,x._2.toInt))
// 创建 dataframe
val df = spark.createDataFrame(rdd_row, schema)
```
### 通过定义样例类创建 DataFrame
```scala
val rdd = sc.parallelize(Seq(("Tom", "13"),("Lily", "25")))
//创建样例类
case class User(name: String, age: Int)
//将 rdd 映射到 rdd[User] 上
val rdd_user = rdd.map(x => User(x._1,x._2.toInt))
// 创建 dataframe
val df = spark.createDataFrame(rdd_user)
// 更简单一点，可以自动推断出 schema 创建 dataframe
val df = rdd_user.toDF()
```
## RDD to DS
### 通过定义样例类创建 Dataset
```scala
val rdd = sc.parallelize(Seq(("Tom", "13"),("Lily", "25")))
//创建样例类
case class User(name: String, age: Int)
//将 rdd 映射到 rdd[User] 上
val rdd_user = rdd.map(x => User(x._1,x._2.toInt))
// 创建 dataframe
val ds = spark.createDataset(rdd_user)
// 更简单一点，可以自动推断出 schema 创建 dataset
val ds = rdd_user.toDS()
```
## DS/DF to RDD
```scala
val df = spark.read.csv("file.csv")
// 获取 rdd
val rdd = df.rdd
```
## DF to DS
```scala
//创建样例类
case class User(name: String, age: Int)
val ds = DataFrame.map(x=> User(x.getAs(0), x.getAs(1)))
```
## DS to DF
```scala
val df = DataSet[DataTypeClass].toDF()
```