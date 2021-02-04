---
layout:     post
title:      Spark
subtitle:   Spark学习笔记
date:       2015-12-03
author:     owl city
header-img: img/post-bg-lamplight.jpg
catalog: true
tags:
    - 大数据
    - Spark
    - 分布式计算
---

> - Create Date: 2019-12-03
> - Update Date: 2019-12-03

> **[上层URL: 大数据学习笔记](http://owlcity.top/2019/12/01/TopBigData-BigdataLearning/)**

## 架构详解
#### 几个基本概念
1、Application(应用程序)：是指我们编写的Spark应用程序，包含驱动程序(Driver)和分布在及群众多个节点上运行的Executor代码，在执行过程中由一个或多个job组成。

2、Driver(驱动程序)：Spark中的Driver即运行Application的main方法，并且创建SparkContext，创建SparkContext的目的是为了准备Spark应用程序的运行环境。在Spark中由SparkContext负责与ClusterManager通信，进行资源的申请、任务的分配和监控。当Executor部分运行完毕后，Driver负责将SparkContext关闭。通常用SparkContext代表Driver。

3、ClusterManager(集群资源管理器)：是指在集群上获取资源的外部服务，目前常用的有以下几种：

(1)standalone：Spark自带的资源管理，由Master负责资源的管理和调度。

(2)HadoopYARN：由YARN中的ResourceManager负责资源的管理。

(3)Mesos：由Mesos中的MesosMaster负责资源的管理。

4、Worker(工作节点)：集群中任何可以运行Application代码的节点，类似于YARN中的NodeManager节点。在Standalone模式中指的是通过Slave文件配置的Worker节点。

5、Master：SparkStandalone模式下的主节点，负责管理和分配集群资源来运行SparkApplication。

6、Executor：Application运行在Worker节点上的一个进程，该进程负责运行Task，并负责将数据存在内存或者磁盘上，每个Application都有各自独立的一批Executor。


## Structured Streaming(结构化流)

#### 概览
Structured Streaming是一个基于Spark SQL引擎,可扩展的且支持容错的流处理引擎.
其编程模型之关键思想为**将持续不断的数据当做一个不断追加的表**

#### Quick Start
例:
```scala
import org.apache.spark.sql.functions._
import org.apache.spark.sql.SparkSession

val spark = SparkSession
  .builder
  .appName("StructuredNetworkWordCount")
  .getOrCreate()

import spark.implicits._

val lines = spark.readStream
    .format("socket")
    .option("host", "localhost")
    .option("port", 9999)
    .load()

val words = lines.as[String].flatMap(_.split(" "))
val wordCounts = words.groupBy("value").count()

// Start running the query that prints the running counts to the console
val query = wordCounts.writeStream
  .outputMode("complete")
  .format("console")
  .start()

query.awaitTermination()
```
