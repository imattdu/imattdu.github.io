

## 时间语义



### 俩种时间

#### 概述

处理时间机器计算的时间

事件时间数据产生的时间



#### 使用场景：

处理时间是我们计算效率的衡量标准，而事件时间会更符合我们的业务计算逻辑



处理时间语义，一般用在对实时性要求极高、而对计算准确性要求不太高的场景，反之就是事件时间

flink1.12默认是事件时间

## 水位线



### 概述

- 水位线是基于数据的时间戳生成的， 向下游传递特殊的数据 ，用来表示当前事件时间的进展
- 水位线是单调递增
- 可以设置延迟，处理迟到数据
- 水位线为t, 则表示<t的数据均已到达





### 如何生成水位线





```java
public interface WatermarkStrategy<T> extends TimestampAssignerSupplier<T>,WatermarkGeneratorSupplier<T>{
		@Override
   TimestampAssigner<T> createTimestampAssigner(TimestampAssignerSupplier.Context context);
	@Override
  WatermarkGenerator<T> createWatermarkGenerator(WatermarkGeneratorSupplier.Context context);
}
```



水位线相关类的介绍

TimestampAssigner:时间戳分配器，指定哪个字段作为事件时间字段



WatermarkGenerator:主要负责按照既定的方式，基于时间戳生成水位线。在 WatermarkGenerator 接口中，主要又有两个方法:onEvent()和 onPeriodicEmit()。



onEvent:每个事件(数据)到来都会调用的方法，它的参数有当前事件、时间戳， 以及允许发出水位线的一个 WatermarkOutput，可以基于事件做各种操作



onPeriodicEmit:周期性调用的方法，可以由WatermarkOutput发出水位线。周期时间为处理时间，可以调用环境配置的.setAutoWatermarkInterval()方法来设置，默认为200ms。







#### 系统内置水位线

``` java
package com.matt.apitest.window;

import com.matt.apitest.beans.Event;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import java.time.Duration;

public class Watermark {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        // >=1.12 不需要设置开启watermark
        // 100ms 触发一次水位线生成
        env.getConfig().setAutoWatermarkInterval(100);
        DataStream<Event> dataStream = env.fromElements(
                new Event("a1", "1", 100L),
                new Event("a2", "2", 200L),
                new Event("a3", "3", 300L),
                new Event("a4", "4", 400L))
               /* // 有序
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forMonotonousTimestamps()
                // 指定时间字段 ms
                .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                    @Override
                    public long extractTimestamp(Event event, long l) {
                        return event.timestamp;
                    }
                })*/
                // 乱序 延迟
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ofMinutes(2))
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event event, long l) {
                                return event.timestamp;
                            }
                        }));
        env.execute("matt");
    }
}

```



水位线 = 当前最大时间戳 – 延迟时间 – 1

水位线为7 表示不会再有<7的数据来



#### 自定义水位线

自定义数据源 发送水位线

```java
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.SourceFunction;
import org.apache.flink.streaming.api.watermark.Watermark;
import java.util.Calendar;
import java.util.Random;
public class EmitWatermarkInSourceFunction {
   public static void main(String[] args) throws Exception {
      StreamExecutionEnvironment env =
      StreamExecutionEnvironment.getExecutionEnvironment();
             env.setParallelism(1);
             env.addSource(new ClickSourceWithWatermark()).print();
             env.execute();
   }
// 泛型是数据源中的类型
public static class ClickSourceWithWatermark implements SourceFunction<Event> {
       private boolean running = true;
       @Override
  public void run(SourceContext<Event> sourceContext) throws Exception { Random random = new Random();
      String[] userArr = {"Mary", "Bob", "Alice"};
      String[] urlArr = {"./home", "./cart", "./prod?id=1"};
      while (running) {
          long currTs = Calendar.getInstance().getTimeInMillis(); // 毫秒时
          String username = userArr[random.nextInt(userArr.length)]; String url = urlArr[random.nextInt(urlArr.length)];
          Event event = new Event(username, url, currTs);
          // 使用 collectWithTimestamp 方法将数据发送出去，并指明数据中的时间戳的字段
          sourceContext.collectWithTimestamp(event, event.timestamp);
          // 发送水位线
          sourceContext.emitWatermark(new Watermark(event.timestamp - 1L)); 
            Thread.sleep(1000L);
			} 
   }
   @Override
   public void cancel() {
      running = false;
   }
} 

}
```





### 为水位线传递

如果上游有三个分区发来 则取最小的作为 水位线 并发往下游







## 窗口





### 窗口分类

#### 按照驱动分

