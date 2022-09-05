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
3. [下载](https://archive.apache.org/dist/hadoop/common/)安装 Hadoop，配置 `HADOOP_HOME` 环境变量，将 `HADOOP_HOME/bin` 添加至 `Path` 环境变量中。若在 Windows 上搭建，则还需要根据具体的 Hadoop 版本[下载](https://github.com/steveloughran/winutils)对应的 `winutils.exe` 和 `hadoop.dll` 文件，以避免以下报错信息：
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