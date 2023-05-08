





之前所有的操作均是在一条流， 真实业务中会存在多条流的情况





## 分流



### 普通方法

fliter过滤 

split 分流 需要保证输入和输出类型一致 目前已经淘汰





### 通用方法 

使用侧输出流

``` java
package com.matt.apitest.mulstream;

import com.matt.apitest.beans.Event;
import com.matt.apitest.source.SourceTest4_UDF;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.ProcessFunction;
import org.apache.flink.util.Collector;
import org.apache.flink.util.OutputTag;
import org.mortbay.util.ajax.JSON;

import java.time.Duration;

/**
 * @author matt
 * @create 2023-03-02 22:54
 * @desc xxx
 */
public class SplitStreamTest {

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

        // 定义输出标签
        OutputTag<Tuple2<String, String>> t1 = new OutputTag<Tuple2<String, String>>("t1") {
        };
        OutputTag<Tuple2<String, String>> t2 = new OutputTag<Tuple2<String, String>>("t2") {
        };

        SingleOutputStreamOperator<Event> s = stream.process(new ProcessFunction<Event, Event>() {
            @Override
            public void processElement(Event event, ProcessFunction<Event, Event>.Context context, Collector<Event> collector) throws Exception {
                if (event.user.equals("matt")) {
                    context.output(t1, Tuple2.of(event.user, event.url));
                } else if (event.user.equals("jack")) {
                    context.output(t2, Tuple2.of(event.user, event.url));
                } else {
                    collector.collect(event);
                }
            }
        });

        s.getSideOutput(t1).print("t1");
        s.getSideOutput(t2).print("t2");

        s.print("main");
        env.execute();
    }
}

```



## 合流



多条流的合并



### 合并 union



``` java
package com.matt.apitest.mulstream;

import com.matt.apitest.beans.Event;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.ProcessFunction;
import org.apache.flink.util.Collector;

import java.time.Duration;

/**
 * @author matt
 * @create 2023-03-02 23:09
 * @desc union 取2个流水位线最小值
 */
public class UnionTest {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        SingleOutputStreamOperator<Event> s1 = env.socketTextStream("localhost", 777)
                .map(d -> {
                    String[] fArr = d.split(" ");
                    if (fArr.length < 3) {
                        return new Event();
                    }
                    return new Event(fArr[0], fArr[1], Long.valueOf(fArr[2]));
                })
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ofSeconds(2))
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event event, long l) {
                                return event.timestamp;
                            }
                        }));


        SingleOutputStreamOperator<Event> s2 = env.socketTextStream("localhost", 888)
                .map(d -> {
                    String[] fArr = d.split(" ");
                    if (fArr.length < 3) {
                        return new Event();
                    }
                    return new Event(fArr[0], fArr[1], Long.valueOf(fArr[2]));
                })
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ofSeconds(10))
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event event, long l) {
                                return event.timestamp;
                            }
                        }));



        s1.union(s2).process(new ProcessFunction<Event, String>() {
            @Override
            public void processElement(Event event, ProcessFunction<Event, String>.Context context, Collector<String> collector) throws Exception {
                collector.collect("wk" + context.timerService().currentWatermark());
            }
        }).print();

        env.execute();
    }
}

```

取了并集 多条流数据类型需要保持一致， union方法 支持>=1个流

水位线以多条流最小的为准



### 连接 connect 



#### connetcedStream

2条流可以是不同数据类型 

connetc = connetcedStream

``` java
package com.matt.apitest.mulstream;

import org.apache.flink.streaming.api.datastream.ConnectedStreams;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.co.CoMapFunction;

/**
 * @author matt
 * @create 2023-03-02 23:40
 * @desc xxx
 */
public class ConnectTest {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        DataStreamSource<Integer> s1 = env.fromElements(10, 20, 30);
        DataStreamSource<Long> s2 = env.fromElements(11L, 22L, 31L);

        ConnectedStreams<Integer, Long> connect = s1.connect(s2);
        SingleOutputStreamOperator<String> map = connect.map(new CoMapFunction<Integer, Long, String>() {
            @Override
            public String map1(Integer integer) throws Exception {
                return String.valueOf(integer) + "s1";
            }

            @Override
            public String map2(Long v) throws Exception {
                return String.valueOf(v) + "s2";
            }
        });

        map.print();
        env.execute();
    }
}

```



