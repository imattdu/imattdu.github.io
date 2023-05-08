## 处理函数



### 概述

处理函数定义数据流的转换操作



之前的算子存在的问题：

- map算子，实现mapFunction 只能获取转换后的数据
- 窗口聚合算子，aggregateFunction中出数据外，还可以获取状态
- 富函数，RichMapFunction调用getRuntimeContext() 可以获得状态，运行信息



处理函数具有上面所有算子的功能另外还可以获取水位线和注册定时器





### processFunction 介绍

#### 概述

ProcessFunction 继承了 AbstractRichFunction，泛型中第一个参数是输入类型，第二个参数是输出类型



内部定义了2个方法

processElement：来一条数据就会调用一次该方法

onTimer():定时器

#### 方法

##### processElement



value:输入的元素



context:表示当前运行的上下文，可以获取到当前的时间戳，并提供了用于查询时间和注册定时器的“定时服务”(TimerService)，以及可以将数据发送到“侧输出流”(side output)的方法.output()。



out:数据收集器，out.Collect()





##### onTimer 定时器

只在keyedStream使用



### 分类



1.ProcessFunction

最基本的处理函数，基于 DataStream 直接调用.process()时作为参数传入。

2.KeyedProcessFunction

对流按键分区后的处理函数，基于 KeyedStream 调用.process()时作为参数传入。要想使用 定时器，比如基于 KeyedStream。

3.ProcessWindowFunction

开窗之后的处理函数，也是全窗口函数的代表。基于 WindowedStream 调用.process()时作 为参数传入。

4.ProcessAllWindowFunction

同样是开窗之后的处理函数，基于 AllWindowedStream 调用.process()时作为参数传入。

5.CoProcessFunction

合并(connect)两条流之后的处理函数，基于 ConnectedStreams 调用.process()时作为参 数传入。关于流的连接合并操作

6.ProcessJoinFunction

间隔连接(interval join)两条流之后的处理函数，基于 IntervalJoined 调用.process()时作为 参数传入。

7.BroadcastProcessFunction

广播连接流处理函数，基于 BroadcastConnectedStream 调用.process()时作为参数传入。这 里的“广播连接流”BroadcastConnectedStream，是一个未 keyBy 的普通 DataStream 与一个广 播流(BroadcastStream)做连接(conncet)之后的产物。

8.KeyedBroadcastProcessFunction

按键分区的广播连接流处理函数，同样是基于 BroadcastConnectedStream 调用.process()时 作为参数传入。与 BroadcastProcessFunction 不同的是，这时的广播连接流，是一个 KeyedStream 与广播流(BroadcastStream)做连接之后的产物。







## keyedProcessFunction

### 概述

首先对流进行keyBy -> keyedStream （逻辑上分流，相同的key在同一分区计算）

keyedStream支持定时器



### 定时器

#### 概述

ProcessFunction 的上下文(Context)中提供了.timerService()方法，可以直接返回一个 TimerService 对象



TimeService支持的方法

```java
// 获取当前的处理时间
long currentProcessingTime();
// 获取当前的水位线(事件时间) 
long currentWatermark();
// 注册处理时间定时器，当处理时间超过 time 时触发 void 
registerProcessingTimeTimer(long time);
// 注册事件时间定时器，当水位线超过 time 时触发 void 
registerEventTimeTimer(long time);
// 删除触发时间为 time 的处理时间定时器
void deleteProcessingTimeTimer(long time);
// 删除触发时间为 time 的处理时间定时器 
void deleteEventTimeTimer(long time);
```

#### 注意

定时器 会以 key+时间 作为唯一键 去重 即 每个key&时间戳即是调用多次onTimer，但是只会触发一次



.onTimer()和.processElement()方法是同步调用的(synchronous)，所以不会有并发问题





### 使用



#### 处理时间 processingTime



