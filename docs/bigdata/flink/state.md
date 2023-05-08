





数据计算的一个结果可认为一个状态



## 状态



### 算子分类

无状态算子： 不依赖之前的数据，直接计算

有状态算子：依赖之前的数据，比如sum



### 状态管理



权限：同一个分区可能会有多个key,如何保证该状态不被非预期的修改

容错性：故障恢复，需要备份

扩展：并发不足可以扩容





### 状态分类

#### 托管状态和原始状态

托管状态就是 由 Flink 统一管理的，状态的存储访问、故障恢复和重组等一系列问题都由Flink 实现，我们只要调接口就可以

原始状态则是自定义的，相当于就是开辟了一块内存，需要我们自己管理，实现状态的序列化和故障恢复





托管状态分为俩类

算子状态

状态作用范围：当前算子任务实例



按键分区状态

仅对当前key有效， 需要再keyBy之后使用



## 按键分区状态



不同的key对应的keyedState会组成键组， 当并行度发生改变时，键组也会发生改变，键组的大小为定义的最大并行度





``` java
package com.matt.apitest.state;

import com.matt.apitest.beans.Event;
import com.matt.apitest.beans.SensorReading;
import com.matt.apitest.source.SourceTest4_UDF;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.functions.*;
import org.apache.flink.api.common.state.*;
import org.apache.flink.api.common.time.Time;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.checkpoint.ListCheckpointed;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.timestamps.BoundedOutOfOrdernessTimestampExtractor;
import org.apache.flink.util.Collector;
import org.apache.flink.util.OutputTag;

import java.time.Duration;
import java.util.Collections;
import java.util.List;

public class StateTest1 {

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
        stream.keyBy(d -> d.user)
                .flatMap(new MyFlatMap()).print();

        env.execute("my");
    }


    public static class MyFlatMap extends RichFlatMapFunction<Event, String> {
        ValueState<Event> myVState = null;
        ListState<Event> eventListState;
        MapState<Event, Long> eventMapState;
        ReducingState<Event> reducingState;
        AggregatingState<Event, String> aggregatingState;

        @Override
        public void open(Configuration parameters) throws Exception {
            ValueStateDescriptor<Event> vDesc = new ValueStateDescriptor<>("v", Event.class);
            myVState = getRuntimeContext().getState(vDesc);
            eventListState = getRuntimeContext().getListState(new ListStateDescriptor<Event>("list", Event.class));
            eventMapState = getRuntimeContext().getMapState(new MapStateDescriptor<Event, Long>("map", Event.class,
                    Long.class));
            reducingState = getRuntimeContext().getReducingState(new ReducingStateDescriptor<Event>("r1", new ReduceFunction<Event>() {
                @Override
                public Event reduce(Event t1, Event t2) throws Exception {
                    return new Event(t1.user + t2.user, t1.url, t1.timestamp);
                }
            }, Event.class));
            aggregatingState = getRuntimeContext().getAggregatingState(new AggregatingStateDescriptor<Event, Long, String>("agg", new AggregateFunction<Event, Long, String>() {
                @Override
                public Long createAccumulator() {
                    return 0L;
                }

                @Override
                public Long add(Event event, Long acc) {
                    return acc + 1;
                }

                @Override
                public String getResult(Long acc) {
                    return String.valueOf(acc);
                }

                @Override
                public Long merge(Long aLong, Long acc1) {
                    return null;
                }
            }, Long.class));

            // 处理时间
            StateTtlConfig ttlConfig = StateTtlConfig.newBuilder(Time.hours(1L))
                    // 更新类型 什么时候可以更新失效时间 读取的时候可以更新ttl
                    .setUpdateType(StateTtlConfig.UpdateType.OnReadAndWrite)
                    // 状态可见性 默认永远不返回失效数据
                    .setStateVisibility(StateTtlConfig.StateVisibility.ReturnExpiredIfNotCleanedUp)
                    .build();

            vDesc.enableTimeToLive(ttlConfig);
        }

        @Override
        public void flatMap(Event e, Collector<String> collector) throws Exception {
            myVState.update(e);
            eventListState.add(e);
            eventMapState.put(e, (eventMapState.get(e) == null ? 1 : eventMapState.get(e)) + 1);
            reducingState.add(e);
            aggregatingState.add(e);
            System.out.println(eventMapState.values());
            System.out.println(reducingState.get());
            System.out.println(aggregatingState.get());
        }
    }
}

```



