---
layout:     post
title:      Hadoop
subtitle:   Hadoop学习笔记
date:       2018-12-24
author:     owl city
header-img: img/post-bg-lamplight.jpg
catalog: true
tags:
    - Hadoop
    - 大数据
    - 分布式
---

> **[上层URL: 大数据学习笔记](http://owlcity.top/2019/01/01/TopBigData-BigDataLearning/)**

## Kafka
#### 本地安装与部署
- 环境要求：java环境，brew工具
    - 1.brew install kafka
    - 2.启动kafka(kafka依赖zookeepee,需先启动zookeeper)：
        ```shell
        zookeeper-server-start /usr/local/etc/kafka/zookeeper.properties & kafka-server-start /usr/local/etc/kafka/server.properties
        ```

    - 3.创建topic：kafka-topic-create -zookeeper localhost:2181 -replication-factor 1 -partitions 1 -topic test
    - 4.发送消息： kafka-console-producer -broker-list localhost:9092 -topic test
    - 5.消费消息：kafka-console-consumer -bootstrap-server localhost:9092 -topic test -from-beginning