```java
package com.matt.apitest.processfunction;

import com.matt.apitest.beans.Event;
import com.matt.apitest.source.SourceTest4_UDF;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.KeyedProcessFunction;
import org.apache.flink.util.Collector;

import java.sql.Timestamp;
import java.time.Duration;

public class ProcessingTimeTimerTest1 {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        SingleOutputStreamOperator<Event> stream = env.addSource(new SourceTest4_UDF.ParallelCustomSource());


        stream.keyBy(d -> d.user)
                .process(new KeyedProcessFunction<String, Event, String>() {
                    @Override
                    public void processElement(Event event, KeyedProcessFunction<String, Event, String>.Context context, Collector<String> collector) throws Exception {
                        long pTs = context.timerService().currentProcessingTime();
                        collector.collect(context.getCurrentKey() + "数据到达时间".concat(String.valueOf(pTs)));

                        // 注册10s 定时器
                        context.timerService().registerProcessingTimeTimer(pTs + 10 * 1000);
                    }

                    @Override
                    public void onTimer(long ts, KeyedProcessFunction<String, Event, String>.OnTimerContext ctx, Collector<String> out) throws Exception {
                        out.collect(ctx.getCurrentKey() + "定时器触发了， 触发时间" + new Timestamp(ts));
                    }
                })
                .print();

        env.execute();
    }
}

```



#### 事件时间 eventTime



``` java
package com.matt.apitest.processfunction;

import com.matt.apitest.beans.Event;
import com.matt.apitest.source.SourceTest4_UDF;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.KeyedProcessFunction;
import org.apache.flink.streaming.api.functions.ProcessFunction;
import org.apache.flink.streaming.api.functions.source.SourceFunction;
import org.apache.flink.util.Collector;

import javax.transaction.TransactionRequiredException;
import java.sql.Timestamp;
import java.time.Duration;
import java.util.concurrent.TransferQueue;

/**
 * @author matt
 * @create 2023-02-28 23:54
 * @desc xxx
 */
public class EventTimeTimerTest1 {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        SingleOutputStreamOperator<Event> stream = env.addSource(new CustomSource())
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ZERO)
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event event, long l) {
                                return event.timestamp;
                            }
                        }));

        stream.keyBy(d -> d.user)
                .process(new KeyedProcessFunction<String, Event, String>() {
                    @Override
                    public void processElement(Event event, KeyedProcessFunction<String, Event, String>.Context context, Collector<String> collector) throws Exception {
                        long ts = context.timestamp();
                        String outD = context.getCurrentKey() + new Timestamp(ts);
                        outD += "wk" + context.timerService().currentWatermark();
                        collector.collect(outD);

                        // 注册10s 定时器
                        context.timerService().registerEventTimeTimer(ts + 10 * 1000);
                    }

                    @Override
                    public void onTimer(long ts, KeyedProcessFunction<String, Event, String>.OnTimerContext ctx, Collector<String> out) throws Exception {
                        out.collect(ctx.getCurrentKey() + "定时器触发了， 触发时间" + new Timestamp(ts) + "wk"
                                + ctx.timerService().currentWatermark());
                    }
                })
                .print();

        env.execute();
    }

    public static class CustomSource implements SourceFunction<Event> {

        @Override
        public void run(SourceContext<Event> ctx) throws Exception {
            ctx.collect(new Event("d1", "/detail", 1000L));
            Thread.sleep(5000L);

            ctx.collect(new Event("d2", "/detail", 11000L));
            Thread.sleep(5000L);

            ctx.collect(new Event("d2", "/detail", 11001L));
            Thread.sleep(5000L);
        }

        @Override
        public void cancel() {
        }
    }
}

```







## processWindowFunction







### 概述

ProcessWindowFunction 继承 AbstractRichFunction

进行keyBy

``` java
ProcessWindowFunction<Long, UrlCntBO, String, TimeWindow>
```

in, out, key, winType



``` java
process(String s, ProcessWindowFunction<Long, UrlCntBO, String, TimeWindow>.Context ctx, Iterable<Long> iterable, Collector<UrlCntBO> collector
```

key, ctx, elements, collector







ProcessAllWindowFunction 使用

```java
stream.windowAll(TumblingEventTimeWindows.of(Time.seconds(10)) )
       .process(new MyProcessAllWindowFunction())
```





## 案例

### topN

查找top2的用户





#### win

UrlCntAgg: 统计每个窗口内url访问次数

UrlCntResult：输出url,开始时间，结束时间，url访问次数

TopNProcessResult： 使用结束时间作为K, 将 inputD -> listState -> 设置定时器 -> 定时器触发从listState 获取top2url