#### CoProcessFunction



connect -> process (CoProcessFunction)



processElement1:处理第一条流

processElement2:处理第一条流

``` java
package com.matt.apitest.mulstream;

import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.state.ValueState;
import org.apache.flink.api.common.state.ValueStateDescriptor;
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.tuple.Tuple3;
import org.apache.flink.api.java.tuple.Tuple4;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.co.CoProcessFunction;
import org.apache.flink.util.Collector;

import java.time.Duration;

/**
 * @author matt
 * @create 2023-03-03 00:01
 * @desc xxx
 */
public class BillCheckCase {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        
        SingleOutputStreamOperator<Tuple3<String, String, Long>> s1 = env.fromElements(
                        Tuple3.of("order-1", "app", 1000L),
                        Tuple3.of("order-2", "app", 1000L),
                        Tuple3.of("order-3", "app", 2000L))
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Tuple3<String, String, Long>>forBoundedOutOfOrderness(Duration.ZERO)
                        .withTimestampAssigner(new SerializableTimestampAssigner<Tuple3<String, String, Long>>() {
                            @Override
                            public long extractTimestamp(Tuple3<String, String, Long> t, long l) {
                                return t.f2;
                            }
                        }));

        SingleOutputStreamOperator<Tuple4<String, String, String, Long>> s2 = env.fromElements(
                        Tuple4.of("order-1", "3", "suc", 2000L),
                        Tuple4.of("order-3", "3", "suc", 4000L))
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Tuple4<String, String, String, Long>>forBoundedOutOfOrderness(Duration.ZERO)
                        .withTimestampAssigner(new SerializableTimestampAssigner<Tuple4<String, String, String, Long>>() {
                            @Override
                            public long extractTimestamp(Tuple4<String, String, String, Long> t, long l) {
                                return t.f3;
                            }
                        }));

        // 检测在同一个订单 俩条流是否匹配 5s 等待时间
        s1.keyBy(d -> d.f0)
                .connect(s2.keyBy(d -> d.f0))
                .process(new OrderMatchResult())
                .print();


        //s1.connect(s2).keyBy(d -> d.f0, d -> d.f0);
        env.execute();
    }

    public static class OrderMatchResult extends CoProcessFunction<Tuple3<String, String, Long>,
            Tuple4<String, String, String, Long>, String> {

        // 状态变量
        private ValueState<Tuple3<String, String, Long>> state1;
        private ValueState<Tuple4<String, String, String, Long>> state2;

        @Override
        public void open(Configuration conf) throws Exception {
            state1 = getRuntimeContext().getState(new ValueStateDescriptor<Tuple3<String, String, Long>>(
                    "state1", Types.TUPLE(Types.STRING, Types.STRING, Types.LONG)));
            state2 = getRuntimeContext().getState(new ValueStateDescriptor<Tuple4<String, String, String, Long>>(
                    "state2", Types.TUPLE(Types.STRING, Types.STRING, Types.STRING, Types.LONG)));
        }

        @Override
        public void processElement1(Tuple3<String, String, Long> v, CoProcessFunction<Tuple3<String, String, Long>, Tuple4<String, String, String, Long>, String>.Context ctx, Collector<String> collector) throws Exception {
            if (state2.value() != null) {
                collector.collect("s1对账成功" + v + " " + state2.value());
                state2.clear();
            } else {
                state1.update(v);
                ctx.timerService().registerEventTimeTimer(v.f2 + 5000L);
            }
        }

        @Override
        public void processElement2(Tuple4<String, String, String, Long> v, CoProcessFunction<Tuple3<String, String, Long>, Tuple4<String,
                String, String, Long>, String>.Context ctx, Collector<String> collector) throws Exception {
            if (state1.value() != null) {
                collector.collect("s2对账成功" + state1.value() + " " + v);
                state1.clear();
            } else {
                state2.update(v);
                ctx.timerService().registerEventTimeTimer(v.f3);
            }
        }


        @Override
        public void onTimer(long timestamp, CoProcessFunction<Tuple3<String, String, Long>, Tuple4<String, String, String, Long>, String>.OnTimerContext ctx, Collector<String> out) throws Exception {
            // 如果某个状态为空 存在一条流没到
            if (state1.value() != null) {
                out.collect(state1.value() + "对账失败");
            } else if (state2.value() != null) {
                out.collect(state2.value() + "对账失败");
            }
            state1.clear();
            state2.clear();
        }
    }
}

```





