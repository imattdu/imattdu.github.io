## 处理函数



### 概述

处理函数定义数据流的转换操作



之前的算子存在的问题：

- map算子，实现mapFunction 只能获取转换后的数据
- 窗口聚合算子，aggregateFunction中出数据外，还可以获取状态
- 富函数，RichMapFunction调用getRuntimeContext() 可以获得状态，运行信息



之前所有的算子是无法获取事件时间水位线以及使用定时器

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