``` java
package com.matt.apitest.processfunction;

import com.matt.apitest.beans.Event;
import com.matt.apitest.model.UrlCntBO;
import com.matt.apitest.source.SourceTest4_UDF;
import com.matt.apitest.window.UrlCntCase;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.functions.AggregateFunction;
import org.apache.flink.api.common.functions.RuntimeContext;
import org.apache.flink.api.common.state.ListState;
import org.apache.flink.api.common.state.ListStateDescriptor;
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.KeyedProcessFunction;
import org.apache.flink.streaming.api.functions.windowing.ProcessAllWindowFunction;
import org.apache.flink.streaming.api.windowing.assigners.SlidingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;
import org.apache.flink.util.Collector;

import java.time.Duration;
import java.util.*;

/**
 * @author matt
 * @create 2023-03-01 00:35
 * @desc 访问最多的2个用户
 */
public class TopNCase {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        SingleOutputStreamOperator<Event> stream = env.addSource(new SourceTest4_UDF.ParallelCustomSource())
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ZERO)
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event event, long l) {
                                return event.timestamp;
                            }
                        }));

        SingleOutputStreamOperator<UrlCntBO> urlCntStream = stream.keyBy(d -> d.url)
                .window(SlidingEventTimeWindows.of(Time.seconds(10), Time.seconds(5)))
                .aggregate(new UrlCntCase.UrlCntAgg(), new UrlCntCase.UrlCntResult());

        urlCntStream.keyBy(d -> d.winEnd)
                .process(new TopNProcessResult(2))
                .print();


        env.execute();
    }

  

    public static class TopNProcessResult extends KeyedProcessFunction<Long, UrlCntBO, String> {

        public int n;
        // 状态
        private ListState<UrlCntBO> urlCntBOListState;

        public TopNProcessResult(int n) {
            this.n = n;
        }

        // 环境中获取状态
        @Override
        public void open(Configuration parameters) throws Exception {
            urlCntBOListState = getRuntimeContext().getListState(
                    new ListStateDescriptor<UrlCntBO>("url-cnt-list", Types.POJO(UrlCntBO.class))
            );
        }

        @Override
        public void processElement(UrlCntBO v, KeyedProcessFunction<Long, UrlCntBO, String>.Context context, Collector<String> collector) throws Exception {
            urlCntBOListState.add(v);
            // 定时器
            context.timerService().registerProcessingTimeTimer(context.getCurrentKey() + 1);
        }

        @Override
        public void onTimer(long timestamp, KeyedProcessFunction<Long, UrlCntBO, String>.OnTimerContext ctx, Collector<String> out) throws Exception {
            List<Tuple2<String, Long>> tList = new ArrayList<>();
            for (UrlCntBO u : urlCntBOListState.get()) {
                tList.add(Tuple2.of(u.url, u.cnt));
            }
            tList.sort(new Comparator<Tuple2<String, Long>>() {
                @Override
                public int compare(Tuple2<String, Long> o1, Tuple2<String, Long> o2) {
                    long diff = o2.f1 - o1.f1;
                    if (diff > 0) {
                        return 1;
                    } else if (diff == 0) {
                        return 0;
                    } else {
                        return -1;
                    }
                }
            });
            String sb = "";
            if (tList.size() >= 2) {
                sb = tList.get(0).f0 + tList.get(0).f1 +
                        tList.get(1).f0 + tList.get(1).f1;
            }

            out.collect(String.valueOf(sb));
        }
    }

}

```











```java
public static class UrlCntAgg implements AggregateFunction<Event, Long , Long> {

        @Override
        public Long createAccumulator() {
            return 0L;
        }

        @Override
        public Long add(Event event, Long acc) {
            return acc + 1;
        }

        @Override
        public Long getResult(Long acc) {
            return acc;
        }

        @Override
        public Long merge(Long aLong, Long acc1) {
            return null;
        }
    }


    public static class UrlCntResult extends ProcessWindowFunction<Long, UrlCntBO, String, TimeWindow> {

        @Override
        public void process(String s, ProcessWindowFunction<Long, UrlCntBO, String, TimeWindow>.Context ctx, Iterable<Long> iterable, Collector<UrlCntBO> collector) throws Exception {
            Long start = ctx.window().getStart();
            Long end = ctx.window().getEnd();
            Long uv = iterable.iterator().next();
            collector.collect(new UrlCntBO(s, uv, start, end));
        }
    }
```





#### allwin