#### 广播连接流 BroadcastConnectedStream



定义一个广播流， 需先定义一个描述器

```java
MapStateDescriptor<String, Rule> ruleStateDescriptor = new MapStateDescriptor<>(...);
BroadcastStream<Rule> ruleBroadcastStream = ruleStream
                    .broadcast(ruleStateDescriptor);
```



连接

```java
DataStream<String> output = stream.connect(ruleBroadcastStream)
              .process( new BroadcastProcessFunction<>() {...} );
```



``` java
public abstract void processElement(IN1 value, ReadOnlyContext ctx, Collector<OUT> out) throws Exception;
public abstract void processBroadcastElement(IN2 value, Context ctx, Collector<OUT> out) throws Exception;
```





dataStream 和广播流连接就是 BroadcastConnectedStream

对数据流调用过 keyBy 进行了按键分区，那么就是 KeyedBroadcastProcessFunction;



## 联结 join



上述合流操作均是取了并集，有一些情况可能需要a&b 合并为1条数据



### 窗口连接

``` java
package com.matt.apitest.mulstream;

import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.functions.JoinFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.api.java.tuple.Tuple3;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;

import java.time.Duration;

/**
 * @author matt
 * @create 2023-03-05 14:56
 * @desc xxx
 */
public class WinJoinTest {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        SingleOutputStreamOperator<Tuple2<String, Long>> s1 = env.fromElements(
                Tuple2.of("t1", 1000L),
                Tuple2.of("t3", 9999L),
                Tuple2.of("t2", 8000L),
                Tuple2.of("t4", 11000L)
        ).assignTimestampsAndWatermarks(WatermarkStrategy.<Tuple2<String, Long>>forBoundedOutOfOrderness(Duration.ZERO)
                .withTimestampAssigner(new SerializableTimestampAssigner<Tuple2<String, Long>>() {
                    @Override
                    public long extractTimestamp(Tuple2<String, Long> t, long l) {
                        return t.f1;
                    }
                }));
        
        SingleOutputStreamOperator<Tuple2<String, Long>> s2 = env.fromElements(
                Tuple2.of("t1", 2002L),
                Tuple2.of("t3", 5000L),
                Tuple2.of("t2", 9000L),
                Tuple2.of("t4", 22000L)
        ).assignTimestampsAndWatermarks(WatermarkStrategy.<Tuple2<String, Long>>forBoundedOutOfOrderness(Duration.ZERO)
                .withTimestampAssigner(new SerializableTimestampAssigner<Tuple2<String, Long>>() {
                    @Override
                    public long extractTimestamp(Tuple2<String, Long> t, long l) {
                        return t.f1;
                    }
                }));

        s1.join(s2)
                .where(d -> d.f0)
                .equalTo(d -> d.f0)
                .window(TumblingEventTimeWindows.of(Time.seconds(5L)))
                .apply(new JoinFunction<Tuple2<String, Long>, Tuple2<String, Long>, String>() {
                    @Override
                    public String join(Tuple2<String, Long> f1, Tuple2<String, Long> f2) throws Exception {

                        return f1 + "->" + f2;
                    }
                }).print();

        env.execute();
    }
}

```



join joinedStream -> 

where指定第一条流的k, equalTo指定第一条流的k 俩条流k相等才会进行匹配 ->

window 指定窗口类型 ->

apply 合并数据  **第一条流和第二条流匹配的数据会做笛卡尔积**





### 间隔联结



a流a1 b流b2 a1,b2虽然不在一个窗口内但仍希望进行联结