时间窗口：根据时间驱动来划分窗口

计数窗口：根据数据个数来划分窗口





#### 按照窗口分配的数据规则

##### 滚动窗口

1:00 - 1:10， 左必右开



##### 滑动窗口

需要定义窗口大小以及每次滑动的步长

1:00 - 1:10

1:05 - 1:15



##### 会话窗口：

超过多少时间没有数据 就关闭会话吗, 会话只有时间没有会话计数窗口



会话窗口特殊处理：

如果来个数据时间间隔gap 大于指定的size 任务他们属于不同会话，但是如果有一个迟到数据在他们二者之间就会把这个平衡打破， 



所以每来一个新的数据，都会创建一个新的会话窗口;然后判断已有窗口之间的距离，如果小于给定的 size，就对它们进行合并(merge) 操作



##### 全局窗口

同一个key 会公用一个窗口



### 窗口API

#### 概述

会有按键分区和非按键分区

按键分区会基于键分为多条逻辑流， 而非按键分区只是一条流，推荐使用按键分区



按键分区

``` java
stream.keyBy(...)
      .window(...)
  
  
```

非按键分区

``` java
stream.windowAll(...)
```

#### 具体使用

窗口分配器(Window Assigners)和窗口函数(Window Functions)。

``` java
stream.keyBy(<key selector>) 
  .window(<window assigner>) 
  .aggregate(<window function>)
```

window()方法需要传入一个窗口分配器，它指明了窗口的类型;

aggregate() 方法传入一个窗口函数作为参数，它用来定义窗口具体的处理逻辑。











```java
dataStream.map(new MapFunction<Event, Tuple2<String, Long>>() {
                    @Override
                    public Tuple2<String, Long> map(Event event) throws Exception {
                        return Tuple2.of(event.user, 1L);
                    }
                }).keyBy(d -> d.f0)
                // 1个参数滚动 2个参数滑动
                //.countWindow()
                // 会话
                //.window(EventTimeSessionWindows.withGap(Time.seconds(2)))
                // 滑动事件时间窗口
                //.window(SlidingEventTimeWindows.of(Time.hours(1), Time.hours(1)))
                // 滚动事件时间窗口
                .window(TumblingEventTimeWindows.of(Time.seconds(10)))
                //.sum(1).print();
                // 相同数据类型
                .reduce(new ReduceFunction<Tuple2<String, Long>>() {
                    @Override
                    public Tuple2<String, Long> reduce(Tuple2<String, Long> t1, Tuple2<String, Long> t2) throws Exception {
                        return Tuple2.of(t1.f0, t1.f1 + t2.f1);
                    }
                }).print();
```



### 窗口分配器



#### 时间窗口

##### 滚动事件时间窗口

支持2个参数， p1: 滚动窗口大小 p2:偏移量



偏移量解释： 窗口大小为1min, 正常是10:00:00 - 10:01:00

偏移量为1 那么窗口开启时间为10:00:00.001

``` java
 .window(TumblingEventTimeWindows.of(Time.seconds(10)))
```



##### 滑动事件时间窗口

2个参数

p1: 滑动窗口大小, p2:每次滑动步长

p3:可选， 和滚动窗口类似, 可指定偏移量

``` java
// 滑动事件时间窗口
.window(SlidingEventTimeWindows.of(Time.hours(1), Time.hours(1)))
```



##### 会话事件时间窗口

表示会话的超时时间，也就是最 小间隔 session gap。我们这里创建了静态会话超时时间为 10 秒的会话窗口。

``` java
// 会话
//.window(EventTimeSessionWindows.withGap(Time.seconds(2)))
```



处理时间窗口

和事件时间类似，Event改成Processing， 用法一致





#### 计数窗口



##### 滚动滑动计数窗口

``` java
// 1个参数滚动 2个参数滑动
 //.countWindow()
```

##### 全局窗口

注意使用全局窗口，必须自行定义触发器才能实现窗口计算，否则起不到任何作用。

``` java
stream.keyBy(...)
.window(GlobalWindows.create());
```



### 窗口函数

分为2类 增量聚合函数和全窗口函数



#### 增量聚合函数

ReduceFunction 和 AggregateFunction。

##### ReduceFunction

中间聚合的状态和输出的结果，都和输入的数据类型是一样的

``` java
 .reduce(new ReduceFunction<Tuple2<String, Long>>() {
                    @Override
                    public Tuple2<String, Long> reduce(Tuple2<String, Long> t1, Tuple2<String, Long> t2) throws Exception {
                        return Tuple2.of(t1.f0, t1.f1 + t2.f1);
                    }
                })
```