### 支持的状态

#### 值状态 valueState



T value():获取当前状态的值;

update(Tvalue):对状态进行更新





#### 列表状态 listState

 ListState<T>



- Iterable<T>get():获取当前的列表状态，返回的是一个可迭代类型Iterable<T>;
- update(List<T>values):传入一个列表values，直接对状态进行覆盖;
- add(Tvalue):在状态列表中添加一个元素value;
- addAll(List<T>values):向列表中添加多个元素，以列表values形式传入。



#### 映射状态 mapState



- UVget(UKkey):传入一个key作为参数，查询对应的value值;
- put(UKkey,UVvalue):传入一个键值对，更新key对应的value值;
- putAll(Map<UK,UV>map):将传入的映射map中所有的键值对，全部添加到映射状态中;
- remove(UKkey):将指定key对应的键值对删除;
- boolean contains(UK key):判断是否存在指定的 key，返回一个 boolean 值
- Iterable<Map.Entry<UK, UV>> entries():获取映射状态中所有的键值对;
- Iterable<UK>keys():获取映射状态中所有的键(key)，返回一个可迭代Iterable类型;
- Iterable<UV> values():获取映射状态中所有的值(value)，返回一个可迭代 Iterable类型;
- booleanisEmpty():判断映射是否为空，返回一个boolean值。



#### 规约状态

需要实现ReduceFunction



#### 聚合状态 



### 状态参数配置



``` java
// 处理时间
StateTtlConfig ttlConfig = StateTtlConfig.newBuilder(Time.hours(1L))
        // 更新类型 什么时候可以更新失效时间 读取的时候可以更新ttl
        .setUpdateType(StateTtlConfig.UpdateType.OnReadAndWrite)
        // 状态可见性 默认永远不返回失效数据
        .setStateVisibility(StateTtlConfig.StateVisibility.ReturnExpiredIfNotCleanedUp)
        .build();

vDesc.enableTimeToLive(ttlConfig);
```



.newBuilder()

返回一个 Builder 之后再调用.build()方法就可以 得到 StateTtlConfig 



.setUpdateType()

更新ttl类型,

OnCreateAndWrite 表示只有创建状态和更改状态(写操作)时更新失效时间, 默认配置

OnReadAndWrite 则表 示无论读写操作都会更新失效时间，也就是只要对状态进行了访问，就表明它是活跃的，从而 延长生存时间





setStateVisibility

NeverReturnExpired 是默认行为，表示从不返回过期值

ReturnExpireDefNotCleanedUp，就是如果过期状态还存在，就返回它的值





## 算子状态



在同一个分区的同一个算子状态是共享的



### 支持的数据类型



#### 列表状态ListState

一组数据



#### 联合列表状态 UnionListState

也是存储一组数据，和列表状态区别在于 当并行度发生改变时，获取状态方式不一致



列表状态是轮询

联合列表状态 是广播的方式，然后选择自己需要使用和丢弃的状态



#### 广播状态

如果希望所有并行子任务保持同一份全局状态 ，可以使用广播状态





### 使用

实现CheckpointedFunction



每次应用保存检查点做快照时，都会调用.snapshotState()方法，将状态进行外部持久化。

算子任务进行初始化时，会调用. initializeState()方法(1.应用第一次启动2.应用重启时，从检查点(checkpoint)或者保存点(savepoint)中读取之前状态的快照，并赋给本地状态)





