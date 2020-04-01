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
