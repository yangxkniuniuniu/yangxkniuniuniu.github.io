---
layout:     post
title:      Flink
subtitle:   Flink-DataStream API
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
> - Update Date: 2021-08-24

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

> example: 实现统计当前小时的word count，在未达到窗口结束时间前，每1分钟或者读取到每个key的元素数量>=100时进行窗口计算并输出

在keyedStream流程使用状态的时候需要注意，flink会为每个key在特定的窗口上都会维护一个状态数据。`TriggerContext.getPartitionedState(StateDescriptor<S, ?> stateDescriptor)`方法的源码注释上有这样一句话
`Retrieves a {@link State} object that can be used to interact with fault-tolerant state that is scoped to the window and key of the current trigger invocation.` 。所以当运行从窗口1到窗口2后，会重新生成一个状态数据。

主程序:
```java
object FlinkStreamDemo {
  def main(args: Array[String]): Unit = {
    val properties = new Properties()
    properties.setProperty("bootstrap.servers", "localhost:9092")
    properties.setProperty("group.id", "test")
    val myConsumer = new FlinkKafkaConsumer[String]("test2", new SimpleStringSchema(), properties)
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    val src: DataStream[String] = env.addSource(myConsumer)
    val windowCounts = src.flatMap(_.split("\\s"))
      .map(x => (x, 1))
      .keyBy(_._1)
      .window(TumblingProcessingTimeWindows.of(Time.hours(1L)))
      .trigger(new CustomTrigger[TimeWindow](100, 60000))
      .reduce(new CustomReduceFuncion(), new CustomWindowFunction())
    windowCounts.addSink(new CustomPrintSinkFunction())
    env.execute("Kafka Window WordCount")
  }
}

class CustomPrintSinkFunction extends RichSinkFunction[String] {
  override def invoke(value: String, context: SinkFunction.Context): Unit = {
    println(s"${value}")
  }
}

class CustomReduceFuncion extends ReduceFunction[(String, Int)] {
  override def reduce(value1: (String, Int), value2: (String, Int)): (String, Int) = {
    (value1._1, value1._2 + value2._2)
  }
}

class CustomWindowFunction extends ProcessWindowFunction[(String, Int), String, String, TimeWindow] {
  override def process(key: String, context: Context, elements: Iterable[(String, Int)], out: Collector[String]): Unit = {
    for (x <- elements) {
      val newEle = s"${context.window.getStart.toString} ${x._1} ${x._2}"
      out.collect(newEle)
    }
  }
}
```

自定义Trigger
```java
class CustomTrigger[T <: Window](val maxNumber: Long, val inteval: Long) extends Trigger[Object, T]{
  private final val countStateDesc = new ReducingStateDescriptor[Long]("count-fire", new Sum(), TypeInformation.of(new TypeHint[Long] {}))
  // 使用Min()函数来保证预设的间隔和窗口的正常触发时间不冲突且窗口能正常触发，如下一个定时器的时间>窗口的触发时间，这时就要保证窗口能正常被触发
  private final val timeStateDesc = new ReducingStateDescriptor[Long]("time-fire", new Min(), TypeInformation.of(new TypeHint[Long] {}))
  override def onElement(element: Object, timestamp: Long, window: T, ctx: Trigger.TriggerContext): TriggerResult = {
    val countState = ctx.getPartitionedState(countStateDesc)
    countState.add(1L)

    // 当前key下元素数量超出预设值后触发窗口计算
    if (countState.get() >= maxNumber) {
      countState.clear()
      return TriggerResult.FIRE
    }

    // 当元素进入到一个新的窗口时，注册新的定时器
    val fireTimestampState = ctx.getPartitionedState(timeStateDesc)
    if (fireTimestampState.get() == null.asInstanceOf[Long]) {
      val start = timestamp - (timestamp % inteval)
      val end = start + inteval
      ctx.registerProcessingTimeTimer(end)
      fireTimestampState.add(end)
    }
    TriggerResult.CONTINUE
  }

  override def onProcessingTime(time: Long, window: T, ctx: Trigger.TriggerContext): TriggerResult = {
    val fireTimestampState = ctx.getPartitionedState(timeStateDesc)
    val fireTimestamp = fireTimestampState.get()
    // 实际测试时发现time并不能完全等于定时器的设定时间，如fireTimestamp=1642060800000，time=1642060799999，所以这地方暂时允许0.1s内的误差
    // 定时事件到达，触发窗口计算并将下一次触发时间写入状态中
    if (fireTimestamp != null.asInstanceOf[Long] && (fireTimestamp - time).abs < 100) {
      fireTimestampState.clear()
      fireTimestampState.add(fireTimestamp + inteval)
      ctx.registerProcessingTimeTimer(fireTimestamp + inteval)
      return TriggerResult.FIRE
    }
    return TriggerResult.CONTINUE
  }

  override def onEventTime(time: Long, window: T, ctx: Trigger.TriggerContext): TriggerResult = {
    TriggerResult.CONTINUE
  }

  override def clear(window: T, ctx: Trigger.TriggerContext): Unit = {
    val countState = ctx.getPartitionedState(countStateDesc)
    val fireTimestampState = ctx.getPartitionedState(timeStateDesc)
    val fireTimestamp = fireTimestampState.get()
    if (countState.get() != null.asInstanceOf[Long]) {
      countState.clear()
    }
    if (fireTimestamp != null.asInstanceOf[Long]) {
      ctx.deleteProcessingTimeTimer(fireTimestamp)
      fireTimestampState.clear()
    }
  }

  override def canMerge: Boolean = true

  override def onMerge(window: T, ctx: Trigger.OnMergeContext): Unit = {
    ctx.mergePartitionedState(timeStateDesc)
    ctx.mergePartitionedState(countStateDesc)
  }

  private class Sum extends ReduceFunction[Long] {
    override def reduce(value1: Long, value2: Long): Long = {
      return value1 + value2
    }
  }

  private class Min extends ReduceFunction[Long] {
    override def reduce(value1: Long, value2: Long): Long = {
      return Math.min(value1, value2)
    }
  }
}
```