UrlHashMapCountAgg: hashMap累加器 k:url v:urlCnt -> getResult 遍历累加器 将结果写入list<Tuple2<string, cnt>> url,cnt, sort  输出结构

UrlAllWinResult: 获取top2

``` java
  

 // 方式一： 没有分组
       /* stream.map(d -> d.user)
                .windowAll(SlidingEventTimeWindows.of(Time.seconds(10L), Time.seconds(5L)))
                .aggregate(new UrlHashMapCountAgg(), new UrlAllWinResult())
                .print();*/



public static class UrlHashMapCountAgg implements AggregateFunction<String, HashMap<String, Long>, ArrayList<Tuple2<String, Long>>> {

        @Override
        public HashMap<String, Long> createAccumulator() {
            return new HashMap<>();
        }

        @Override
        public HashMap<String, Long> add(String s, HashMap<String, Long> acc) {
            if (!acc.containsKey(s)) {
                acc.put(s, 0L);
            }
            acc.put(s, acc.get(s) + 1);
            return acc;
        }

        @Override
        public ArrayList<Tuple2<String, Long>> getResult(HashMap<String, Long> acc) {
            ArrayList<Tuple2<String, Long>> tList = new ArrayList<>();
            for (Map.Entry<String, Long> entry : acc.entrySet()) {
                tList.add(Tuple2.of(entry.getKey(), entry.getValue()));
            }
            tList.sort(new Comparator<Tuple2<String, Long>>() {
                @Override
                public int compare(Tuple2<String, Long> o1, Tuple2<String, Long> o2) {
                    long diff = o2.f1 - o1.f1;
                    if (diff > 0) {
                        return 1;
                    } else if (diff == 0) {
                        return 0;
                    } else {
                        return -1;
                    }
                }
            });
            return tList;
        }

        @Override
        public HashMap<String, Long> merge(HashMap<String, Long> stringLongHashMap, HashMap<String, Long> acc1) {
            return null;
        }
    }


    public static class UrlAllWinResult extends ProcessAllWindowFunction<ArrayList<Tuple2<String, Long>>, String, TimeWindow> {
        @Override
        public void process(ProcessAllWindowFunction<ArrayList<Tuple2<String, Long>>, String, TimeWindow>.Context context, Iterable<ArrayList<Tuple2<String, Long>>> iterable, Collector<String> collector) throws Exception {
            ArrayList<Tuple2<String, Long>> tList = iterable.iterator().next();
            StringBuilder res = new StringBuilder();
            for (int i = 0; i < 2; i++) {
                res.append(tList.get(i));
            }
            collector.collect(String.valueOf(res));
        }
    }

```





## 侧输出流



之前的迟到数据写入到侧边输出流 也是基于该方法 



``` java
package com.matt.apitest.processfunction;

import com.matt.apitest.beans.SensorReading;
import org.apache.flink.api.common.state.ValueState;
import org.apache.flink.api.common.state.ValueStateDescriptor;
import org.apache.flink.api.java.tuple.Tuple;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.KeyedProcessFunction;
import org.apache.flink.streaming.api.functions.ProcessFunction;
import org.apache.flink.util.Collector;
import org.apache.flink.util.OutputTag;

public class Test3_Sideoutputcase {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        DataStream<String> inputStream = env.socketTextStream("localhost", 777);
        // 转换成SensorReading类型
        DataStream<SensorReading> dataStream = inputStream.map(line -> {
            String[] fields = line.split(",");
            return new SensorReading(fields[0], new Long(fields[1]), new Double(fields[2]));
        });
        
        // 匿名
        OutputTag<SensorReading> outputTag = new OutputTag<SensorReading>("lowTemp"){};

        SingleOutputStreamOperator<SensorReading> highTempStream = dataStream.process(new ProcessFunction<SensorReading, SensorReading>() {
            @Override
            public void processElement(SensorReading sensorReading, ProcessFunction<SensorReading, SensorReading>.Context context, Collector<SensorReading> collector) throws Exception {
                if (sensorReading.getTemperatrue() > 30) {
                    collector.collect(sensorReading);
                } else {
                    // 输出到侧边流
                    context.output(outputTag, sensorReading);
                }
            }
        });

        highTempStream.print("high");
        highTempStream.getSideOutput(outputTag).print("lowTem");
        env.execute("my");
    }
}

```

