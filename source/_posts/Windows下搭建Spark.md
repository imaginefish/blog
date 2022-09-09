---
title: Windows 下搭建 Spark
date: 2022-09-05 13:49:20
categories:
- 大数据
tags: 
- Hadoop
- Spark
toc: true
---
## 版本选择
Spark 部署模式分为本地单机（local）和集群模式，本地单机模式常用于本地开发程序与调试。集群模式又分为 Standalone 模式、Yarn 模式、Mesos 模式
通过测试发现，以下版本组合报错信息最少
|组件|版本|
|:--:|:--:|
|Spark|3.2.2|
|Hadoop|3.3.1|
|Scala|2.12.15|
|JDK|1.8|
<!--more-->
## Spark 依赖库
`Spark 3.2.2` 的依赖库版本如下：
|依赖库|版本|
|:--:|:--:|
|Scala|2.12.15|
|Hadoop|3.3.1|

## 安装步骤
1. [下载](https://www.oracle.com/java/technologies/downloads/archive/)安装 JDK，配置 `JAVA_HOME` 环境变量，将 `JAVA_HOME/bin` 添加至 `Path` 环境变量中。
2. [下载](https://www.scala-lang.org/download/all.html)安装 Scala，根据具体的操作系统，按照官网推荐的方式安装，无需配置 `SCALA_HOME` 环境变量。
3. [下载](https://archive.apache.org/dist/hadoop/common/)安装 Hadoop，配置 `HADOOP_HOME` 环境变量，将 `HADOOP_HOME/bin` 添加至 `Path` 环境变量中。若在 Windows 上搭建，则还需要根据具体的 Hadoop 版本[下载](https://github.com/kontext-tech/winutils)对应的 `winutils.exe` 和 `hadoop.dll` 文件，放入 `HADOOP_HOME/bin` 路径下，以获得在 Windows 上运行 Spark 的支持，避免以下报错信息：
```java
Exception in thread “main” java.lang.UnsatisfiedLinkError: org.apache.hadoop.io.nativeio.NativeIO$Windows.access0(Ljava/lang/String;I)Z
```
4. [下载](https://archive.apache.org/dist/spark/)安装 Spark，配置 `SPARK_HOME` 环境变量，将 `SPARK_HOME/bin` 添加至 `Path` 环境变量中。进入 Spark 目录下的 conf 子目录下，根据需要修改 `log4j.properties` 等配置文件。`log4j.properties` 常见的配置如下：
```
# 在终端输出 WARN 级别的日志，避免输出过多日志，影响查看
log4j.rootCategory=WARN, console
# 避免 ERROR ShutdownHookManager: Exception while deleting Spark temp dir 报错
log4j.logger.org.apache.spark.util.ShutdownHookManager=OFF
log4j.logger.org.apache.spark.SparkEnv=ERROR
```
## 安装 PySpark
如果上述安装步骤都已完成，就可以开始使用 Java 或 Scala 开发 Spark 程序了。对于 Python 用户，Spark 也提供了语言支持，只需要在 Spark 安装配置完成后，继续安装 PySpark 就可以使用 Python 开发 Spark 程序了：
- 使用 PyPI 安装
```bash
pip install pyspark=3.2.2
```
- 使用 Conda 安装
```bash
conda install -c conda-forge pyspark=3.2.2
```
**注意：** PySpark 的版本需要和 Spark 的版本保持一致，想要了解更多安装详情可以参考[官方文档](https://spark.apache.org/docs/latest/api/python/getting_started/install.html)

### 报错解决
- 编码错误
```bash
UnicodeDecodeError: 'gbk' codec can't decode byte 0x82 in position 120: illegal multibyte sequence
```
通过添加以下环境变量解决：
```
PYTHONIOENCODING=utf8
```
- 任务运行时报错
```bash
org.apache.spark.SparkException: Python worker failed to connect back
```
通过添加以下环境变量解决：
```
PYSPARK_PYTHON=python
```