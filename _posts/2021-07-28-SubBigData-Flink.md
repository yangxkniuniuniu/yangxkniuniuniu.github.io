---
layout:     post
title:      Flink
subtitle:   Flink
date:       2021-07-28
author:     owl city
header-img: img/post-bg-lamplight.jpg
catalog: true
tags:
    - 大数据
    - Flink
    - 分布式计算
---

> - Create Date: 2021-07-28
> - Update Date: 2021-07-28

> **[上层URL: 大数据学习笔记](http://owlcity.top/2019/12/01/TopBigData-BigdataLearning/)**

## 架构详解

## Flink窗口计算
#### window生命周期
一般来说，每一个窗口会有一个`Trigger`和一个`Function`。`Function`决定了窗口里面的数据会被如何进行计算处理，而`Trigger`指定了何时出发窗口计算的条件。触发器同时也可以清除任何在窗口创建后和移除前时间段内的数据，这个地方需要注意，触发器仅会清除窗口内的元素，而不会清除窗口的元数据信息，因此，新的数据仍然可以加入到窗口中。
除此之类，还可以指定`Evictor`用于在窗口被触发后、窗口计算前，进行数据的筛选移除操作，类似于`filter`操作。

#### Keyed and Non-Keyed Windows
在定义window前必须要做的操作：指定是`keyedStream`还是`nonKeyedStream`，一般使用`keyBy()`算子来区分。使用`keyedStream`可以将任务以多并行度进行运行，每个逻辑`keyedStream`都可以独立于其余部分进行计算，有相同键值的元素会被发送到同一个并行任务上运行。而`nonKeyedStream`对应的窗口计算会在同一个任务里面进行，即并行度为1

#### Window Assigners
window assigner决定了数据被如何分配到相应的窗口中，在`window()`或`windowAll()`中指定相应的`WindowAssigner`。Flink提供了绝大多数场景使用的几个Assigner。
- `Tumbing Windows`：滚动窗口
![滚动窗口](https://tva1.sinaimg.cn/large/008i3skNgy1gsxvy4pwa2j30lm0csaax.jpg)
```java
    input
    .keyBy(<key selector>)
    .window(TumblingEventTimeWindows.of(Time.days(1), Time.hours(-8)))
    .<windowed transformation>(<window function>)
```

- `Sliding Windows`: 滑动窗口
![滑动窗口](https://tva1.sinaimg.cn/large/008i3skNgy1gsxvzaalswj30lg0czmyh.jpg)
```java
    input
    .keyBy(<key selector>)
    .window(SlidingProcessingTimeWindows.of(Time.hours(12), Time.hours(1), Time.hours(-8)))
    .<windowed transformation>(<window function>)
```

- `Session Windows`: 回话窗口
Session窗口中，数据不会重复落入多个窗口中，且窗口的大小不固定。相反，在一段时间内没有收到数据后，窗口会被关闭
![Session窗口](https://tva1.sinaimg.cn/large/008i3skNgy1gsxvzs4s7nj30lh0cnq3t.jpg)
```java
val input: DataStream[T] = ...
// event-time session windows with static gap
input
    .keyBy(<key selector>)
    .window(EventTimeSessionWindows.withGap(Time.minutes(10)))
    .<windowed transformation>(<window function>)
// event-time session windows with dynamic gap
input
    .keyBy(<key selector>)
    .window(EventTimeSessionWindows.withDynamicGap(new SessionWindowTimeGapExtractor[String] {
      override def extract(element: String): Long = {
        // determine and return session gap
      }
    }))
    .<windowed transformation>(<window function>)
```

- `Global Windows`: 全局窗口
全局窗口将所有的数据都分配给一个窗口，这个窗口仅在自定义`Trigger`的时候才有用，否则不会执行任何窗口计算。
![全局窗口](https://tva1.sinaimg.cn/large/008i3skNgy1gsxwbppn1lj30lm0c2gm1.jpg)
```java
input
    .keyBy(<key selector>)
    .window(GlobalWindows.create())
    .<windowed transformation>(<window function>)
```

#### Window Function
WindowFunction一般有三种：`ReduceFunction`、 `AggregateFunction`、 `ProcessWindowFunction`。前两种执行效率会更高，因为Flink会进行增量的计算，而`ProcessWindowFunction`会得到窗口里的所有元素以及窗口的元数据信息。

- `ReduceFunction`
```java
input
    .keyBy(<key selector>)
    .window(<window assigner>)
    .reduce { (v1, v2) => (v1._1, v1._2 + v2._2) }
```

- `AggregateFunction`
```java
class AverageAgg extends AggregateFunction[(String, Long), (Long, Long), Double] {
  override def createAccumulator(): (Long, Long) = (0L, 0L)
  override def add(value: (String, Long), accumulator: (Long, Long)): (Long, Long) = (accumulator._1 + value._2, accumulator._1 + 1L)
  override def getResult(accumulator: (Long, Long)): Double = accumulator._1 / accumulator._2
  override def merge(a: (Long, Long), b: (Long, Long)): (Long, Long) = (a._1 + b._1, a._2 + b._2)
}
```

- `ProcessWindowFunction`
```java
class MyProcessWindowFunction extends ProcessWindowFunction[(String, Long), String, String, TimeWindow] {
  override def process(key: String, context: ProcessWindowFunction[(String, Long), String, String, TimeWindow]#Context, elements: lang.Iterable[(String, Long)], out: Collector[String]): Unit = {
    var count = 0L
    elements.forEach(x => count+=1)
    out.collect(s"Window ${context.window()} count: $count")
  }
}
```

#### Triggers
Triggers决定窗口何时进行计算，每一个WindowAssigner都有一个默认的Trigger，当默认的Trigger不满足需求的时候，可以使用自定义Trigger。
Flink提供的Triggers:
- `EventTimeTrigger`：event-time window assigner默认使用的触发器
- `ProcessTimeTrigger`：processing-time window assigner默认使用的触发器
- `NeverTrigger`: `GlobalWindow`默认的触发器
- `ContinuousEventTimeTrigger`: 根据间隔时间周期性触发窗口或Window的结束时间小于当前EndTime触发窗口计算
- `ContinuousProcessingTimeTrigger`: ...
- `CountTrigger`: 根据接入的数据量是否超过设定的阈值判断是否进行窗口计算
- `DeltaTrigger`:  根据接入数据计算出来的Delta指标是否超过指定的Threshold去判断是否触发窗口计算
- `PurgingTrigger`: 可以将任意触发器作为参数转换为Purge类型的触发器，计算完成后数据将被清理


Tirgger需要实现以下5个方法：
- `onElement()` : 每一个元素被加入到窗户时调用
- `onEventTime()` : 基于事件时间，当定时器被触发时调用
- `onProcessingTime()` : 基于处理时间，当定时器被触发时调用
- `onMerge()` : 在两个触发器的状态窗口合并的时候执行，比如session window
- `clear()` : 执行窗口及状态数据的清除

窗口触发返回结果的类型：
- `CONTINUE`: 不进行操作，等待
- `FIRE`: 触发计算且数据保留
- `PRUGE`: 窗口内部数据清除且不触发计算
- `FIRE_AND_PURGE`: 触发计算并清除对应的数据

## Joining
#### Window Join
sample:
```java
stream1.join(stream2)
    .where(_._1)
    .equalTo(_._1)
    .window(TumblingEventTimeWindows.of(Time.Seconds(5)))
    .apply(<JoinFunction>)
```
- Tumbling Window Join
![Tumbling](https://tva1.sinaimg.cn/large/008i3skNgy1gtr1cqsoqgj60u60asdg702.jpg)

- Sliding Window Join
![Sliding](https://tva1.sinaimg.cn/large/008i3skNgy1gtr1dowrdnj60uk0bidga02.jpg)

- Session Window Join
![Session](https://tva1.sinaimg.cn/large/008i3skNgy1gtr1eagosoj60zo0as74o02.jpg)

#### Interval Join
![Interval join](https://tva1.sinaimg.cn/large/008i3skNgy1gtr1bb8h4qj60z60am0t702.jpg)

```java
val stream1 = ...
val stream2 = ...
stream1
    .keyBy(_._1)
    .intervalJoin(stream2.keyBy(_._1))
    .between(Time.seconds(-2), Time.seconds(1))
    .process(ProcessJoinFunction())
```

## Flink算子

#### Flink物理分区
- `GlobalPartitioner`: 将数据输出到下游算子的第一个实例

- `ShufflePartitioner`: 将数据随机输出到下游算子的并发实例
```java
public int selectChannel(SerializationDelegate<StreamRecord<T>> record) {
		return random.nextInt(numberOfChannels);
	}
```

- `RebalancePartitioner`: 会先随机选择一个下游算子的实例，然后用轮询(round-robin)的方式从该实例开始循环输出，保证下游完全的负载均衡，常用来处理有倾斜的源数据流
```java
private int nextChannelToSendTo;
@Override
public void setup(int numberOfChannels) {
    super.setup(numberOfChannels);
    nextChannelToSendTo = ThreadLocalRandom.current().nextInt(numberOfChannels);
}
// stream.rebalance()
@Override
public int selectChannel(SerializationDelegate<StreamRecord<T>> record) {
    // 进行轮询，保证完全负载均衡
    nextChannelToSendTo = (nextChannelToSendTo + 1) % numberOfChannels;
    return nextChannelToSendTo;
}
```

- `KeyGroupStreamPartitioner`: `keyBy()`算子的底层采用的分区方式
```java
private final KeySelector<T, K> keySelector;
private int maxParallelism;
public KeyGroupStreamPartitioner(KeySelector<T, K> keySelector, int maxParallelism) {
    Preconditions.checkArgument(maxParallelism > 0, "Number of key-groups must be > 0!");
    this.keySelector = Preconditions.checkNotNull(keySelector);
    this.maxParallelism = maxParallelism;
}
public int getMaxParallelism() {
    return maxParallelism;
}
@Override
public int selectChannel(SerializationDelegate<StreamRecord<T>> record) {
    K key;
    try {
        key = keySelector.getKey(record.getInstance().getValue());
    } catch (Exception e) {··
        throw new RuntimeException("Could not extract key from " + record.getInstance().getValue(), e);
    }
    return KeyGroupRangeAssignment.assignKeyToParallelOperator(key, maxParallelism, numberOfChannels);
}
// 根据key, 最大并行度以及算子并行度获得分区ID
public static int assignKeyToParallelOperator(Object key, int maxParallelism, int parallelism) {
    Preconditions.checkNotNull(key, "Assigned key must not be null!");
    return computeOperatorIndexForKeyGroup(maxParallelism, parallelism, assignToKeyGroup(key, maxParallelism));
}
// 三步：先对key进行hashCode(),再调用murmurHash，然后将哈希值对最大并行对取余，最后乘以算子并行度，再除最大并并行度
public static int assignToKeyGroup(Object key, int maxParallelism) {
    Preconditions.checkNotNull(key, "Assigned key must not be null!");
    return computeKeyGroupForKeyHash(key.hashCode(), maxParallelism);
}
public static int computeKeyGroupForKeyHash(int keyHash, int maxParallelism) {
    return MathUtils.murmurHash(keyHash) % maxParallelism;
}
public static int computeOperatorIndexForKeyGroup(int maxParallelism, int parallelism, int keyGroupId) {
    return keyGroupId * parallelism / maxParallelism;
}
```

- `BroadcastPartitioner`: broadcast专用分区器，由于broadcast发挥作用必须靠`DataStream.connect()`与正常的数据流连接，广播数据总会投递给下游算子的所有并发，因此`selectChannel`就不必实现了
```java
// Note: Broadcast mode could be handled directly for all the output channels.in record writer, so it is no need to select channels via this method.
@Override
public int selectChannel(SerializationDelegate<StreamRecord<T>> record) {
    throw new UnsupportedOperationException("Broadcast partitioner does not support select channels.");
}
```

- `RescalePartitioner`: 从`selectChannel`层面来看和rebalance没有太大的区别，但是StreamGraph -> JobGraph的过程中，会对`RescalePartitioner`和`ForwardPartitioner`进行特殊处理。`POINTWISE`模式下在中间结果下发给下游节点时，会根据并行度的比值来轮询分配给下游算子实例的子集，对TaskMananger来说本地性会比较好，而在`ALL_TO_ALL`模式下是真正意义上的全局轮询分配，这样节点间的数据交换更加频繁。
```java
private int nextChannelToSendTo = -1;
@Override
public int selectChannel(SerializationDelegate<StreamRecord<T>> record) {
    if (++nextChannelToSendTo >= numberOfChannels) {
        nextChannelToSendTo = 0;
    }
    return nextChannelToSendTo;
}
// 特殊处理
if (partitioner instanceof ForwardPartitioner || partitioner instanceof RescalePartitioner) {
            jobEdge = downStreamVertex.connectNewDataSetAsInput(
                headVertex,
                DistributionPattern.POINTWISE,
                resultPartitionType);
        } else {
            jobEdge = downStreamVertex.connectNewDataSetAsInput(
                    headVertex,
                    DistributionPattern.ALL_TO_ALL,
                    resultPartitionType);
```

- `ForwardPartitioner`: 从`selectChannel`层面和global没有太大区别，但是同样它是走`POINTWISE`模式的，它会将数据输出到本地运行的下游算子的第一个实例上。
在上下游算子并行度相同的情况下，默认使用`ForwardPartitioner`, 当上下游算子并行度不同时，默认使用`RebalancePartitioner`
```java
@Override
public int selectChannel(SerializationDelegate<StreamRecord<T>> record) {
    return 0;
}
```

- `CustomPartitionerWrapper`: 自定义逻辑分区，继承`Partitioner`接口自己实现
```java
dtaSource.partitionCustom(new Partitioner[String] {
  override def partition(key: String, numPartitions: Int): Int = key.length % numPartitions
}, _._1)
```

## Process Function
#### 介绍
ProcessFunction 函数是低阶流处理算子，可以访问流应用程序所有（非循环）基本构建块：
- 事件 (数据流元素)
ProcessFunction 可以被认为是一种提供了对 KeyedState 和定时器访问的 FlatMapFunction。每在输入流中接收到一个事件，就会调用来此函数来处理。

- 状态 (容错和一致性)
对于容错的状态，ProcessFunction 可以通过 RuntimeContext 访问 KeyedState，类似于其他有状态函数访问 KeyedState。

- 定时器 (事件时间和处理时间)
定时器可以对处理时间和事件时间的变化做一些处理。每次调用 `processElement()` 都可以获得一个 Context 对象，通过该对象可以访问元素的事件时间戳以及 `TimerService`. `TimerService` 可以为尚未发生的事件时间/处理时间实例注册回调。当定时器到达某个时刻时，会调用 `onTimer()` 方法。在调用期间，所有状态再次限定为定时器创建的键，允许定时器操作 KeyedState。

```java
stream.keyBy(...).process(new MyProcessFunction)
```

#### 低阶Join
要在两个输入上实现低阶操作，应用程序可以使用 CoProcessFunction。这个函数绑定了两个不同的输入，并为来自两个不同输入的记录分别调用 `processElement1()` 和 `processElement2()`。
实现低阶 Join 通常遵循以下模式：
- 为一个输入（或两个）创建状态对象。
- 在从输入中收到元素时更新状态。
- 在从其他输入收到元素时扫描状态对象并生成 Join 结果。

例如，你可能会将客户数据与金融交易数据进行 Join，并将客户数据存储在状态中。如果你比较关心无序事件 Join 的完整性和确定性，那么当客户数据流的 Watermark 已经超过交易时间时，你可以使用定时器来计算和发出交易的 Join。

#### 实例
在以下示例中，KeyedProcessFunction 为每个键维护一个计数，并且会把一分钟(事件时间)内没有更新的键/值对输出：

- 计数，键以及最后更新的时间戳会存储在 ValueState 中，ValueState 由 key 隐含定义。
- 对于每条记录，KeyedProcessFunction 增加计数器并修改最后的时间戳。
- 该函数还会在一分钟后调用回调（事件时间）。
- 每次调用回调时，都会检查存储计数的最后修改时间与回调的事件时间时间戳，如果匹配则发送键/计数键值对（即在一分钟内没有更新）

```java
val dtaSource= env.socketTextStream("localhost", 9999)
    .flatMap(x => x.split("\\s"))
    .map((_, 1))
    .keyBy(_._1)
    .process(new CustomFunc())
// case class
case class CountWithT(key: String, count: Long, lastModify: Long)
/** keyed process function
* @param <K> Type of the key.
* @param <I> Type of the input elements.
* @param <O> Type of the output elements.
**/
class CustomFunc extends KeyedProcessFunction[String, (String, Int), (String, Long)] {
    lazy val state: ValueState[CountWithT] = getRuntimeContext.getState(new ValueStateDescriptor[CountWithT]("mystate", classOf[CountWithT]))
    // 每来一条数据会处理一次且会设置一个60s的Timer
    override def processElement(value: (String, Int), ctx: KeyedProcessFunction[String, (String, Int), (String, Long)]#Context, out: Collector[(String, Long)]): Unit = {
      val current = state.value() match {
        case null => CountWithT(value._1, 1, ctx.timestamp())
        case CountWithT(key, count, lastModify) => CountWithT(key, count + 1, ctx.timestamp())
      }
      state.update(current)
      ctx.timerService().registerEventTimeTimer(current.lastModify + 60000)
    }
    // 60s后会进行回调, 如果一分钟内没有更新，那键值对会被发送出去
    override def onTimer(timestamp: Long, ctx: KeyedProcessFunction[String, (String, Int), (String, Long)]#OnTimerContext, out: Collector[(String, Long)]): Unit = {
      state.value() match {
        case CountWithT(key, count, lastModify) if (timestamp == lastModify + 60000) => out.collect((key, count))
        case _ =>
      }
    }
}
```

#### 定时器
TimerService 在内部维护两种类型的定时器（处理时间和事件时间定时器）并排队执行。
TimerService 会删除每个键和时间戳重复的定时器，即每个键在每个时间戳上最多有一个定时器。如果为同一时间戳注册了多个定时器，则只会调用一次 `onTimer()` 方法。
>Flink同步调用 onTimer() 和 processElement() 方法。因此，用户不必担心状态的并发修改。
