## 概述



### 流程

- 获取执行环境get environment
- 读取数据源 source
- 数据的转换 tansfrom
- 数据的输出 sink
- 程序执行 execute





## 执行环境





### 创建执行环境

1.只能判断 如果是本地则使用本地 集群则使用集群环境

```java
StreamExecutionEnvironment env =StreamExecutionEnvironment.getExecutionEnvironment();
```

2.指定  

**createLocalEnvironment** 指定本地环境， 默认并行度cpu核数

```java
StreamExecutionEnvironment localEnv = StreamExecutionEnvironment.createLocalEnvironment();
```

**createRemoteEnvironment** 使用集群执行环境

```java
StreamExecutionEnvironment remoteEnv = StreamExecutionEnvironment.createRemoteEnvironment("host", // JobManager 主机名
1234, // JobManager 进程端口号 "path/to/jarFile.jar" // 提交给 JobManager 的 JAR 包
);
```



### 执行模式 execution mode

``` java
// 批处理环境
ExecutionEnvironment batchEnv = ExecutionEnvironment.getExecutionEnvironment(); 
// 流处理环境
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
```

1.12 批流统一, 自定义设置即可批流切换



#### 执行模式分类

流执行模式(STREAMING)，默认使用流模式



批执行模式(BATCH)， 不会持续计算的有界数据



自动模式， automatic , 程序根据输入数据的数据源是否有界， 来自动选择执行模式





#### 模式配置

Batch模式配置

```
bin/flink run -Dexecution.runtime-mode=BATCH ...
```



```
StreamExecutionEnvironment  env =StreamExecutionEnvironment.getExecutionEnvironment();
env.setRuntimeMode(RuntimeExecutionMode.BATCH);
```



### 触发执行

```java
env.execute();
```



## 源算子