```java
package com.matt.apitest.state;

import com.matt.apitest.beans.Event;
import com.matt.apitest.source.SourceTest4_UDF;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.state.ListState;
import org.apache.flink.api.common.state.ListStateDescriptor;
import org.apache.flink.runtime.state.FunctionInitializationContext;
import org.apache.flink.runtime.state.FunctionSnapshotContext;
import org.apache.flink.streaming.api.checkpoint.CheckpointedFunction;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.sink.SinkFunction;

import java.time.Duration;
import java.util.ArrayList;
import java.util.List;

/**
 * @author matt
 * @create 2023-03-12 16:19
 * @desc 算子状态
 */
public class BufferingSinkCase {
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

        stream.addSink(new BufferingSinkRes(10));

        env.execute();
    }

    public static class BufferingSinkRes implements SinkFunction<Event>, CheckpointedFunction {

        public int threshold;

        public BufferingSinkRes(int threshold) {
            this.threshold = threshold;
            this.bufferedEvents = new ArrayList<>();
        }

        private List<Event> bufferedEvents;
        private ListState<Event> checkPointedState;

        @Override
        public void snapshotState(FunctionSnapshotContext functionSnapshotContext) throws Exception {
            checkPointedState.clear();
            for (Event e: bufferedEvents) {
                checkPointedState.add(e);
            }
        }

        @Override
        public void initializeState(FunctionInitializationContext ctx) throws Exception {
            ListStateDescriptor<Event> lDesc = new ListStateDescriptor<Event>("list", Event.class);
            checkPointedState = ctx.getOperatorStateStore().getListState(lDesc);

            // 故障恢复
            if (ctx.isRestored()) {
                for (Event e : checkPointedState.get()) {
                    bufferedEvents.add(e);
                }
            }
        }

        @Override
        public void invoke(Event event, Context context) throws Exception {
            bufferedEvents.add(event);

            if (bufferedEvents.size() >= threshold) {
                for (Event e : bufferedEvents) {
                    System.out.println(e);
                }
                System.out.println("----");
                bufferedEvents.clear();
            }
        }
    }
}

```







## 广播流

```java
package com.matt.apitest.state;

import org.apache.flink.api.common.state.*;
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.BroadcastStream;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.co.KeyedBroadcastProcessFunction;
import org.apache.flink.util.Collector;

/**
 * @author matt
 * @create 2023-03-13 23:15
 * @desc 广播流案例
 */
public class BehaviorPatternDetectCase {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        env.enableCheckpointing(1000L);

        DataStreamSource<Action> actionStream = env.fromElements(
                new Action("1", "t1"),
                new Action("2", "t1"),
                new Action("2", "t1"),
                new Action("1", "t2"),
                new Action("2", "t3"),
                new Action("1", "t1"),
                new Action("1", "t2")
        );

        DataStreamSource<Pattern> patternStream = env.fromElements(
                new Pattern("t1", "t2")
        );

        MapStateDescriptor<Void, Pattern> patternDesc = new MapStateDescriptor<>("pattern", Types.VOID, Types.POJO(Pattern.class));

        BroadcastStream<Pattern> broadcastStream = patternStream.broadcast(patternDesc);
        SingleOutputStreamOperator<Tuple2<String, Pattern>> process = actionStream.keyBy(d -> d.id)
                .connect(broadcastStream)
                .process(new PatternDetector());
        process.print();
        env.execute();
    }

    // 用户行为
    public static class Action {
        public String id;
        public String action;

        public Action() {
        }

        public Action(String id, String action) {
            this.id = id;
            this.action = action;
        }

        @Override
        public String toString() {
            return "Action{" +
                    "id='" + id + '\'' +
                    ", action='" + action + '\'' +
                    '}';
        }
    }

    public static class Pattern {
        public String ac1;
        public String ac2;

        public Pattern() {
        }

        public Pattern(String ac1, String ac2) {
            this.ac1 = ac1;
            this.ac2 = ac2;
        }

        @Override
        public String toString() {
            return "Pattern{" +
                    "ac1='" + ac1 + '\'' +
                    ", ac2='" + ac2 + '\'' +
                    '}';
        }
    }


    public static class PatternDetector extends KeyedBroadcastProcessFunction<String, Action, Pattern, Tuple2<String, Pattern>> {

        // 上一个行为
        ValueState<String> preActionState;

        @Override
        public void open(Configuration parameters) throws Exception {
            preActionState = getRuntimeContext().getState(new ValueStateDescriptor<String>("pre", String.class));
        }

        @Override
        public void processElement(Action v, KeyedBroadcastProcessFunction<String, Action, Pattern, Tuple2<String, Pattern>>.ReadOnlyContext ctx, Collector<Tuple2<String, Pattern>> collector) throws Exception {
            ReadOnlyBroadcastState<Void, Pattern> p1 = ctx.getBroadcastState(new MapStateDescriptor<>("pattern", Types.VOID, Types.POJO(Pattern.class)));
            Pattern pattern = p1.get(null);

            String preAct = preActionState.value();
            if (pattern != null && preAct != null) {
                if (pattern.ac1.equals(preAct) && pattern.ac2.equals(v.action)) {
                    collector.collect(new Tuple2<>(ctx.getCurrentKey(), pattern));
                }
            }
            // 更新章台
            preActionState.update(v.action);
        }

        @Override
        public void processBroadcastElement(Pattern pattern, KeyedBroadcastProcessFunction<String, Action, Pattern, Tuple2<String, Pattern>>.Context ctx, Collector<Tuple2<String, Pattern>> collector) throws Exception {
            BroadcastState<Void, Pattern> p1 = ctx.getBroadcastState(new MapStateDescriptor<>("pattern", Types.VOID, Types.POJO(Pattern.class)));
            p1.put(null, pattern);
        }
    }
}

```