##### AggregateFunction 聚合函数

输入数据、中间状态、输出结果三者类型都可以不同



``` java
package com.matt.apitest.window;

import com.matt.apitest.beans.Event;
import com.matt.apitest.source.SourceTest4_UDF;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.functions.AggregateFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.assigners.SlidingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;


import java.time.Duration;
import java.util.HashSet;

// pv uv
public class WinAggregateTest2 {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        // >=1.12 不需要设置开启watermark
        // 100ms 触发一次水位线生成
        //env.getConfig().setAutoWatermarkInterval(100);
        DataStream<Event> dataStream = env.addSource(new SourceTest4_UDF.ParallelCustomSource())
                .setParallelism(1)
                // 乱序 延迟
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ZERO)
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event event, long l) {
                                return event.timestamp;
                            }
                        }));

        // 所有数据放在一起 可以根据url分开
        dataStream.keyBy(d -> true)
                .window(SlidingEventTimeWindows.of(Time.seconds(10), Time.seconds(2)))
                .aggregate(new AvgPvAggregate())
                .print();

        env.execute("matt");
    }

    public static class AvgPvAggregate implements AggregateFunction<Event, Tuple2<Long, HashSet<String>>, Double> {

        @Override
        public Tuple2<Long, HashSet<String>> createAccumulator() {
            return Tuple2.of(0L, new HashSet<>());
        }

        @Override
        public Tuple2<Long, HashSet<String>> add(Event event, Tuple2<Long, HashSet<String>> tuple2) {
            tuple2.f1.add(event.user);
            return Tuple2.of(tuple2.f0 + 1, tuple2.f1);
        }

        @Override
        public Double getResult(Tuple2<Long, HashSet<String>> tuple2) {
            return tuple2.f0 * 1.0 / tuple2.f1.size();
        }

        @Override
        public Tuple2<Long, HashSet<String>> merge(Tuple2<Long, HashSet<String>> longHashSetTuple2, Tuple2<Long, HashSet<String>> acc1) {
            return null;
        }
    }

}

```





- createAccumulator: 创建一个累加器，每个聚合任务只会调用一次
- add:每来一条数据都是调用该方法
- getResult： 获取聚合后的结果
- merge:合并俩个累加器， 常用于会话窗口





#### 全窗口函数

##### 概述

与增量聚合函数不同，全窗口函数需要先收集窗口中的数据，并在内部缓存起来，等到窗口要输出结果的时候再取出数据进行计算。



为什么要有全窗口函数

我们要做的计算必须基于全部的 数据才有效，这时做增量聚合就没什么意义了

输出的结果有可能要包含上下文中的一 些信息(比如窗口的起始时间)，这是增量聚合函数做不到的



分类

WindowFunction 和 ProcessWindowFunction。



##### WindowFunction



WindowFunction 能提供的上下文信息较少，也没有更高级的功能。 事实上，它的作用可以被 ProcessWindowFunction 全覆盖，一般在 实际应用，直接使用 ProcessWindowFunction 就可以了



``` java
stream
   .keyBy(<key selector>)
   .window(<window assigner>)
   .apply(new MyWindowFunction());
```

##### ProcessWindowFunction



``` java
package com.matt.apitest.window;

import com.matt.apitest.beans.Event;
import com.matt.apitest.source.SourceTest4_UDF;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.ProcessFunction;
import org.apache.flink.streaming.api.functions.windowing.ProcessWindowFunction;
import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;
import org.apache.flink.util.Collector;

import java.time.Duration;
import java.util.HashSet;
import java.util.Iterator;

public class WindowProcessTest1 {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        // >=1.12 不需要设置开启watermark
        // 100ms 触发一次水位线生成
        //env.getConfig().setAutoWatermarkInterval(100);
        DataStream<Event> dataStream = env.addSource(new SourceTest4_UDF.ParallelCustomSource())
                .setParallelism(1)
                // 乱序 延迟
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ZERO)
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event event, long l) {
                                return event.timestamp;
                            }
                        }));


        dataStream.keyBy(data -> true)
                .window(TumblingEventTimeWindows.of(Time.seconds(10)))
                .process(new UVCountByWindow())
                .print();

        env.execute("全窗口函数");
    }

    public static class UVCountByWindow extends ProcessWindowFunction<Event, String, Boolean, TimeWindow> {

        @Override
        public void process(Boolean key, ProcessWindowFunction<Event, String, Boolean, TimeWindow>.Context ctx, Iterable<Event> iterable, Collector<String> collector) throws Exception {
            HashSet<String> usernameSet = new HashSet<>();
            for (Event e : iterable) {
                usernameSet.add(e.user);
            }
            collector.collect(ctx.window().getStart() / 1000 + "-" + ctx.window().getEnd() / 1000 + "uv->" + usernameSet.size());
        }
    }
}

```