![](https://raw.githubusercontent.com/imattdu/img/main/img/202303080047408.png)





使用nc 获取数据源 并行度设置为1

```
DataStream<String> stream = env.socketTextStream("localhost", 7777);
```







### 前提

有一个如下的实体类

``` java
package com.matt.apitest.model;

import java.sql.Timestamp;

public class UrlCntBO {

    public String url;
    public Long cnt;
    public Long winStart;
    public Long winEnd;

    public UrlCntBO(String url, Long cnt, Long winStart, Long winEnd) {
        this.url = url;
        this.cnt = cnt;
        this.winStart = winStart;
        this.winEnd = winEnd;
    }

    public UrlCntBO() {
    }

    @Override
    public String toString() {
        return "UrlCntBO{" +
                "url='" + url + '\'' +
                ", cnt=" + cnt +
                ", winStart=" + new Timestamp(winStart) +
                ", winEnd=" + new Timestamp(winEnd) +
                '}';
    }
}

```



- 类是公有的 public
- 有一个无参的构造方法
- 所有属性是公有的 或者提供get,set 方法
- 所有属性可序列化
- 提供一个**toString** 方法 可选



### 集合



``` java
package com.matt.apitest.source;

import com.matt.apitest.beans.Event;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import java.util.Arrays;

/**
 * @author matt
 * @create 2022-01-16 14:36
 */
public class SourceTest1_Collection {
    
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(5);
        DataStream<Event> events = env.fromCollection(Arrays.asList(
                new Event("m1", "a1", 1672472280000L),
                new Event("m2", "a2", 1672472279514L)
        ));

        events.print("collection");
        env.execute("my");
    }
}

```



也可以使用如下方法

``` java
DataStreamSource<Event> stream2 = env.fromElements(
    new Event("Mary", "./home", 1000L),
    new Event("Bob", "./cart", 2000L)
);
```



### 文件



``` java
package com.matt.apitest.source;

import com.matt.apitest.beans.SensorReading;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import java.util.Arrays;

/**
 * @author matt
 * @create 2022-01-16 14:54
 */
public class SourceTest2_File {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(10);
        // /Users/matt/workspace/java/stu/stu-flink/src/main/resources
        DataStream<String> dataStream = env.readTextFile("/Users/matt/workspace/java/stu/stu-flink/src/main/resources/sensor.txt");
        dataStream.print("file");

        // job name
        env.execute("f_source");
    }
}

```



如果是hdfs 可添加如下依赖

``` xml
<dependency>
   <groupId>org.apache.hadoop</groupId>
   <artifactId>hadoop-client</artifactId>
   <version>2.7.5</version>
   <scope>provided</scope>
</dependency>
```

### socket



需要将并行度设置为1

``` java
DataStream<String> stream = env.socketTextStream("localhost", 7777);
```



### kafka



添加依赖 kafka版本>=0.10.0

``` xml
<dependency>
   <groupId>org.apache.flink</groupId>
   <artifactId>flink-connector-kafka_${scala.binary.version}</artifactId>
   <version>${flink.version}</version>
</dependency>
```



``` java
package com.matt.apitest.source;

import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;

import java.util.Properties;

/**
 * @author matt
 * @create 2022-01-16 15:05
 */
public class SourceTest3_Kafka {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        Properties properties = new Properties();
        properties.setProperty("bootstrap.servers", "localhost:9092");
        properties.setProperty("group.id", "consumer-group");
        properties.setProperty("key.deserializer",
                "org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("value.deserializer",
                "org.apache.kafka.common.serialization.StringDeserializer");
        
      	// 
      	properties.setProperty("auto.offset.reset", "latest");

        DataStream<String> dataStream = env.addSource( new
                FlinkKafkaConsumer<String>("first", new SimpleStringSchema(), properties));


        dataStream.print("kafka");
        env.execute("kafka_job");
    }
}

```

auto.offset.reset=latest ： 从上一个消费者提交的offset 开始消费

auto.offset.reset=earliest : 从头开始消费



### 自定义数据源

``` java
package com.matt.apitest.source;

import com.matt.apitest.beans.Event;
import org.apache.flink.api.common.time.Time;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.ParallelSourceFunction;
import org.apache.flink.streaming.api.functions.source.SourceFunction;

import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Random;

/**
 * 自定义数据源
 *
 * @author matt
 * @create 2022-01-16 15:50
 */
public class SourceTest4_UDF {
    
    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //env.setParallelism(1);

        DataStream<Event> dataStream = env.addSource(new ParallelCustomSource()).setParallelism(2);

        dataStream.print("kafka");

        // job name
        env.execute("cu");


    }

    public static class ParallelCustomSource implements ParallelSourceFunction<Event> {
        // 标志位 控制数据的产生
        private boolean running = true;

        /**
         * 功能：
         *
         * @param ctx 1
         * @author matt
         * @date 2022/1/16
         */
        @Override
        public void run(SourceContext<Event> ctx) throws InterruptedException {
            // 随机数发生器
            Random random = new Random();
            List<String> userList = Arrays.asList("matt", "jack", "lisi", "lb", "df");
            List<String> urlList = Arrays.asList("/save", "/remove", "/update", "/list", "/detail");

            while (running) {
                for (int i = 0; i < 5; i++) {
                    int randV = random.nextInt(5);
                    ctx.collect(new Event(userList.get(randV), urlList.get(randV), System.currentTimeMillis()));
                    Thread.sleep(1000L);
                }
            }
        }

        @Override
        public void cancel() {
            running = false;
        }
    }
}

```









### 支持的数据类型



#### 1.基本数据类型

Java 基本类型及其包装类，再加上 Void、String、Date、BigDecimal 和 BigInteger。

#### 2.数组类型

包括基本类型数组(PRIMITIVE_ARRAY)和对象数组(OBJECT_ARRAY)

#### 3.复合数据类型

- Java 元组类型(TUPLE):这是 Flink 内置的元组类型，是 Java API 的一部分。最多25 个字段，也就是从 Tuple0~Tuple25，不支持空字段

- Scala 样例类及 Scala 元组:不支持空字段
- 行类型(ROW):可以认为是具有任意个字段的元组,并支持空字段

- POJO:Flink 自定义的类似于 Java bean 模式的类


#### 4.辅助类型

Option、Either、List、Map 等



#### 5.泛型类型

不过如果没有按照上面 POJO 类型的要求来定义，

就会被 Flink 当作泛型类来处理。Flink 会把泛型类型当作黑盒，无法获取它们内部的属性;它 们也不是由 Flink 本身序列化的，而是由 Kryo 序列化的





pojo要求

1. 类是公共的(public)和独立的(standalone，也就是说没有非静态的内部类);
2. 类有一个公共的无参构造方法;
3. 类中的所有字段是public且非final的;或者有一个公共的getter和setter方法，这些方法需要符合 Java bean 的命名规范。





类型提示 防止泛型查除

```
.map(word -> Tuple2.of(word, 1L))
.returns(Types.TUPLE(Types.STRING, Types.LONG));


returns(new TypeHint<Tuple2<Integer, SomeType>>(){})
```







## 转换算子 transform





### 基本转换算子



``` java
package com.matt.apitest.transform;

import org.apache.flink.api.common.functions.FilterFunction;
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

/**
 * @author matt
 * @create 2022-01-16 17:16
 * map flatmap filter
 */
public class Base {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(16);
        DataStream<String> source = env.readTextFile("/Users/matt/workspace/java/stu/stu-flink/src/main/resources/sensor.txt");

        // 1 map string -> len(string) 1:1
        // 方-》园
        DataStream<Integer> mapStream = source.map(new MapFunction<String, Integer>() {
            @Override
            public Integer map(String s) throws Exception {
                return s.length();
            }
        });


        DataStream<Integer> lMapStream = source.map(data -> data.length());

        // flatMap 按逗号切分字端 1:n
        DataStream<String> flatMapStream = source.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public void flatMap(String s, Collector<String> collector) throws Exception {
                String[] fields = s.split(",");
                for (String field : fields) {
                    collector.collect(field);
                }
            }
        });

        // 3.filter 过滤 筛选某个数据 1:[0,1]
        DataStream<String> filterStream = source.filter(new FilterFunction<String>() {
            @Override
            public boolean filter(String s) throws Exception {
                // true 要 false 不要这个数据
                return s.startsWith("sensor_1");
            }
        });

        mapStream.print("map");
        flatMapStream.print("flatMap");
        filterStream.print("filter");

        // job name
        env.execute("trans-form");
    }


}

```



### 聚合算子 aggregation



#### 按键分区

``` java
package com.matt.apitest.transform;

import com.matt.apitest.beans.Event;
import org.apache.flink.api.java.functions.KeySelector;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

/**
 * @author matt
 * @create 2022-01-17 22:14
 */
public class Aggr {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(16);

        DataStream<Event> stream = env.fromElements(new Event("a1", "/1", 1L),
                new Event("a1", "/2", 2L),
                new Event("a2", "/1", 3L),
                new Event("a2", "/2", 2L));


        KeyedStream<Event, String> keyedSteam = stream.keyBy(new KeySelector<Event, String>() {
            @Override
            public String getKey(Event event) throws Exception {
                return event.user;
            }
        });

        keyedSteam.max("timestamp").print("1");

        // max 只更新统计的字段，其他字段都是第一个数据的值  maxBy 所有字段均更新
        stream.keyBy(e -> e.user).maxBy("timestamp")
                        .print("maxBy");
        env.execute();
    }

}

```



stream.keyBy 得到的是一个状态 不是一个算子 





#### 简单聚合



sum

min

max

minBy: min()只计算指定字段的最小值，其他字段会保留最初第一个数据的值;而 minBy()则会返回包含字段最小值的整条数据

maxBy: 和minBy类似



#### 规约聚合



更加通用化 对已有数据不断做规约



```java
package com.matt.apitest.transform;

import com.matt.apitest.beans.Event;
import com.matt.apitest.beans.SensorReading;
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.java.tuple.Tuple;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

/**
 * @author matt
 * @create 2022-01-17 22:48
 * 高级聚合
 */
public class Reduce {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(16);

        DataStream<Event> stream = env.fromElements(new Event("a1", "/1", 1L),
                new Event("a1", "/2", 2L),
                new Event("a2", "/3", 3L),
                new Event("a2", "/2", 3L),
                new Event("a3", "/1", 3L),
                new Event("a2", "/3", 3L),
                new Event("a3", "/1", 3L),
                new Event("a2", "/2", 2L));

        // 2. 用户点击次数
        DataStream<Tuple2<String, Long>> userClickCntStream = stream.map(new MapFunction<Event, Tuple2<String, Long>>() {
            @Override
            public Tuple2<String, Long> map(Event event) throws Exception {
                return Tuple2.of(event.user, 1L);
            }
        }).keyBy(d -> d.f0).reduce(new ReduceFunction<Tuple2<String, Long>>() {
            @Override
            public Tuple2<String, Long> reduce(Tuple2<String, Long> a, Tuple2<String, Long> b) throws Exception {

                return Tuple2.of(a.f0, a.f1 + b.f1);
            }
        });

        // 3.当前最活跃的用户
        userClickCntStream.keyBy(d -> "xx").reduce((x, y) ->
                x.f1 >= y.f1 ? x : y
        ).print("maxActiveUser");

        // job name
        env.execute("reduce");
    }
}


```



但是对于像 flatMap() 这样的函数，它的函数签名 void flatMap(IN value, Collector<OUT> out) 被 Java 编译器编译成了 void flatMap(IN value, Collector out)，也就是说将 Collector 的泛 型信息擦除掉了。这样 Flink 就无法自动推断输出的类型信息了。





### 用户自定义函数



上述基本转换算子&聚合函数 都有对应的抽象类 可以写 匿名匿名内部类 or lambda表达式



#### 富函数

介绍

map,filter,reduce 都有对应的富函数版本 RichMapFunction、RichFilterFunction、 RichReduceFunction

富函数类可以获取运行环境的上下文，并拥有一些生命周期方法，所以可以实现 更复杂的功能。



方法：

open()方法，是RichFunction的初始化方法。所以像文件 IO 的创建，数据库连接的创建，配置文件的读取等等这样一次性的工作，都适合在 open()方法中完成。



close()方法，是生命周期中的最后一个调用的方法，一般用来做一 些清理工作





code 

``` java
package com.matt.apitest.transform;

import com.matt.apitest.beans.SensorReading;
import org.apache.flink.api.common.functions.RichMapFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

/**
 * @author matt
 * @create 2022-01-24 23:55
 */
public class RichFunction {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(4);
        DataStream<String> inputStream = env.readTextFile("/Users/matt/workspace/java/stu/stu-flink/src/main/resources/sensor.txt");

        DataStream<SensorReading> dataStream = inputStream.map(s -> {
            String[] fields = s.split(",");
            return new SensorReading(fields[0], new Long(fields[1]), new Double(fields[2]));
        });



        DataStream<Tuple2<String, Integer>> resStream = dataStream.map(
                new MyMapper()
        );
        resStream.print();

        // job name
        env.execute("trans-form");
    }
    
    public static class MyMapper extends RichMapFunction<SensorReading, Tuple2<String, Integer>> {

        @Override
        public Tuple2<String, Integer> map(SensorReading v) throws Exception {
            return new Tuple2<>(v.getId(), getRuntimeContext().getIndexOfThisSubtask());
        }

        public MyMapper() {
            super();
        }

        @Override
        public void open(Configuration parameters) throws Exception {
            System.out.println("init....");
        }

        @Override
        public void close() throws Exception {
            System.out.println("clear...");
        }
    }

}

```





### 物理分区



分区种类



1.随机分区 shuffle

把流中的数据随机打乱 均匀传递到下游分区



2.轮询分区 round-robin

比如来了 a,b,c,d,e,f 6条数据 有分区 p1, p2, p3

则 a,d 进到p1 b,e 进到p2 c,f进到p3



3.重缩放分区 rescale

比如上游有2分区pp1,pp2 下游有6个分区cp1, cp2 ...cp6

pp1 会发到cp1-cp3 然后再cp1-cp3 做轮询分区

pp2 -> cp4-cp6



4.广播 broadcast

将数据广播到下游所有分区中



5.全局分区 global

将上游多个分区 传递到下游1个分区， 把并行度设置为1



6.自定义分区

``` java
package com.matt.apitest.transform;

import com.matt.apitest.beans.Event;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.functions.Partitioner;
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.java.functions.KeySelector;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.ParallelSourceFunction;
import org.apache.flink.streaming.api.functions.source.RichParallelSourceFunction;

public class Partition {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(16);

        DataStream<Event> stream = env.fromElements(new Event("a1", "/1", 1L),
                new Event("a1", "/2", 2L),
                new Event("a2", "/3", 3L),
                new Event("a2", "/2", 3L),
                new Event("a3", "/1", 3L),
                new Event("a2", "/3", 3L),
                new Event("a3", "/1", 3L),
                new Event("a2", "/2", 2L));

        // 2.1 keyBy 逻辑分区 shuffle 随机分区
        //stream.shuffle().print().setParallelism(4);

        // 2.2轮询分区 2 3 1 4 # 2 3 1 4
        //stream.rebalance().print("rebalance").setParallelism(4);

        // 2.3 rescale 重缩放分区 分组，组内轮询 上游分区2 2 个分区内内部轮询
        DataStream<Integer> rescaleStream = env.addSource(new RichParallelSourceFunction<Integer>() {
            @Override
            public void run(SourceContext<Integer> ctx) throws Exception {
                for (int i = 1; i <= 8; i++) {
                    if (i % 2 == getRuntimeContext().getIndexOfThisSubtask()) {
                        ctx.collect(i);
                    }
                }
            }

            @Override
            public void cancel() {
            }
        }).setParallelism(2);
        // 上游分区a 发到下游a1 a2 a1,a2 轮询 b -> b1, b2 b1,b2所有分区轮询
        // 3,4 均是奇书 1,2 均是偶数
        //rescaleStream.rescale().print("rescale").setParallelism(4);

        // 2.4 广播 一份数据向每个分区都发送
        //stream.broadcast().print("broadcast").setParallelism(4);

        //2.5 全局分区 等价并行度为1
        //stream.global().print("global").setParallelism(4);

        // 2.6 自定义重分区
        env.fromElements(1, 2, 3, 4, 5).partitionCustom(new Partitioner<Integer>() {
            @Override
            public int partition(Integer k, int cnt) {
                return k % 2;
            }
        }, new KeySelector<Integer, Integer>() {
            @Override
            public Integer getKey(Integer v) throws Exception {
                return v;
            }
        }).print("custom").setParallelism(4);

        // job name
        env.execute("partition");
    }

}

```







## 输出算子





### 文件





行编码:StreamingFileSink.forRowFormat(basePath，rowEncoder)。 路径&数据编码格式

批量编码:StreamingFileSink.forBulkFormat(basePath，bulkWriterFactory)。





``` java
package com.matt.apitest.sink;

import com.matt.apitest.beans.Event;
import org.apache.flink.api.common.serialization.SimpleStringEncoder;
import org.apache.flink.core.fs.Path;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.sink.filesystem.StreamingFileSink;
import org.apache.flink.streaming.api.functions.sink.filesystem.rollingpolicies.DefaultRollingPolicy;

import java.util.concurrent.TimeUnit;


public class Sink2File {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        DataStream<Event> stream = env.fromElements(new Event("a1", "/1", 1L),
                new Event("a1", "/2", 2L),
                new Event("a2", "/3", 3L),
                new Event("a2", "/2", 3L),
                new Event("a3", "/1", 3L),
                new Event("a2", "/3", 3L),
                new Event("a3", "/1", 3L),
                new Event("a2", "/2", 2L));

        StreamingFileSink<String> streamingFileSink = StreamingFileSink.<String>forRowFormat(new Path("./output"),
                new SimpleStringEncoder<>("UTF-8"))
                // 滚动策略
                .withRollingPolicy(DefaultRollingPolicy.builder()
                        .withMaxPartSize(1024 * 1024 * 1024)
                        // ms
                        .withRolloverInterval(TimeUnit.MINUTES.toMillis(15))
                        // 没有数据来
                        .withInactivityInterval(TimeUnit.MINUTES.toMillis(5))
                        .build())
                .build();
        stream.map(data -> data.toString()).addSink(streamingFileSink);
        env.execute();
    }
}

```



分区文件

- 至少包含15分钟的数据
- 最近5分钟没有收到新的数据
- 文件大小已达到 1 GB






### kafka





FlinkKafkaProducer 继承了抽象类 TwoPhaseCommitSinkFunction，这是一个实现了“两阶段提交”的 RichSinkFunction。两阶段提交提供了 Flink 向 Kafka 写入数据的事务性保证，能够真正做到精确一次(exactly once)的状态一致性





``` java
package com.matt.apitest.sink;

import com.matt.apitest.beans.Event;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaProducer;

import java.util.Properties;


/**
 * @author matt
 * @create 2022-01-25 23:33
 */
public class Sink2Kafka {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        Properties properties = new Properties();
        properties.setProperty("bootstrap.servers", "localhost:9092");
        properties.setProperty("group.id", "consumer-group");
        properties.setProperty("key.deserializer",
                "org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("value.deserializer",
                "org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("auto.offset.reset", "latest");

        DataStream<String> kafkaStream = env.addSource( new
                FlinkKafkaConsumer<String>("first", new SimpleStringSchema(), properties));

        //
        SingleOutputStreamOperator<String> res = kafkaStream.map(new MapFunction<String, String>() {
            @Override
            public String map(String s) throws Exception {
                String[] fileds = s.split(",");
                return new Event(fileds[0], fileds[1], Long.valueOf(fileds[2])).toString();
            }
        });

        res.addSink(new FlinkKafkaProducer<String>("localhost:9092", "test", new SimpleStringSchema()));
        // job name
        env.execute("SINK_KAFKA");
    }
}

```



### redis



导入依赖

``` xml
<dependency>
<groupId>org.apache.bahir</groupId>
  <artifactId>flink-connector-redis_2.11</artifactId>
  <version>1.0</version>
</dependency>
```



``` java
package com.matt.apitest.sink;

import com.matt.apitest.beans.Event;
import com.matt.apitest.beans.SensorReading;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.redis.RedisSink;
import org.apache.flink.streaming.connectors.redis.common.config.FlinkJedisPoolConfig;
import org.apache.flink.streaming.connectors.redis.common.mapper.RedisCommand;
import org.apache.flink.streaming.connectors.redis.common.mapper.RedisCommandDescription;
import org.apache.flink.streaming.connectors.redis.common.mapper.RedisMapper;

/**
 * @author matt
 * @create 2022-01-25 23:57
 */
public class Sink2Redis {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<Event> dataStream = env.fromElements(
                new Event("a1", "1", 1L),
                new Event("a2", "2", 1L),
                new Event("a3", "3", 1L),
                new Event("a4", "4", 1L));

        FlinkJedisPoolConfig config = new FlinkJedisPoolConfig.Builder()
                .setHost("127.0.0.1")
                .setPort(6379)
                .build();
        dataStream.addSink(new RedisSink<>(config, new MyRedisMapper()));

        env.execute("redis");
    }

    public static class MyRedisMapper implements RedisMapper<Event> {
        // 保存到 redis 的命令，存成哈希表
        public RedisCommandDescription getCommandDescription() {
            return new RedisCommandDescription(RedisCommand.HSET, "test");
        }
        // key
        public String getKeyFromData(Event data) {
            return data.user;
        }
        // v
        public String getValueFromData(Event data) {
            return data.url;
        }
    }
}
```





redisSink 提供2个参数

jedis的连接配置

实现RedisMapper, 提供add, getK getV 实现





### es



flink1.13 支持es7

添加依赖

``` xml
<dependency>
   <groupId>org.apache.flink</groupId>
<artifactId>flink-connector-elasticsearch7_${scala.binary.version}</artifactId>
   <version>${flink.version}</version>
</dependency>
```







``` java
package com.matt.apitest.sink;

import com.matt.apitest.beans.Event;
import com.matt.apitest.beans.SensorReading;
import org.apache.flink.api.common.functions.RuntimeContext;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.elasticsearch.ElasticsearchSinkFunction;
import org.apache.flink.streaming.connectors.elasticsearch.RequestIndexer;
import org.apache.flink.streaming.connectors.elasticsearch6.ElasticsearchSink;
import org.apache.http.HttpHost;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.client.Requests;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.Objects;

/**
 * @author matt
 * @create 2022-01-26 0:21
 */
public class Sink2ES {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<Event> dataStream = env.fromElements(
                new Event("a1", "1", 1L),
                new Event("a2", "2", 1L),
                new Event("a3", "3", 1L),
                new Event("a4", "4", 1L));

        ArrayList<HttpHost> httpHosts = new ArrayList<>();
        httpHosts.add(new HttpHost("localhost", 9200));
        dataStream.addSink(new ElasticsearchSink.Builder<Event>(httpHosts,new MyEsSinkFunction()).build());

        // job name
        env.execute("sink_es");

    }

    public static class MyEsSinkFunction implements
            ElasticsearchSinkFunction<Event> {
        @Override
        public void process(Event element, RuntimeContext ctx, RequestIndexer
                indexer) {
            HashMap<String, String> dataSource = new HashMap<>();
            dataSource.put("user", element.user);
            dataSource.put("url", element.url);
            dataSource.put("ts", String.valueOf(element.timestamp));
            IndexRequest indexRequest = Requests.indexRequest()
                    .index("tt4")
                    .type("tt")
                    .source(dataSource);
            indexer.add(indexRequest);
        }
    }
}

```



### mysql



```xml
 <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
        <!--版本太低可能会失败-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.28</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-jdbc_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
```





```java
package com.matt.apitest.sink;

import com.matt.apitest.beans.Event;
import com.matt.apitest.beans.SensorReading;

import org.apache.flink.configuration.Configuration;
import org.apache.flink.connector.jdbc.JdbcConnectionOptions;
import org.apache.flink.connector.jdbc.JdbcExecutionOptions;
import org.apache.flink.connector.jdbc.JdbcSink;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.sink.RichSinkFunction;

import java.sql.DriverManager;
import java.sql.Connection;
import java.sql.PreparedStatement;

/**
 * @author matt
 * @create 2022-01-26 1:41
 */
public class Sink2MySQL {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<Event> dataStream = env.fromElements(
                new Event("a1", "1", 1L),
                new Event("a2", "2", 1L),
                new Event("a3", "3", 1L),
                new Event("a4", "4", 1L));


        String sql = "INSERT INTO `event` (user, url) VALUES(?, ?)";
        dataStream.addSink(JdbcSink.sink(sql, ((statement, event) -> {
                    statement.setString(1, event.user);
                    statement.setString(2, event.url);
                }),
                //JdbcExecutionOptions.builder()
                //        .withBatchSize(1000)
                //        .withBatchIntervalMs(200)
                //        .withMaxRetries(5)
                //        .build(),
                new JdbcConnectionOptions.JdbcConnectionOptionsBuilder()
                        .withUrl("jdbc:mysql://localhost:3306/stu_go?characterEncoding=utf8&useSSL=false")
                        .withDriverName("com.mysql.cj.jdbc.Driver")
                        .withUsername("root")
                        .withPassword("rootroot")
                        .build()
        ));

        // job name
        env.execute();

    }


    public static class MyJdbcSink extends RichSinkFunction<SensorReading> {
        Connection conn = null;
        PreparedStatement insertStmt = null;
        PreparedStatement updateStmt = null;

        // open 主要是创建连接
        @Override
        public void open(Configuration parameters) throws Exception {
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/stu_flink",
                    "root", "root");

            // 创建预编译器，有占位符，可传入参数
            insertStmt = conn.prepareStatement("INSERT INTO sensor_temp (id, temp) VALUES ( ?, ?)");
            updateStmt = conn.prepareStatement("UPDATE sensor_temp SET temp = ? WHERE id = ? ");
        }

        // 调用连接，执行 sql
        @Override
        public void invoke(SensorReading value, Context context) throws Exception {
            // 执行更新语句，注意不要留 super
            updateStmt.setDouble(1, value.getTemperatrue());
            updateStmt.setString(2, value.getId());
            updateStmt.execute();
            // 如果刚才 update 语句没有更新，那么插入
            if (updateStmt.getUpdateCount() == 0) {
                insertStmt.setString(1, value.getId());
                insertStmt.setDouble(2, value.getTemperatrue());
                insertStmt.execute();
            }
        }

        @Override
        public void close() throws Exception {
            insertStmt.close();
            updateStmt.close();
            conn.close();
        }
    }
}
```