## 状态持久化和状态后端



### 状态持久化



检查点默认是禁用的， 定时触发 保存所有任务的状态， 发生故障时从上次保存的状态恢复应用状态

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getEnvironment();
env.enableCheckpointing(1000); // ms
```



保存点：保存点在原理和形式上 跟检查点完全一样，也是状态持久化保存的一个快照;区别在于，保存点是自定义的镜像保存





### 状态后端



![](https://raw.githubusercontent.com/imattdu/img/main/img/202305090133307.png)



#### 概述

一是本地的状态管理，二是将检查点(checkpoint)写入远程的持久化存储。



#### 状态后端分类

一种是“哈希表状态后端”(HashMapStateBackend)，另 一种是“内嵌 RocksDB 状态后端”(EmbeddedRocksDBStateBackend)



##### HashMapStateBackend

内存保存



##### EmbeddedRocksDBStateBackend

会将处理中的数据全部放入 RocksDB 数据库中，RocksDB 默认存储在 TaskManager 的本地数据目录里。







#### 配置



方式一

flink-conf.yaml  配置 

hashmap -> HashMapStateBackend;

EmbeddedRocksDBStateBackend -> rocksdb

也可以自己实现 StateBackendFactory，配置全类名即可

``` yaml
# 默认状态后端
state.backend: hashmap
# 存放检查点的文件路径
state.checkpoints.dir: hdfs://namenode:40010/flink/checkpoints
```



方式二



``` java
StreamExecutionEnvironment                        env                        = StreamExecutionEnvironment.getExecutionEnvironment();
env.setStateBackend(new HashMapStateBackend());
```





```java
StreamExecutionEnvironment                        env                        = StreamExecutionEnvironment.getExecutionEnvironment();
env.setStateBackend(new EmbeddedRocksDBStateBackend());
```



如果想在 IDE 中使用 EmbeddedRocksDBStateBackend，需要为 Flink 项目添加 依赖(flink集群已经有该lib,所以集群运行就不需要该依赖)

```xml
<dependency>
   <groupId>org.apache.flink</groupId>
<artifactId>flink-statebackend-rocksdb_${scala.binary.version}</artifactId>
   <version>1.13.0</version>
</dependency>
```