Event, String, Boolean, TimeWindow

输入数据 输出数据 key 窗口类型





全窗口函数 和 增量聚合函数结合使用

``` java
package com.matt.apitest.window;

import com.matt.apitest.beans.Event;
import com.matt.apitest.source.SourceTest4_UDF;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.functions.AggregateFunction;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.windowing.ProcessWindowFunction;
import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;
import org.apache.flink.util.Collector;

import java.time.Duration;
import java.util.HashSet;

// 增量 全窗口 组合
public class UVCntExample {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        // >=1.12 不需要设置开启watermark
        // 100ms 触发一次水位线生成
        //env.getConfig().setAutoWatermarkInterval(100);
        DataStream<Event> dataStream = env.addSource(new SourceTest4_UDF.ParallelCustomSource())
                .setParallelism(1)
                // 乱序 延迟
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ZERO)
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event event, long l) {
                                return event.timestamp;
                            }
                        }));


        dataStream.keyBy(d -> true)
                .window(TumblingEventTimeWindows.of(Time.seconds(10)))
                .aggregate(new UvAgg(), new UvCountResult())
                .print();
        env.execute();
    }

    public static class UvAgg implements AggregateFunction<Event, HashSet<String>, Long> {

        @Override
        public HashSet<String> createAccumulator() {
            return new HashSet<>();
        }

        @Override
        public HashSet<String> add(Event event, HashSet<String> usernameSet) {
            usernameSet.add(event.user);
            return usernameSet;
        }

        @Override
        public Long getResult(HashSet<String> acc) {
            return (long) acc.size();
        }

        @Override
        public HashSet<String> merge(HashSet<String> strings, HashSet<String> acc1) {
            return null;
        }
    }

    public static class UvCountResult extends ProcessWindowFunction<Long, String, Boolean, TimeWindow> {
        @Override
        public void process(Boolean aBoolean, ProcessWindowFunction<Long, String, Boolean, TimeWindow>.Context ctx, Iterable<Long> iterable, Collector<String> collector) throws Exception {
            Long uv = iterable.iterator().next();
            collector.collect(ctx.window().getStart() / 1000 + "-" + ctx.window().getEnd() / 1000 + "uv->" + uv);
        }
    }

}

```





## 迟到数据







### 设置水位线延迟时间



比如当前水位线真实是 15s 那么生效后的水位线是10s （延迟5s）

水位线 = eventTime - watermarkDealy - 1



### 允许窗口处理迟到的数据





``` java
package com.matt.apitest.window;

import com.matt.apitest.beans.Event;
import com.matt.apitest.model.UrlCntBO;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.OutputTag;

import java.time.Duration;

// 延迟处理
public class LateDataTest {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        // >=1.12 不需要设置开启watermark
        // 100ms 触发一次水位线生成
        //env.getConfig().setAutoWatermarkInterval(100);
        env.setParallelism(1);
        DataStream<Event> dataStream = env.socketTextStream("localhost", 777)
                .map(
                        new MapFunction<String, Event>() {
                            @Override
                            public Event map(String line) throws Exception {
                                String[] fArr = line.split(" ");
                                return new Event(fArr[0], fArr[1], Long.valueOf(fArr[2]));
                            }
                        }
                )
                // 乱序 延迟
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ofSeconds(2L))
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event event, long l) {
                                return event.timestamp;
                            }
                        }));

        dataStream.print("in");
        // 测输出流标签 {} 防止泛型查出 内部列
        OutputTag<Event> late = new OutputTag<Event>("late"){};

        SingleOutputStreamOperator<UrlCntBO> res = dataStream.keyBy(d -> d.url)
                .window(TumblingEventTimeWindows.of(Time.seconds(10)))
                // 1分钟延迟 水位线查60s 内 仍然会计算
                .allowedLateness(Time.minutes(1L))
                // 没来的数据放到
                .sideOutputLateData(late)
                .aggregate(new UrlCntCase.UrlCntAgg(), new UrlCntCase.UrlCntResult());

        res.print("res");
        DataStream<Event> sideOutput = res.getSideOutput(late);
        sideOutput.print("late");
        env.execute();
    }


}

```





如何设置迟到时间过了 还是会有部分数据没来， 可以设置将迟到的数据写到侧输出流