输出结果如下:
```csv
1642060800000 e 1
1642060800000 e 2
1642060800000 e 3
1642060800000 e 5
1642060800000 e 5
1642060800000 c 1
1642060800000 e 5
```


## Flink算子

#### Joining
**Window Join**
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

**Interval Join**
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

**容错**：定时器具有容错能力，并且与应用程序的状态一起进行快照。如果故障恢复或从保存点启动应用程序，就会恢复定时器。

**定时器合并**：由于 Flink 仅为每个键和时间戳维护一个定时器，因此可以通过降低定时器的频率来进行合并以减少定时器的数量。对于频率为1秒的定时器（事件时间或处理时间），我们可以将目标时间向下舍入为整秒数。定时器最多提前1秒触发，但不会迟于我们的要求，精确到毫秒。因此，每个键每秒最多有一个定时器。
```java
val coalescedTime = ((ctx.timestamp + timeout) / 1000) * 1000
ctx.timerService().registerEventTimeTimer(coalescedTime)
```

**定时器删除**： 如果没有给当前Key指定时间戳注册定时器，那么停止定时器没有任何效果
```java
val timestampOfTimerToStop = ...
ctx.timerService.deleteEventTimeTimer(timestampOfTimerToStop)
```

## 用户自定义Functions&累加器、计数器
#### 累加器
注意事项：
- 首先，在需要使用累加器的用户自定义的转换 function 中创建一个累加器对象（此处是计数器）。
- 其次，必须在 rich function 的 `open()` 方法中注册累加器对象。也可以在此处定义名称。
- 然后，在操作 function 中的任何位置（包括 `open()` 和 `close()` 方法中）使用累加器。
- 最后，整体结果会存储在由执行环境的 execute() 方法返回的 JobExecutionResult 对象中（当前只有等待作业完成后执行才起作用）。

>单个作业的所有累加器共享一个命名空间。因此你可以在不同的操作 function 里面使用同一个累加器。Flink 会在内部将所有具有相同名称的累加器合并起来。

```java
val env = StreamExecutionEnvironment.getExecutionEnvironment
val dtaSource= env.socketTextStream("localhost", 9999)
  .flatMap(x => x.split("\\s"))
  .map((_, 1))
  .keyBy(_._1)
  .process(new CustomFunc())
val jobExecutionResult: JobExecutionResult = env.execute("Join Example")
jobExecutionResult.getAccumulatorResult("number-lines")
// 用户自定义Function并注册累加器对象
class cusMapFunc extends RichMapFunction[String, String] {
  private val numLines = new IntCounter()
  // 在rich function的open方法中注册累加器
  override def open(parameters: Configuration): Unit = {
    getRuntimeContext.addAccumulator("number-lines", this.numLines)
    super.open(parameters)
  }
  override def map(value: String): String = {
    this.numLines.add(1)
    value.substring(2)
  }
}
```

## 旁路输出
使用旁路输出时，首先需要定义用于标识旁路输出流的 OutputTag：
`val outputTag = OutputTag[String]("side-output")`
可以通过以下方法将数据发送到旁路输出：
- ProcessFunction
- KeyedProcessFunction
- CoProcessFunction
- KeyedCoProcessFunction
- ProcessWindowFunction
- ProcessAllWindowFunction

```java
val input: DataStream[Int] = ...
val outputTag = OutputTag[String]("side-output")
val mainDataStream = input
  .process(new ProcessFunction[Int, Int] {
    override def processElement(
        value: Int,
        ctx: ProcessFunction[Int, Int]#Context,
        out: Collector[Int]): Unit = {
      // 发送数据到主要的输出
      out.collect(value)
      // 发送数据到旁路输出
      ctx.output(outputTag, "sideout-" + String.valueOf(value))
    }
  })
// 获取旁路输出
val sideOutputStream: DataStream[String] = mainDataStream.getSideOutput(outputTag)
```

