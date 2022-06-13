---
layout:     post
title:      Kafka
subtitle:   Kafka学习笔记
date:       2015-01-02
author:     owl city
header-img: img/post-bg-lamplight.jpg
catalog: true
tags:
    - Hadoop
    - 大数据
    - 分布式
---

> - Create Date: 2019-12-03
> - Update Date: 2019-12-03

> **[上层URL: 大数据学习笔记](http://owlcity.top/2019/12/01/TopBigData-BigdataLearning/)**

## Kafka
#### 本地安装与部署
- 环境要求：java环境，brew工具
    - 1.brew install kafka
    - 2.启动kafka(kafka依赖zookeepee,需先启动zookeeper)：
        ```shell
        brew services start zookeeper
        brew services start kafka
        ```

    - 3.创建topic：`kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test`
    - 4.发送消息： `kafka-console-producer -broker-list localhost:9092 -topic test`
    - 5.消费消息：`kafka-console-consumer -bootstrap-server localhost:9092 -topic test -from-beginning`
    - 6.查看所有topic: `kafka-topics --list --zookeeper localhost:2181`

#### kafka常用消费代码示例
![kafka消费](https://tva1.sinaimg.cn/large/e6c9d24egy1h1jynzqokfj20yi0is0vd.jpg)

#### kafka rebalance
[consumer group & rebalance](https://www.lixueduan.com/post/kafka/11-consumer-group-rebalance/)

## Kafka参数配置
#### kafka生产者配置参数
`boostrap.servers`：用于建立与kafka集群连接的host/port组。数据将会在所有servers上均衡加载，不管哪些server是指定用于bootstrapping。这个列表格式：host1:port1,host2:port2,…

`acks`：此配置实际上代表了数据备份的可用性。

`acks=0`： 设置为0表示producer不需要等待任何确认收到的信息。副本将立即加到socket buffer并认为已经发送。没有任何保障可以保证此种情况下server已经成功接收数据，同时重试配置不会发生作用

`acks=1`： 这意味着至少要等待leader已经成功将数据写入本地log，但是并没有等待所有follower是否成功写入。这种情况下，如果follower没有成功备份数据，而此时leader又挂掉，则消息会丢失。

`acks=all`： 这意味着leader需要等待所有备份都成功写入日志，这种策略会保证只要有一个备份存活就不会丢失数据。这是最强的保证。

`buffer.memory`：producer可以用来缓存数据的内存大小。如果数据产生速度大于向broker发送的速度，producer会阻塞或者抛出异常，以“block.on.buffer.full”来表明。

`compression.type`：producer用于压缩数据的压缩类型。默认是无压缩。正确的选项值是none、gzip、snappy。压缩最好用于批量处理，批量处理消息越多，压缩性能越好。

`retries`：设置大于0的值将使客户端重新发送任何数据，一旦这些数据发送失败。注意，这些重试与客户端接收到发送错误时的重试没有什么不同。允许重试将潜在的改变数据的顺序，如果这两个消息记录都是发送到同一个partition，则第一个消息失败第二个发送成功，则第二条消息会比第一条消息出现要早。

`batch.size`：producer将试图批处理消息记录，以减少请求次数。这将改善client与server之间的性能。这项配置控制默认的批量处理消息字节数。

`client.id`：当向server发出请求时，这个字符串会发送给server。目的是能够追踪请求源头，以此来允许ip/port许可列表之外的一些应用可以发送信息。这项应用可以设置任意字符串，因为没有任何功能性的目的，除了记录和跟踪。

`linger.ms`：producer组将会汇总任何在请求与发送之间到达的消息记录一个单独批量的请求。通常来说，这只有在记录产生速度大于发送速度的时候才能发生。

`max.request.size`：请求的最大字节数。这也是对最大记录尺寸的有效覆盖。注意：server具有自己对消息记录尺寸的覆盖，这些尺寸和这个设置不同。此项设置将会限制producer每次批量发送请求的数目，以防发出巨量的请求。

`receive.buffer.bytes`：TCP receive缓存大小，当阅读数据时使用。

`send.buffer.bytes`：TCP send缓存大小，当发送数据时使用。

`timeout.ms`：此配置选项控制server等待来自followers的确认的最大时间。如果确认的请求数目在此时间内没有实现，则会返回一个错误。这个超时限制是以server端度量的，没有包含请求的网络延迟。

`block.on.buffer.full`：当我们内存缓存用尽时，必须停止接收新消息记录或者抛出错误。默认情况下，这个设置为真，然而某些阻塞可能不值得期待，因此立即抛出错误更好。设置为false则会这样：producer会抛出一个异常错误：BufferExhaustedException， 如果记录已经发送同时缓存已满。

`metadata.fetch.timeout.ms`：是指我们所获取的一些元素据的第一个时间数据。元素据包含：topic，host，partitions。此项配置是指当等待元素据fetch成功完成所需要的时间，否则会抛出异常给客户端。

`metadata.max.age.ms`：以微秒为单位的时间，是在我们强制更新metadata的时间间隔。即使我们没有看到任何partition leadership改变。

`metric.reporters`：类的列表，用于衡量指标。实现MetricReporter接口，将允许增加一些类，这些类在新的衡量指标产生时就会改变。JmxReporter总会包含用于注册JMX统计

`metrics.num.samples`：用于维护metrics的样本数。

`metrics.sample.window.ms`：metrics系统维护可配置的样本数量，在一个可修正的window size。这项配置配置了窗口大小，例如。我们可能在30s的期间维护两个样本。当一个窗口推出后，我们会擦除并重写最老的窗口。

`recoonect.backoff.ms`：连接失败时，当我们重新连接时的等待时间。这避免了客户端反复重连。

`retry.backoff.ms`：在试图重试失败的produce请求之前的等待时间。避免陷入发送-失败的死循环中。

#### kafka消费者配置参数
`group.id`：用来唯一标识consumer进程所在组的字符串，如果设置同样的group id，表示这些processes都是属于同一个consumer group。

`zookeeper.connect`：指定zookeeper的连接的字符串，格式是hostname：port, hostname：port…

`consumer.id`：不需要设置，一般自动产生

`socket.timeout.ms`：网络请求的超时限制。真实的超时限制是max.fetch.wait+socket.timeout.ms。默认3000

`socket.receive.buffer.bytes`：socket用于接收网络请求的缓存大小。默认64*1024。

`fetch.message.max.bytes`：每次fetch请求中，针对每次fetch消息的最大字节数。默认1024*1024。这些字节将会督导用于每个partition的内存中，因此，此设置将会控制consumer所使用的memory大小。这个fetch请求尺寸必须至少和server允许的最大消息尺寸相等，否则，producer可能发送的消息尺寸大于consumer所能消耗的尺寸。

`num.consumer.fetchers`：用于fetch数据的fetcher线程数。默认1

`auto.commit.enable`：如果为真，consumer所fetch的消息的offset将会自动的同步到zookeeper。这项提交的offset将在进程挂掉时，由新的consumer使用。默认true。

`auto.commit.interval.ms`：consumer向zookeeper提交offset的频率，单位是秒。默认60*1000。

`queued.max.message.chunks`：用于缓存消息的最大数目，每个chunk必须和fetch.message.max.bytes相同。默认2。

`rebalance.max.retries`：当新的consumer加入到consumer group时，consumers集合试图重新平衡分配到每个consumer的partitions数目。如果consumers集合改变了，当分配正在执行时，这个重新平衡会失败并重入。默认4

`fetch.min.bytes`：每次fetch请求时，server应该返回的最小字节数。如果没有足够的数据返回，请求会等待，直到足够的数据才会返回。

`fetch.wait.max.ms`：如果没有足够的数据能够满足fetch.min.bytes，则此项配置是指在应答fetch请求之前，server会阻塞的最大时间。默认100

`rebalance.backoff.ms`：在重试reblance之前backoff时间。默认2000

`refresh.leader.backoff.ms`：在试图确定某个partition的leader是否失去他的leader地位之前，需要等待的backoff时间。默认200

`auto.offset.reset`：**zookeeper中没有初始化的offset时**，如果offset是以下值的回应：
    - lastest：自动复位offset为lastest的offset
    - earliest：自动复位offset为earliest的offset
    - none：向consumer抛出异常

`consumer.timeout.ms`：如果没有消息可用，即使等待特定的时间之后也没有，则抛出超时异常

`exclude.internal.topics`：是否将内部topics的消息暴露给consumer。默认true。

`paritition.assignment.strategy`：选择向consumer 流分配partitions的策略，可选值：range，roundrobin。默认range。循环的partition分配器分配所有可用的partitions以及所有可用consumer线程。它会将partition循环的分配到consumer线程上。如果所有consumer实例的订阅都是确定的，则partitions的划分是确定的分布。循环分配策略只有在以下条件满足时才可以：（1）每个topic在每个consumer实力上都有同样数量的数据流。（2）订阅的topic的集合对于consumer group中每个consumer实例来说都是确定的

`client.id`：是用户特定的字符串，用来在每次请求中帮助跟踪调用。它应该可以逻辑上确认产生这个请求的应用。

`zookeeper.session.timeout.ms`：zookeeper 会话的超时限制。默认6000。如果consumer在这段时间内没有向zookeeper发送心跳信息，则它会被认为挂掉了，并且reblance将会产生

`zookeeper.connection.timeout.ms`：客户端在建立通zookeeper连接中的最大等待时间。默认6000

`zookeeper.sync.time.ms`：ZK follower可以落后ZK leader的最大时间。默认1000

`offsets.storage`：用于存放offsets的地点： zookeeper或者kafka。默认zookeeper。

`offset.channel.backoff.ms`：重新连接offsets channel或者是重试失败的offset的fetch/commit请求的backoff时间。默认1000

`offsets.channel.socket.timeout.ms`：当读取offset的fetch/commit请求回应的socket 超时限制。此超时限制是被consumerMetadata请求用来请求offset管理。默认10000。

`offsets.commit.max.retries`：重试offset commit的次数。这个重试只应用于offset commits在shut-down之间。默认5。

`dual.commit.enabled`：如果使用“kafka”作为offsets.storage，你可以二次提交offset到zookeeper(还有一次是提交到kafka）。在zookeeper-based的offset storage到kafka-based的offset storage迁移时，这是必须的。对任意给定的consumer group来说，比较安全的建议是当完成迁移之后就关闭这个选项