``` java
package com.matt.apitest.mulstream;

import com.matt.apitest.beans.Event;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.co.ProcessJoinFunction;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;

import java.time.Duration;

/**
 * @author matt
 * @create 2023-03-05 15:26
 * @desc xxx
 */
public class IntervalJoinTest {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        SingleOutputStreamOperator<Event> s1 = env.fromElements(
                new Event("a1", "/u1", 100L),
                new Event("a2", "/u1", 200L),
                new Event("a3", "/u2", 300L),
                new Event("a4", "/u3", 400L)
        ).assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ZERO)
                .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                    @Override
                    public long extractTimestamp(Event t, long l) {
                        return t.timestamp;
                    }
                }));


        SingleOutputStreamOperator<Event> s2 = env.fromElements(
                new Event("a1", "/bill", 100L),
                new Event("a2", "/b2", 200L),
                new Event("a3", "/b3", 300L),
                new Event("a4", "/b4", 400L)
        ).assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ZERO)
                .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                    @Override
                    public long extractTimestamp(Event t, long l) {
                        return t.timestamp;
                    }
                }));

        s1.keyBy(d -> d.user)
                .intervalJoin(s2.keyBy(d -> d.user))
                .between(Time.seconds(-5), Time.seconds(5))
                .process(new ProcessJoinFunction<Event, Event, String>() {
                    @Override
                    public void processElement(Event e1, Event e2, ProcessJoinFunction<Event, Event, String>.Context context, Collector<String> collector) throws Exception {
                        collector.collect(e1 + "->" + e2);
                    }
                }).print();

        env.execute();
    }
}

```

between 指定窗口 上下界

winStart + low <= t < winEnd + up 只要2条流满足这个条件均可以连接





### 窗口同组联结

apply：CoGroupFunction

一次传入匹配的数据集 由用户自己去定义匹配 不在是暴力笛卡尔积

``` java
package com.matt.apitest.mulstream;

import com.matt.apitest.beans.Event;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.functions.CoGroupFunction;
import org.apache.flink.api.common.functions.JoinFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.co.ProcessJoinFunction;
import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;

import java.time.Duration;

/**
 * @author matt
 * @create 2023-03-05 15:50
 * @desc xxx
 */
public class CoGroupTest {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        SingleOutputStreamOperator<Tuple2<String, Long>> s1 = env.fromElements(
                Tuple2.of("t1", 1000L),
                Tuple2.of("t3", 9999L),
                Tuple2.of("t2", 8000L),
                Tuple2.of("t4", 11000L)
        ).assignTimestampsAndWatermarks(WatermarkStrategy.<Tuple2<String, Long>>forBoundedOutOfOrderness(Duration.ZERO)
                .withTimestampAssigner(new SerializableTimestampAssigner<Tuple2<String, Long>>() {
                    @Override
                    public long extractTimestamp(Tuple2<String, Long> t, long l) {
                        return t.f1;
                    }
                }));


        SingleOutputStreamOperator<Tuple2<String, Long>> s2 = env.fromElements(
                Tuple2.of("t1", 2002L),
                Tuple2.of("t1", 2003L),
                Tuple2.of("t3", 5000L),
                Tuple2.of("t2", 9000L),
                Tuple2.of("t4", 22000L)
        ).assignTimestampsAndWatermarks(WatermarkStrategy.<Tuple2<String, Long>>forBoundedOutOfOrderness(Duration.ZERO)
                .withTimestampAssigner(new SerializableTimestampAssigner<Tuple2<String, Long>>() {
                    @Override
                    public long extractTimestamp(Tuple2<String, Long> t, long l) {
                        return t.f1;
                    }
                }));

        s1.coGroup(s2)
                .where(d -> d.f0)
                .equalTo(d -> d.f0)
                .window(TumblingEventTimeWindows.of(Time.seconds(5L)))
                .apply(new CoGroupFunction<Tuple2<String, Long>, Tuple2<String, Long>, String>() {
                    @Override
                    public void coGroup(Iterable<Tuple2<String, Long>> s1Iterable, Iterable<Tuple2<String, Long>> s2Iterable, Collector<String> collector) throws Exception {
                        collector.collect(s1Iterable + "->" + s2Iterable);
                    }
                }).print();

        env.execute();
    }
}

```