## 状态与容错
#### State
[Flink状态与容错](https://awslzhang.top/2021/01/02/Flink%E7%8A%B6%E6%80%81%E7%AE%A1%E7%90%86/)

#### Keyed State
**状态有效期TTL**
任何类型的 keyed state 都可以有 有效期 (TTL)。如果配置了 TTL 且状态值已过期，则会尽最大可能清除对应的值
```java
val ttlConfig = StateTtlConfig
    .newBuilder(Time.seconds(1))
    .setUpdateType(StateTtlConfig.UpdateType.OnCreateAndWrite)
    .setStateVisibility(StateTtlConfig.StateVisibility.NeverReturnExpired)
    .build
val stateDescriptor = new ValueStateDescriptor[String]("text state", classOf[String])
stateDescriptor.enableTimeToLive(ttlConfig)
```

TTL配置有几个选项：
-  `newBuilder` 表示数据的有效期，必选项
-  TTL更新策略：
    - `StateTtlConfig.UpdateType.OnCreateAndWrite`: 仅在创建和写入时更新(默认)
    - `StateTtlConfig.UpdateType.OnReadAndWrite`: 读取写入时更新
- 数据在过期但未被清理时的可见性配置:
    - `StateTtlConfig.StateVisibility.NeverReturnExpired`: 不返回过期数据(默认)
    - `StateTtlConfig.StateVisibility.ReturnExpireIfNotCleanedUp`: 会返回过期但未清理的数据

> - 状态的修改时间会和数据一起保存在state backend中，因此开启此特性会增加状态数据的存储
> - 暂只支持`processing time`的TTL
> - 尝试从 checkpoint/savepoint 进行恢复时，TTL 的状态（是否开启）必须和之前保持一致，否则会遇到 `StateMigrationException`
> - TTL 的配置并不会保存在 checkpoint/savepoint 中，仅对当前 Job 有效
> - 当前开启 TTL 的 map state 仅在用户值序列化器支持 null 的情况下，才支持用户值为 null。如果用户值序列化器不支持 null， 可以用 `NullableSerializer` 包装一层
> - State TTL 当前在 PyFlink DataStream API 中还不支持

#### Operator State
**示例: 自定义sinkFunction,在chekpointedFunction中进行数据缓存，然后统一下发到下游**
```java
class BufferingSink(threshold: Int = 0) extends SinkFunction[(String, Int)] with CheckpointedFunction {
  private var checkpointState: ListState[(String, Int)] = _
  private val bufferedElements = ListBuffer[(String, Int)]()
  override def invoke(value: (String, Int), context: SinkFunction.Context): Unit = {
    bufferedElements += value
    if (bufferedElements.size == threshold) {
      for (ele <- bufferedElements) {
        // send to sink
      }
  }
    bufferedElements.clear()
  }
  // 进行checkpoint时会调用
  override def snapshotState(context: FunctionSnapshotContext): Unit = {
    checkpointState.clear()
    for (ele <- bufferedElements) {
      checkpointState.add(ele)
    }
  }
  // 初始化宝库第一次自定义函数初始化和从之前的checkpoint恢复
  override def initializeState(context: FunctionInitializationContext): Unit = {
    val descriptor = new ListStateDescriptor[(String, Int)]("bufferedEleStatee", TypeInformation.of(new TypeHint[(String, Int)] {}))
    checkpointState = context.getOperatorStateStore.getListState(descriptor)
    if (context.isRestored) {
      val itero = checkpointState.get().iterator()
      while (itero.hasNext) {
        bufferedElements += itero.next()
      }
    }
  }
}
```

#### Broadcast State

#### checkpointing
Flink 中的每个方法或算子都能够是有状态的。 状态化的方法在处理单个 元素/事件 的时候存储数据，让状态成为使各个类型的算子更加精细的重要部分。 为了让状态容错，Flink 需要为状态添加 checkpoint（检查点）。Checkpoint 使得 Flink 能够恢复状态和在流中的位置，从而向应用提供和无故障执行时一样的语义。

```java
val env = StreamExecutionEnvironment.getExecutionEnvironment
// 每1000ms开始一次checkpoint
env.enableCheckpointing(1000)
// 设置模式为精确一次
env.getCheckpointConfig.setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE)
// 确认checkpoints之间的时间会进行500ms
env.getCheckpointConfig.setMinPauseBetweenCheckpoints(500)
// checkpoint必须在一分钟之内完成，否则会被抛弃
env.getCheckpointConfig.setCheckpointTimeout(1000)
// 如果task的checkpoint发生错误，会租车task失败，checkpoint仅仅会被抛弃并将错误信息回到给checkpoint coordinator
env.getCheckpointConfig.setFailOnCheckpointingErrors(false)
// 同一时间只允许一个checkpoint进行
env.getCheckpointConfig.setMaxConcurrentCheckpoints(1)
```

**State Backend**
默认情况下，状态始终保存在TaskManagers的内存中，checkpoint保存JobManager的内存中，为了合适的持久化大体量状态，可以将checkpoint状态存储到其他state backends上。`StreamExecutionEnvironment.setStateBackend()`来配置

## Table API & SQL
