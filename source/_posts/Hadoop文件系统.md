---
title: Hadoop 文件系统
date: 2022-09-08 18:23:59
categories:
- 大数据
tags: 
- Hadoop
- HDFS
toc: true
---
Hadoop 有一个抽象的文件系统概念。Java 抽象类 `org.apache.hadoop.fs.FileSystem` 定义了 Hadoop 中一个文件系统的客户端接口，并且该抽象类有几个具体实现，其中常用的如下表：
|文件系统|URI 方案|Java 实现|描述|
|:--:|:--:|:--:|:--:|
|Local|file:///path|fs.LocalFileSystem|使用客户端检验和的本地磁盘文件系统。使用 `RawLocalFileSystem` 表示无校验和的本地磁盘文件系统。|
|HDFS|hdfs://host/path|hdfs.DistributedFileSystem|Hadoop 的分布式文件系统|
|FTP|ftp://host/path|fs.ftp.FTPFileSystem|由 FTP 服务器支持的文件系统|
|SFTP|sftp://host/path|fs.sftp.SFTPFileSystem|由 SFTP 服务器支持的文件系统|
<!--more-->
其中 `Local` 文件系统的 URI 方案比较特殊，冒号后有三个斜杠 (`///`)。这是因为 URL 标准规定 file URL 采用 `file://<host>/<path>` 形式。作为一个特例，当主机是本机时，<host> 是空字符串。因此，本地 file URL 通常具有三个斜杠。

## 从 Hadoop URL 读取数据
